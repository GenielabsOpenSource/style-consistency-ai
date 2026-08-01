# Approach 2 · Reference Images & the Character Atlas

Visual anchoring without a full dataset system. Two techniques, usually combined:

## A. The master reference / character atlas

A single clean reference sheet that reinforces the character's *core identity* in every generation. It provides a consistent visual anchor, reinforces defining features, reduces dependency on picking the "right" reference each time, and increases consistency across very different prompts.

**Include:**
- The character from several clear angles (front, side, back at minimum)
- Important facial features and proportions
- Distinctive clothing and accessories
- Unique hands or finger counts, shown clearly
- Ears, hair, horns, tails, other identifying features
- Close-ups of details the model repeatedly gets wrong

**Avoid:**
- Complex backgrounds (use one neutral background, light gray works well)
- Unnecessary scene elements
- Overlapping characters
- Excessive text labels
- Small or unclear details — if it matters, make it big
- Too many poses crammed into one image

**Building one:** composite the best existing images onto one neutral canvas, or generate the panels and hand-pick the most on-model results. Keep it to one clean sheet; a cluttered atlas dilutes the anchor. When a trait keeps drifting, add a dedicated close-up panel of that trait.

If the user has only one good image, that image *is* the master reference for now — use it on every generation and suggest growing it into an atlas.

## B. Closest-reference selection (small edits win)

With even a handful of references, don't attach one at random: look at them and pick the one closest to the request, matching in priority order:

1. Pose and body position
2. Camera angle and orientation
3. Facial expression
4. Action

The model preserves identity best when it only has to change a little. "Bob waving, seen from the side" is better served by a side-view with a raised arm than by a perfect front portrait.

## Combining references per generation

- Model accepts **multiple references** → attach the closest match *and* the atlas as identity anchor.
- Model accepts **one reference** → atlas for identity-critical requests (face close-ups, hands, feature-heavy shots); closest pose match otherwise.
- **Editing an existing image** → the input image is the primary reference; atlas as secondary anchor if supported; prompt only the requested change.
- When the model exposes fidelity/strength settings, bias toward preserving the reference.

Always pair with Approach 1 guidelines in the prompt.

## When this rung is enough — and when it isn't

Enough: consistent identity for moderate prompt variety; most casual and small-team use.

Not enough: production use demanding wide pose/angle/expression range (no single sheet covers it — move to Approach 3), or identity that holds alone but collapses in scenes (move to Approach 4's two-stage split).
