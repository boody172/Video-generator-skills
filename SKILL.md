---
name: video-generator-skills
description: >
  Produce professional, cinematic video ads/commercials/promos from scratch
  inside the user's LOCAL Blender installation via the BlenderMCP connector
  (raw bpy Python scripting) — completely free, no cloud AI credits, no
  subscriptions — for any brand, product, or idea across any industry.
  Combines user-supplied photos, a single focal brand logo, optional
  voiceover script and/or music, and Arabic and/or English text overlays,
  cut together with editor-grade transitions (match cuts, whip pans, speed
  ramps, light leaks, glitch wipes, film-grain grades) so the result reads
  like it came out of a cinematographer/editor working in After Effects,
  Premiere Pro, or DaVinci Resolve — not a slideshow. Use this whenever the
  user asks to build a video ad, commercial, promo reel, product video, or
  any short cinematic video from still images + a logo (+ captions/voice)
  using Blender, or says things like "اعملي فيديو منتاج احترافي", "فيديو
  دعائي", "أنيميشن للمنتج", or mentions driving Blender directly (not
  through Higgsfield/Magnific/cloud generation). Also covers verifying the
  local Blender connection (localhost:9876) before any work starts.
---

# Video Generator Skills — Professional Local Blender Production

## 0. What this is (and isn't)

This skill drives the user's **own local Blender app** through
`Blender:execute_blender_code` / `bl_execute` (raw `bpy` Python, via the
BlenderMCP add-on listening on `localhost:9876`). Because it's just
remote-controlling software the user already owns, **this path costs
nothing** — no Higgsfield or Magnific credits, no cloud render fees.
Rendering happens on the user's own machine/GPU, so render time depends on
their hardware, not on any API quota.

Keep this distinction explicit any time cost comes up:
- Driving Blender via `bl_execute` → free, always.
- Any Higgsfield/Magnific AI generation call (new AI images/video, upscaling,
  etc.) → costs credits. Only bring those in if the user explicitly asks for
  AI-generated shots in addition to their own photos.

The bar for every deliverable is **client-ready** — this has to look good
enough to hand to a paying client in any niche (fashion, food, real estate,
tech, industrial, healthcare, etc.), not a rough draft. That means: correct
color grading, no jittery camera moves, readable typography, clean audio
mix, and transitions that feel intentional — never a plain hard cut between
every shot.

## 1. Always check the connection FIRST

```python
import bpy
result = {"blender_version": bpy.app.version_string, "scene": bpy.context.scene.name,
          "objects": [o.name for o in bpy.data.objects]}
```
Call via `Blender:execute_blender_code` / `bl_execute`.

- **Success** → proceed to §2.
- **Error "Cannot connect to Blender at localhost:9876"** → tell the user to:
  1. Open Blender itself (it must be actively running).
  2. Press `N` in the 3D Viewport to open the side panel.
  3. Find the **BlenderMCP** tab and click **Start Server / Connect**.
  4. Confirm it's listening on port 9876.
  Then retry the ping. Don't proceed to scene work until this succeeds.

## 2. Gather the inputs (ask once, up front)

Before touching the scene, collect:

1. **Images** — the product/project photos. The user attaches them in chat;
   note their file paths under `/mnt/user-data/uploads/`. If no photos are
   supplied, offer to build the shot entirely in 3D (product mockup,
   backdrop, lighting) from a description of the brand/idea instead.
2. **Logo(s)** — how many brand logos exist in this project. **Default
   behavior: focus on a single logo** (the primary brand mark) — it appears
   as a clean, consistent watermark/bumper (intro sting and/or corner mark)
   throughout. Only show more than one logo if the user explicitly asks for
   a co-branded/multi-partner video — see `references/logo-handling.md` for
   how to pick the "hero" logo and how to handle a multi-logo request when
   asked.
3. **On-screen text per image — map explicitly, never guess.** This is the
   single most common way an AI-driven edit gets caught: a caption landing
   on the wrong shot. Never infer which line of text belongs to which photo
   from context or ordering. For every image, get its filename/order and
   its exact caption paired 1:1, and **read the mapping back to the user
   before building anything** (e.g. "شكل 1 (اسم الملف) → النص كذا، شكل 2 →
   النص كذا... تمام كده؟"). Language(s) per caption: Arabic, English, or
   bilingual (see `references/arabic-text-overlay.md`). Full intake
   precision protocol and the anti-"looks AI-made" QA pass before
   rendering: `references/quality-precision-checklist.md` — follow it on
   every project, not just when something looks off.
4. **Voiceover script — ask, don't assume.** Some brands/videos genuinely
   read better silent-with-music-and-captions (fashion, luxury, food,
   atmosphere-driven content); others need a spoken script to land the
   message (explainers, testimonials, service/tech pitches, anything with a
   claim or CTA that must be *heard*, not just read). Ask the user directly:
   "عايز سكريبت متقال (voiceover) ولا الفيديو يفضل صامت بموسيقى وكابشِنز؟" —
   and if they're unsure, recommend based on the brand/content type (see
   `references/audio-and-script.md` for the decision table and for how to
   actually write and deliver the script). Never silently assume either way.
