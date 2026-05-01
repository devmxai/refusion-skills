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

## General Video Transition Compositor Contract

ReFusion's professional transition compositor is general. Do not describe or
author a private renderer for each transition. Every transition should lower
into one shared `ProfessionalVideoTransitionRenderPlan` with:

- `definitionId`
- stable transition id
- canvas dimensions
- seam timing and leading/trailing durations
- source list with timeline/source ranges
- required capabilities
- transition parameters
- sampling policy
- edge policy
- motion blur policy

If a transition needs a new capability, declare the missing capability and
extend the engine. Do not fake support with thumbnails, still frames, decorative
lines, Gaussian blur, or transformed single-surface previews.

The native compositor has a renderer registry foundation. It may know a
definition name while still rejecting it until the real renderer and required
capabilities exist. Treat registry statuses literally: unsupported definition,
missing capabilities, or renderer not implemented are blockers, not invitations
to invent a visual fallback.

## Strict Authoring Gate

Do not create a new transition preset, manual transition lane, or AI-generated
transition draft while the native compositor reports the foundation/unavailable
capability set.

The app may show `Preset`, `Manual`, and `AI Transition` as locked roadmap
entries, but they must not become clickable authoring paths until the native
capability bridge reports all of these:

- dual video sampling;
- temporal motion blur;
- mirror-edge tiling;
- preview parity;
- Live Scrub parity;
- playback parity;
- export parity.

This rule applies to every video transition, including apparently simple ones
such as Cross Dissolve or Fade Black. Do not ship a separate fallback for each
transition. First complete the general compositor, then expose transitions above
that compositor.

## Native Render Session Contract

Every accepted `ProfessionalVideoTransitionRenderPlan` must become one strict
native render session before rendering:

- exactly two sources: outgoing A and incoming B;
- positive canvas size;
- non-negative seam timing and positive transition duration;
- source roles must be `[outgoing, incoming]`;
- every source needs a stable clip id, asset id, timeline range, source start,
  and source duration;
- outgoing A must reach the seam and cover the leading transition window;
- incoming B must start at or before the seam and cover the trailing transition
  window.

If any of these are missing, the correct answer is an explicit invalid or
unsupported result. Do not repair the plan by freezing a boundary frame,
stretching a thumbnail, or inventing hidden source timing.

## Native Video Source Binding Contract

Before exact decode requests can become a real decoder workload, every outgoing
and incoming source must be bound to a concrete media URI or path.

Each source binding must preserve:

- source role: `outgoing` or `incoming`;
- stable clip id;
- stable asset id;
- concrete `sourceUri`;
- timeline start/end;
- source start/duration;
- `requiresConcreteSourceUri=true`;
- `allowAssetIdOnlyDecode=false`;
- `allowGeneratedProxyDecode=false`.

`assetId` is identity metadata for project bookkeeping. It is not a video decode
source. A future decoder must open the concrete `sourceUri` for each side of the
transition. If a source URI is missing, the compositor must block the decoder
path with `native_video_source_uri_missing`.

Do not describe or author a transition that decodes from asset ids alone,
generated proxies, cached thumbnails, timeline posters, or inferred media
locations. The professional path is: source URI binding, exact frame samples,
exact decode requests, dual-video decoder tracks, temporal accumulation,
mirror-edge tiling when required, render-pass graph, output surface, and parity
outputs.

Flutter production code must not hand-assemble compositor source maps inside
large editor screens. Use the source-bound render-plan adapter contract:
adjacent `TimelineClipData` clips plus a `sourceUri` resolver become one strict
`ProfessionalVideoTransitionRenderPlan`. If either side lacks a concrete
`sourceUri`, enough visible/source handle for the leading/trailing window, or
explicit source-rate support, the adapter must fail closed with a blocker
instead of emitting a plan that would force frozen frames or ambiguous media
sampling.

Before any UI, agent, or script claims that a video transition can be exposed,
it must run the full readiness preflight. The readiness chain is: native
capabilities, strict render-session preparation, concrete source binding,
frame samples, exact decode requests, dual-video decoder, temporal accumulator,
mirror-edge tiler, render-pass graph, output surface, and
preview/live-scrub/playback/export parity. A single green stage is not
permission to ship a transition. Every stage must be able to advance.

Any UI or agent-facing explanation of transition readiness must use the formal
readiness presentation model. Do not collapse readiness into a vague "not
ready" or "missing capabilities" string. Name the blocked stages in order so a
future agent can tell whether the problem is source binding, exact decode,
temporal accumulation, mirror-edge tiling, output-surface ownership, or
preview/live-scrub/playback/export parity.

After source URI binding and before exact frame decode, every transition plan
must pass a real video source probe. The probe must prove that each bound
`sourceUri` is openable as video and contains a video track. Do not let an
agent, UI, or renderer skip this step by using asset ids, generated proxies,
thumbnails, poster frames, or boundary-frame stills. If the probe is not
implemented or fails for either source, the readiness blocker is
`sourceMediaProbe` and the transition must remain unavailable.

## Native Frame Sample Contract

