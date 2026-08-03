# Flow

A camera-reactive visual toy. Optical flow drives everything.

## How it works
The video is drawn into a hidden 160x120 canvas. A 32x24 grid of cells is
compared against the previous frame: the spatial gradient and the temporal
difference at each point give a velocity, smoothed over time into a fluid
field.

That field drives three layers:
- **feedback** — the previous frame redrawn scaled, rotated and pushed by the
  drift of the flow. This is what makes tunnels and smears.
- **mesh** — the motion grid drawn as cells displaced by the local flow, with
  the RGB channels pulled apart.
- **particles** — carried by the field, streaked along their velocity.

## Seeing yourself
With ME ONLY on, your silhouette is drawn directly: a glowing rim plus a soft
inner bloom, built from the segmentation mask at 192x144 and scaled up. This
matters more than any of the weighting — the abstract layers alone give no
clue that they are reacting to your body, so without something on screen
shaped like a person the whole thing reads as random.

The **your outline** slider controls how strongly it is drawn. The BODY preset
is built around it.

## ME ONLY
Optional. Loads MediaPipe Image Segmenter from a CDN (~1MB, cached after the
first load) and builds a person mask each frame. With it on:

- background motion is suppressed, so the effect only reacts to you
- the mesh warps hardest across your silhouette
- particles respawn onto you instead of anywhere on screen
- the **edge bias** slider controls how much your outline is emphasised over
  your interior

Entirely optional. If the CDN is unreachable the toggle switches itself off
and everything else carries on as plain motion.

There is no true depth: iOS does not expose the TrueDepth sensor to Safari
and there is no browser depth API. This is subject isolation, which is what
makes the effect feel deliberate rather than random.

## Controls
Thirteen sliders, all read fresh every frame, so nothing needs a restart.
Five presets: SMOKE, TUNNEL, OIL, GHOST, SHATTER.

## Requirements
Camera access needs **HTTPS** and Safari proper — in-app browsers are blocked.
Nothing is recorded and nothing leaves the device.


## Why it stopped burning out
Additive light with a feedback factor f reaches a steady state of
add/(1-f). At f=0.93 that is 1.43 — past clipping within about 30 frames,
which is why everything went white and the body read as a black hole.

Three changes:
- a **bleed** slider subtracts a little from the feedback buffer each frame,
  so accumulated light has somewhere to go. Steady state is now 0.28.
- the body is **cut out** of the accumulated light rather than filled, so the
  light lives around you and your silhouette is a clean window.
- mesh cells were opaque rectangles at full alpha; they are now 33% of that
  brightness.

The rim is drawn as three offset copies in three hues. The split and the
thickness both scale with how much you are moving, so it breathes rather than
sitting at a fixed offset.
