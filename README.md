# Refusion Understanding Scene

Status: agent authoring handbook  
Purpose: teach any external coding/design agent how to write professional ReFusion scenes that validate, import, preview, and remain editable.  
Output format: JSON only.

This repository is a single-source reference for generating ReFusion scene scripts. Give this document to any agent before asking it to create a scene.

## 1. Core Rule

Return one JSON object only.

Do not return Markdown, JSX, JavaScript, TypeScript, HTML, CSS, Dart, shader code, remote imports, functions, comments, or prose around the JSON.

The preferred output shape is:

```json
{
  "directorPlan": {},
  "sceneProgram": {}
}
```

`directorPlan` explains the choreography.  
`sceneProgram` is the editable executable scene.

The app validates both. If the plan says one thing and the scene does another, the scene can be rejected.

## 2. Required Root Shape

Always use this wrapper:

```json
{
  "directorPlan": {
    "schemaVersion": "refusion.motion-director/v1",
    "name": "Readable Plan Name",
    "durationMs": 4000,
    "frameRate": 30,
    "canvasWidth": 1080,
    "canvasHeight": 1920,
    "beats": [],
    "components": [],
    "primitives": []
  },
  "sceneProgram": {
    "schemaVersion": "refusion.scene-program/v1",
    "name": "Readable Scene Name",
    "durationMs": 4000,
    "frameRate": 30,
    "layers": []
  }
}
```

Use numeric JSON values for all timing fields:

```json
{ "startMs": 0, "durationMs": 2400, "timeMs": 900 }
```

Never write timing numbers as strings:

```json
{ "startMs": "0" }
```

Avoid aliases such as `start`, `startTimeMs`, `duration`, or `lengthMs`. ReFusion can repair simple aliases with warnings, but professional output must use `startMs` and `durationMs`.

## 3. Professional Authoring Order

Think in this order before producing JSON:

1. Define the visual idea in your own internal reasoning.
2. Split it into intentional beats.
3. Define semantic components.
4. Add primitives for each component inside its owning beat.
5. Compile the primitives into real Scene Program layers, elements, channels, and keyframes.
6. Return JSON only.

Good:

```text
Beat: prompt text types on
Component: promptText
Primitive: typewriterProgress from 0.0 to 1.0
Scene Program: one text element with one typewriterProgress channel
```

Bad:

```text
Create one text layer per character and fade each character randomly.
```

## 4. Motion Director Plan

The Director Plan is the choreography contract.

### Beats

Each beat is a narrative time block:

```json
{
  "id": "text-type",
  "label": "Type prompt text",
  "startMs": 900,
  "endMs": 2100,
  "intent": "Reveal the input text with keyboard typing.",
  "componentRefs": ["promptText"]
}
```

Rules:

- Beats must have positive duration.
- Beats must stay inside `durationMs`.
- Every beat must include explicit `componentRefs`.
- Beats may overlap only when `componentRefs` are explicit and disjoint.
- Same-component overlap is ambiguous and should be one intentional beat with multiple primitives.
- Use readable holds for text/UI states before major transformations.

Professional beat example:

```text
0-400ms      background enters
220-1000ms   prompt shell enters
900-2100ms   text types on
2100-2500ms  readable hold
2500-2900ms  send button press
2900-4000ms  circle expands to cover screen
```

This is valid because the early overlap is on different components.

### Components

Components are semantic targets. Use stable IDs.

```json
{
  "id": "promptText",
  "role": "text.typewriter",
  "label": "Typed prompt text"
}
```

Common roles:

- `background.canvas`
- `promptInputBar.shell`
- `text.typewriter`
- `button.send`
- `icon.send`
- `circle.cover`
- `shape.line`
- `card.container`
- `image.hero`

### Primitives

Primitives describe one intentional motion.

