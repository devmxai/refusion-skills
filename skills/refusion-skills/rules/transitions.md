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
- playback parity.

Export parity is a separate later release gate. Do not block interactive
preview/scrub/playback transition authoring on export once the native
interactive compositor is complete, but do not claim export support until the
export renderer joins the same output contract.

This rule applies to every video transition, including apparently simple ones
such as Cross Dissolve or Fade Black. Do not ship a separate fallback for each
transition. First complete the general compositor, then expose transitions above
that compositor.

No diagnostic transition exception is currently active. `Zoom In Pro` was tried
as a live-surface experiment and is now closed because a transformed single
native preview surface cannot provide dual-video sampling, temporal shutter
motion blur, mirror-edge tiling, or stable preview/Live Scrub/playback parity.
Agents must not expose or recommend it as an available preset.

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
  mirror-edge tiling when required, render-pass graph, graph execution, output
  surface, surface renderer, frame render commands, renderer backend, renderer
  draw loop, transition shader evaluation, transition pixel workload, transition
  pixel frame buffer, transition pixel frame-buffer writer, pixel render
  execution, native pixel output proof, transition surface endpoint, and parity
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
  mirror-edge tiler, render-pass graph, graph execution, output surface, surface
  renderer, frame render commands, renderer backend, renderer draw loop,
  transition shader evaluation, transition pixel workload, transition pixel
  frame buffer, transition pixel frame-buffer writer, transition pixel render
  execution, native pixel output proof, transition surface endpoint, and
  preview/live-scrub/playback parity.
  A single green stage is not
permission to ship a transition. Every stage must be able to advance.

Before preview, playback, or Live Scrub parity can claim success, the native
pixel frame-buffer, frame-buffer writer, pixel output proof, and transition
surface endpoint gates must pass.
The frame-buffer gate must prove a native canvas-sized `rgba8888` buffer exists for
`nativeTransitionCanvasSurface`, and must explicitly forbid synthetic pixels,
poster frames, thumbnails, and boundary-frame freezes. The output proof must
prove that a real source-derived frame was carried into the native output proof
path, and it must explicitly forbid Flutter overlays, timeline overlays,
transformed platform-view previews, poster frames, thumbnails, or any other
non-native output fallback. The transition surface endpoint must attach an
offscreen native `ImageReader`/`Surface` endpoint and accept the uploaded
canvas-sized frame-buffer packet before parity is allowed to continue. If shader
inputs or pixel workloads exist but no real output frame has been written, the
blocker is
`native_transition_pixel_output_proof_missing`.

An offscreen endpoint upload is still not an interactive transition. It proves
the native endpoint can receive the frame-buffer packet, but presets remain
locked until preview, Live Scrub, and playback consume the same compositor
output through their real interactive surfaces.

The offscreen proof endpoint must be named and treated as
`offscreenNativeProofSurface`. It must never be described as the preview,
Live Scrub, or playback output. Each interactive parity mode must instead bind
its own `interactiveNativeTransitionSurface`; otherwise the readiness report
must expose a mode-specific blocker such as
`native_transition_preview_interactive_surface_missing`,
`native_transition_liveScrub_interactive_surface_missing`, or
`native_transition_playback_interactive_surface_missing`.

Each interactive parity mode must also receive a source-derived frame-buffer
packet through its own `interactiveNativeTransitionSurface`. The readiness
report must record delivered frame state, byte count, checksum, and reason for
preview, Live Scrub, and playback independently. Missing delivery is a blocker
such as `native_transition_preview_interactive_surface_frame_missing`,
`native_transition_liveScrub_interactive_surface_frame_missing`, or
`native_transition_playback_interactive_surface_frame_missing`. A shared
offscreen proof frame or endpoint upload is not enough to expose or describe a
transition preset.

Accepted delivery is still not enough. Each interactive endpoint must present
an acquired native frame after delivery. The readiness report must record
presented image count, byte count, checksum, and reason for preview, Live
Scrub, and playback independently. Missing presentation is a blocker such as
`native_transition_preview_interactive_surface_presentation_missing`,
`native_transition_liveScrub_interactive_surface_presentation_missing`, or
`native_transition_playback_interactive_surface_presentation_missing`.

Accepted presentation on a proof endpoint is still not production parity. A
temporary `ImageReader` proof endpoint must identify itself as
`interactiveNativePresentationProofSurface`; it is not a real
`interactiveNativeTransitionSurface`. Preview, Live Scrub, and playback remain
blocked with `native_transition_<mode>_production_surface_missing` until each
mode binds and presents through its own production interactive transition
surface. Do not expose, describe, or recommend any transition preset based on a
proof endpoint, even if byte counts and checksums match.

