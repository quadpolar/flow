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


## Build 22 — softening the interior only

Build 21 fixed the interior being erased, and overshot: at full sharpness the
luminance resolves skin texture, which reads as a processed photograph rather
than a form.

Softening it cannot be done with the existing blur, because that one shapes
the silhouette — turning it up destroys the outline. The shading now gets its
own blur with its own radius, applied after the silhouette blur and
independent of it.

Measured against a test luminance carrying both coarse facial structure
(period about 39 cells) and fine skin texture (about 4 cells):

  soften 0   variation 116.1   everything, including texture
  soften 1    98.1
  soften 2    44.6             texture gone, features intact
  soften 3    31.1             the default
  soften 5    27.5
  soften 8     0.1             features gone too

The local-contrast pass is also scaled to a quarter in bloom mode. It exists
to sharpen fine detail for the streamlines, which is the opposite of what
this mode wants.

The background no longer carries the room's luminance at all — it was leaking
shelves and furniture into the field.

New slider: **bloom soften**.