```json
{
  "id": "prompt-text-type",
  "beatId": "text-type",
  "targetComponentId": "promptText",
  "kind": "typewriter",
  "property": "typewriterProgress",
  "startMs": 900,
  "endMs": 2100,
  "fromValue": 0.0,
  "toValue": 1.0,
  "easing": "linear"
}
```

Primitive rules:

- Every primitive must stay inside its owning beat.
- Every primitive must target an existing component.
- Every primitive should map to a real Scene Program channel.
- Typewriter/typing primitives must use `property: "typewriterProgress"`, `fromValue: 0.0`, and `toValue: 1.0`.
- Do not reverse typewriter values unless the prompt explicitly asks for deletion/backspace.
- Do not create random opacity flicker, random scale pulses, or unrelated simultaneous movement.

Common primitive kinds:

- `fade`
- `slide`
- `move`
- `scale`
- `press`
- `typewriter`
- `lineGrow`
- `widthGrow`
- `cover`
- `blur`
- `colorPulse`

## 5. Scene Program

The Scene Program is the executable editable scene. It must implement the Director Plan.

### Layers

Each layer has:

```json
{
  "id": "prompt-shell-layer",
  "name": "Prompt Shell",
  "kind": "shape",
  "startMs": 0,
  "durationMs": 4000,
  "elements": []
}
```

Supported layer kinds:

- `shape`
- `text`
- `image`

Keep scenes compact. Prefer 3 to 8 visible layers unless the user asked for a complex composition.

### Elements

Supported element kinds:

- `shape`
- `solid`
- `text`
- `image`
- `icon`

Supported shape kinds:

- `rectangle`
- `roundedRectangle`
- `circle`
- `line`

Shape example:

```json
{
  "id": "prompt-shell",
  "kind": "shape",
  "properties": {
    "shapeKind": "roundedRectangle",
    "position": { "x": 0, "y": 120 },
    "width": 820,
    "height": 104,
    "cornerRadius": 52,
    "color": "#121826",
    "opacity": 1
  }
}
```

Text example:

```json
{
  "id": "prompt-text",
  "kind": "text",
  "text": "hello world",
  "properties": {
    "position": { "x": -210, "y": 120 },
    "fontSize": 42,
    "letterSpacing": 0,
    "color": "#F7FAFF",
    "opacity": 1,
    "typewriterProgress": 0
  }
}
```

Icon example:

```json
{
  "id": "send-icon",
  "kind": "icon",
  "properties": {
    "icon": "send",
    "position": { "x": 324, "y": 120 },
    "width": 28,
    "height": 28,
    "color": "#08111F",
    "opacity": 1
  }
}
```

## 6. Coordinate System

Use a center-origin canvas.

Default portrait canvas:

```json
{ "canvasWidth": 1080, "canvasHeight": 1920 }
```

Coordinate meaning:

```text
center       x=0    y=0
left edge    x=-540
right edge   x=540
top edge     y=-960
bottom edge  y=960
```

Negative `x` moves left. Positive `x` moves right.  
Negative `y` moves up. Positive `y` moves down.

Keep text and important controls inside the safe visual area:

```text
x between -440 and 440
y between -760 and 760
```

## 7. Supported Properties

Use these properties in `properties` and `channels`:

- `position`: `{ "x": 0, "y": 0 }`
- `positionX`
- `positionY`
- `scale`: number or `{ "x": 1, "y": 1 }`
- `scaleX`
- `scaleY`
- `rotation`
- `opacity`: `0.0` to `1.0`
- `blur`
- `color`: `"#RRGGBB"` or `"#AARRGGBB"`
- `width`
- `height`
- `cornerRadius`
- `fontSize`
- `letterSpacing`
- `typewriterProgress`
- `typingProgress`
- `reveal`

Preferred canonical names:

- Use `color`, not `backgroundColor`, unless you need compatibility.
- Use `cornerRadius`, not `radius`.
- Use `typewriterProgress`, not one text element per character.
- Use `startMs`, `durationMs`, and `timeMs` as numeric values.