Production interactive surfaces must be explicit render-plan data. Use
`interactiveSurfaceBindings` with one binding for each mode: `preview`,
`liveScrub`, and `playback`. Each binding must carry a stable `surfaceId`, set
`attached=true`, and declare `surfaceKind=interactiveNativeTransitionSurface`.
Missing bindings, unknown modes, proof-surface kinds, or detached surfaces are
not professional parity and must remain blocked.

When an interactive professional transition is active in preview, the
transition surface owns the canvas. The normal single-video preview surface is a
competing native surface and must be suppressed for that transition window, so
the user sees the compositor's A/B result rather than the untransitioned video.
If the transition surface is not registered or presentation is delayed, retry
the interactive render and report the native blocked reasons. Do not replace
that failure with thumbnails, poster frames, or Flutter fake zoom overlays.

The current `Distortion Zoom Transition In V1` implementation must not render
on every playback or Live Scrub tick. Its temporary renderer extracts source
video frames and composes bitmaps per request; device logs showed repeated
codec start/stop/release churn when it was driven by playback. Until the engine
has a cached, nonblocking native decoder/frame pipeline, this transition is
stationary-preview only. Playback and Live Scrub must keep using the normal
video preview path and must not repeatedly invoke the heavy transition renderer.
Fail closed for unsupported modes rather than retrying until the app stalls.

The user-facing replacement is `Zoom In Camera`. For the current mobile engine,
author it as a real-video surface transition: transform the playing video
surface with seam-aware scale, soft blur, and edge-safe overscan. Do not extract
source frames per tick, do not freeze thumbnails, and do not call the temporary
`distortionZoomInV1` renderer for playback or Live Scrub. The outgoing side
zooms into the seam, the incoming side settles from a zoomed-in state, and scale
must stay at or above `1.0` so the surface behaves like a motion-tiled zoom
without black borders. A future fully native version must use cached dual-video
decoder surfaces instead of repeated codec start/stop/release.

Any UI or agent-facing explanation of transition readiness must use the formal
readiness presentation model. Do not collapse readiness into a vague "not
ready" or "missing capabilities" string. Name the blocked stages in order so a
future agent can tell whether the problem is source binding, exact decode,
temporal accumulation, mirror-edge tiling, output-surface ownership, or
preview/live-scrub/playback parity.

After source URI binding and before exact frame decode, every transition plan
must pass a real video source probe. The probe must prove that each bound
`sourceUri` is openable as video and contains a video track. Do not let an
agent, UI, or renderer skip this step by using asset ids, generated proxies,
thumbnails, poster frames, or boundary-frame stills. If the probe is not
implemented or fails for either source, the readiness blocker is
`sourceMediaProbe` and the transition must remain unavailable.

The Android implementation probes `file://` and `content://` sources with
`MediaExtractor` and may report video MIME type, dimensions, duration, and frame
rate. Passing the source probe is only source truth; it is not permission to
render. Exact decode, dual-video decoder, temporal shutter accumulation,
mirror-edge tiling, output-surface ownership, and preview/live-scrub/playback/
parity must still pass. Export remains a later release gate.

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
- concrete source probe readiness;
- video MIME type, dimensions, duration, and frame rate when available;
- center sample source time;
- exact frame decode probe status and decoded frame time;
- decoded buffer probe status;
- decoded buffer count;
- whether every decoded output buffer is readable;
- center-sample decoded buffer byte count and checksum;
- `requiresExactFrameDecode=true`;
- `allowThumbnailFallback=false`;
- `allowBoundaryFreeze=false`.

The decoder session must not advance from metadata alone. It must require a
real source probe, a native exact-frame decode probe, and readable decoded
`MediaCodec` output buffers for outgoing and incoming shutter samples. A
decoded presentation timestamp without a readable output buffer is not
professional transition readiness. If either side cannot decode or cannot prove
readable decoded buffers, it must block with a native decoder reason such as
`native_dual_video_decoder_not_ready`, `native_exact_frame_decode_timeout`,
`native_exact_frame_decode_failed`, or
`native_exact_frame_output_buffer_not_ready`.
Passing this stage proves dual-video sampling readiness only; it is not
permission to expose a transition preset until temporal accumulation,
mirror-edge tiling, output surface, and parity stages also pass.

The decoder session must also prove a continuous live decode window. Exact
decoded shutter samples are not enough. Outgoing must cover the leading
transition window up to the seam, and incoming must cover the trailing window
from the seam. Agents and renderers must look for:

- `requiresContinuousFrameStream=true`;
- `liveDecodeWindowTimelineStartMs`;
- `liveDecodeWindowTimelineEndMs`;
- `liveDecodeWindowSourceStartMs`;
- `liveDecodeWindowSourceEndMs`;
- `liveDecodeCoverageSourceTimesMs`;
- `liveDecodeCoverageDecodedSampleCount`;
- `liveDecodeCoverageDecodedBufferCount`;
- `liveDecodeWindowReady=true`;
- `liveDecodeStreamProbeImplemented=true`;
- `liveDecodeStreamDecodedFrameCount`;
- `liveDecodeStreamReadableBufferCount`;
- `liveDecodeStreamFirstFrameTimeMs`;
- `liveDecodeStreamLastFrameTimeMs`;
- `liveDecodeStreamMinRequiredFrameCount`;
- `liveDecodeStreamCoverageReady=true`;
- `continuousSampleCoverageReady=true`.

