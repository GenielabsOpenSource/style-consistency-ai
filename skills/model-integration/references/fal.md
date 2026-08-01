# fal.ai

fal hosts fast, consistency-friendly models — **Flux Kontext** (reference-conditioned editing) and **Seedream**, among many others. If this session already exposes a **`fal-generate` skill or MCP tool, prefer that** over the raw SDK: keys and I/O are already wired. Use the SDK below when no tool is available.

## Setup

```bash
pip install fal-client
export FAL_KEY="..."
```

## Text-to-image

```python
import fal_client

result = fal_client.subscribe(
    "fal-ai/flux-pro/v1.1",
    arguments={"prompt": "A red fox curled asleep on a mossy stone, soft dawn light",
               "image_size": "landscape_4_3"},
)
print(result["images"][0]["url"])   # download this URL to a local file
```

## Reference-conditioned edit (the consistency call) — Flux Kontext

Kontext edits *from* an input image with a delta-only instruction — ideal for keeping a character/object on-model.

```python
result = fal_client.subscribe(
    "fal-ai/flux-pro/kontext",
    arguments={
        "prompt": "Same character, now waving with the right hand, three-quarter view. Keep everything else identical.",
        "image_url": "https://.../hero_atlas.png",   # or upload a local file first (see below)
    },
)
```

Upload a local reference to get a URL fal can read:

```python
url = fal_client.upload_file("references/hero_atlas.png")
```

## Notes

- Model IDs live under the `fal-ai/...` namespace; browse fal's model gallery for current Kontext/Seedream/video endpoints.
- Outputs are returned as URLs — always download to a local path the user can open, don't just report the URL.
