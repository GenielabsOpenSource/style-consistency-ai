# Gemini / Nano Banana / Veo (Google)

Google's Gemini image models — marketed as **"Nano Banana"** (`gemini-2.5-flash-image`) and **"Nano Banana Pro"** (`gemini-3-pro-image`) — are strong at reference-following and multi-image conditioning, which makes them a top pick for character/object/style consistency. **Veo** (`veo-3.1`) handles video.

## Setup

```bash
pip install google-genai
export GEMINI_API_KEY="..."   # or GOOGLE_API_KEY
```

## Text-to-image

```python
import os
from google import genai

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])

resp = client.models.generate_content(
    model="gemini-3-pro-image",              # Nano Banana Pro; use gemini-2.5-flash-image for the cheaper tier
    contents=["A red fox curled asleep on a mossy stone, soft dawn light"],
)

# Image bytes come back as inline_data parts:
for part in resp.candidates[0].content.parts:
    if getattr(part, "inline_data", None):
        with open("out.png", "wb") as f:
            f.write(part.inline_data.data)
```

## Reference-conditioned edit (the consistency call)

Pass one or more reference images alongside a **delta-only** prompt — describe what changes, not the art style (the model reads style from the references). Nano Banana models accept multiple references.

```python
from google.genai import types

def load(path):
    return types.Part.from_bytes(data=open(path, "rb").read(), mime_type="image/png")

resp = client.models.generate_content(
    model="gemini-3-pro-image",
    contents=[
        "Same character, now waving with the right hand, three-quarter view. Keep everything else identical.",
        load("references/hero_atlas.png"),   # closest reference from the consistency skill
    ],
)
```

## Video (Veo)

```python
op = client.models.generate_videos(
    model="veo-3.1",
    prompt="The fox stretches and yawns, camera slowly pushes in",
    # image=load("first_frame.png"),  # for image-to-video
)
# Veo is async: poll op until done, then download the returned video asset.
```

## Notes

- Multiple reference images → keep the *closest* one first; too many loosely-related refs can dilute identity.
- If `gemini-3-pro-image` errors as unknown, list models (`client.models.list()`) and fall back to `gemini-2.5-flash-image`.
