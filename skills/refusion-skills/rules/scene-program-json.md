# Scene Program JSON

Use this rule when writing or repairing a `refusion.scene-program/v1` scene.

## Root Shape

Always return this wrapper:

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

Return complete JSON from the first `{` to the final `}`.

## Numeric Timing

Use numeric values, never strings:

```json
{ "startMs": 0, "durationMs": 2400, "timeMs": 900 }
```

Avoid aliases such as `start`, `startTimeMs`, `duration`, or `lengthMs`.

## Coordinates

Default portrait canvas:

```json
{ "canvasWidth": 1080, "canvasHeight": 1920 }
```

Coordinate system:

```text
center       x=0    y=0
left edge    x=-540
right edge   x=540
top edge     y=-960
bottom edge  y=960
```

Keep important text inside:

```text
x between -440 and 440
y between -760 and 760
```

## Layers

Supported layer kinds:

- `shape`
- `text`
- `image`

Layer shape:

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

Keep scenes compact. Prefer 3 to 8 visible layers unless the prompt asks for a
complex composition.

## Elements

Supported element kinds:

- `shape`
- `solid`
- `text`
- `image`
- `icon`
- `mask`

Supported shape kinds:

- `rectangle`
- `roundedRectangle`
- `circle`
- `line`
- `mask`

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

Mask reveal example:

```json
{
  "id": "title-wipe-mask",
  "kind": "mask",
  "properties": {
    "maskTarget": "title-text",
    "maskMode": "alpha",
    "revealDirection": "leftToRight",
    "position": { "x": -320, "y": 0 },
    "width": 90,
    "height": 180,
    "color": "#FFFFFF",
    "opacity": 1
  },
  "channels": [
    {
      "property": "movingMaskReveal",
      "keyframes": [
        { "timeMs": 900, "value": 0.0, "easing": "linear" },
        { "timeMs": 1500, "value": 1.0, "easing": "easeOutCubic" }
      ]
    },
    {
      "property": "position",
      "keyframes": [
        { "timeMs": 900, "value": { "x": -320, "y": 0 }, "easing": "easeOutCubic" },
        { "timeMs": 1500, "value": { "x": 320, "y": 0 }, "easing": "easeOutCubic" }
      ]
    }
  ]
}
```

## Supported Properties

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
- `morphSize`: `{ "width": 96, "height": 96 }`
- `roundness`
- `movingMaskReveal`
- `maskReveal`
- `fontSize`
- `letterSpacing`
- `typewriterProgress`
- `typingProgress`
- `reveal`

Preferred canonical names:

- use `color`, not `backgroundColor`;
- use `cornerRadius`, not `radius`;
- use `width`/`height`/`cornerRadius` for canonical shape morphs, or
  `morphSize`/`roundness` when describing a circle-to-bar style morph;
- use `mask` elements with `maskTarget`, `maskMode`, and `movingMaskReveal`
  when describing a travelling matte/wipe. Do not fake a mask by making many
  one-frame text or shape layers;
- use `typewriterProgress`, not one text element per character;
- use `startMs`, `durationMs`, and `timeMs` as numeric values.

## Channels And Keyframes

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

- if layer starts at `startMs: 1000`, keyframe `timeMs: 0` appears at project
  time `1000`;
- if a layer has `durationMs: 800`, local keyframes must be from `0` to `800`.

If using absolute project times, set:

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

Never mix local and project time in the same channel.

## Easing

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

Use `linear` for typewriter progress. Use `easeOutCubic` for entrances.
Use `easeInOutCubic` for press/transform moments. Use `easeOutQuint` for large
final cover/reveal motion.

## Core Icon Pack

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