## 8. Channels And Keyframes

Every animation must be a channel:

```json
{
  "property": "position",
  "keyframes": [
    { "timeMs": 0, "value": { "x": 0, "y": 180 }, "easing": "easeOutCubic" },
    { "timeMs": 800, "value": { "x": 0, "y": 120 }, "easing": "easeOutCubic" }
  ]
}
```

Default keyframe time is local layer time:

- If layer starts at `startMs: 1000`, a keyframe `timeMs: 0` appears at project time 1000.
- If a layer has `durationMs: 800`, its local keyframes must be from `0` to `800`.

If you write absolute project times, set:

```json
{
  "property": "opacity",
  "timeBasis": "project",
  "keyframes": [
    { "timeMs": 1000, "value": 0.0 },
    { "timeMs": 1600, "value": 1.0 }
  ]
}
```

Recommended for agents:

- Use local time for simple layers.
- Use `timeBasis: "project"` only when you intentionally author everything on the global timeline.
- Never mix local and project time inside the same channel.
- Sort keyframes by ascending `timeMs`.
- Layer `durationMs` must cover its latest local keyframe.

## 9. Easing

Safe easing values:

- `linear`
- `easeIn`
- `easeOut`
- `easeInOut`
- `easeInCubic`
- `easeOutCubic`
- `easeInOutCubic`
- `easeOutQuint`
- `spring`

Use `linear` for typewriter progress.  
Use `easeOutCubic` for entrances.  
Use `easeInOutCubic` for press/transform moments.  
Use `easeOutQuint` for large final cover/reveal motion.

## 10. Core Design Pack

Use the built-in lightweight offline icon pack.

Supported icons:

```text
arrow-down, arrow-left, arrow-right, arrow-up,
bookmark, camera, check, chevron-left, chevron-right, close,
comment, crop, heart, image, lock, mic, music, paperclip,
pause, play, plus, search, send, settings, share, sparkles,
text, user, verified, video, volume
```

Accepted aliases:

```text
attach -> paperclip
attachment -> paperclip
microphone -> mic
voice -> mic
submit -> send
profile -> user
verification -> verified
favorite -> heart
chat -> comment
done -> check
add -> plus
```

If the prompt asks for an unsupported icon, choose the nearest supported icon and keep the scene valid.

## 11. Typewriter Rules

For keyboard typing:

- Use one text element with the complete text.
- Add one `typewriterProgress` channel.
- Animate from `0.0` to `1.0`.
- Use `linear` easing.
- Do not create one layer or element per character.

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

Wrong:

```json
[
  { "id": "char-h", "text": "h" },
  { "id": "char-e", "text": "e" }
]
```

Wrong unless the user asks for deletion:

```json
{
  "property": "typewriterProgress",
  "keyframes": [
    { "timeMs": 0, "value": 1.0 },
    { "timeMs": 1200, "value": 0.0 }
  ]
}
```

## 12. Professional Scene Quality

A good generated scene should feel like a compact motion-graphics composition:

- clear visual hierarchy,
- one main subject at a time,
- readable pauses,
- intentional transitions,
- no random simultaneous motion,
- no hidden elements moving for no reason,
- no off-screen text unless it is entering or exiting,
- no flickering opacity unless explicitly requested,
- no excessive layers,
- no unsupported assets.

For a prompt input animation, a strong structure is:

1. background fades in,
2. prompt bar enters,
3. text types on,
4. send button presses,
5. shape expands or transforms into the next scene.

## 13. Validation Checklist

Before returning JSON, verify:

- Root object has `directorPlan` and `sceneProgram`.
- `directorPlan.schemaVersion` is `refusion.motion-director/v1`.
- `sceneProgram.schemaVersion` is `refusion.scene-program/v1`.
- `durationMs`, `startMs`, `endMs`, `timeMs`, and `frameRate` are numbers, not strings.
- Every beat has `componentRefs`.
- Overlapping beats have disjoint `componentRefs`.
- Every primitive target exists in `components`.
- Every primitive stays inside its beat.
- Every Scene Program layer has `id`, `kind`, `startMs`, `durationMs`, and `elements`.
- Every keyframe is inside its owning layer duration unless `timeBasis` is `project`.
- Typewriter uses one full text element and one `typewriterProgress` channel.
- Scene Program implements the Director Plan components and primitives.
- No executable or remote-code fields exist.

Forbidden keys anywhere:

```text
code, script, function, eval, imports, remoteImports, shaderSource
```

## 14. Agent Prompt Template

Use this instruction before asking an agent to generate a ReFusion scene:

```text
You are a ReFusion Scene Director.
Read the Refusion Understanding Scene documentation.
Return JSON only.
Return exactly one object with directorPlan and sceneProgram.
Use schemaVersion refusion.motion-director/v1 for directorPlan.
Use schemaVersion refusion.scene-program/v1 for sceneProgram.
Use numeric startMs, durationMs, endMs, timeMs, frameRate values.
Use center-origin 1080x1920 canvas unless asked otherwise.
Plan ordered beats first, then semantic components, then primitives, then compile to editable layers/elements/channels/keyframes.
Typewriter text must be one complete text element with typewriterProgress 0.0 -> 1.0.
Do not use executable code, JSX, CSS, markdown, comments, or URLs.
```

## 15. Complete Valid Example

This example creates a professional prompt bar animation:

