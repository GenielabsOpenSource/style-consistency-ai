# Approach 5 · Post-Processing & Training

The heaviest rung. Only enter here when Approaches 1–4 are in place and specific errors still persist — and only when the new layer solves a clear, measurable problem.

## A. Post-processing refinement pass

Run an image-to-image editing model (a Kontext-style editor or similar) *after* the main generation to correct repeated misses in: colors, anatomy, facial features, body proportions, materials, textures, line quality, and final polish. Feed it the raw output, a short refinement prompt targeting the known misses, and the atlas if supported.

**Tradeoff:** two models in sequence roughly doubles generation time. Use the single-pass pipeline when speed matters more than small inconsistencies. Make the pass skippable.

## B. Training the refinement pass (editing LoRA on corrected pairs)

The refinement pass can be specialized to the character. Collect paired examples covering the recurring failures:

- Incorrect color → corrected color
- Incorrect anatomy → corrected anatomy
- Incorrect proportions → corrected proportions
- Missing detail → restored detail
- Rough output → polished output

Then: train an editing LoRA on the pairs (hosted training platforms support this), deploy it, configure a fixed refinement prompt, test on fresh outputs, and add more pairs wherever problems remain.

## C. Training a dedicated character model

When reference-conditioned generation tops out entirely, two common routes:

- **Character LoRA on a base image model.** Powerful but expect real experimentation: dataset selection, captions, training parameters, trigger words, prompt phrasing, LoRA strength. Validate against real production use cases before integrating.
- **Fine-tuning a trainer-friendly model.** Several hosted platforms offer simple character fine-tuning — fast and accurate with a well-prepared dataset: the Approach 3 dataset plus a clear caption for every image. Generate captions with a captioning tool, then review each one for clarity and consistency.

Either way, the Approach 3 dataset standards are the training-set standards; a dataset that fails reference-conditioned generation will fail training too.

## Scope the trained model narrowly

A fine-tuned character model is usually *narrower* than the general pipeline — often text-to-image, character alone, backgrounds added downstream via the two-stage split. Its routing rule (Approach 4) must state what it must **not** do:

"When the user requests this character alone through text-to-image, use the fine-tuned model. Do not activate when an input image is attached or when the user asks to edit an existing image — route image-to-image requests to the designated supported model."

Configure the operational side first (model access, identifier, supported generation modes, default parameters, prompt/trigger configuration), then test both directions: activation on supported requests, and staying out of unsupported ones (input images, edits, multiple characters).
