# Approach 1 · Corrective Guidelines

The cheapest and often most effective fix: short, targeted text rules for exactly what the model gets wrong, plus delta-only prompting. No assets required beyond whatever the user already generates with.

## Write only what the model gets wrong

Do **not** describe the art style or the whole character up front. When any reference image is in play, the model learns style and identity from the image; long descriptions fight it and create *inconsistent* results. Even in pure text-to-image, over-specification dilutes the corrections that matter.

Start minimal. Generate, observe the failures, then add one short line per failure. Keep a guideline only when it does one of these:

- Corrects a repeated generation problem
- Reinforces an important characteristic
- Clarifies an unusual visual feature
- Prevents a common artifact
- Improves consistency across outputs

**Example.** Every reference shows three fingers, but outputs keep showing five. Add exactly: "The character has three fingers." Not a paragraph about the hands.

## Delta-only prompting

When editing or generating from a reference, the prompt should describe only what changes: the new pose, action, expression, or setting. Everything not mentioned should come from the reference. If the user's prompt re-describes the character or style, trim it before sending.

## Keep guidelines persistent

Store the corrective lines in a `guidelines.md` next to the character's assets (or pin them in the conversation) and include them in every generation for that character. On platforms with per-character tags/guidelines fields, put them there.

## When this rung is enough — and when it isn't

Enough: one-off images, a small number of recurring detail errors, users without reference assets.

Not enough: broad style drift, pose/angle range failures, identity collapse in scenes. Those need visual anchoring — move to Approach 2 (references/atlas) or Approach 3 (dataset).
