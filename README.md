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
