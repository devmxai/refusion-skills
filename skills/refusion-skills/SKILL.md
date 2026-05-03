---
name: refusion-skills
description: Use when generating, repairing, validating, or reviewing ReFusion motion scenes, Scene Program JSON, Director Plans, tutorial-derived motion capabilities, professional timing, editable keyframes, or agent-authored animation scripts for the ReFusion engine.
metadata:
  tags: refusion, motion-graphics, scene-program, keyframes, timing, choreography
---

# ReFusion Skills

Use this skill whenever a task involves ReFusion scene generation, motion
script authoring, tutorial-to-capability extraction, timing review, or JSON
repair.

## Core Rule

Return complete ReFusion JSON only when the user asks for a scene. Do not return
Markdown around the JSON. Do not use executable code, JSX, CSS, functions,
imports, shader code, remote code, or comments.

Preferred root shape:

```json
{
  "directorPlan": {},
  "sceneProgram": {}
}
```

`directorPlan` is the choreography contract. `sceneProgram` is the editable
executable scene.

## Required Workflow

1. Understand the visual goal.
2. Plan ordered beats.
3. Define semantic components.
4. Define motion primitives inside beats.
5. Compile those primitives into real layers, elements, channels, and keyframes.
6. Validate timing, holds, contrast, and editability.
7. Return only complete JSON.

## Load Rules As Needed

- For schema, coordinates, supported properties, icons, and examples, read
  [rules/scene-program-json.md](rules/scene-program-json.md).
- For professional time ownership, holds, completion, layer lifetime, and
  future contract requirements, read
  [rules/professional-timing-contract.md](rules/professional-timing-contract.md).
- For motion direction, readable holds, handoffs, and anti-random animation
  rules, read [rules/choreography.md](rules/choreography.md).
- For composition-first workflow, Scene Clip containers, root/scene/layer scope
  responsibilities, and editing existing scenes or mentioned elements, read
  [rules/composition-workspace.md](rules/composition-workspace.md).
- For transitions between clips/scenes, boundary-frame truth, and the
  professional Zoom In Camera contract, read
  [rules/transitions.md](rules/transitions.md).
- For supported and planned capabilities, categories, and how new tutorial
  tools should be registered, read
  [rules/capability-registry.md](rules/capability-registry.md).
- For converting After Effects or motion tutorials into reusable engine
  capability, read [rules/tutorial-intake.md](rules/tutorial-intake.md).
- For preflight checks before returning JSON, read
  [rules/validation.md](rules/validation.md).

If your environment cannot open these relative rule files, use the repository
root file `REFUSION_SCENE_SKILL_FULL.md` instead. It contains this skill, every
rule file, and the example JSON in one document.

## Non-Negotiable Professional Rules

- Do not create random simultaneous animation.
- Do not let a scene end before all child motion completes.
- Important text must have a readable hold unless the user asks for kinetic
  typography.
- If two motions touch the same component/property, they must be one ordered
  track or an explicit handoff.
- A layer's visual lifetime must cover its keyframes.
- Typewriter text uses one full text element plus `typewriterProgress`.
- Scene Program must implement the Director Plan; they may not disagree.
- Every visible motion must be represented by editable keyframes.
- A generated scene should become one editable Scene Clip container; do not
  scatter one scene across many root timeline clips.
- Existing-scene or `@mention` edits must target stable existing IDs and must
  not create unrelated new elements.
- Transitions must be seam-aware boundary effects. For live video presets such
  as Zoom In Camera, animate the live video surface over the transition window
  rather than freezing boundary-frame thumbnails. Boundary frames are exact
  seeds/fallbacks for AI and non-live transitions, not a substitute for live
  playback.
- New preset/AI video transition creation is locked until the native
  professional compositor reports complete interactive readiness: dual video
  sampling, temporal motion blur, mirror-edge tiling, preview parity, Live
  Scrub parity, and playback parity. Manual may open only as an authoring
  scope in the existing focused TimelinePanel. Legacy native
  `manualTransform` compositor rendering must stay disabled for Manual
  authoring in preview/liveScrub/playback. Manual lanes must not suppress
  Stage5 native preview ownership in any interactive mode. Manual transition
  authoring still writes real graph/keyframe data; runtime visual execution for
  preview/playback/Live Scrub must come from the Stage5 master path before
  parity can be claimed. Do not claim export parity until export consumes the
  same compositor output.
  Current runtime status: Manual Transition transform/opacity now route through a
  dedicated Stage5 visual runtime state bridge (Master Clock -> master
  evaluation -> live scrub program -> Stage5 runtime state), and the runtime
  state is applied to both scrub overlay and visible player-surface ownership
  paths. Runtime follow-up rule: evaluate Manual Transition visuals against the
  effective motion project plus explicit preview-time clock override, push
  runtime visual source config during active scrub sessions (do not wait for
  scrub end), and reject Add Key when the playhead is outside the real active
  transition window (no silent clamp). Manual FX shader parity still requires
  dedicated native/Media3 slices.
- Professional Canva layer unification is the next binding architecture rule:
  video, image, text, shape, masks, and future generated objects must all
  resolve to composition-layer truth before Animate/FX/Key/Value/Graph can
  claim parity. Video must resolve through `MotionLayerKind.video` +
  `MotionElementKind.videoClip` + source binding, not raw transport/player clip
  identity. Manual Transition Add Key must use the same transition-local time
  domain shown to the user. Manual Transition Scale/Opacity/Position/Rotation
  must target outgoing/incoming/both layer instances through a unified target
  resolver, then flow through Master Clock, Value Truth, and a shared Visual
  Layer Program before preview/playback/Live Scrub/export renderer adapters.
  Do not treat a PlayerView transform, thumbnail, poster, or boundary-frame
  preview as final layer parity.

## Current Engine Boundary

ReFusion is moving toward a Professional Scene Timing Contract. Until that
contract is fully implemented, be conservative:

- use explicit beats and components;
- use readable holds;
- keep same-property motion in one channel when authoring Scene Program JSON;
- avoid ambiguous overlaps;
- avoid claiming unsupported effects are real.

## Prompt Template

```text
You are a ReFusion Scene Director.
Read the ReFusion Skills instructions.
Return exactly one complete JSON object with directorPlan and sceneProgram.
Use schemaVersion refusion.motion-director/v1 and refusion.scene-program/v1.
Use numeric startMs, endMs, durationMs, timeMs, and frameRate.
Use center-origin 1080x1920 canvas unless asked otherwise.
Plan ordered beats, semantic components, primitives, then editable layers,
elements, channels, and keyframes.
Do not use executable code, Markdown, comments, URLs, JSX, CSS, or imports.
```
