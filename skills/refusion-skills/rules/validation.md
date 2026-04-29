# Validation

Use this checklist before returning ReFusion JSON.

## JSON Integrity

- Return one complete JSON object.
- No Markdown around JSON.
- No comments.
- No trailing commas.
- No truncated fragments.
- Root has `directorPlan` and `sceneProgram`.
- `directorPlan.schemaVersion` is `refusion.motion-director/v1`.
- `sceneProgram.schemaVersion` is `refusion.scene-program/v1`.

## Security

Forbidden keys anywhere:

```text
code, script, function, eval, imports, remoteImports, shaderSource
```

No URLs unless the user and engine explicitly support that asset path.

## Timing

- `durationMs`, `startMs`, `endMs`, `timeMs`, and `frameRate` are numbers.
- Beats stay inside `durationMs`.
- Primitives stay inside owning beats.
- Primitive `beatId` points to an existing beat.
- The owning beat includes the primitive `targetComponentId` in
  `componentRefs`.
- Layer `durationMs` covers all local keyframes.
- Project-time channels declare `timeBasis: "project"`.
- No empty channels. Remove channels without keyframes.
- No same-target/same-property overlap unless deliberately authored as one
  ordered channel.
- Scene does not end before all child motion completes.
- Visible component final motion should not land exactly on the final scene
  boundary without a resolve/hold moment.
- Text has readable hold when important.

## Director Plan Alignment

- Every beat references existing components.
- Every primitive targets an existing component.
- Every primitive maps to a real Scene Program channel.
- Background fade primitives are implemented by an actual opacity channel, not
  only static opacity properties.
- Typewriter primitive maps to `typewriterProgress`.

## Scene Program

- Every layer has `id`, `name`, `kind`, `startMs`, `durationMs`, and
  `elements`.
- Every element has `id`, `kind`, and valid `properties`.
- Every animation is represented by a channel with sorted keyframes.
- Use stable semantic IDs, not random UUID-like names.
- Do not create excessive layers for a simple scene.

## Typewriter

Correct:

```json
{
  "property": "typewriterProgress",
  "keyframes": [
    { "timeMs": 0, "value": 0.0, "easing": "linear" },
    { "timeMs": 1200, "value": 1.0, "easing": "linear" }
  ]
}
```

Wrong unless deletion/backspace is explicitly requested:

```json
{
  "property": "typewriterProgress",
  "keyframes": [
    { "timeMs": 0, "value": 1.0 },
    { "timeMs": 1200, "value": 0.0 }
  ]
}
```

## Professional Quality

Reject and rewrite if:

- animation feels random;
- next motion starts before the prior motion resolves;
- text appears in the wrong contrast;
- visible motion is not editable;
- the Director Plan and Scene Program disagree;
- a scene relies on unsupported effects but presents them as real.
