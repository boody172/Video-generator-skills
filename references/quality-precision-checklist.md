# Quality & Precision Checklist — Intake and Pre-Render QA

The goal is a video a client can't distinguish from one cut by a human
editor/cinematographer. That requires precision at intake (so nothing is
mismatched by construction) and a deliberate QA pass before the final
render (so nothing that "looks AI-made" ships unnoticed).

## Part A — Intake precision (before touching Blender)

Never infer, assume, or pattern-match any of the following. If it isn't
stated explicitly, ask.

1. **Image ↔ caption mapping.** Ask the total image count up front, then
   have the user upload **one image at a time, each with its caption/
   voiceover line given in the same message** — this is the preferred
   protocol because it makes a batch mismatch structurally impossible
   (only one pairing is ever in flight). Track progress explicitly
   ("صورة 2 من 5 — استلمتها ✅") so nothing gets lost mid-sequence. If the
   user instead sends several photos together, get every exact caption
   paired 1:1 by filename or explicit order — never "the third image
   probably goes with the third line." Read the full mapping back to the
   user as a numbered list and get an explicit confirmation before
   building any planes or text. If the user supplies images and captions
   separately (e.g. images in chat, captions in a paragraph), do not
   guess the pairing from the order they arrived in — ask.
2. **Language per caption**, not just per project — a bilingual project
   can still have some captions pure-English and others pure-Arabic;
   confirm per line if mixed.
3. **Which file is the hero logo**, if more than one logo file is present
   (see `references/logo-handling.md`).
4. **Exact on-screen spelling/formatting** — brand names, product names,
   prices, taglines: copy the user's exact text verbatim into the caption
   render step. Do not "clean up," retranslate, auto-correct, or
   paraphrase copy the user gave you — a respelled brand name or an
   auto-corrected product name is a real client-facing error.
5. **Any brand color/font constraints** — if the user has brand colors or
   a specific font for captions/logo, use exactly those; don't substitute
   a similar-looking default because it wasn't attached.
6. **Ambiguous instructions** — if a request could reasonably mean two
   different things (e.g. "ضيف اللوجو في الآخر" could mean *the closing
   bumper* or *a persistent mark throughout*), ask which, rather than
   picking one silently.

When in doubt on any of the above, ask a short, specific question rather
than proceeding on a best guess — a wrong guess here is a mismatched
caption or wrong logo baked into a render that takes time to redo.

## Part B — Pre-render QA pass (on the preview frame(s))

Run this against every still-frame preview before committing to a full
animation render. This is what catches the "looks AI-generated" tells a
real editor would never let through:

- [ ] **Caption-to-image match** — re-verify against the confirmed mapping
      from Part A on the actual rendered frame, not just in code.
- [ ] **Logo integrity** — not stretched/skewed off its original aspect
      ratio, not pixelated (check the source PNG resolution is high enough
      for its render size), edges clean (no visible white/black fringing
      from a bad alpha channel).
- [ ] **Text legibility & placement** — sufficient contrast against its
      background (add a subtle drop shadow or semi-opaque backing bar if a
      caption sits on a busy part of the photo), not clipped by frame
      edges, not overlapping the logo or another caption.
- [ ] **Arabic rendering** — letters correctly joined and right-to-left
      (per `references/arabic-text-overlay.md`), no isolated/backwards
      glyphs. Zoom into the actual rendered pixels, don't just trust the
      generation step ran without error.
- [ ] **Lighting/shadow consistency** — if photos were shot under
      different lighting, either grade them toward a consistent look or
      accept the difference deliberately (real productions do this too) —
      but never leave one shot jarringly blown-out or flat next to well-lit
      others by accident.
- [ ] **Camera motion smoothness** — no jitter or popping between
      keyframes; scrub a few frames around each keyframe to confirm the
      easing reads as smooth, not mechanical/linear (see the easing note
      in `references/cinematic-transitions.md`).
- [ ] **Transition timing** — the transition doesn't cut off part of a
      word/caption mid-fade, and doesn't linger so long it feels like a
      stall.
- [ ] **Color grade consistency** across every shot (same lift/gamma/gain
      or curve applied timeline-wide, not per-shot).
- [ ] **No stray default-scene leftovers** — Blender's default Cube/Light/
      Camera objects removed or repurposed, not accidentally left visible
      in a corner of frame.
- [ ] **Audio sync** (if voiceover is in use) — captions/on-screen text
      timed to appear with their spoken line, not drifting ahead/behind.
- [ ] **Aspect ratio/safe area** — key text isn't sitting right at the
      extreme edge where a platform's UI (e.g. Instagram/TikTok overlay
      buttons) would cover it on a 9:16 delivery.

Only after this pass reads clean — and after telling the user what was
checked — proceed to the full animation render.

## Part C — When something is genuinely ambiguous mid-build

If a decision comes up that intake didn't cover (e.g. the user's photos
have very different aspect ratios and cropping one will cut off part of
the product), stop and ask rather than silently picking a resolution.
Flag it plainly: what the ambiguity is, and the 1-2 reasonable options,
so the user picks rather than discovering the choice after a full render.
