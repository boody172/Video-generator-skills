# Logo Handling — Single Focal Logo by Default

## Default: one hero logo

Unless the user explicitly asks for a multi-brand/co-branded video, a
project should showcase **exactly one logo** — the primary brand mark. This
is the professional norm: a client-facing ad that flashes several logos
reads as cluttered and undermines brand recall.

If the project folder/assets actually contain more than one logo file
(e.g. an old version, a sub-brand mark, a partner's logo), **ask the user
which one is the hero logo** before building anything — don't guess, and
don't include the others "just in case."

```
"شايف أكتر من ملف لوجو في اللي بعتهولي — عايز أستخدم أنهي واحد كلوجو أساسي
للفيديو؟"
```

## How the hero logo appears in the edit

A clean, repeatable pattern used across professional commercials:

1. **Opening bumper (optional, 1–2s)** — logo alone on a clean background
   or subtle particle/light sweep, animated in (scale + fade, or a wipe
   reveal), then holds briefly before cutting to the first shot.
2. **Persistent corner mark** — a small, semi-transparent version of the
   logo in a fixed corner (commonly bottom-right or top-left) for the
   *entire* runtime after the bumper, so the brand is present without
   competing with the product footage.
3. **Closing bumper (1–2s)** — logo returns full-frame, often with a
   tagline/CTA caption beneath it, held on the last frame(s) so viewers can
   read it before the video ends.

Implementation in Blender:
- Bumper: logo image plane centered in frame, keyframe `scale`/`alpha`
  (via a mix node or object color) for the animate-in, `location`/`scale`
  keyframes on the *camera* or logo plane for a subtle push.
- Corner mark: a small always-visible plane, `location` fixed relative to
  camera space (e.g. parent it to the camera, or precompute its screen-space
  position for the given resolution), material alpha around 0.6–0.85 so it
  doesn't overpower the footage.
- Use the **same logo asset** (single source PNG, ideally with alpha) for
  all three placements — don't re-derive or re-crop it per placement, to
  avoid visible inconsistency in edge anti-aliasing or color.

## When the user *does* want multiple logos

Only build a multi-logo layout when explicitly asked (e.g. a co-branded
partnership spot, an event sponsor reel, an agency showreel across several
clients). In that case:
- Ask which logo is primary (gets the bumper treatment) vs. secondary
  (appears in a shared "in partnership with" / sponsor strip, typically in
  the closing bumper only, not as a persistent corner mark for the whole
  runtime).
- Keep secondary logos visually equal in size/treatment to each other
  (same plane size, same alpha) so no unintended hierarchy is implied unless
  the user asks for one.