Every renderer must sample live source time through the shared native frame
sample contract before it draws pixels.

For any requested timeline frame inside the transition window, the compositor
must resolve:

- normalized transition progress;
- outgoing source time from source A's real source range;
- incoming source time from source B's real source range;
- temporal shutter sample timeline times;
- outgoing temporal source sample times;
- incoming temporal source sample times;
- source roles `[outgoing, incoming]`.

If the requested frame time is outside the transition window, reject it. If the
render plan does not cover the full source window, reject it. Do not clamp the
missing interval into a frozen image and call it motion blur.

Temporal motion blur means multiple real temporal samples from the outgoing and
incoming videos, derived from shutter angle, frame rate, and sample count.
Gaussian blur, poster-frame blur, speed-line shapes, or thumbnail stretching are
not valid substitutes.

## Native Decode Request Contract

After frame samples are planned, the compositor must turn them into explicit
decode requests before any renderer can draw.

Each decode request must carry:

- source role: `outgoing` or `incoming`;
- stable clip id;
- stable asset id;
- timeline sample time;
- source sample time;
- sample index;
- decode mode `exactVideoFrame`;
- whether it is the center sample for the evaluated timeline frame.

The decode request contract must explicitly forbid:

- thumbnail fallback;
- generic poster frame fallback;
- boundary-frame freezing;
- hidden clamping to make missing source handles appear valid.

If exact video frame decode is unavailable, the transition remains locked. Do
not author or describe a visual workaround as if it were the real compositor.

## Native Dual-Video Decoder Session Contract

After exact decode requests exist, the compositor must group them into a
dual-video decoder session before any render pass can claim support.

The decoder session must include exactly two video tracks:

- outgoing;
- incoming.

Each track must preserve:

- clip id;
- asset id;
- exact decode request ids;
- sample count;
- `requiresExactFrameDecode=true`;
- `allowThumbnailFallback=false`;
- `allowBoundaryFreeze=false`.

The decoder session may be planned before the actual decoder exists, but it
must report `decoderImplemented=false`. A planned decoder session is not
permission to expose a transition preset.

## Native Temporal Sample Accumulator Contract

After a dual-video decoder session exists, the compositor must bind outgoing
and incoming decoder tracks into temporal sample accumulators before any
transition can claim real motion blur support.

Each accumulator must preserve:

- source role: `outgoing` or `incoming`;
- input decoder track role;
- sample count;
- deterministic sample weights;
- normalization mode, initially `weightedAverage`;
- `requiresTemporalShutter=true` when the render plan requests temporal shutter
  motion blur;
- `requiresExactFrameDecode=true`;
- `allowGaussianFallback=false`;
- `allowDecorativeSpeedLines=false`.

The accumulator session may be planned before a real implementation exists, but
it must report `accumulatorImplemented=false` and block transition exposure with
`native_temporal_sample_accumulator_missing`.

This is a hard quality boundary. Gaussian blur, poster-frame blur, line overlays,
radial decorative strokes, or any still-image substitute are not motion blur.

## Native Mirror-Edge Tiling Contract

After temporal sample accumulation exists, camera/zoom/push transitions that can
expose canvas borders must bind the outgoing and incoming accumulated frames
into mirror-edge tile plans before shader/effect evaluation.

Each tile plan must preserve:

- source role: `outgoing` or `incoming`;
- input accumulator id;
- edge mode, usually `mirrorTile`;
- output overscan scale;
- `clipToCanvas=true`;
- `allowBlackBorders=false`;
- `allowFlutterOverlay=false`;
- `allowTimelineOverlay=false`.

The tiling session may be planned before a real implementation exists, but it
must report `tilerImplemented=false` and block transition exposure with
`native_mirror_edge_tiler_missing`.

Do not solve edge gaps by shrinking the video, stretching a thumbnail, drawing a
fake background, or placing transition pixels in the timeline area. The only
professional path is canvas-clipped native mirror-edge tiling.

## Native Render Pass Graph Contract

After exact decode requests, temporal accumulators, and required mirror-edge
tiling plans exist, the compositor must lower the frame into a renderer-agnostic
pass graph before any concrete transition implementation can claim support.

The pass graph must include the general phases required by professional video
transitions:

- exact video frame decode;
- temporal sample accumulation for outgoing and incoming sources;
- mirror-edge tile when the edge policy requires it;
- transition shader/effect evaluation;
- composition to the transition output surface.

The pass graph may be planned before a renderer exists, but it must report that
the renderer is not implemented. Planning a graph is not permission to expose a
transition preset. A preset becomes valid only when the concrete native renderer
executes this graph for preview, Live Scrub, playback, and export parity.

## Native Output Surface Contract

After a render pass graph exists, the compositor must bind it to a single
professional output target: a native transition canvas surface clipped to the
preview/export canvas.

The output-surface contract must explicitly forbid:

- Flutter overlay rendering;
- timeline overlay rendering;
- transformed PlatformView fallback rendering;
- drawing transition video into the timeline area;
- any renderer path that is not canvas-clipped and native-owned.

