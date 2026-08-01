---
name: character-consistency
description: Maintain character identity and style accuracy when generating or editing images of a recurring character with multimodal image models (e.g., Nano Banana, GPT Image, Flux Kontext, Seedream, or any reference-conditioned model). Use this skill whenever the user asks to generate, edit, or iterate on images of a named or established character — especially when they mention keeping the character "on-model", consistent, accurate, or in-style, when generations keep drifting from the character's design, or when they want to set up, improve, or debug a character-generation workflow, dataset, reference sheet, or pipeline. Trigger even if the user just says a character "doesn't look right" across generations.
---

# Character Consistency

Keep a recurring character on-model across generations and edits: proportions, colors, distinctive marks, finger counts, facial structure — every trait that makes the character *that* character.

Two principles govern everything in this skill:

1. **Simplest fix first.** There is a ladder of approaches from a one-line guideline to a fine-tuned model. Always start at the lowest rung that plausibly solves the user's problem, and add a heavier layer only when the previous one demonstrably falls short *and* the new layer solves a clear, measurable problem.
2. **Aim for the smallest possible edit.** Image models preserve identity best when the reference material is already close to the target and the prompt only describes the delta. Every approach on the ladder is a way of shrinking what the model has to invent.

## The ladder of approaches

| # | Approach | Effort | What it is | Reference file |
|---|----------|--------|------------|----------------|
| 1 | **Corrective guidelines** | Minutes | Short, targeted rules for what the model gets wrong; delta-only prompting | `references/approach-1-guidelines.md` |
| 2 | **Reference images & atlas** | An hour | Attach a master reference / character atlas; with a few references, pick the closest one per request | `references/approach-2-references.md` |
| 3 | **Full dataset system** | Hours–days | Curated 10–20 image dataset + tags + closest-reference selection per generation | `references/approach-3-dataset-system.md` |
| 4 | **Pipelines & model routing** | Hours | Two-stage identity/composition split, systematic model comparison, narrow routing rules | `references/approach-4-pipelines-routing.md` |
| 5 | **Post-processing & training** | Days+ | Refinement pass with an editing model; character LoRA or fine-tune | `references/approach-5-postprocess-training.md` |

Approaches stack: a dataset system still uses corrective guidelines; a fine-tuned model still benefits from an atlas and routing rules.

## Triage: choose the right approach with the user

Don't assume — ask. Before recommending or executing anything, establish the user's situation (skip questions the conversation already answers):

1. **What do they have?** Nothing but a description • one good image • a handful of references • a curated dataset • a trained model
2. **What's breaking?** One recurring detail (fingers, a mark, a color) • overall style drift • pose/angle failures • identity collapses in scenes/edits • nothing yet, they're setting up
3. **What's the volume?** A few one-off images • ongoing production use with many prompts
4. **What can they run?** Chat model only • API/MCP image tools • a platform with datasets and routing • training infrastructure

Then map to a starting rung:

- **One-off images, or a single recurring mistake** → Approach 1. A one-line corrective guideline often fixes what a whole dataset can't.
- **Has 1+ good images, identity drifts broadly** → Approach 2. A master reference/atlas is the biggest consistency win per unit of effort.
- **Production use, many varied prompts (poses, angles, expressions)** → Approach 3. Range requires a dataset.
- **Identity fine alone but collapses in scenes/backgrounds, or different models win different jobs** → Approach 4.
- **Approaches 1–4 applied and specific errors still persist** → Approach 5. Never start here.

State the recommendation and the reasoning briefly, confirm with the user, then read the matching reference file and execute. If the user's ask is a generation request and assets already exist, run the per-generation loop below at whatever rung they're on.

## The per-generation loop (applies at every rung)

Whenever actually generating or editing a character image:

1. **Classify the request** — text-to-image or image-to-image; character alone or in a scene; which pose/angle/expression/action.
2. **Gather the best conditioning available at the current rung** — guidelines (rung 1), atlas or closest reference (rungs 2–3), pipeline stage plan (rung 4).
3. **Prompt the delta only.** Describe what should change from the reference; never re-describe the art style — reference-conditioned models learn style from images, and long style descriptions fight the reference and cause drift.
4. **Invoke whatever image capability is available** — an MCP image tool, a configured API (fal, Replicate, OpenAI, Google, etc.), or the platform's built-in generation — adapting the package to its interface. If several models are available, prefer the strongest at reference-following and say which was used.
5. **Verify the output** against the character's defining traits (build a checklist from the references if none exists), checking historically-wrong traits first. State what's on-model and what drifted.
6. **On drift, fix at the cheapest level:** sharpen a guideline → swap the reference → adjust the pipeline → try another model → fix the assets → only then consider the next rung of the ladder. Tell the user which fix you're applying and why.

## Debugging drift: inspect the whole flow

When outputs keep going off-model, review every stage before blaming the model or escalating: the original request → the prompt actually sent → the guidelines/tags included → the reference(s) selected → intermediate generations (in multi-stage pipelines) → the final result. Most "model problems" are a bloated prompt, a wrong reference, or a dirty asset. Detailed debugging and the model-comparison rubric live in `references/approach-4-pipelines-routing.md`.

## Recommended asset layout (a convention, not a requirement)

```
<character-name>/
├── guidelines.md     # rung 1: short corrective rules
├── atlas.png         # rung 2: master reference sheet
├── dataset/          # rung 3: 10–20 varied but consistent images
└── identity.md       # optional: checklist of defining traits for verification
```

Adapt to whatever the user actually has — loose folders, a single image, or images pasted in chat all work.
