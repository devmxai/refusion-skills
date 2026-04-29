# Capability Registry

Use this rule when a tutorial introduces a new tool or when a scene needs an
effect.

## Registry Principle

Every new capability must live in one official category. Do not create one-off
tutorial-specific effects.

```text
Transform
  position
  scale
  rotation
  anchorPoint
  opacity

Text
  typewriter
  wordReveal
  letterReveal
  rangeSelector
  tracking
  lineReveal

Shape
  size
  cornerRadius
  fill
  stroke
  trimPath
  morph

Mask
  maskPath
  maskFeather
  maskExpansion
  invertedMask
  movingMaskReveal

Effects
  blur
  shadow
  glow
  tint
  gradientRamp
  noise
  lightSweep

Layout
  alignCenter
  safeArea
  padding
  gap
  align
  justify
  readableHold

Composition
  precompose
  parentGroup
  nullTransform
  adjustmentLayer

Choreography
  beat
  handoff
  leaderFollower
  completionPolicy
  contrastPolicy
```

## Capability Entry Contract

Before a capability is advertised as supported, document:

```text
id
category
professionalName
aliases
supportedTargets
parameters
defaultTiming
defaultEasing
editableInScope
previewSupport
exportSupport
sceneProgramSyntax
directorPrimitiveSyntax
motionPatchSyntax
validationRules
unsupportedCases
goldenExamples
status
```

Status values:

```text
planned
documented
implemented-domain
preview-ready
export-ready
blocked
```

## Current Support Snapshot

Preview/editing support changes over time. Treat this list as conservative.

Supported basics:

- transform position/scale/rotation/opacity;
- width/height/cornerRadius for shapes;
- shape morph aliases `morphSize` and `roundness` lower to editable
  width/height/cornerRadius channels;
- line trim path controls `trimStart`, `trimEnd`, and `trimOffset` lower to
  editable shape channels and preview for `shapeKind: "line"`;
- `mask` scene elements and `movingMaskReveal`/`maskReveal` lower to editable
  `mask.revealProgress` graph channels with preserved mask metadata;
- color;
- blur;
- soft shadow/drop shadow for shape/icon preview and editable scalar shape
  scope controls;
- typewriter progress;
- text range selector aliases for word/letter reveal lower to editable
  `revealProgress`, and tracking aliases lower to editable `letterSpacing`;
- layout/parent metadata (`parentId`, `containerId`, `parentGroup`,
  `layoutRole`, `layoutMode`, `padding`, `gap`, `align`, `justify`, `anchor`,
  `safeArea`, `constraints`, `zIndex`) is validated during authoring and
  preserved on lowered elements for future Scene Scope, Layer Scope, mention,
  preview, and export adapters;
- core icon pack;
- scene clip container and nested editable layers.

Needs dedicated engine work before being treated as real:

- trim path export parity, circular progress, arcs, dashed paths, and
  wraparound offset rendering;
- advanced per-word/per-character transform offsets beyond the current visible
  text reveal and tracking foundation;
- text shadow and export-perfect authored visual shadows;
- glow;
- light sweep;
- gradient ramp;
- preview/export compositing for moving masks;
- parent-group transform evaluation and inherited child transforms;
- 2.5D/3D camera, lights, z-depth;
- counters and chart primitives.

## Tutorial Tool Mapping

When a tutorial says:

```text
Gradient Ramp
```

Map to:

```text
Effects > gradientRamp
```

When a tutorial says:

```text
animated line reveal with trim paths
```

Map to:

```text
Shape > trimPath
```

When a tutorial says:

```text
pre-compose and parent to null
```

Map to:

```text
Composition > precompose
Composition > parentGroup
Composition > nullTransform
```

Do not build these as `tutorial001Gradient` or `designRevealSpecialLine`.