If `continuousSampleCoverageReady` is false, the transition is still blocked
with `native_dual_video_live_decode_not_ready`. If the sequential stream probe
does not cover the live window, it also blocks with
`native_dual_video_live_decode_stream_not_ready` or the exact probe reason such
as `native_video_decode_stream_end_gap` or
`native_video_decode_stream_output_buffer_not_ready`. Do not treat decoded
center frames, boundary frames, thumbnails, poster frames, or shutter sample
probes as a playing transition. Android currently proves this stage with both
strict start/mid/end coverage samples and a sequential `MediaCodec` live-stream
probe for each live decode window; it is still decoder readiness only, not
permission to expose presets before renderer, output surface, and
preview/scrub/playback parity pass.

## Native Temporal Sample Accumulator Contract

After a dual-video decoder session exists, the compositor must bind outgoing
and incoming decoder tracks into temporal sample accumulators before any
transition can claim real motion blur support.

Each accumulator must preserve:

- source role: `outgoing` or `incoming`;
- input decoder track role;
- sample count;
- decoded sample count;
- decoded buffer count;
- whether all input shutter samples are decodable;
- whether all decoded input buffers are readable;
- deterministic sample weights;
- normalization mode, initially `weightedAverage`;
- `requiresTemporalShutter=true` when the render plan requests temporal shutter
  motion blur;
- `requiresExactFrameDecode=true`;
- `allowGaussianFallback=false`;
- `allowDecorativeSpeedLines=false`.

The accumulator session may be planned before a real implementation exists, but
it must report `accumulatorImplemented=false` and block transition exposure with
`native_temporal_sample_accumulator_missing`. If the decoder stage cannot prove
that every shutter sample is decodable, it must also block with
`native_temporal_sample_decode_not_ready`. If the decoder stage cannot prove
readable decoded buffers for every decoded sample, it must also block with
`native_temporal_sample_buffer_not_ready`.
If the decoder stage cannot prove a live decode stream through the transition
window, the accumulator must also block with
`native_temporal_live_decode_stream_not_ready`. Temporal shutter accumulation
cannot be considered professional when it only receives isolated decoded
samples; the source stream itself must remain playable through the outgoing and
incoming windows.

This is a hard quality boundary. Gaussian blur, poster-frame blur, line overlays,
radial decorative strokes, or any still-image substitute are not motion blur.

## Native Mirror-Edge Tiling Contract

After temporal sample accumulation exists, camera/zoom/push transitions that can
expose canvas borders must bind the outgoing and incoming accumulated frames
into mirror-edge tile plans before shader/effect evaluation.

Each tile plan must preserve:

- source role: `outgoing` or `incoming`;
- input accumulator id;
- sample count, decoded sample count, and whether input samples are decodable;
- `liveDecodeStreamCoverageReady`;
- `continuousSampleCoverageReady`;
- edge mode, usually `mirrorTile`;
- output overscan scale;
- `clipToCanvas=true`;
- `allowBlackBorders=false`;
- `allowFlutterOverlay=false`;
- `allowTimelineOverlay=false`.

The tiling session may be planned before a real implementation exists, but it
must report `tilerImplemented=false` and block transition exposure with
`native_mirror_edge_tiler_missing`. If temporal inputs are not decodable, it
must also block with `native_mirror_edge_input_samples_not_ready`.
If temporal inputs are not backed by live stream coverage, it must also block
with `native_mirror_edge_live_decode_stream_not_ready`.

Do not solve edge gaps by shrinking the video, stretching a thumbnail, drawing a
fake background, or placing transition pixels in the timeline area. The only
professional path is canvas-clipped native mirror-edge tiling.

## Native Render Pass Graph Contract

After live stream decode readiness, exact decode requests, temporal
accumulators, and required mirror-edge tiling plans exist, the compositor must
lower the frame into a renderer-agnostic pass graph before any concrete
transition implementation can claim support.

The pass graph must include the general phases required by professional video
transitions:

- live video stream decode for outgoing and incoming sources;
- exact video frame decode;
- temporal sample accumulation for outgoing and incoming sources;
- mirror-edge tile when the edge policy requires it;
- transition shader/effect evaluation;
- composition to the transition output surface.

Temporal accumulation passes must depend on the live stream decode pass as well
as exact decode request ids. A graph that contains exact frame decode but no
`decodeLiveVideoStreams` pass is not allowed to claim professional motion blur
or live video transition readiness.

