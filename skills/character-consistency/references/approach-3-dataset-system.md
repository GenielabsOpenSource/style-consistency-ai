# Approach 3 · The Full Dataset System

For production use with many varied prompts. A curated dataset teaches the model the character's *range* — poses, angles, expressions — while the atlas (Approach 2) keeps anchoring core identity. Per generation, the closest dataset image is selected and sent as reference so the model performs a small edit rather than an invention.

## Build the dataset: 10–20 varied but consistent images

**Vary across the set:**
- Poses and body positions
- Facial expressions
- Actions
- Camera angles
- Orientations (facing left/right, front/back)
- Character proportions in frame (full body, half, close-up)

**Keep identical across the set:**
- Dimensions and aspect ratio (a square around 1024–1440 px works well)
- The same neutral background
- Character shown clearly, no background clutter
- Consistent rendering quality and sharpness

One outlier poisons the well: a single unusually tall or narrow image can make the model repeatedly generate stretched, elongated characters. Audit for outliers in dimensions, style, and quality first.

## Tags per image, guidelines per character

Give each image a short factual tag (pose/angle/expression/action) so the closest match is findable. Character-level guidelines follow Approach 1 exactly: only what the model gets wrong, nothing about style.

## Per-generation selection

For each request, look at the dataset and pick the closest image (pose → angle → expression → action). For large datasets, skim filenames/tags first, then inspect the top candidates closely. Attach the atlas too when multiple references are supported.

If nothing in the dataset is remotely close to the request, that's a **dataset gap** — note it to the user; it maps to rung 5 of the fix ladder below.

## Iterate against real use cases

Test with the prompts production will actually use — real poses, camera angles, actions, compositions, expressions. Then evaluate: what works, which prompts repeatedly fail, where artifacts appear, what's production-ready. A dataset is done when real use cases pass, not when it looks nice.

## The dataset fix ladder (smallest correction first)

When a recurring problem appears, apply the cheapest fix that could plausibly work:

1. Add or update a short, specific guideline
2. Improve the tag of a relevant dataset image
3. Replace an image that introduces an unwanted pattern
4. Remove an image that reduces dataset consistency
5. Add a stronger example of a missing pose, angle, or feature

## When this rung is enough — and when it isn't

Enough: wide prompt coverage with strong identity for character-alone generation.

Not enough: scene/background degradation, or when different models clearly win different jobs → Approach 4. Persistent trait errors that survive dataset fixes → Approach 5.
