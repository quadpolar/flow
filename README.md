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


## Build 13 — the face surface

Two things were limiting facial detail.

**Only outlines were being drawn.** The landmarker was supplying eyes, brows,
nose, lips and the face oval — roughly 220 line segments describing feature
BOUNDARIES. FACE_LANDMARKS_TESSELATION is the full triangulated mesh over all
468 points, about 2,600 connections covering the actual surface: cheeks, jaw,
forehead, the bridge of the nose. That is now drawn first, very faint, under
the features. There is a **face mesh** slider for it.

**The outlines were swamping everything else.** A feature ridge sat at roughly
1.4 in field terms against an interior of 0.3-0.7, so streamlines collapsed
onto the ridges and the surface between them stayed empty — the face read as a
mask with holes cut in it. Ridge weights are roughly halved and the luminance
term is up from 1.5 to 2.4, so real skin shading competes.

Presets rebuilt around RIBBON, which is now the default state. ETCH and SKIN
are new: both lean hard on the mesh and local contrast rather than the feature
outlines.

## On depth
There is no way to reach the TrueDepth sensor from Safari. iOS exposes no
depth API to the web at all, and no permission unlocks it — a native app can,
a web app cannot. Everything here is derived from the colour image: person
segmentation, face landmarks, and local contrast.