Planning an output surface is still not permission to expose a transition. The
surface must report `rendererImplemented=false` until a concrete native renderer
executes the graph for preview, Live Scrub, playback, and export parity.

## Native Parity Output Contract

After the output surface exists, the compositor must prove that the same
transition output contract is valid for every required mode:

- preview;
- Live Scrub;
- playback;
- export.

An agent must not describe a transition as implemented if one mode uses the
native compositor while another mode uses a fallback. All four modes must share
the same native transition output surface contract, and all four modes must be
renderable before a preset/manual/AI transition is exposed.

## Cross Dissolve Primitive Contract

For `crossDissolve`, reason as a true two-source alpha blend:

- both outgoing A and incoming B must be real video sources;
- outgoing A must sample the playing end of A, and incoming B must sample the
  playing beginning of B. Do not confuse scene placement time with media source
  time;
- both sources should cover the full transition window, not just their normal
  non-overlapping clip ranges;
- use a two-second symmetric dissolve as the professional default. Very short
  windows below about 600ms read as quick fades, not smooth cross dissolves;
- progress is normalized from the transition start to the transition end;
- outgoing opacity is `1 - progress`;
- incoming opacity is `progress`;
- outgoing and incoming source times must be sampled from their real
  timeline/source ranges.

If either source does not cover the transition window, declare the missing
source coverage. Do not solve it by freezing the first/last frame, stretching a
thumbnail, or using a poster image. The host app now has a
`ProfessionalCrossDissolveCompositorPlanner` that exposes this coverage truth,
and a renderer must not enable the dissolve when coverage is false.

Until the full dual-video native compositor renders both streams directly, the
preview bridge may use exact boundary frames only as seam anchors:

- before the seam, live native playback is outgoing A and the incoming first
  boundary frame fades in;
- after the seam, live native playback is incoming B and the outgoing last
  boundary frame fades out;
- the incoming boundary frame must be source B's first visible source frame, not
  a late frame caused by B's placement on the composition timeline;
- boundary-frame anchors should be requested at the composition canvas
  resolution, not generic low-resolution thumbnail size.

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

The current ReFusion engine intentionally gates all new video transition
authoring out of the picker until a real professional video transition
compositor exists. Existing saved Zoom transitions must not draw fake speed
lines, frozen cards, or Gaussian transition blur.

The current app contains a strict `ProfessionalZoomCameraCompositorPlanner`
contract for future native rendering. Agents must describe Zoom In Camera in
terms of real outgoing/incoming source-time samples, shutter-angle temporal
sampling, and mirror-edge tiling. Do not describe it as an overlay, a card, a
blurred thumbnail, or a decorative speed-line effect.

The app also has a native capability bridge at
`com.refusion.app/professional_video_transition_compositor.getCapabilities`.
No transition preset, manual transition path, or AI transition path may become
clickable when that native response is missing any required capability: dual
video sampling, temporal motion blur, mirror-edge tiling, preview parity, live
scrub parity, playback parity, and export parity. Current builds report these
as unavailable until the real native renderer ships.

The current bridge defines generic `prepareRenderPlan`. Zoom In Camera is only
one definition lowered into that general handoff shape: canvas dimensions, seam
timing, outgoing/incoming source ranges, shutter settings, and mirror-edge tile
overscan must be present. Current builds return `unsupported` from native code
by design; do not work around that with a Flutter overlay, frozen frame,
Gaussian blur, or speed-line decoration.

The current native foundation also defines `planFrameSamples`. This endpoint is
not a renderer and does not unlock transitions. It proves that the engine can
turn a timeline frame inside a transition window into exact outgoing/incoming
source samples and temporal shutter samples. Agents should treat this as the
mandatory sampling truth for every future transition renderer.

The current native foundation also defines `planFrameDecodeRequests`. This
endpoint converts those samples into exact video decode requests for both
sources, with thumbnail fallback and boundary freeze explicitly disabled. It is
still planning only; it does not mean transitions are renderable.

The current native foundation also defines `planDualVideoDecoderSession`. This
endpoint groups exact outgoing/incoming decode requests into two native decoder
tracks and keeps `decoderImplemented=false`. Agents must not substitute
MediaMetadataRetriever thumbnails, cached posters, or frozen boundary stills
for this contract.

The current native foundation also defines `planRenderPassGraph`. This endpoint
turns exact decode requests into a pass graph and keeps
`rendererImplemented=false`. Agents must not treat a planned graph as a usable
transition effect until the concrete renderer is implemented and the capability
bridge reports full parity.

The current native foundation also defines `planOutputSurface`. This endpoint
binds the pass graph to a canvas-clipped native transition output target and
forbids Flutter overlays, timeline overlays, and PlatformView transform
fallbacks. It also keeps `rendererImplemented=false`, so it cannot unlock a
transition preset by itself.

The current native foundation also defines `planParityOutputs`. This endpoint
requires preview, Live Scrub, playback, and export to share the same output
surface contract and keeps every mode blocked until a concrete native renderer
can render all four modes without fallback divergence.

Do not promise transition support until preview, live scrub, playback, and
export all use the same compositor contract.
