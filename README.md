# Flow

A camera-reactive visual toy built around streamline fields.

## The look
The references for this are flow-field portraits: thousands of thin curves
following the contours of a face, bright on black. That needs three things:

1. **A smooth scalar field.** MediaPipe gives a hard person mask, whose
   gradient is zero everywhere except one pixel. The mask is combined with
   the camera's own luminance so the interior has structure, then blurred
   heavily over five passes.
2. **Bilinear sampling.** Nearest-neighbour quantises the field and lets
   tracers wander off their isoline. Measured drift dropped from 0.25 to
   0.053 once sampling was made continuous.
3. **Contour direction, not gradient direction.** A streamline follows the
   perpendicular of the gradient, which keeps it on a line of constant value.

Hundreds of tracers walk that field each frame, each drawing a short curve
and respawning. Overlapping curves accumulate additively, which is what
builds density.

Optical flow still runs, and nudges each line as it is drawn — so moving
warps the whole field rather than just adding motion on top.

## Controls
Fourteen sliders. Five presets: GRAIN, EMBER, GHOST, RIBBON, STORM.

The **bleed** slider matters most: additive light with feedback f settles at
add/(1-f), which clips to white without it.

## Requirements
Camera needs HTTPS and Safari proper. Nothing is recorded, nothing leaves the
device. The person field is a ~1MB model from a CDN, cached after first load.


## Build notes: aspect and resolution

The camera is 4:3 and a phone screen is roughly 9:19.5. Drawing the mask
across the full screen stretched it — measured, that squashed the horizontal
axis to 0.35 of its correct width, which is why the silhouette looked
compressed. Everything now uses a cover-fit: scale to fill, crop the
overflow, with an exact inverse for lookups.

Resolution raised throughout:
  sampling    160x120 -> 256x192
  field        96x72  -> 176x132   (23,232 cells, was 6,912)
  silhouette  192x144 -> 384x288

The interior of the field was nearly flat, so the only contours were on the
silhouette edge and no lines appeared inside the body. Luminance now dominates
inside the mask, taking cells with signal from 2,575 to 9,308.

That finer detail broke contour tracking until the blur was replaced with a
wide separable box blur — drift went 0.19 -> 0.108.


## Build 18 — BLOOM

A fifth mode with a completely separate render path: no lines, no dots, no
feedback. It is the same app rather than a second one so it reuses the
segmentation model already loaded.

The reference breaks into four separable parts, and it is built that way:

1. **soft field** — the person mask sampled through a drifting noise offset,
   so the silhouette warps and bleeds rather than sitting as a hard matte
2. **wide blur** — three passes at radius 4, with a much taller vertical
   kernel than horizontal. That asymmetry is the slit-scan look
3. **gradient map** — the scalar remapped through a thermal ramp: deep blue,
   violet, hot pink, orange, cream. The ramp phase cycles over time, which is
   what makes the colour continuously shift
4. **columnar bleed** — the result drawn several times at increasing vertical
   extent with falling alpha, so light bleeds up and down from the figure

Built at 96x170 as raw pixels and upscaled, which gives the softness for free
rather than paying for a large blur.

Five sliders: bloom warp, bloom smear, bloom hue, bloom gain, bloom drift.

Verified: 10 distinct colours across the ramp, the full buffer written every
frame, wide channel ranges (R 26-255, G 32-226, B 104-232), and three smear
layers composited per frame.
