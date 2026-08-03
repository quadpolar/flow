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


## Build 10 — facial features

MediaPipe Face Landmarker runs alongside the segmenter, sharing the same
fileset. 478 tracked points give real eye, lip, brow, nose and jaw geometry.

The contours are rasterised into a buffer at field resolution and folded into
the scalar field before the blur, weighted by how hard each feature should
deflect a line: eyes and lips hardest, the face oval softest. The nose bridge
and wings are drawn from explicit landmark indices, since they have no
connector set of their own.

Because the field is what streamlines follow, a raised contour at an eyelid
deflects a line exactly the way the body silhouette does — the features are
part of the same flow rather than an overlay.

Measured with deliberately flat lighting, where luminance gives nothing to
follow:

  no landmarks   : 0 cells with any usable gradient
  with landmarks : 268 cells

That is the case the landmarker exists for. Under flat or backlit conditions
the shading-driven field is empty and the face reads as a blank silhouette.

The **face detail** slider scales the effect; the PORTRAIT preset is built
around it — slow, dense, high face weight, low body weight.

Costs a second model download (cached) and a second inference per frame. If
the landmarker fails to load, the segmenter and everything else carry on.
