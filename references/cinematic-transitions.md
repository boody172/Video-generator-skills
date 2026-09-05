# Editor-Grade Transitions in Blender

Goal: cuts between shots should feel like they came out of an editor
working in After Effects / Premiere / DaVinci — never a bare hard cut on
every single change, and never a random grab-bag of gimmicks either. Pick
1–2 transition types per project and repeat them consistently; that
repetition (not variety) is what reads as "produced" rather than
"experimental."

All of these are built two ways depending on what's being transitioned:
**(A) compositor node setups** driving cross-fades/color/blur effects
between two shot strips, or **(B) keyframed object transforms** on the
image planes/camera themselves. Use whichever is simpler for the specific
transition.

## Setting up the base: compositor cross-fade rig

For any dissolve-family transition, put each shot's image plane on its own
render layer (or simply keyframe two overlapping planes' material alpha),
then in the Compositor:

```python
import bpy
scene = bpy.context.scene
scene.use_nodes = True
tree = scene.node_tree
tree.nodes.clear()

rlayers_a = tree.nodes.new("CompositorNodeRLayers")
rlayers_b = tree.nodes.new("CompositorNodeRLayers")  # from a second view layer, or reuse alpha-over on planes
alpha_over = tree.nodes.new("CompositorNodeAlphaOver")
composite = tree.nodes.new("CompositorNodeComposite")

tree.links.new(rlayers_a.outputs["Image"], alpha_over.inputs[1])
tree.links.new(rlayers_b.outputs["Image"], alpha_over.inputs[2])
tree.links.new(alpha_over.outputs["Image"], composite.inputs["Image"])

# Animate the fac (blend factor) across the transition's frame range
alpha_over.inputs["Fac"].default_value = 0.0
alpha_over.inputs["Fac"].keyframe_insert("default_value", frame=transition_start)
alpha_over.inputs["Fac"].default_value = 1.0
alpha_over.inputs["Fac"].keyframe_insert("default_value", frame=transition_end)
```

In practice it's often simpler to keep everything in one 3D scene and just
keyframe each shot plane's material alpha (via a mix shader or the
object's `color` alpha channel) directly instead of a two-layer compositor
rig — do that when it's not worth the extra render-layer setup.

## 1. Crossfade / dissolve (safe default, always works)

Overlap the outgoing and incoming shot planes for ~8–15 frames (at 30fps,
roughly 0.3–0.5s), fade alpha 1→0 on the outgoing, 0→1 on the incoming,
eased (not linear — see easing note below).

## 2. Light-leak dissolve (premium/lifestyle feel)

Same as the crossfade, but composite a warm light-leak/flare overlay
(an emissive plane with a soft radial gradient texture, additive blend
mode) at full brightness right at the crossfade's midpoint, fading in/out
around it. Gives the "shot on film" feel common in lifestyle/fashion ads.

```python
leak = bpy.data.objects["LightLeakPlane"]
leak.data.materials[0].node_tree.nodes["Emission"].inputs["Strength"].default_value = 0.0
# keyframe strength: 0 at transition_start, peak (~3-5) at midpoint, 0 at transition_end
```

## 3. Whip pan (energetic, good for fashion/sport/hype reels)

Snap-pan the camera hard in one direction while motion-blurred, timed so
the outgoing shot exits and the incoming shot enters mid-blur:

```python
cam = bpy.data.objects["Camera"]
scene.render.use_motion_blur = True
scene.render.motion_blur_shutter = 0.8   # higher = more blur streak

cam.rotation_euler[2] = 0.0
cam.keyframe_insert("rotation_euler", index=2, frame=transition_start)
cam.rotation_euler[2] = 1.4   # ~80 degrees, fast whip
cam.keyframe_insert("rotation_euler", index=2, frame=transition_start + 6)
cam.rotation_euler[2] = 0.0   # snap back for the next shot, or continue into its own motion
cam.keyframe_insert("rotation_euler", index=2, frame=transition_start + 7)
```
Set the two keyframes' interpolation to `EASE_IN`/`LINEAR` (fast, not
smoothed) so the whip actually reads as a snap, not a slow swing —
see the easing note below on setting interpolation via `fcurve.keyframe_
points[i].interpolation`.

## 4. Speed ramp (emphasis cut, product reveals)

Ramp the scene's playback speed down into a cut for emphasis (slow-mo just
before impact/reveal, then snap to normal speed on the next shot). Control
via the **Time Remap** in the sequencer, or by keyframing a
`Speed Control` compositor node / retiming the animation curve itself:

```python
# Simplest: keyframe an object's animation "speed" via a Time Warp modifier
# on the relevant f-curve, or just hand-keyframe closer-spaced keys near
# the cut point so the apparent motion slows there.
```
Use sparingly — one speed ramp per video, right before the hero
reveal/logo bumper, is the professional amount; more than that reads as
gimmicky.

## 5. Glitch wipe (tech/gaming/energetic brand feel)

Composite a short (3–5 frame) RGB-channel-split + block-displacement burst
right at the cut point:

```python
# Compositor: Translate nodes on separate R/G/B split copies of the image,
# offset by a few pixels in opposite directions, held for 3-5 frames only,
# then snap back to normal for the incoming shot.
```
Only use this if the brand is explicitly tech/gaming/high-energy — it's
wrong for anything premium/luxury/calm.

## 6. Match cut (most "director" of all, use when shots allow it)

Time the outgoing shot's camera move and the incoming shot's camera move so
a shape, motion direction, or subject position lines up across the cut
(e.g. both shots end/begin with the subject centered and moving the same
screen direction). This needs no compositor trick at all — it's purely
about *choosing which frame you cut on* and matching the camera keyframes
of both shots so the motion continues unbroken. When the source photos/
framing allow it, this reads as the most premium option; use it for the
key hero-product reveal cut if only one "special" transition is wanted.

## Easing — never leave transitions on linear interpolation

By default Blender inserts `BEZIER` keys, which is usually fine, but for
snappy transitions (whip pans, glitch bursts) force faster easing
explicitly:

```python
fcurve = cam.animation_data.action.fcurves.find("rotation_euler", index=2)
for kp in fcurve.keyframe_points:
    kp.interpolation = 'EASE_IN' if kp.co[0] == transition_start else 'EASE_OUT'
```
For camera dolly/pan *within* a shot (not at a cut), prefer smooth
`EASE_IN_OUT` — real camera operators don't start/stop moves abruptly.

## Color grade consistency across cuts

Apply one grade to the whole composite (after all shot/transition nodes,
right before the `Composite` output node) rather than per-shot — a Color
Balance node (lift/gamma/gain) or a Curves node with a subtle S-curve for
contrast. Keep it identical across the whole timeline; inconsistent grade
between shots is the fastest way to look amateur, and is the single most
common failure mode to check for before rendering the final pass.
