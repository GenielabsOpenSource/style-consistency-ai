---
name: model-integration
description: Connect to and invoke image and video generation models across providers (Gemini / Nano Banana, fal.ai, Replicate, OpenAI GPT Image, xAI Grok Imagine) from an agent. Use this skill whenever a task needs to actually generate or edit an image or video and it is unclear which capability is available or how to call it — setting up API keys, detecting configured providers or MCP image tools, choosing the right model for a job (especially reference-following for style/character consistency), or writing the call. This is the execution layer the consistency skills delegate to when they say "invoke whatever image capability is available".
---

# Model Integration

The consistency skills in this repo decide *what* to prompt and *which reference* to condition on. This skill owns everything below that line: finding a way to reach a model, choosing the right one for the job, making the call correctly, and recovering when it fails.

Treat it as a contract with four clauses:

1. **Discover before you call.** The environment defines what's possible — never assume a provider.
2. **Route by capability.** Models differ most where consistency cares most: how faithfully they follow a reference image.
3. **Invoke reproducibly.** Same inputs should give comparable outputs; outputs land as local files, never bare URLs.
4. **Fail loudly and cheaply.** Diagnose from the error, retry only what retrying can fix, and report exactly what ran.

## Clause 1 — Discovery

Work down this list and stop at the first hit:

1. **Session tools.** An MCP image tool or generation skill already wired into the session (for example a fal-backed generation skill) beats raw SDK calls — auth, upload, and download are solved. Enumerate what the session exposes before writing any provider code.
2. **Credentials in the environment.** Probe, don't guess:

   ```bash
   env | grep -oE '^(GEMINI_API_KEY|GOOGLE_API_KEY|FAL_KEY|REPLICATE_API_TOKEN|OPENAI_API_KEY|XAI_API_KEY)=' | tr -d '='
   ```

   (Prints only the *names* of configured keys, never values.)

   | Credential | Unlocks | Reference |
   |---|---|---|
   | `GEMINI_API_KEY` / `GOOGLE_API_KEY` | Nano Banana image models, Veo video | `references/gemini.md` |
   | `FAL_KEY` | Flux Kontext, Seedream, fal video endpoints | `references/fal.md` |
   | `REPLICATE_API_TOKEN` | Replicate's hosted catalog (Flux, video models) | `references/replicate.md` |
   | `OPENAI_API_KEY` | GPT Image generation + masked edits | `references/openai-xai.md` |
   | `XAI_API_KEY` | Grok Imagine text-to-image | `references/openai-xai.md` |

3. **Empty environment.** Stop and present the trade-space instead of picking for the user: name each provider, what it's uniquely good at, and roughly what setup involves. Once they choose, walk them through only that provider's reference file.

Handle keys by name only — read them from the environment inside the call, never echo values into logs, code, or chat.

## Clause 2 — Routing

`references/model-routing.md` holds the full rubric, capability matrix, and the two-stage identity/composition pattern. The short version, ordered by the question you should ask first:

- **Preserving an established identity** (character, object, style)? → a reference-follower: Nano Banana (Pro) or Flux Kontext. This is the default path for everything in this repo.
- **Legible text inside the image**? → GPT Image.
- **Local edit under a mask** (fix a region, swap a background)? → GPT Image or a Flux inpaint endpoint.
- **Motion**? → Veo via Gemini; Replicate/fal video endpoints as alternates.
- **No reference, just a scene**? → whatever is configured; Grok Imagine and GPT Image are quick and strong.
- **Only one provider available?** Use it — and if it's weak for the job, say so in one line ("no reference support here; identity may drift").

Always name the model you used in your reply. When two qualify, prefer the stronger reference-follower and note the runner-up so the user can ask for a comparison.

## Clause 3 — Invocation

Each provider reference gives: one-time setup, a minimal text-to-image call, and the reference-conditioned edit call (the one consistency work lives on). Beyond copying the pattern:

- **Prompt the delta.** When a reference image is attached, describe only what changes. Re-describing the art style fights the reference — this is the core rule inherited from the consistency skills.
- **Pin what you can.** Pass a `seed` where the API accepts one and record it; on Replicate, version-pin the model hash. When the user later says "like the last one but…", a pinned seed + model is the difference between iteration and lottery.
- **Land outputs locally.** Download every result to a project path with a name that encodes intent — `out/fox_wave_3q_seed42.png`, not `download (3).png`. Report the path.
- **Batch deliberately.** For an exploratory request, 2–4 variants at low cost beats one expensive render; for a locked-in consistency edit, one careful call beats a spray of variants.

## Clause 4 — Failure playbook

Diagnose from the error class, not by blind retry:

| Symptom | Likely cause | Cheapest fix |
|---|---|---|
| 401 / 403 | Key missing, expired, or wrong env var name | Re-run discovery; confirm the exact variable the SDK reads |
| Unknown / retired model ID | Provider renamed the model | List models via the SDK and pick the nearest current ID — IDs drift fast (Nano Banana → Pro; `gpt-image-1` → successors) |
| 429 / rate limit | Burst too fast | Back off with jitter; drop batch size to 1 |
| Content-policy rejection | Prompt or reference tripped the filter | Rephrase neutrally; if a reference image triggers it, try a different reference before a different provider |
| Empty / malformed response | Transient provider fault | One retry; then switch provider for this call and note it |
| Output ignores the reference | Wrong model class for the job | Re-route (Clause 2) — this is a routing bug, not a prompt bug |

After any generation, control returns to the calling consistency skill's loop: verify against the identity checklist, and on drift fix at the cheapest level — prompt, then reference, then model. Switching providers is a mid-ladder move, never the reflex.
