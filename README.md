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


## Build 23 — why it read as low resolution

The blur was not the problem. Blurry and low-resolution are independent: a
blur on a large image is soft but every pixel is uniquely computed. Three
things were making it undersampled rather than merely soft.

**Colour was applied before upscaling.** The field was mapped to RGB at
160x284 and then scaled 2.4x to the screen, so every boundary between palette
bands was smeared across two or three screen pixels, and the in-between
values were RGB interpolations belonging to neither stop. The palette lookup
now happens per output pixel at CSS resolution, 390x844, while the field
stays small because it is genuinely low frequency. That is the crisp-and-soft
combination.

**No dithering.** Large smooth gradients band visibly in 8 bits. A small
ordered offset before quantisation breaks the contours.

**Fake depth.** The mask was flat inside, so there was no volume to read. A
pyramid of successively halved and blurred copies, summed back, gives a wide
smooth falloff from the silhouette edge — high deep inside the body, falling
away at every boundary. Measured on a head-and-shoulders figure: 0.94 at the
centre, 0.03 near the edge, 0.005 outside. Combined with luminance for
surface detail, that reads as a depth map.

**Palette.** Sampling the reference gives one saturated colour holding about
a fifth of the frame and everything else between 0.22 and 0.44 saturation.
The old ramp was seven stops all above 0.55 spanning the full hue circle,
then cycled — a spectrum, not a palette. Four designed palettes replace it.

**Cost.** The blur is now a running-sum box, two adds per output regardless
of radius. The bilinear samplers were removed from the hot loops. A full
bloom frame measures about 40ms in Node; Safari's JIT is usually faster on
typed-array loops, and the **bloom detail** slider scales the output
resolution if it needs to come down.
