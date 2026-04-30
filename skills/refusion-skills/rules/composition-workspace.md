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
- interpolation.

Scene Contents Media:

- belongs to the open source scene, not the root composition;
- opens only video/image import for scene-local media layers;
- must not include Text, Shape, Audio, Null, or Adjustment in the media sheet;
- Text and Shape are direct Scene Contents dock commands;
- Audio, Null, and Adjustment are visible planned dock commands until their
  engines are wired;
- must not be confused with Layer Scope keyframe/property Add tools.

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