5. **Music — ask every time, per project.** Always ask: "عايز أضيف موسيقى
   خلفية؟" Don't default it on or off. If yes, ask for a track (or agree on
   mood: upbeat/corporate, cinematic/emotional, minimal/luxury) so audio
   levels (music bed under voiceover, or fuller music-only mix) get set up
   right in §7.
6. **Duration** — total ad length in seconds (or per-shot duration).
7. **Aspect ratio** — vertical `9:16` (1080×1920, social) or horizontal
   `16:9` (1920×1080), or square `1:1`.
8. **Camera & transition style** — dolly-in, slow pan, orbit around
   product, static with subtle push, or "surprise me" (pick something
   clean and commercial). Pair this with a transition style from
   `references/cinematic-transitions.md` — never leave transitions as bare
   hard cuts on a "professional" request; pick match cuts / whip pans /
   speed ramps / light-leak dissolves appropriate to the pacing.

Don't block on every single field — reasonable defaults are fine (e.g.
9:16, 15–20s, gentle push-in, single focal logo, ask-first on script/music)
as long as you state the assumption back to the user.

## 2a. New video request: same brand, or a new one — always check first

The moment the user asks for another video (a new request in the same
conversation, or a returning one later), **don't assume**. Decide which of
these two paths applies before asking anything else:

- **New brand / new project** → treat it as a completely fresh intake.
  Discard every previous answer (old logo, colors, tone, duration, aspect
  ratio, transition choice, script/music decision) — none of it carries
  over by default. Run the full §2 question set again from scratch as if
  this were the first video ever requested.
- **Same brand, new footage/attachments** → if the brand is recognizable
  from earlier in the conversation (or the user says "نفس البراند"),
  **confirm it explicitly by name** first — "يعني ده لسه لبراند [X] اللي
  شغالين عليه؟" — don't silently assume a match just because the request
  sounds similar. Once confirmed, it's fine to reuse the brand-level
  decisions already locked in (hero logo, brand colors/font, tone,
  script-vs-music decision, aspect ratio) *if the user doesn't say
  otherwise* — but still treat the following as fresh, every time:
  - **The new images/attachments themselves** — run the full image↔caption
    mapping confirmation from `references/quality-precision-checklist.md`
    on this new batch; never assume the new photos map the same way the
    old ones did.
  - **The transition style** — ask which transitions to use on this new
    cut (e.g. "عايز نفس الـ transitions اللي استخدمناها ولا نجرب حاجة
    مختلفة المرة دي؟") rather than silently reusing the last video's
    choice — the user's own phrasing above flags this explicitly as
    something that changes per video even on the same brand.

If it's ambiguous whether this is the same brand or not, ask — don't guess
either direction.

## 3. Getting user files INTO Blender

Files the user uploads land in Claude's sandbox
(`/mnt/user-data/uploads/...`), which is **not** the same machine as their
running Blender. `bl_execute` only runs Python text — it can't read Claude's
sandbox filesystem directly. To get an image onto the user's disk where
Blender can load it, base64-encode the file content in the sandbox and have
the injected Blender-side code decode and write it locally, then load it:

```python
# (sandbox) read + base64-encode the file first, then:
code = f'''
import base64, tempfile, os, bpy
data = base64.b64decode("{b64_string}")
path = os.path.join(tempfile.gettempdir(), "{filename}")
with open(path, "wb") as f:
    f.write(data)
img = bpy.data.images.load(path)
result = {{"loaded": img.name, "path": path}}
'''
```
Call this through `Blender:execute_blender_code`. Do this once per image and
for the logo. Keep images reasonably sized (downscale very large photos
first) since base64 payloads bloat the tool call.

## 4. Arabic text overlays — don't use Blender's native Text object for Arabic

Blender's built-in Text object does **not** shape Arabic correctly (no
letter-joining, no RTL layout). Render Arabic/bilingual captions to a
transparent PNG first (arabic_reshaper + python-bidi + Pillow), then import
that PNG as an image plane. Full technique, fonts, and bilingual handling:
`references/arabic-text-overlay.md`. Pure English captions can use
Blender's native `FONT` Text object directly.

## 5. Building the scene

Typical structure per shot:
- An **image plane** (mesh plane with the photo as an emission/base-color
  texture) sized to the target aspect ratio.