The pass graph may be planned before a renderer exists, but it must propagate
upstream blockers from decode, temporal accumulation, and mirror-edge tiling,
then report that the renderer is not implemented. Planning a graph is not
permission to expose a transition preset. A preset becomes valid only when the
concrete native renderer executes this graph for preview, Live Scrub, and
playback parity. Export remains a separate later phase until explicitly built.

## Native Render Graph Executor Contract

After the pass graph is planned, the native compositor must validate graph
execution ownership before any concrete renderer draws pixels.

The executor must prove:

- pass order matches the professional sequence:
  `decodeLiveVideoStreams`, `decodeExactVideoFrames`,
  outgoing/incoming `temporalSampleAccumulator`, optional `mirrorEdgeTile`,
  `transitionShaderEvaluation`, and `composeToTransitionSurface`;
- pass dependencies only point to earlier passes in the graph;
- the final pass is `composeToTransitionSurface`;
- graph ownership is ready before any preview, Live Scrub, or playback mode can
  claim parity.

The executor may exist before a renderer exists. In that state it must return
ordered pass execution states and `graphOwnershipReady=true` when the graph is
well formed, but `canExecuteGraph=false` while `rendererImplemented=false`.
It must also report `drawsPixels=false`. Do not expose a transition preset just
because the executor can own the graph; a concrete native renderer must still
attach and render the graph for preview, Live Scrub, and playback parity.

## Native Output Surface Contract

After a render pass graph exists, the compositor must bind it to a single
professional output target: a native transition canvas surface clipped to the
preview/export canvas. This surface must be bound to the final
`composeToTransitionSurface` pass in the graph, not inferred from a loose
surface id.

The output-surface contract must explicitly forbid:

- Flutter overlay rendering;
- timeline overlay rendering;
- transformed PlatformView fallback rendering;
- drawing transition video into the timeline area;
- any renderer path that is not canvas-clipped and native-owned.

The output-surface plan must preserve:

- render pass count;
- output pass id;
- output pass type, which must be `composeToTransitionSurface`;
- output pass inputs;
- `outputPassBound`;
- `renderGraphOutputReady`.

If the graph has no valid output pass, the compositor must block with
`native_transition_output_pass_missing`. Do not route through a side surface,
Flutter overlay, timeline overlay, or PlatformView transform to repair a
missing graph output.

Planning an output surface is still not permission to expose a transition. The
surface must inherit upstream blockers and report `rendererImplemented=false`
until a concrete native renderer executes the graph for preview, Live Scrub,
and playback parity. Export remains a separate later phase.

## Native Surface Renderer Skeleton Contract

After the graph executor and output surface are planned, the compositor may
create a native surface-renderer skeleton. This skeleton must attach exactly
three things to one contract:

- the ordered render graph executor;
- the canvas-clipped native transition output surface;
- the final `composeToTransitionSurface` output pass.

The skeleton is still not a concrete renderer. Until real pixels are drawn, it
must report:

- `surfaceRendererImplemented=true`;
- `outputSurfaceAttached=true`;
- `rendererImplemented=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderSurface=false`.

The required blocker is
`native_transition_surface_renderer_pixels_missing`. Do not expose a transition
preset, manual editor, or AI transition path just because a native surface is
attached. The next professional milestone is a concrete native renderer that
draws real transition pixels through this attached graph and surface. Flutter
overlays, timeline-area drawing, transformed PlatformViews, still frames, fake
motion blur, and decorative substitutes remain forbidden.

## Native Frame Render Command Contract

Before a concrete renderer draws, the compositor must lower the ordered render
graph into explicit per-frame native render commands.

Each command must preserve:

- command id;
- source pass id;
- source pass type;
- source pass role;
- pass index;
- input pass ids;
- output target;
- whether it writes to the final native transition canvas surface;
- `requiresRealPixels=true`;
- command-level blockers.

The command buffer may exist before the renderer exists. In that state it must
report:

- `rendererCommandBufferImplemented=true`;
- `rendererImplemented=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canSubmitCommands=false`;
- `canRenderFrame=false`.

The required blocker is
`native_transition_frame_command_renderer_missing`. Do not treat a complete
command graph as a visual transition. It is only the final planning layer before
a real native renderer submits commands and writes pixels to the
`nativeTransitionCanvasSurface`.

## Native Renderer Backend Contract

After frame render commands exist, the compositor must attach them to the native
renderer backend before any transition can claim it can draw.

The renderer backend gate must preserve:

- renderer backend id;
- frame render command buffer id;
- output surface id;
- output target, which must be `nativeTransitionCanvasSurface`;
- GPU/OpenGL ES availability;
- command-buffer readiness;
- output-surface attachment;
- whether a real draw loop exists;
- whether a concrete renderer writes pixels.

The backend may report `rendererBackendImplemented=true`,
`gpuContextAvailable=true`, `backendReady=true`, and `canSubmitCommands=true`
when the platform path can accept the command buffer, but this still does not
unlock transitions. A renderer backend is not a pixel renderer. Until the next
draw-loop stage and pixel renderer exist, it must still report:

- `drawLoopImplemented=false`;
- `rendererImplemented=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

Do not replace this stage with a Flutter overlay, timeline-area drawing,
transformed PlatformView, Gaussian blur, speed-line shapes, still-frame zoom, or
any renderer-specific fake.

## Native Renderer Draw Loop Contract

After the renderer backend accepts the command buffer, the compositor must create
an ordered draw-loop submission plan before any shader or transition pixel
program can run.

The draw-loop contract must preserve:

- renderer draw-loop id;
- renderer backend id;
- frame render command buffer id;
- ordered draw submissions;
- each submission's command id, pass id, pass type, output target, and whether it
  writes to the final native transition canvas surface;
- `requiresRealPixels=true` for every submission.

This stage may report `drawLoopImplemented=true` and `canSubmitCommands=true`
when commands are ordered and can be submitted to the backend. It still must not
claim visual transition support. This stage proves submission order only; shader
evaluation and pixel rendering are separate downstream gates. It must report:

- `shaderEvaluatorImplemented=false`;
- `pixelRendererImplemented=false`;
- `rendererImplemented=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

When the backend and commands are ready, the draw loop should not carry shader
or pixel-renderer blockers. A draw-loop submission plan is not a rendered
transition; it is the final scheduling layer before shader evaluation.

## Native Transition Shader Evaluation Contract

After ordered draw submissions exist, the compositor may evaluate the transition
shader/effect program for the current transition frame. This is still not a
pixel renderer.

The shader-evaluation gate must preserve:

- transition shader evaluation id;
- transition shader program id;
- shader family / transition definition id;
- renderer draw-loop id;
- every bound shader input with its submission id, command id, pass id, pass
  type, output target, and `requiresRealPixels=true`;
- whether temporal shutter samples are required;
- whether mirror-edge tiling is required.

This stage may report:

- `shaderEvaluatorImplemented=true`;
- `shaderProgramReady=true`;
- `shaderInputsBound=true`;
- `canEvaluateShader=true`.

It still must report:

- `pixelRendererImplemented=false`;
- `rendererImplemented=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

Do not expose a transition preset just because the shader can be evaluated. The
next professional milestone is a concrete native pixel renderer that consumes
the shader inputs and writes real pixels to `nativeTransitionCanvasSurface`.
Still-frame zoom, transformed native preview surfaces, Gaussian blur, fake
speed lines, Flutter overlays, and timeline-area rendering remain forbidden.

## Native Transition Pixel Renderer Contract

After shader evaluation is ready, the compositor must bind shader inputs into a
pixel workload for the final native transition canvas surface. This is a
workload-binding gate, not proof that pixels were rendered.

The pixel-workload gate must preserve:

- transition pixel renderer id;
- pixel program id;
- transition shader evaluation id;
- transition shader program id;
- shader family / transition definition id;
- every pixel input with its shader input id, draw submission id, command id,
  pass id, pass type, output target, and `requiresRealPixels=true`;
- `pixelWorkloadBound`;
- temporal shutter and mirror-edge requirements inherited from the shader plan.

If the workload is bound, the gate may advance while still reporting:

- `pixelRendererImplemented=false`;
- `pixelRendererReady=false`;
- `rendererImplemented=false`;
- `canRenderPixels=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

Do not expose any transition preset, manual transition editor, or AI-generated
transition just because shader inputs and pixel workload are bound. The next
professional milestone is a native pixel frame buffer, a native frame-buffer
writer that writes temporal video pixels, followed by pixel render execution and
native pixel output proof.

## Native Transition Pixel Frame Buffer Contract

After pixel workload binding, the compositor must allocate and own a native
frame buffer for the final transition canvas before it attempts pixel execution.
This is the contract that prevents a renderer from treating thumbnails, poster
frames, cached boundary images, or synthetic placeholders as renderable video.
The current Android foundation allocates bounded `DirectByteBuffer` storage for
valid canvas-sized `rgba8888` buffers; allocation alone is not rendering and
must not be treated as temporal pixel output.

The pixel-frame-buffer gate must preserve:

- transition pixel frame buffer id;
- transition pixel renderer id;
- pixel program id;
- output surface id;
- output target and output framebuffer target, both targeting
  `nativeTransitionCanvasSurface`;
- frame buffer width and height matching the composition canvas;
- frame buffer format, currently `rgba8888`;
- frame buffer byte count;
- frame buffer memory class, currently `directByteBuffer` when allocation
  succeeds;
- frame buffer allocation reason when allocation fails;
- `pixelWorkloadBound`;
- `outputFramebufferBound`;
- `frameBufferAllocated`;
- `frameBufferReady`;
- `allowsSyntheticPixels=false`;
- `allowsPosterFrame=false`;
- `allowsThumbnailFallback=false`;
- `allowsBoundaryFreeze=false`.

