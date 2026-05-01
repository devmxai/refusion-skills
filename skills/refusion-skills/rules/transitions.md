# Transition Authoring Rule

Use this rule when a prompt asks for a transition between two clips, scenes, or
Scene Clip containers.

## Boundary Truth

A transition is a boundary effect between an outgoing source and an incoming
source. It is not an independent fake clip.

For camera, zoom, blur, push, whip, flash, or matte transitions, always reason
from the seam:

- outgoing source A contributes the playing end of A before the seam;
- incoming source B contributes the playing beginning of B after the seam;
- exact boundary frames are still useful as AI seeds and non-live fallbacks, but
  a live-video transition must not replace playback with frozen thumbnails;
- progress is normalized over the transition window;
- the seam is normally at `0.5` unless an explicit alignment says otherwise;
- all motion curves are evaluated from this one progress value.

## Zoom In Camera Contract

For a professional zoom-in camera transition:

- keep the visual action fast, centered, and seam-aware;
- outgoing A should accelerate into the cut;
- incoming B should continue the perceived camera motion and settle after the
  cut;
- avoid slow dissolves unless the user explicitly asks for a dissolve;
- hide the cut with peak velocity, motion blur, and a small deterministic
  impact shake;
- require a real dual-video compositor. Do not fake this transition with
  Gaussian blur, speed-line shapes, static boundary frames, or a transformed
  single Android video surface.

Baseline normalized recipe:

```text
duration: 4 seconds by default, 1.2s..5.0s allowed
seam: 0.5
outgoing scale: 1.0 -> 3.0 by seam, ease-in
incoming scale: 0.28 -> 1.0 after seam, ease-out
opacity handoff: 0.42..0.58
incoming lead: none as frozen pre-roll; B becomes live at the seam
shutter angle: 180..360 degrees
motion blur: temporal shutter sampling or equivalent native/GPU compositor
mirror edges: enabled before transform sampling
impact shake: 5
bridge darkness: 0.12
```

## Professional Timing Rules

- Do not show a transition before both boundary sources are known.
- Do not use the first frame of source A for an outgoing seam preview; use the
  last visible frame of A.
- Do not use a generic thumbnail as the final transition source if a
  boundary-frame source is available.
- Do not use boundary-frame stills for Zoom In Camera when live video playback
  is available.
- Do not let the incoming layer appear as a rounded card unless the prompt asks
  for a card transition.
- Motion blur and shake must be deterministic so preview and export can match.
  Gaussian blur and decorative speed-line shapes are not valid motion blur for
  this preset.

## Current Engine Support

The current ReFusion engine intentionally gates `Zoom In Camera` out of the
preset picker until a real professional video transition compositor exists.
Existing saved Zoom transitions must not draw fake speed lines, frozen cards, or
Gaussian transition blur.

The current app contains a strict `ProfessionalZoomCameraCompositorPlanner`
contract for future native rendering. Agents must describe Zoom In Camera in
terms of real outgoing/incoming source-time samples, shutter-angle temporal
sampling, and mirror-edge tiling. Do not describe it as an overlay, a card, a
blurred thumbnail, or a decorative speed-line effect.

Do not promise Zoom In Camera support until preview, live scrub, playback, and
export all use the same compositor contract.
