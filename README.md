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


## Build 14 — dots, mesh, and instant response

**Three render modes**, cycled by the mode button:

- **FLOW** — the streamline field, as before
- **DOTS** — all 468 tracked landmarks as points, sized and hue-shifted by
  their z depth so the face still reads as a surface rather than a flat
  constellation
- **MESH** — the same dots plus the full tessellation as a wireframe

DOTS and MESH are drawn straight from the landmarks in screen space. There is
no field, no temporal smoothing and no feedback in that path, so they track
the camera frame exactly — nothing to catch up.

**The lag in FLOW mode** came from four delays compounding: field smoothing at
0.14, mask smoothing at 0.5, face-field smoothing at 0.55, and feedback at
0.78 holding old frames on screen.

Smoothing is now adaptive. It exists to absorb jitter while you are still, but
the same easing reads as lag when you move — so it opens in proportion to how
much of the frame is actually moving:

  still        0.23s to settle
  small move   0.13s
  big move     0.07s

Previously a flat 0.53s regardless. The mask and face buffers snap harder too.