When allocation succeeds but no concrete native renderer has filled the buffer,
this gate must report:

- `frameBufferAllocated=true`;
- `frameBufferReady=true`;
- `rendererImplemented=false`;
- `canRenderPixels=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

The frame-buffer allocation stage may advance once allocation succeeds, but the
next writer gate must remain blocked until real temporal video pixels are
written. If allocation itself fails, use the specific allocation blocker such as
`native_transition_pixel_frame_buffer_invalid_size`,
`native_transition_pixel_frame_buffer_too_large`, or
`native_transition_pixel_frame_buffer_allocation_failed`. Do not let a later
stage skip this gate.

## Native Transition Pixel Frame Buffer Writer Contract

After the native pixel frame buffer is allocated, the compositor must bind a
native writer to that exact buffer before pixel execution. This gate exists so
agents cannot call an allocated buffer "rendered" while no live video pixels
were copied, mixed, or accumulated into it.

The writer gate must preserve:

- transition pixel frame-buffer writer id;
- transition pixel frame buffer id;
- transition pixel renderer id;
- output framebuffer target, which must be `nativeTransitionCanvasSurface`;
- frame buffer width, height, format, byte count, and memory class;
- `writerBoundToFrameBuffer`;
- `requiresTemporalSamples=true`;
- `requiresDualSourceSamples=true` for two-clip transitions;
- `allowsStillFrameWrite=false`;
- `allowsSyntheticPixels=false`;
- `allowsPosterFrame=false`;
- `allowsThumbnailFallback=false`;
- `allowsBoundaryFreeze=false`;
- `writerImplemented`;
- `writerReady`;
- `canWriteTemporalPixels`;
- `wroteTemporalPixels`;
- `frameBufferContainsRealPixels`.
- writer temporal sample count;
- writer extracted frame count;
- writer frame-buffer write byte count;
- writer frame-buffer checksum;
- writer source-frame extractor;
- writer canvas fill mode;
- writer reason when blocked.

The current Android foundation binds a native writer and writes real
outgoing/incoming source-frame pixels into the allocated `DirectByteBuffer`.
The writer extracts temporal shutter samples with
`MediaMetadataRetriever.getFrameAtTime`, composites them into a canvas-sized
`rgba8888` buffer with center-crop-fill, and records sample count,
extracted-frame count, write byte count, checksum, extractor, and fill mode.
This is source-pixel proof only. It is not final transition rendering and must
not unlock presets, manual transitions, AI transitions, preview parity, Live
Scrub parity, or playback parity by itself.

If the source-frame writer cannot extract temporal pixels, this gate must
report:

- `writerReady=false`;
- `canWriteTemporalPixels=false`;
- `wroteTemporalPixels=false`;
- `frameBufferContainsRealPixels=false`;
- `rendererImplemented=false`;
- `canRenderPixels=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

The required blockers after successful frame-buffer allocation are whichever
specific writer reasons apply, such as
`native_transition_pixel_frame_buffer_writer_missing`,
`native_transition_temporal_dual_source_pixels_missing`,
`native_transition_pixel_frame_buffer_write_failed`, or
`native_transition_pixel_frame_buffer_pixels_missing`. Pixel render execution
must depend on this writer gate. It may not read a poster, thumbnail, boundary
still, or single frozen sample.

## Native Transition Pixel Render Execution Contract

After the native pixel frame buffer is allocated and the writer has filled it
with real temporal video pixels, the compositor must bind that workload to the
native output framebuffer and attempt the concrete pixel-render execution path.
This is the stage that remains blocked until real pixels are written.

The pixel-render execution gate must preserve:

- transition pixel render execution id;
- pixel output frame id;
- transition pixel renderer id;
- pixel program id;
- output surface id;
- output target and output framebuffer target, which must be
  `nativeTransitionCanvasSurface`;
- `pixelWorkloadBound`;
- `outputFramebufferBound`;
- `frameBufferReady`;
- `writerReady`;
- `wroteTemporalPixels`;
- `frameBufferContainsRealPixels`;
- pixel output source frame-buffer id;
- pixel output write mode;
- pixel output byte count;
- pixel output checksum;
- pixel output reason;
- whether pixel output was written.

The current Android foundation can consume the temporal frame-buffer writer
output into an offscreen native pixel-output frame. This removes only the
"nothing was written to an output frame" gap when source-derived temporal
pixels exist. It must not unlock a transition preset, because an offscreen frame
is not the same as writing the final native preview/playback surface.

Pixel output proof must also build a surface-upload packet contract from that
offscreen output. The packet must preserve:

- output surface upload packet id;
- output surface upload packet readiness;
- output surface upload source frame-buffer id;
- output surface upload byte count;
- output surface upload checksum;
- surface upload renderer implementation status;
- output surface upload reason when blocked.

