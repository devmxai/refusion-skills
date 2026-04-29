# Professional Scene Timing Contract

Use this rule when planning professional motion timing or reviewing generated
scene timing.

## Why This Exists

Professional motion is not a collection of attractive shapes. It is deterministic
time ownership.

ReFusion is moving toward a first-class timing contract inspired by professional
systems such as Remotion sequences, After Effects compositions/precomps, Lottie
in/out points, and Rive state-machine transitions.

## Core Contract

Every generated scene must be explainable as:

```text
scene clock
-> beats / markers
-> component lifetimes
-> property tracks
-> keyframes
-> holds / handoffs / completion
```

If an animation cannot be explained by this chain, rewrite the Director Plan
before writing Scene Program JSON.

## Required Timing Concepts

### Scene Clock

The scene must declare:

- `durationMs`
- `frameRate`
- canvas size

The scene duration must cover all beats, primitives, layer spans, readable
holds, and final transitions.

### Beat

A beat is an intentional narrative block:

```json
{
  "id": "text-type",
  "label": "Type prompt text",
  "startMs": 1000,
  "endMs": 2200,
  "intent": "Reveal prompt text with keyboard typing.",
  "componentRefs": ["promptText"]
}
```

Rules:

- beats stay inside scene duration;
- beats have positive duration;
- every beat declares componentRefs;
- important text/UI states need a readable hold beat;
- overlaps are intentional only when components are disjoint or the handoff is
  explicit and same-property conflict does not exist.

### Component Lifetime

Each visible component must have an intentional lifetime:

```text
enter -> active/readable -> action/transform -> exit/resolve
```

Current ReFusion Scene Program layers still use `startMs` and `durationMs`.
Author them honestly:

- do not give every layer full-scene duration unless it truly exists for the
  full scene;
- layer duration must cover every local keyframe;
- if a component should hold after animation, keep it visible and stable;
- if it should disappear, animate opacity/scale/position intentionally.

### Property Track Ownership

All motion for one target/property should be one ordered track.

Good:

```json
{
  "property": "position",
  "keyframes": [
    { "timeMs": 0, "value": { "x": 0, "y": 320 } },
    { "timeMs": 800, "value": { "x": 0, "y": 0 } },
    { "timeMs": 1400, "value": { "x": -260, "y": 0 } },
    { "timeMs": 2200, "value": { "x": 260, "y": 0 } }
  ]
}
```

Bad:

```text
three separate position channels for the same target
```

Separate channels for the same target/property are fragile until the engine's
compiler/lowerer merge contract is complete.

### Hold

A hold means the viewer can read or understand the state.

Use holds for:

- completed text;
- UI labels;
- important icons;
- final result moments;
- before destructive transitions.

Minimum practical hold:

```text
short label: 250-400ms
sentence/prompt text: 500-900ms
complex UI card: 900-1400ms
```

### Handoff

A handoff is when one motion gives control to another.

Rules:

- do not start a new same-property motion before the old one has reached its
  intended value;
- if overlap is needed, make it a different property group or explicitly model
  the handoff;
- never hide text before its readable hold unless the prompt asks for fast
  kinetic typography.

### Completion

Every scene must end cleanly:

- all child animations completed;
- final frame is intentional;
- no layer cuts before its final motion;
- no hidden random motion after the final beat;
- final state has enough visual contrast.

## Current Engine Support

The ReFusion engine now includes an initial domain-only
`ProfessionalSceneTimingContractValidator`.

Current enforced rules:

- text reveal/typewriter primitives need an explicit readable hold beat after
  the reveal;
- a text reveal may not end exactly at the scene boundary;
- overlapping same-target/same-property primitives are invalid;
- component timing lifetimes are derived for future compiler use;
- Scene Program timing checks can detect duplicate channels and keyframes
  outside a layer span.

Current caution:

- avoid multiple channels for the same target/property;
- avoid ambiguous overlaps;
- prefer longer, explicit keyframe tracks;
- keep Scene Program and Director Plan aligned;
- make layer spans honest and large enough.

Future engine updates must document the exact supported contract here.
