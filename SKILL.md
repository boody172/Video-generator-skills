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

Before touching the scene, collect — in this order, since later answers
often depend on the earlier ones:

1. **Project/brand name.** Always ask first: "اسم المشروع أو البراند اللي
   هنشتغل عليه إيه؟" This is also the identifier used later for the
   same-brand-vs-new-project check in §2a.
2. **Video type and goal.** What kind of video (product ad, brand
   awareness/launch, explainer, event promo, testimonial/case study, social
   teaser, etc.) and what it needs to *achieve* (drive sales, build brand
   recall, explain a feature, announce an event...). This single answer
   drives most of the downstream creative choices — pacing, whether a
   voiceover script is likely needed (see `references/audio-and-script.md`'s
   decision table), transition energy level, and tone.
3. **Hero logo = this project's own logo, by default.** Each project the
   user brings has its own logo, and that's the one to focus on for that
   project — don't ask "standalone or partner?" as if it were open every
   time. Only raise the partner/co-branded question if the user's own
   framing signals a shared/sponsored piece (e.g. "فيديو مشترك مع..."), or
   if more than one logo file actually shows up in the attachments for a
   single project (then ask which one is the hero, per
   `references/logo-handling.md`). If this project/brand has been worked on
   earlier in the conversation (or the user confirms "نفس البراند"), reuse
   the hero logo/identity already established rather than asking again.
4. **Images** — the product/project photos. The user attaches them in chat;
   note their file paths under `/mnt/user-data/uploads/`. If no photos are
   supplied, offer to build the shot entirely in 3D (product mockup,
   backdrop, lighting) from a description of the brand/idea instead.
5. **On-screen text per image — map explicitly, never guess.** This is the
   single most common way an AI-driven edit gets caught: a caption landing
   on the wrong shot. Never infer which line of text belongs to which photo
   from context or ordering. **Preferred method: ask the total image count
   first, then have the user upload one image at a time, each paired
   immediately with its own caption/voiceover line in the same message** —
   this removes batch-mismatch risk entirely, since there's never more than
   one image-to-text pairing in flight at once. If the user instead sends
   several images together, get filename/order and exact caption paired
   1:1 for every one, and **read the full mapping back before building
   anything** (e.g. "شكل 1 (اسم الملف) → النص كذا، شكل 2 → النص كذا...
   تمام كده؟"). Language(s) per caption: Arabic, English, or bilingual
   (see `references/arabic-text-overlay.md`). Full intake
   precision protocol and the anti-"looks AI-made" QA pass before
   rendering: `references/quality-precision-checklist.md` — follow it on
   every project, not just when something looks off.
6. **Voiceover script — ask, don't assume.** Some brands/videos genuinely
   read better silent-with-music-and-captions (fashion, luxury, food,
   atmosphere-driven content); others need a spoken script to land the
   message (explainers, testimonials, service/tech pitches, anything with a
   claim or CTA that must be *heard*, not just read). Ask the user directly:
   "عايز سكريبت متقال (voiceover) ولا الفيديو يفضل صامت بموسيقى وكابشِنز؟" —
   and if they're unsure, recommend based on the brand/content type (see
   `references/audio-and-script.md` for the decision table and for how to
   actually write and deliver the script). Never silently assume either way.
7. **Music — ask every time, per project.** Always ask: "عايز أضيف موسيقى
   خلفية؟" Don't default it on or off. If yes, ask for a track (or agree on
   mood: upbeat/corporate, cinematic/emotional, minimal/luxury) so audio
   levels (music bed under voiceover, or fuller music-only mix) get set up
   right in §7.
8. **Duration** — total ad length in seconds (or per-shot duration).
9. **Aspect ratio** — vertical `9:16` (1080×1920, social) or horizontal
   `16:9` (1920×1080), or square `1:1`.
10. **Camera & transition style** — dolly-in, slow pan, orbit around
   product, static with subtle push, or "surprise me" (pick something
   clean and commercial). Pair this with a transition style from
   `references/cinematic-transitions.md` — never leave transitions as bare
   hard cuts on a "professional" request; pick match cuts / whip pans /
   speed ramps / light-leak dissolves appropriate to the pacing.

Don't block on every single field — reasonable defaults are fine (e.g.
9:16, 15–20s, gentle push-in, single focal logo, ask-first on script/music)
as long as you state the assumption back to the user.

## 2b. Recurring video type: project progress / milestone update

A common goal (§2 item 2) is showing a client/project owner how far a
project has come — completed work so far, company logo attached to that
work, meant to build confidence in the project's progress. Treat this as
its own pattern, distinct from a straight product ad:

- **Structure**: order shots to read as a progression, not just a gallery —
  chronological by completion stage if the user can tell you the order
  (foundations → structure → finishing, or whatever the project's actual
  phases are), not just the order files were uploaded in. Ask which order
  reflects real progress if it isn't obvious from filenames/context.
- **Per-shot labeling**: since this is about *progress*, a short label per
  shot naming the phase/date/milestone (e.g. "المرحلة الأولى — التأسيس",
  or a date) reads far more credibly to an owner than an unlabeled photo —
  ask whether the user has phase names/dates for each image, and map them
  with the same 1:1 rigor as any other caption (§2 item 5).
- **Logo placement**: the company logo functions here almost like a stamp
  of ownership/certification on the completed work — a persistent corner
  mark throughout (per `references/logo-handling.md`) reads better than a
  logo that only shows up in a bumper, since the point is "this is our
  work."
- **Tone/pacing**: measured and confident, not hyped — steady dolly/pan
  moves, crossfade or a simple wipe between phases (a wipe reads well here
  as "moving forward in time"); skip glitch/whip-pan style transitions,
  which suit energetic consumer ads, not a progress report to a stakeholder.
- **Voiceover vs. captions**: per `references/audio-and-script.md`'s
  table this sits closest to the real-estate/industrial/B2B row — a short
  voiceover summarizing progress (or strong on-screen phase labels if no
  voiceover) plus a restrained music bed reads as the professional default;
  confirm with the user rather than assuming.
- **Closing shot**: end on the logo (closing bumper) over the most complete
  /impressive shot available, optionally with a one-line status caption
  ("تم إنجاز X% من المشروع" or similar, only if the user gives you the
  real figure — never invent a completion percentage or claim).

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