This packet is still not final rendering. It only proves which native
source-derived frame should be uploaded to `nativeTransitionCanvasSurface`.
Transitions remain locked until a concrete surface-upload renderer writes that
packet into the final native surface and parity confirms preview, Live Scrub,
and playback all read the same output.

The surface-upload renderer contract may be structurally ready once a real
packet exists. That is still not enough. The final blocker must move to the
native surface endpoint: `native_transition_surface_endpoint_missing`. This
keeps the risky endpoint attachment separate from packet creation and renderer
readiness.

Until a concrete native surface renderer exists, this gate must report:

- `pixelOutputReady=false`;
- `rendererImplemented=false`;
- `canRenderPixels=false`;
- `rendersRealPixels=false`;
- `drawsPixels=false`;
- `canRenderFrame=false`.

The required blockers include `native_transition_pixel_frame_buffer_not_ready`,
`native_transition_pixel_frame_buffer_writer_missing`,
`native_transition_pixel_frame_buffer_temporal_pixels_missing`,
`native_transition_pixel_frame_buffer_pixels_missing`,
`native_transition_pixel_renderer_missing`,
`native_transition_pixel_output_missing`,
`native_transition_pixel_output_not_ready`, and
`native_transition_renderer_pixels_missing`. Do not expose transitions before
this gate can render real pixels and the pixel output proof can verify that the
result is written to the native output surface.

## Native Transition Pixel Output Proof Contract

After pixel render execution, the compositor must prove that the executed
workload produced a real output frame on `nativeTransitionCanvasSurface`. This
is the anti-fallback gate before preview, Live Scrub, and playback parity.

The proof must preserve:

- transition pixel output proof id;
- transition pixel render execution id;
- pixel output frame id;
- output surface id;
- output target and output framebuffer target;
- `outputSurfaceIsNative`;
- `writesOnlyToNativeSurface`;
- `forbidsFlutterOverlay`;
- `forbidsTimelineOverlay`;
- `forbidsPlatformViewTransform`;
- `pixelOutputWritten`;
- `pixelOutputReady`;
- `outputProofReady`.

If no concrete frame has been written, the blocker is
`native_transition_pixel_output_proof_missing`. Do not mark parity ready while
this proof is false, even when every previous planning gate is green.

## Native Parity Output Contract

After native pixel output proof passes, the compositor must prove that the same
transition output contract is valid for every required mode:

- preview;
- Live Scrub;
- playback;
- future export.

An agent must not describe a transition as implemented if one mode uses the
native compositor while another mode uses a fallback. Preview, Live Scrub, and
playback must share the same native transition output surface contract and the
same final `composeToTransitionSurface` output pass before a preset/manual/AI
transition is exposed.

Each parity output must carry:

- output pass id;
- output pass type;
- output pass inputs;
- `outputPassBound`;
- `renderGraphOutputReady`;
- pixel output proof id;
- `outputProofReady`;
- upload packet readiness;
- surface-upload renderer readiness;
- final surface endpoint attachment state.

Mode-level `canRender` is false if the output pass binding is missing, even if
the output surface id matches. Mode-level `canRender` is also false while
`outputProofReady=false` or `outputSurfaceEndpointAttached=false`; the expected
blocker is `native_transition_<mode>_surface_endpoint_missing` until the real
native endpoint exists. Export must later attach to the same contract; it must
not fork into a different renderer.

## Native Surface Endpoint Preflight Stage

The readiness preflight must expose a dedicated `transitionSurfaceEndpoint`
stage between `transitionPixelOutputProof` and `parityOutputs`.

This stage advances only when all of the following are true:

- `outputSurfaceUploadPacketReady=true`;
- `surfaceUploadRendererReady=true`;
- `outputSurfaceEndpointAttached=true`.

If the endpoint is missing, the blocker must be
`native_transition_surface_endpoint_missing`. Do not hide this state inside a
generic parity failure, and do not expose any preset/manual/AI transition while
this stage is blocked.

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

## Distortion Zoom Transition In V1 Contract

`Distortion Zoom Transition In V1` is the first zoom/distortion preset on the
native transition renderer path.

Canonical ids:

- built-in transition definition: `distortion_zoom_in_v1`;
- native compositor definition: `distortionZoomInV1`;
- display label: `Distortion Zoom Transition In V1`.

Default timing:

- duration: `4000ms`;
- seam: center of the transition window;
- outgoing A contributes the playing tail before the seam;
- incoming B contributes the playing head after the seam;
- the transition must never jump to the middle or end of incoming B.

Required parameters:

- `outgoingBoostScale`: default `3.0`;
- `incomingStartScale`: default `0.25`;
- `lensDistortionPeak`: default around `0.32`;
- `chromaticAberrationPeak`: default around `0.08`;
- `motionTileOutputScaleX`: default `4.0`;
- `motionTileOutputScaleY`: default `4.0`.

Required rendering behavior:

- sample real source video frames from A and B over the transition window;
- accumulate temporal shutter samples for motion blur;
- apply mirror-edge tiling before transform/distortion so scaled-down B never
  reveals black canvas edges;
- apply lens-style distortion from source pixels;
- optional RGB/chromatic split must be derived from source pixels, not drawn as
  decorative lines;
- output must go through the native transition surface contract used by
  preview, Live Scrub, and playback.

Forbidden behavior:

- no thumbnail zoom;
- no frozen poster-frame zoom;
- no boundary-frame freeze;
- no fake radial speed lines;
- no Gaussian-only blur pretending to be motion blur;
- no Flutter overlay or timeline-area rendering;
- no transformed single Android video surface as the transition.

## Current Engine Support

The current ReFusion engine intentionally gates new video transition authoring
through the professional native compositor. Existing saved Zoom transitions must
not draw fake speed lines, frozen cards, or Gaussian transition blur.

The `Zoom In Pro` test path is closed. It did not satisfy dual-video sampling,
temporal shutter motion blur, mirror-edge tiling, output-surface ownership, or
parity readiness, and it must not be used as a replacement for Zoom In Camera.

The current app contains a strict professional transition compositor contract
for native rendering. Agents must describe Zoom In Camera and future video
transitions in terms of real outgoing/incoming source-time samples,
shutter-angle temporal sampling, mirror-edge tiling, a render plan, and a real
interactive output surface. Do not describe any professional transition as an
overlay, a card, a blurred thumbnail, or a decorative speed-line effect.

The app also has a native capability bridge at
`com.refusion.app/professional_video_transition_compositor.getCapabilities`.
No transition preset, manual transition path, or AI transition path may become
clickable when that native response is missing any required capability: dual
video sampling, temporal motion blur, mirror-edge tiling, preview parity, live
scrub parity, and playback parity. Current builds report these as unavailable
until the real native renderer ships. Export parity is tracked separately.

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
tracks, requires the Android real-source probe, and performs a native
`MediaCodec` center-sample decode probe on both sides. It now also proves
readable decoded output buffers by reporting decoded buffer counts, center
buffer byte counts, and deterministic checksums. Agents must not substitute
MediaMetadataRetriever thumbnails, cached posters, frozen boundary stills, or
timestamp-only decode metadata for this contract. Even when dual-video
sampling passes, the transition remains locked until temporal accumulation,
mirror-edge tiling, output-surface ownership, and parity outputs pass.

The current native foundation also defines `planTemporalSampleAccumulator`.
This endpoint inherits decoded sample counts and decoded buffer readiness from
the dual decoder, records accumulated decoded-buffer byte counts and checksums,
and reports readiness only when both outgoing and incoming shutter sample sets
are exactly decodable with readable output buffers. Agents must still not claim
user-visible real motion blur until the downstream native renderer executes the
render-pass graph onto the transition output surface.

The current native foundation also defines `planMirrorEdgeTiling`. It consumes
the temporal accumulator readiness signals, records overscan scales, and keeps
black borders, Flutter overlays, timeline overlays, thumbnails, and
single-surface substitutes forbidden. Passing mirror-edge readiness is not
permission to expose a preset; render-pass graph execution, output-surface
ownership, and preview/live-scrub/playback parity must still pass.

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
requires preview, Live Scrub, and playback to share the same output surface
contract and keeps every interactive mode blocked until a concrete native
renderer can render without fallback divergence. Export remains a later phase
and must join the same contract before export support is claimed.

The current interactive foundation also defines a real Flutter-to-Android
surface binding. Flutter creates `ProfessionalVideoTransitionSurface` as a
dedicated Android PlatformView inside the preview canvas and calls
`renderInteractiveFrame` with the active render plan, timeline time, mode, and
surface id. Android accepts a bound surface id only when it was registered by
that visible PlatformView as an `interactiveNativeTransitionSurface`; it must
not allocate an offscreen `ImageReader` for production-bound preview, Live
Scrub, or playback. If the surface is missing, the correct behavior is a hard
blocker such as `native_transition_interactive_surface_not_registered`, not a
fake still-frame preview.

The first selectable transition on this path was `Cross Dissolve`. The first
zoom/distortion preset on the same path is now `Distortion Zoom Transition In
V1`. Do not ask agents to expose `Fade Black`, legacy `Zoom In Camera`, or any
other named transition until each one has its own renderer definition that
produces real source-derived pixels through the same surface contract.

Root and Scene Contents transition bridges must preflight zoom-family
professional presets before staging them. If the outgoing/incoming source URI,
source handle window, transition boundary, or native definition binding cannot
build a real render plan, the app must block with a visible diagnostic. Do not
convert that failure into a Flutter thumbnail zoom, poster-frame fallback, or
empty authored transition.

Do not promise interactive transition support until preview, live scrub, and
playback use the same compositor contract. Do not promise export support until
the export renderer later joins that same contract.
