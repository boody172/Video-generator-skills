# Audio: Script Decision, Writing, and Mixing

## 1. Always ask — never assume — on both script and music

Two separate yes/no questions, every project, regardless of niche:

1. "عايز سكريبت متقال (voiceover) في الفيديو ولا يفضل صامت بكابشِنز وموسيقى؟"
2. "عايز أضيف موسيقى خلفية؟"

These are independent — a video can have voiceover + music, voiceover only,
music only (no spoken script), or neither (captions carry everything).

## 2. If the user is unsure — recommend by content type

Use this as a starting recommendation, not a rule — state it as a
suggestion and let the user confirm or override:

| Content type | Recommended default | Why |
|---|---|---|
| Fashion / luxury / beauty | Silent + music, captions minimal | Mood and visuals sell; a voice can feel like it's undercutting the aspirational tone. |
| Food / hospitality / travel | Silent + music, light caption text | Sensory/atmosphere-driven; music carries pacing. |
| Tech / SaaS / service pitch | Voiceover + subtle music bed | Needs a clear spoken claim/value prop + CTA that a viewer might not read in full. |
| Real estate / industrial / B2B | Voiceover (or strong on-screen text) + music bed | Specs/claims need to be *heard and read* — redundant channels aid retention. |
| Testimonial / case study | Voiceover (often the actual client's words) + very light music | Authenticity matters more than polish; music must stay under the voice. |
| Event / sponsor reel / hype | Silent + upbeat music, bold kinetic captions | Energy and pace over explanation. |

If genuinely unclear which bucket the brand fits, default to **music +
captions, no voiceover** — it's the safer, less-invasive choice, and is
easy to upgrade later.

## 3. Writing the voiceover script (when one is wanted)

- **Budget words to the duration.** Natural spoken pace is roughly
  2.2–2.6 words/second for a clear, unhurried commercial read (slower for
  Arabic MSA, which tends to run a bit more measured). For a 20s spot,
  that's roughly 45–55 words total — leave room for pauses, don't fill
  every second with talking.
- **Structure**: hook (first 2–3s) → value/benefit (middle) → CTA/brand
  line (last 2–3s, timed to land as the closing logo bumper appears).
- **One idea per sentence.** Short, punchy lines cut cleanly to match shot
  changes — write the script *shot by shot* so each line has an obvious
  home in the timeline, rather than one paragraph you chop up after.
- **Match the brand voice** given by the user (formal/corporate vs.
  casual/friendly vs. premium/minimal) — ask if it isn't obvious from the
  brief.
- For Arabic scripts, write in the register the brand actually uses in its
  other marketing (Modern Standard Arabic vs. a specific dialect) — don't
  assume MSA by default if the user's other materials are colloquial.

Deliver the script to the user as text first for approval before recording/
timing — script changes are cheap before voice recording, expensive after.

## 4. Getting the actual voiceover audio

This skill's Blender path is **free and local**, and Blender itself does
not synthesize speech. Options, in order of fit:
1. **User records/provides their own voiceover file** (most common —
   ask for a WAV/MP3, transfer it into Blender's VSE the same base64 way as
   images, per SKILL.md §3).
2. **User has their own TTS tool** — same as above, they hand you the
   rendered audio file.
3. **A Higgsfield/Magnific TTS tool** (`audio_tts` or equivalent) can
   generate a voice — but this is a paid/credit-consuming path outside the
   free local-Blender workflow. Only offer this if the user explicitly asks
   for AI-generated narration and understands it costs credits; state the
   cost distinction clearly before calling it (see SKILL.md §0).

## 5. Mixing in Blender's Video Sequence Editor (VSE)

```python
import bpy
scene = bpy.context.scene
scene.sequence_editor_create()
se = scene.sequence_editor

voice = se.sequences.new_sound("voiceover", filepath="/tmp/voiceover.wav",
                                channel=2, frame_start=1)
music = se.sequences.new_sound("music", filepath="/tmp/music.mp3",
                                channel=1, frame_start=1)

# Duck the music under the voiceover: keyframe the music strip's volume
music.volume = 1.0
music.keyframe_insert("volume", frame=1)
music.volume = 0.25          # ducked while voice is speaking
music.keyframe_insert("volume", frame=voice.frame_start + 5)
music.volume = 0.25
music.keyframe_insert("volume", frame=voice.frame_final_end - 5)
music.volume = 1.0           # back up once voice ends
music.keyframe_insert("volume", frame=voice.frame_final_end + 15)
```

- **Voiceover + music**: duck music to ~20–30% under speech, back to full
  in gaps and at the end, as above.
- **Music-only**: keep music at a comfortable full level (avoid clipping —
  check normalized peak isn't near 0dB if the source file is already hot),
  and fade in/out over the first/last ~1s (`volume` keyframes) rather than
  starting/stopping abruptly.
- Always render with `scene.render.ffmpeg.audio_codec = 'AAC'` (or the
  current Blender equivalent) so the exported MP4 actually carries the
  audio track — verify the rendered file has audio before telling the user
  it's done.