```json
{
  "directorPlan": {
    "schemaVersion": "refusion.motion-director/v1",
    "name": "Prompt Input Send Wipe Plan",
    "durationMs": 4000,
    "frameRate": 30,
    "canvasWidth": 1080,
    "canvasHeight": 1920,
    "beats": [
      {
        "id": "background-enter",
        "label": "Background enters",
        "startMs": 0,
        "endMs": 400,
        "intent": "Fade in a dark canvas.",
        "componentRefs": ["background"]
      },
      {
        "id": "prompt-enter",
        "label": "Prompt bar enters",
        "startMs": 220,
        "endMs": 1000,
        "intent": "Slide and fade the prompt shell and send button into place.",
        "componentRefs": ["promptShell", "sendButton", "sendIcon"]
      },
      {
        "id": "text-type",
        "label": "Prompt text types",
        "startMs": 1000,
        "endMs": 2200,
        "intent": "Reveal the prompt text like keyboard typing.",
        "componentRefs": ["promptText"]
      },
      {
        "id": "readable-hold",
        "label": "Readable hold",
        "startMs": 2200,
        "endMs": 2500,
        "intent": "Let the typed prompt remain readable before the action.",
        "componentRefs": ["promptShell", "promptText", "sendButton", "sendIcon"]
      },
      {
        "id": "send-action",
        "label": "Send button press",
        "startMs": 2500,
        "endMs": 2920,
        "intent": "Compress the send button and icon to show a deliberate tap.",
        "componentRefs": ["sendButton", "sendIcon"]
      },
      {
        "id": "cover-transition",
        "label": "Circle cover transition",
        "startMs": 2920,
        "endMs": 4000,
        "intent": "Expand a circle from the send button until it covers the screen.",
        "componentRefs": ["coverCircle"]
      }
    ],
    "components": [
      {
        "id": "background",
        "role": "background.canvas",
        "label": "Dark background"
      },
      {
        "id": "promptShell",
        "role": "promptInputBar.shell",
        "label": "Rounded prompt shell"
      },
      {
        "id": "promptText",
        "role": "text.typewriter",
        "label": "Typed prompt text"
      },
      {
        "id": "sendButton",
        "role": "button.send",
        "label": "Send button"
      },
      {
        "id": "sendIcon",
        "role": "icon.send",
        "label": "Send icon"
      },
      {
        "id": "coverCircle",
        "role": "circle.cover",
        "label": "Expanding cover circle"
      }
    ],
    "primitives": [
      {
        "id": "background-fade",
        "beatId": "background-enter",
        "targetComponentId": "background",
        "kind": "fade",
        "property": "opacity",
        "startMs": 0,
        "endMs": 400,
        "fromValue": 0.0,
        "toValue": 1.0,
        "easing": "linear"
      },
      {
        "id": "prompt-shell-slide",
        "beatId": "prompt-enter",
        "targetComponentId": "promptShell",
        "kind": "slide",
        "property": "position",
        "startMs": 220,
        "endMs": 1000,
        "fromValue": { "x": 0, "y": 220 },
        "toValue": { "x": 0, "y": 120 },
        "easing": "easeOutCubic"
      },
      {
        "id": "prompt-shell-opacity",
        "beatId": "prompt-enter",
        "targetComponentId": "promptShell",
        "kind": "fade",
        "property": "opacity",
        "startMs": 220,
        "endMs": 760,
        "fromValue": 0.0,
        "toValue": 1.0,
        "easing": "easeOutCubic"
      },
      {
        "id": "text-typewriter",
        "beatId": "text-type",
        "targetComponentId": "promptText",
        "kind": "typewriter",
        "property": "typewriterProgress",
        "startMs": 1000,
        "endMs": 2200,
        "fromValue": 0.0,
        "toValue": 1.0,
        "easing": "linear"
      },
      {
        "id": "send-button-press",
        "beatId": "send-action",
        "targetComponentId": "sendButton",
        "kind": "press",
        "property": "scale",
        "startMs": 2500,
        "endMs": 2920,
        "fromValue": 1.0,
        "toValue": 0.92,
        "easing": "easeInOutCubic"
      },
      {
        "id": "cover-circle-expand",
        "beatId": "cover-transition",
        "targetComponentId": "coverCircle",
        "kind": "cover",
        "property": "scale",
        "startMs": 2920,
        "endMs": 4000,
        "fromValue": 0.01,
        "toValue": 18.0,
        "easing": "easeOutQuint"
      }
    ]
  },
  "sceneProgram": {
    "schemaVersion": "refusion.scene-program/v1",
    "name": "Prompt Input Send Wipe",
    "durationMs": 4000,
    "frameRate": 30,
    "layers": [
      {
        "id": "background-layer",
        "name": "Background",
        "kind": "shape",
        "startMs": 0,
        "durationMs": 4000,
        "elements": [
          {
            "id": "background",
            "kind": "shape",
            "properties": {
              "shapeKind": "rectangle",
              "position": { "x": 0, "y": 0 },
              "width": 1080,
              "height": 1920,
              "color": "#05070C",
              "opacity": 1
            },
            "channels": [
              {
                "property": "opacity",
                "keyframes": [
                  { "timeMs": 0, "value": 0.0, "easing": "linear" },
                  { "timeMs": 400, "value": 1.0, "easing": "linear" }
                ]
              }
            ]
          }
        ]
      },
      {
        "id": "prompt-shell-layer",
        "name": "Prompt Shell",
        "kind": "shape",
        "startMs": 0,
        "durationMs": 4000,
        "elements": [
          {
            "id": "prompt-shell",
            "kind": "shape",
            "properties": {
              "shapeKind": "roundedRectangle",
              "position": { "x": 0, "y": 120 },
              "width": 820,
              "height": 104,
              "cornerRadius": 52,
              "color": "#121826",
              "opacity": 1
            },
            "channels": [
              {
                "property": "position",
                "keyframes": [
                  { "timeMs": 220, "value": { "x": 0, "y": 220 }, "easing": "easeOutCubic" },
                  { "timeMs": 1000, "value": { "x": 0, "y": 120 }, "easing": "easeOutCubic" }
                ]
              },
              {
                "property": "opacity",
                "keyframes": [
                  { "timeMs": 220, "value": 0.0, "easing": "linear" },
                  { "timeMs": 760, "value": 1.0, "easing": "easeOutCubic" }
                ]
              }
            ]
          }
        ]
      },
      {
        "id": "prompt-text-layer",
        "name": "Prompt Text",
        "kind": "text",
        "startMs": 0,
        "durationMs": 4000,
        "elements": [
          {
            "id": "prompt-text",
            "kind": "text",
            "text": "hello world",
            "properties": {
              "position": { "x": -220, "y": 120 },
              "fontSize": 42,
              "letterSpacing": 0,
              "color": "#F7FAFF",
              "opacity": 1,
              "typewriterProgress": 0
            },
            "channels": [
              {
                "property": "typewriterProgress",
                "keyframes": [
                  { "timeMs": 1000, "value": 0.0, "easing": "linear" },
                  { "timeMs": 2200, "value": 1.0, "easing": "linear" }
                ]
              }
            ]
          }
        ]
      },
      {
        "id": "send-control-layer",
        "name": "Send Control",
        "kind": "shape",
        "startMs": 0,
        "durationMs": 4000,
        "elements": [
          {
            "id": "send-button",
            "kind": "shape",
            "properties": {
              "shapeKind": "circle",
              "position": { "x": 324, "y": 120 },
              "width": 72,
              "height": 72,
              "color": "#6B92FF",
              "opacity": 1,
              "scale": 1
            },
            "channels": [
              {
                "property": "scale",
                "keyframes": [
                  { "timeMs": 2500, "value": 1.0, "easing": "linear" },
                  { "timeMs": 2720, "value": 0.92, "easing": "easeInOutCubic" },
                  { "timeMs": 2920, "value": 1.0, "easing": "easeOutCubic" }
                ]
              }
            ]
          },
          {
            "id": "send-icon",
            "kind": "icon",
            "properties": {
              "icon": "send",
              "position": { "x": 324, "y": 120 },
              "width": 28,
              "height": 28,
              "color": "#08111F",
              "opacity": 1,
              "scale": 1
            },
            "channels": [
              {
                "property": "scale",
                "keyframes": [
                  { "timeMs": 2500, "value": 1.0, "easing": "linear" },
                  { "timeMs": 2720, "value": 0.86, "easing": "easeInOutCubic" },
                  { "timeMs": 2920, "value": 1.0, "easing": "easeOutCubic" }
                ]
              }
            ]
          }
        ]
      },
      {
        "id": "cover-circle-layer",
        "name": "Cover Circle",
        "kind": "shape",
        "startMs": 0,
        "durationMs": 4000,
        "elements": [
          {
            "id": "cover-circle",
            "kind": "shape",
            "properties": {
              "shapeKind": "circle",
              "position": { "x": 324, "y": 120 },
              "width": 120,
              "height": 120,
              "color": "#6B92FF",
              "opacity": 0,
              "scale": 0.01
            },
            "channels": [
              {
                "property": "opacity",
                "keyframes": [
                  { "timeMs": 2910, "value": 0.0, "easing": "linear" },
                  { "timeMs": 2920, "value": 1.0, "easing": "linear" }
                ]
              },
              {
                "property": "scale",
                "keyframes": [
                  { "timeMs": 2920, "value": 0.01, "easing": "linear" },
                  { "timeMs": 3400, "value": 2.4, "easing": "easeOutCubic" },
                  { "timeMs": 4000, "value": 18.0, "easing": "easeOutQuint" }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## 16. Final Reminder

The goal is not merely valid JSON. The goal is editable, professional motion.

If a scene cannot be edited after import, it is not acceptable.
If the keyframes do not represent the story, it is not acceptable.
If the Director Plan and Scene Program disagree, it is not acceptable.
If the animation feels random, rewrite the beats first.
