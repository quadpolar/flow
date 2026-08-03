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


## Build 11 — actual detail

The field was being blurred at radius 3 over two passes, an effective support
of 13 cells. At a field width of 176 with a face spanning 40% of frame, that
smoothed across 19% of the face — which is why features read as generalised
no matter how good the landmarks were.

- field resolution 176x132 -> 288x216, sampling 256x192 -> 384x288
- the blur radius is now the **smoothness** slider, defaulting to 1.5 instead
  of 3. The gradient stencil scales with it, so lowering it sharpens the
  field and the streamline response together.
- a **contrast** slider adds a local-contrast pass: a blurred copy of the
  luminance is subtracted from itself, which surfaces cheekbone, nostril,
  eyelid and lip shading. A global range stretch cannot recover those,
  because they are small differences sitting on a large gradient.
- 1100 lines instead of 820, thinner, with a longer step to suit the finer
  field.

## Scrubbing
The panel drops to near-transparent while a slider is being dragged, and
returns 450ms after release — so a change can be seen while it is being made.

## A NaN guard
Buffers are reallocated when the video aspect is detected, and a tracer
surviving across that could read stale geometry; one non-finite value then
propagated through every subsequent step. Positions, contour vectors and flow
samples are all checked now. Verified over repeated runs: no non-finite
coordinates reach the canvas.
