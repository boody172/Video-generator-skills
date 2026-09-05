# Arabic Text Overlay Technique

Blender's native `FONT` (Text) datablock does not shape Arabic script — it
renders isolated, unjoined letters in visual (not logical) order. To get
correctly shaped, right-to-left Arabic captions into a Blender scene, render
the text to a transparent PNG *outside* Blender (in the sandbox) and import
that PNG as an image texture on a plane.

## Steps (run in Claude's sandbox with `bash_tool` / Python)

1. Install dependencies once per session:

```bash
pip install --break-system-packages pillow arabic-reshaper python-bidi
```

2. Render the caption to a transparent PNG:

```python
from PIL import Image, ImageDraw, ImageFont
import arabic_reshaper
from bidi.algorithm import get_display

def render_caption(text, font_path, font_size=90, color=(255, 255, 255, 255),
                    canvas_size=(1080, 300), out_path="/home/claude/caption.png",
                    is_arabic=True):
    if is_arabic:
        reshaped = arabic_reshaper.reshape(text)   # join letters correctly
        text = get_display(reshaped)               # fix visual RTL order

    img = Image.new("RGBA", canvas_size, (0, 0, 0, 0))
    draw = ImageDraw.Draw(img)
    font = ImageFont.truetype(font_path, font_size)

    bbox = draw.textbbox((0, 0), text, font=font)
    w, h = bbox[2] - bbox[0], bbox[3] - bbox[1]
    x = (canvas_size[0] - w) / 2
    y = (canvas_size[1] - h) / 2
    draw.text((x, y), text, font=font, fill=color)

    img.save(out_path)
    return out_path
```

- Use a font that actually has Arabic glyphs (e.g. Cairo, Noto Naskh Arabic,
  Tajawal). Don't assume a default system font covers Arabic — check first.
- For **bilingual** captions (Arabic sentence with an English technical term
  in parentheses), reshape/bidi only the Arabic portion(s) and leave the
  Latin portion as-is; `python-bidi`'s `get_display` generally handles mixed
  runs correctly if you pass the whole mixed string through it, but verify
  the rendered output visually before shipping.

3. Base64-encode the resulting PNG and transfer it into the user's running
   Blender via `Blender:execute_blender_code`, per the main SKILL.md §3
   pattern — decode, write to a local temp path, `bpy.data.images.load()`.

4. In Blender, put this image on its own plane, sized to match the caption's
   pixel aspect ratio, positioned per the ad's layout (typically lower
   third), and parent/keyframe it to appear on the correct frames — and, if
   a voiceover script is in use, timed to appear while its matching line is
   spoken rather than floating independent of the audio.

## Why not just use Blender's Text object with a Unicode string?

Blender *will* accept an Arabic Unicode string in a Text datablock and it
will select an Arabic-capable font if one is loaded, but it renders each
codepoint's isolated form left-to-right, ignoring joining and bidi
reordering. The result is visibly wrong to any Arabic reader — always use
the pre-rendered PNG approach above for Arabic content.