- The **hero logo** composited as its own plane — a short branded
  intro/outro bumper plus a subtle persistent corner mark is the
  professional default (see `references/logo-handling.md`).
- The **caption PNG/Text** positioned per the layout (bottom-third is the
  safe commercial default), timed to the voiceover line it supports if a
  script is in use.
- **Camera** animated with keyframes across the shot's frame range — subtle,
  controlled moves (slow dolly/push, gentle pan, orbit) with a touch of
  depth-of-field (`camera.data.dof.use_dof = True`) for a photographic look.
  Never a static, motionless frame on a "professional" request.
- **Lighting**: 3-point setup or HDRI environment texture — never a flat
  default-gray render.
- **Transitions between shots** — this is what separates a slideshow from a
  produced ad. Build them via the compositor (crossfades, light-leak
  dissolves, glitch wipes) or keyframed plane transforms (whip pan, speed
  ramp, match cut on shape/motion). Full recipes, node graphs, and
  frame-timing patterns: `references/cinematic-transitions.md`. Pick 1–2
  transition types and use them consistently — professional edits repeat a
  small transition vocabulary rather than using a different gimmick every
  cut.
- **Color grade**: apply a consistent grade across the whole timeline via
  compositor Color Balance / Curves nodes (or a subtle LUT-style curve) so
  every shot matches — inconsistent color between shots is the #1 tell of
  an amateur edit.

## 6. Audio: script, voiceover, and music

Follow `references/audio-and-script.md` for:
- The decision framework for script-vs-silent per brand/content type.
- How to write a tight, on-brand voiceover script sized to the chosen
  duration (words-per-second budget).
- Where to source/place a voiceover audio file (user-provided, or flag that
  Claude cannot itself synthesize speech inside this free local-Blender
  path — recommend the user's own TTS/recording, or note that a
  Higgsfield/Magnific audio tool is available but costs credits, only if
  they want that instead).
- Mixing music under voiceover (ducking) vs. music-only mixes, using
  Blender's Video Sequence Editor (`bpy.data.scenes[...].sequence_editor`)
  for the audio strips and level keyframes.

## 7. Render settings

- Set `render.resolution_x/y` to match the chosen aspect ratio.
- Set `render.fps` (30 is a safe default for social video).
- Set `frame_start`/`frame_end` to match total duration × fps.
- Prefer `EEVEE`/`EEVEE Next` for speed unless the user wants Cycles-quality
  lighting and is fine waiting longer.
- Output format `FFMPEG`/`MPEG4`, sensible bitrate (high enough to avoid
  compression artifacts on a client deliverable), output path to a clear
  local folder (e.g. `~/Desktop/ad_output.mp4`). The file lives on **their**
  machine — Claude cannot download or preview it directly, so confirm the
  path with the user.

## 8. Rendering

`bpy.ops.render.render(animation=True)` via `bl_execute` runs synchronously
on the user's machine and can take a long time — warn them before a full
render, and always offer a single representative-frame preview render first
(`animation=False`) to check framing/lighting/text/logo placement and color
grade before committing to the full render.

Before the full render, run the anti-mistake QA pass in
`references/quality-precision-checklist.md` against the preview frame(s) —
confirm every caption is on its intended image, the logo isn't distorted,
and nothing reads as an obvious AI-generated artifact.

## 9. Delivering the result

The final MP4 lives on the user's own computer at the output path set in
§7 — tell them exactly where to find it. Offer to iterate (camera timing,
transition style, text/logo position, color grade, audio mix) by re-running
the relevant `bl_execute` steps.

## Quick checklist for a new video request

- [ ] Ping Blender connection — confirm connected
- [ ] Collect images, confirm single hero logo (or explicit multi-logo ask)
- [ ] Map every caption to its exact image 1:1 and read the mapping back to
      the user for confirmation before building (+ language per caption)
- [ ] Ask: voiceover script or silent-with-music? (recommend per brand if unsure)
- [ ] Ask: add background music? get track/mood if yes
- [ ] Confirm duration, aspect ratio, camera + transition style
- [ ] Transfer images/logo into Blender (base64 → local temp path)
- [ ] Render Arabic captions as PNGs where needed; native Text for English-only
- [ ] Build planes, hero logo bumper/mark, camera animation, lighting
- [ ] Apply editor-grade transitions between shots + a consistent color grade
- [ ] Set up audio (voiceover/music/ducking) in the VSE if requested
- [ ] Set render resolution/fps/frame range/output path
- [ ] Do a still-frame preview render, then run the QA pass in
      `references/quality-precision-checklist.md` before the full render
- [ ] Tell the user the local output path when done
