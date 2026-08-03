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


## Build 12 — density and a panel you can see past

**Why the face read as sparse.** A flat patch of cheek has no gradient, so a
tracer landing there is retired immediately and that area stays empty. The
features were resolving correctly — there simply were not enough surviving
strands to read as a surface.

The field now carries a gentle built-in slope inside the mask: weak enough
that real shading and the landmarks still determine the shape, strong enough
that flow exists everywhere on the body. Measured under perfectly flat
lighting, interior cells carrying usable flow went from patchy to 100%.

Line count raised from 1100 to 3200 by default, ceiling 6000, and strands
survive to a lower field value. Draw calls went from 5,885 to 17,096 per six
frames.

**The panel.** It was rgba(12,10,20,.82) over an 18px backdrop blur, which is
opaque for practical purposes — the scrub-transparency added in build 11 was
fighting a blur that never went away. Now:

- base background .34 alpha, no blur at all
- .06 alpha while a slider is being dragged
- max height 46vh instead of 74vh, tighter rows, so it covers far less
- labels carry a text shadow so they survive on a transparent ground
- tapping the picture closes the panel outright
