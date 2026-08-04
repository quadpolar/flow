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


## Build 24 — real depth

A sixth mode. Depth Anything V2 Small, loaded from a CDN through
transformers.js, inferring dense per-pixel depth from the colour image.

To be exact about what this is: it is a neural network predicting depth from
an ordinary camera frame. No depth sensor is involved. iOS does not expose
TrueDepth to Safari and no permission unlocks it — that requires a native
app. This is the same category as the face mesh z, both predictions from
colour; the difference is coverage. The face mesh gives 468 accurate points
on a face. This gives every pixel in the frame, including the body and the
room, at lower accuracy.

It is slow — hundreds of milliseconds per inference — so it runs in its own
loop and the renderer always uses the most recent result, cross-fading
between successive inferences over about a third of a second. The visuals
stay at full frame rate; the depth lags when you move. That lag is the cost
of dense depth in a browser.

- q8 quantisation to keep the download and memory down
- webgpu where the browser offers it, wasm otherwise
- if the model fails to load the mode says so and everything else carries on
- the depth feeds the same per-pixel palette path built in build 23
