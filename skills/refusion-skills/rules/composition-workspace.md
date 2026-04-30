# Composition Workspace Rule

Use this rule whenever a prompt asks for multiple scenes, editable scenes,
scene sequencing, imported assets, manual layers, `@mentions`, transitions, or
modifying an existing ReFusion scene.

## Core Model

ReFusion is composition-first.

Think in this structure:

```text
Project
  Root Composition
    Scene Clip instances on the root timeline
    Transition clips between adjacent scenes
  Source Compositions / Scenes
    Layers
      Elements
        Effects
        Property Channels
        Keyframes
  Assets
```

Do not treat a scene as loose visual fragments. A generated scene should become
one root Scene Clip whose internals can be opened and edited.

## Root Timeline vs Scene Internals

Root timeline:

- sequence scenes;
- position scene clips in time;
- hold master audio when needed;
- hold transition clips between scenes.

Root `Add > New Scene`:

- creates a new empty Scene Clip container in the root composition;
- when a Scene Clip is selected, inserts the new scene immediately after that
  selected clip;
- shifts later sequential Scene Clips forward to keep the story timeline
  non-overlapping;
- when no Scene Clip is selected, appends the new scene at the end of the
  current scene sequence.

Scene internals:

- video layers;
- image layers;
- text layers;
- shape layers;
- icon/vector layers;
- audio layers;
- null/parent groups when supported;
- adjustment layers when supported.

Layer scope:

- editable properties;
- effect controls;
- channels;
- keyframes;
- image and video layers are real scene layers and may open Unified Layer Scope
  for shared visual graph properties such as position, scale, rotation,
  opacity, and blur;
- do not reject video layer scope just because the source is playable media;
  instead, preserve the layer identity and make renderer/export parity explicit
  when an effect is not yet supported.
- video preview is graph-aware: the active native video surface may be wrapped
  with graph-evaluated transform/opacity/blur samples for preview/scrub/play.
  Export must still declare whether the authored video surface renderer is
  available instead of silently dropping those properties.
- export blockers must name authored visual kinds explicitly. If a scene uses a
  video layer surface, the export contract must carry `videoClip` with its
  source asset identity and graph channels, and the parity gate must say that
  video authored visual rendering is blocked until the native renderer ships.
  Do not collapse video/image/shape/mask export gaps into a vague warning.
- interpolation.

Scene Contents Media:

- belongs to the open source scene, not the root composition;
- opens only video/image import for scene-local media layers;
- treats video and image access as separate device-media permissions;
- must not include Text, Shape, Audio, Null, or Adjustment in the media sheet;
- Text and Shape are direct Scene Contents dock commands;
- Audio, Null, and Adjustment are visible planned dock commands until their
  engines are wired;
- normal video insertion uses one primary video storyline. New video layers are
  appended after the last video in the open scene and displayed on one video
  row. Use future explicit `Video Overlay` only when the user needs layered
  video compositing; do not make every imported video a new default row;
- adjacent video layers in the primary Scene Contents storyline should expose a
  selectable transition bridge. Treat this bridge as a real transition intent
  between the two scene-local video layers, not as empty timeline space or a
  hidden effect on only one layer;
- Scene Contents video clips shown in the editor are scene-layer proxies. They
  may expose video timing, drag, and transition bridges, but they must not be
  authored as fake root media clips or force the source scene timeline to become
  the compact native media program;
- the transition bridge browser order is `Preset`, `Manual`, then
  `AI Transition`. `Preset` should drill into a picker with a back action and
  only list transitions the engine can currently evaluate (`Cross Dissolve`,
  `Fade Black`, `Zoom In Camera` until more real effects are wired);
- applying a Scene Contents preset should return directly to the timeline so
  play and Live Scrub remain available. Open an inspector only when the user
  taps an existing transition or explicitly chooses an edit/manual path;
- manual/preset/AI transition choices may be staged before full renderer parity,
  but the authored scene must preserve a clear boundary, stable left/right
  layer IDs, and enough timing for the host app to open a future transition
  scope without guessing;
- must not be confused with Layer Scope keyframe/property Add tools.

Playback and scrub projection:

- media inside a Scene Clip remains an internal scene layer in the editable
  graph;
- the root timeline should still show one Scene Clip container, not all nested
  media layers;
- preview/playback and Live Scrub may receive a derived media-track projection
  from the currently open Scene Scope or root Scene Clip sequence;
- Scene Contents video insertion must preserve the video asset's natural
  duration. If the asset runs past the current Scene Clip duration, extend the
  source composition and Scene Clip instance together and shift later sequential
  Scene Clips forward;
- Scene Contents media timing has two clocks: authored composition/source time
  and compact native media-program time. Native duration is transport-internal
  only and must never replace the visible timeline duration of a composition,
  source scene, Scene Clip, or scoped layer timeline;
- `MotionLayerModel.visibleRange` owns source-scene placement. Newly inserted
  media elements should use layer-local `localRange` values, normally
  `0..layer.duration`, so moving a layer does not double-offset the element.
  Legacy scene-absolute element ranges may be read for compatibility, but new
  authoring must prefer layer-local ranges;
