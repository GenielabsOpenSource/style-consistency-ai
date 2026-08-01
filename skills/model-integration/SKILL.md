---
name: model-integration
description: Connect to and invoke image and video generation models across providers (Gemini / Nano Banana, fal.ai, Replicate, OpenAI GPT Image, xAI Grok Imagine) from an agent. Use this skill whenever a task needs to actually generate or edit an image or video and it is unclear which capability is available or how to call it — setting up API keys, detecting configured providers or MCP image tools, choosing the right model for a job (especially reference-following for style/character consistency), or writing the call. This is the execution layer the consistency skills delegate to when they say "invoke whatever image capability is available".
---

# Model Integration

Turn "generate this image" into an actual, correct API call — regardless of which provider the user has configured. This skill is the **execution layer** for the consistency skills in this repo (`character-consistency` and the planned `background-`, `object-`, `video-consistency`): they decide *what* to prompt and *which reference* to condition on; this skill handles *how to reach a model and invoke it*.

Two principles:

1. **Detect, don't assume.** Never hardcode a provider. Discover what is actually available — MCP image tools first, then configured API keys — and use the strongest option present.
2. **Match the model to the job.** Providers are not interchangeable. Reference-following (the thing consistency needs) varies enormously between models. Route by capability, not by habit.

## Step 1 — Detect available capabilities (in this order)

1. **MCP / built-in image tools.** If the environment exposes an image-generation tool or skill (e.g. a `fal-generate` skill, a platform image tool), prefer it — keys and plumbing are already handled. List what's available before reaching for raw APIs.
2. **Configured API keys.** Check the environment for provider credentials:

   | Provider | Env var | Reference |
   |----------|---------|-----------|
   | Gemini / Nano Banana / Veo | `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) | `references/gemini.md` |
   | fal.ai | `FAL_KEY` | `references/fal.md` |
   | Replicate | `REPLICATE_API_TOKEN` | `references/replicate.md` |
   | OpenAI GPT Image | `OPENAI_API_KEY` | `references/openai-xai.md` |
   | xAI Grok Imagine | `XAI_API_KEY` | `references/openai-xai.md` |

3. **Nothing configured?** Don't guess a key. Tell the user which providers this skill supports, what each is best at (below), and ask which they want to set up — then point them at the matching reference file for the one-time setup.

Never print, log, or hardcode key values. Read them from the environment at call time.

## Step 2 — Route to the right model

See `references/model-routing.md` for the full rubric. Quick guide:

| Job | Prefer | Why |
|-----|--------|-----|
| **Reference-following / consistency** (character, object, style) | Gemini "Nano Banana" image models, fal Flux Kontext, Seedream | Built for identity-preserving edits from reference images |
| **Text-to-image, no reference** | Any; xAI Grok Imagine and OpenAI GPT Image are strong | Fast, high quality from prompt alone |
| **Text *inside* the image** (posters, UI, labels) | OpenAI GPT Image | Best in-image typography |
| **Mask-based inpaint / outpaint** | OpenAI GPT Image, fal | First-class edit + mask support |
| **Video (text→video, image→video)** | Google Veo (via Gemini), Replicate/fal video models | Veo leads on coherence |
| **Widest model selection under one key** | Replicate | One API, many hosted models |

When several providers qualify, pick the strongest at reference-following and **state which model you used** in your reply.

## Step 3 — Invoke, following the reference file

Each `references/<provider>.md` gives the setup (SDK install + env var), a minimal text-to-image call, and — where the provider supports it — a reference-conditioned edit call (the important one for consistency). Copy the pattern, substitute the prompt and reference image(s), and run it. Save outputs to a path the user can open.

## Step 4 — Hand back to the consistency loop

After generating, return to the calling consistency skill's per-generation loop: **verify** the output against the character/object/scene's defining traits, and **on drift, fix at the cheapest level** — sharpen the prompt or swap the reference before switching models. Trying a different model (Step 2) is a mid-ladder fix, not the first move.

## Notes on model IDs

Model names drift fast (e.g. Nano Banana → Nano Banana Pro; `gpt-image-1` → successors). The reference files name the current-known IDs, but if a call 404s on an unknown-model error, list the provider's available models via its SDK/API and pick the closest match rather than assuming the ID is wrong everywhere.
