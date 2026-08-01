# Replicate

One API key, a huge catalog of hosted models — Flux (incl. **Flux Kontext** for reference edits), SDXL, and many video models. Good default when you want breadth or a specific community model under a single integration.

## Setup

```bash
pip install replicate
export REPLICATE_API_TOKEN="..."
```

## Text-to-image

```python
import replicate

output = replicate.run(
    "black-forest-labs/flux-1.1-pro",
    input={"prompt": "A red fox curled asleep on a mossy stone, soft dawn light",
           "aspect_ratio": "4:3"},
)
# output is typically a list of file objects / URLs:
url = output[0] if isinstance(output, list) else output
```

## Reference-conditioned edit (the consistency call) — Flux Kontext

```python
output = replicate.run(
    "black-forest-labs/flux-kontext-pro",
    input={
        "prompt": "Same character, now waving with the right hand, three-quarter view. Keep everything else identical.",
        "input_image": open("references/hero_atlas.png", "rb"),
    },
)
```

## Video

```python
output = replicate.run(
    "<owner>/<video-model>",   # browse replicate.com for current Veo / Kling / Wan endpoints
    input={"prompt": "The fox stretches and yawns, camera slowly pushes in",
           "image": open("first_frame.png", "rb")},
)
```

## Notes

- Model refs are `owner/name` and can be version-pinned as `owner/name:<version-hash>` for reproducibility; most image models also accept a `"seed"` input — pin both when iterating.
- Downloads: iterate the returned file objects and write them locally; don't report bare URLs.
