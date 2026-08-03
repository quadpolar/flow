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


## Build 8 — smoothness

**Aspect.** The cover-fit was using the sample buffer's 4:3 shape rather than
the video's actual dimensions. iOS often delivers portrait frames, so the mask
was being stretched horizontally — which reads as the subject looking wide.
Every buffer now sizes itself from cam.videoWidth/videoHeight on the first
frame.

**Jitter.** The field is now eased toward its new state rather than replaced.
At the default the settle time is about 0.6s, which absorbs a knock to the
camera and the segmentation edge flickering, without losing detail.

**Camera shake.** A jostle moves every flow cell at once. When more than 35%
of cells are moving together, the mean drift is subtracted — so a shake no
longer reads as the whole world lurching. There is a de-shake slider.

**Speed.** Tracer advance is now a slider rather than a constant, defaulting
to about a quarter of build 7. Hue drift was also running per-frame rather
than per-second, so it was roughly 60x too fast.

**Edges.** The silhouette was a 1-bit mask upscaled, which stair-steps. Each
output pixel now averages a small neighbourhood, giving a soft edge that
survives the scale.

Presets rebuilt for the slower feel: SILK, EMBER, GHOST, RIBBON, STORM.
