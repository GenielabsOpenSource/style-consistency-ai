# Approach 4 · Pipelines & Model Routing

When one model in one pass can't hold both identity and composition — or when different models win different jobs.

## The two-stage identity/composition split

For characters in scenes or custom backgrounds, split the work:

1. **Identity stage:** generate the character *alone on a neutral background* (even if the user asked for a beach) using the model strongest at identity, with atlas/closest reference and guidelines.
2. **Composition stage:** pass that result into an image-to-image step with the model strongest at editing, prompting only the background/scene addition.

Identity survives far better when the model isn't juggling character fidelity and scene composition simultaneously. The same split applies to editing requests: keep identity-preserving edits and scene edits in separate passes when one pass keeps failing.

**Test models per stage, not just per character.** A model's overall ranking is irrelevant — what matters is which model wins each *stage* of the pipeline. For example, one model (say, GPT Image) might be best at generating the character alone on a neutral gray background, while another (say, Nano Banana) is best at the image-to-image pass that adds the background. Run the stage-level comparison separately:

- **Identity stage test:** same reference package + same character-alone prompt through each candidate → judge on character consistency, anatomy, detail preservation.
- **Composition stage test:** same clean character-on-gray input + same background prompt through each candidate → judge on how untouched the character stays, background quality, and edge/blending artifacts.

The winners become the stage assignments in the routing rules below, and they may be different models for different characters or art styles — retest per character rather than assuming one split fits all.

## Comparing models systematically

Run the *same* prompt and reference package through each available model (e.g., "generate 10 images of the character with model X") and compare on:

- Character consistency (the headline metric)
- Prompt understanding
- Pose accuracy
- Facial-expression accuracy
- Anatomy
- Preservation of key details (marks, finger counts, accessories)
- Artifact frequency
- Background handling
- Real production use cases — not just pretty test prompts

Run this comparison both for the whole pipeline and per stage (see the stage-level tests above) — the best single-pass model and the best per-stage combination are often not the same.

Also look beyond the models already in use: a recently released or niche model may perform significantly better for a specific character or art style. Run small tests before any full integration.

## Routing rules: narrow and explicit

Once a winner per job is known, encode it. On platforms with per-character routing/skills, or simply as standing instructions in a workflow, a routing rule must specify:

- The exact character
- Alone or with others
- Text-to-image or image-to-image
- Whether an input image is allowed
- Which model to use
- Whether a background is allowed
- When the rule must **not** activate
- The fallback model

Too broad: "Always use model X for this character."
Precise: "Only when generating this character alone through text-to-image, use model X. Always generate on a neutral gray background even when the user requests another background." (The background then comes from the composition stage.)

Set an organization/default-level model when one model wins for most characters; use narrow per-character rules for exceptions — precise control without changing everything.

**Test both directions:** that a rule activates when it should, and that it stays out of unsupported requests (input images, edits, multiple characters).

## Debugging the whole flow

When drift persists, inspect every stage before escalating:

1. The original user request
2. The prompt actually sent to the model
3. The guidelines and image tags included
4. The reference image(s) selected
5. Intermediate generations (which stage introduced the drift?)
6. The final result

Most "model problems" turn out to be a wrong reference, a bloated prompt, or a dirty asset. Only when the flow is clean and errors persist should Approach 5 enter the conversation.
