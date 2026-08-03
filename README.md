# Flow

A camera-reactive visual toy. Optical flow drives everything — no face
detection, no model download, no libraries.

## How it works
The video is drawn into a hidden 160x120 canvas. A 32x24 grid of cells is
compared against the previous frame: the spatial gradient and the temporal
difference at each point give a velocity, which is smoothed over time into a
fluid field.

That field drives three layers:
- **feedback** — the previous frame redrawn scaled, rotated and pushed by the
  overall drift of the flow. This is what makes tunnels and smears.
- **mesh** — the motion grid drawn as cells displaced by the local flow, with
  the RGB channels pulled apart.
- **particles** — carried by the field, streaked along their velocity.

Because there is no face detection it never fails to find you, works in any
light, and reacts to hands and background as well.

## Controls
Twelve sliders, all read fresh every frame, so nothing needs a restart.
Five presets: SMOKE, TUNNEL, OIL, GHOST, SHATTER.

## Requirements
Camera access needs **HTTPS**. Open it from GitHub Pages, not a local file.
Nothing is recorded and nothing leaves the device.