- Scene Contents native preview/scrub uses scene-local transport time while the
  app clock remains in root composition time. Always map `root <-> scene local`
  at the preview transport boundary;
- if a scoped timeline contains playable video, route scrub start/end through
  the same native settle handoff used by the root timeline. Do not immediately
  mark `scrubSettled` after finger lift while a native video surface is active;
  wait until native transport reaches the mapped scene-local target frame, then
  reveal/adopt the settled player frame;
- Scene Contents layer clips are real source-scene timing. Dragging/moving one
  must work as a direct horizontal layer move, must update the layer visible
  range, child element ranges, and related graph channel/keyframe times, and
  must never be represented as a fake detached timeline chip;
- when nested video layers overlap, preview playback is a visible-video
  projection by draw order (`zIndex`, then insertion order). Hidden elapsed time
  under a higher video remains real elapsed source time for the lower video, so
  when that lower video becomes visible it must continue from its true source
  offset instead of frame zero;
- empty timeline time is real. If the playhead is on a gap before, between, or
  after Scene Contents media layers, preview must be blank/transparent instead
  of falling back to the previous, next, or first visual asset. A native preview
  transport may use a compact media program internally, but raw authored
  source-scene time must never be passed through as compact media-program time.
  If an authored gap needs a native settle coordinate, clamp to the nearest
  compact media-program boundary while keeping the visible preview blank;
- Scene Contents transitions must resolve against scene-video layer proxies,
  not only root media clips. Preserve the proxy's source asset identity so the
  bridge preview can warm outgoing/incoming thumbnails, and keep native preview
  enabled even when the current playhead time is an authored gap;
- do not clear Scene Contents clip/transition selection or open a transition
  inspector while scrub, native handoff, or a structural edit is in progress;
  selection changes must not tear down playback/scrub state mid-gesture;
- project composition aspect is authoritative after composition creation.
  Imported media metadata and native rendered video dimensions must not flip,
  resize, or relock the composition canvas;
- moving a Scene Contents layer may extend the source composition and the
  owning Scene Clip instance when the moved layer ends after the current scene
  duration. Do not clamp a full-duration layer to start time zero just because
  its current duration equals the scene duration;
- Scene Contents row order must match preview draw order: the visually higher
  row is the higher-priority layer (`zIndex`, then insertion order);
- never solve playback by duplicating nested media as fake root timeline clips;
- if a scene contains a video/image layer, preserve its `sourceBinding.assetId`
  and timing so the host can project it into real preview/scrub media
  descriptors.

## When Creating A New Scene

If the user asks for a new scene from scratch, return one complete scene:

```text
one Scene Program
with one coherent duration
with real layers/elements/channels/keyframes
```

The app will place it as one Scene Clip container on the root timeline.

Do not split a single scene into many root clips unless the user explicitly asks
for multiple scenes.

## When Creating Multiple Scenes

If the user asks for several scenes, make each scene independent and clearly
timed.

Each scene must have:

- a name;
- a duration;
- a coherent start/end;
- visible completion or hold;
- no hidden dependency on another scene unless described as a transition.

Transitions between scenes should be described explicitly as transition intent,
not hidden random keyframes.

## When Modifying An Existing Scene

If the user is modifying a selected scene or mentioned elements:

- preserve stable existing IDs when provided;
- target existing `@mentions` by their stable IDs;
- do not create unrelated new elements unless requested;
- do not delete elements unless requested;
- keep every animated value inspectable as channels/keyframes.

For `@mention` motion, return a Motion Patch when requested by the host app
instead of a whole Scene Program.

## Add And Inspector Mental Model

Agents should assume ReFusion has separate responsibilities:

```text
Outliner = hierarchy and selection
Timeline = time, spans, keyframes, transitions
Inspector = selected-object properties
Canvas = visual preview
```

Generated JSON must support that separation. In practice:

- give layers clear names;
- give elements clear stable IDs;
- keep related elements grouped by layer or parent metadata when supported;
- use `layoutRole`, `parentId`, `parentGroup`, `zIndex`, and similar metadata
  when it helps the app show a clean hierarchy;
- do not depend on visual-only ordering that cannot be inspected.

## Timing Rule

Scene time is local to the owning scene. Keyframes must be inside the owning
layer duration.

For project-time choreography, use explicit layer `startMs` and local keyframes
or a documented `timeBasis` when the host schema supports it. Avoid ambiguous
global time values inside deeply nested layers.

## Professional Rejection Rules

Reject or repair your own output before returning it if:

- a generated scene would become many unrelated root tracks;
- a scene ends before its final visible motion resolves;
- important text has no readable hold;
- keyframes fall outside layer duration;
- two motions fight over the same property at the same time;
- an existing-scene request creates an unrelated new scene;
- a transition is hidden inside only one neighboring scene instead of being
  described as a transition between scenes.

## Practical Instruction

When in doubt:

```text
Create one clean Scene Program for one scene.
Use layers/elements/channels/keyframes inside it.
Let the app wrap it as a Scene Clip.
Use Motion Patch only when targeting existing @mentioned elements.
```
