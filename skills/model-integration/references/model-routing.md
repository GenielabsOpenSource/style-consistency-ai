# Model routing rubric

Which model to reach for, and why. The dimension that matters most for this repo is **reference-following** — how faithfully a model preserves identity from a conditioning image. That is what makes or breaks consistency; raw prompt-only quality is secondary.

## Capability matrix

| Model (provider) | Ref-following | Text-in-image | Inpaint/mask | Video | Best for |
|---|---|---|---|---|---|
| Nano Banana Pro `gemini-3-pro-image` (Gemini) | ★★★★★ | ★★★ | ★★★ | — | Character/object/style consistency, multi-reference |
| Nano Banana `gemini-2.5-flash-image` (Gemini) | ★★★★ | ★★ | ★★ | — | Cheaper consistency tier |
| Flux Kontext (fal / Replicate) | ★★★★★ | ★★★ | ★★★★ | — | Delta-only reference edits |
| Seedream (fal) | ★★★★ | ★★ | ★★ | — | Stylized consistency |
| GPT Image `gpt-image-1` (OpenAI) | ★★★ | ★★★★★ | ★★★★★ | — | Text in image, precise masked edits |
| Grok Imagine `grok-2-image` (xAI) | — (no refs) | ★★ | — | — | Fast fresh text-to-image scenes |
| Veo `veo-3.1` (Gemini) | ★★★★ (img→video) | — | — | ★★★★★ | Text/image-to-video coherence |
| Replicate video hub | varies | — | — | ★★★★ | Access to Kling/Wan/others |

Ratings are relative guidance, not benchmarks — verify on the user's own character before committing a production pipeline (see `character-consistency` approach 4, model comparison).

## Decision order

1. **Is there a reference to preserve?** (a character, object, or established scene)
   - **Yes** → a reference-follower: Nano Banana (Pro) or Flux Kontext. This is the consistency path.
   - **No** → any strong text-to-image: Grok Imagine, GPT Image, Flux.
2. **Does the image need legible text inside it?** → GPT Image.
3. **Is it a masked/local edit** (swap background, fix one region)? → GPT Image or Flux (mask/inpaint support).
4. **Is it video?** → Veo first; Replicate/fal video models for specific looks or when Veo is unavailable.
5. **Only one key configured?** → use what's there; note the tradeoff if it's weak for the job (e.g. "used Grok — no reference support, so identity may drift; configure Gemini or fal for on-model results").

## Two-stage pattern (identity + composition)

When one model can't both hold identity *and* nail a complex scene, split the job (this is `character-consistency` approach 4):

1. **Identity stage** — generate/keep the character on-model on a plain background with a reference-follower (Nano Banana / Kontext).
2. **Composition stage** — place that on-model output into the scene with a mask-capable editor (GPT Image / Flux inpaint), prompting only the environment.

Route each stage to the model that wins that stage, not one model for both.
