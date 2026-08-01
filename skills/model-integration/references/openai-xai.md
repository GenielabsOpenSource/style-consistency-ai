# OpenAI GPT Image & xAI Grok Imagine

Two providers that share an SDK shape (both use the `openai` Python client; xAI is OpenAI-compatible via a custom `base_url`).

- **OpenAI GPT Image** (`gpt-image-1`, or the latest `gpt-image-*`) — best at **text rendered inside images** (posters, UI, labels) and first-class **mask-based inpaint/outpaint**. Solid image-to-image, but for tight character/object identity, Nano Banana or Flux Kontext usually follow references more faithfully.
- **xAI Grok Imagine** (`grok-2-image`) — fast, high-quality **text-to-image**. Text-to-image only; no reference-conditioning, so not a consistency tool — use it for fresh scenes.

## OpenAI — setup

```bash
pip install openai
export OPENAI_API_KEY="..."
```

### Text-to-image

```python
import base64, os
from openai import OpenAI

client = OpenAI()
resp = client.images.generate(
    model="gpt-image-1",
    prompt="A poster reading 'DAWN' over a sleeping red fox on mossy stone",
    size="1024x1024",
)
with open("out.png", "wb") as f:
    f.write(base64.b64decode(resp.data[0].b64_json))
```

### Edit / inpaint (reference + optional mask)

```python
resp = client.images.edit(
    model="gpt-image-1",
    image=open("references/hero_atlas.png", "rb"),
    mask=open("mask.png", "rb"),   # optional; transparent areas get repainted
    prompt="Replace the background with a snowy forest; keep the character untouched.",
)
```

## xAI — setup

```bash
pip install openai   # reuse the OpenAI SDK
export XAI_API_KEY="..."
```

```python
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["XAI_API_KEY"], base_url="https://api.x.ai/v1")
resp = client.images.generate(model="grok-2-image", prompt="A red fox curled asleep on a mossy stone")
url = resp.data[0].url   # download to a local file
```

## Notes

- If `gpt-image-1` is rejected as unknown, list models via the SDK and pick the current `gpt-image-*`.
- xAI image model IDs (e.g. `grok-2-image-1212`) get versioned suffixes — verify against xAI's current docs if a call fails.
