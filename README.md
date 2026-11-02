# Flowr Android — 0.45.0

A local Canvas/WebGL motion editor wrapped with Capacitor. This archive contains source code, not a compiled APK.

## Latest: synchronized video cutouts in preview and export

Fixed a shared frame-timing problem affecting Roto Brush and Auto Track Mask: the displayed video could still be an earlier decoded frame while the matte/path had advanced to the timeline clock. The renderer now uses a paired source snapshot and its presented media timestamp. Rapid seeks finish in order and redraw after decoding; playback no longer reuses a stale seek target. Tracked mask paths and roto mattes follow the displayed source frame, including during video export. Manual mask animation and layer transforms keep their own timeline behavior.

**Existing roto mattes:** open **Edit Rotoscope Brush → Freeze & Apply** once. This refreshes older stored sample timestamps without rebrushing or rerunning segmentation. **Existing Auto Track Mask results:** run Analyze and Apply again to generate keys with corrected source timestamps. Saved legacy results are identified in the Masks drawer.

See [video frame synchronization and existing-project repair](docs/video-frame-sync.md).

## Sharper mobile roto preview and local motion propagation

The Rotoscope workspace now matches the main app's **520px frame** (or its full mobile width). Source previews use a separate high-resolution image, up to 2048 pixels on the longest side, rather than enlarging the small segmentation raster. Source/Overlay/Cutout and inspection backgrounds retain source detail while strokes and boundaries stay in matte coordinates.

Propagation combines foreground feature motion with bounded RGB patch matching and local deformation. Low output sample rates now use smaller internal tracking steps to reduce jumps. Automatic interior seeds are checked against the preceding foreground/background appearance so occlusion can remove stale foreground. Failed analysis retains the last accepted draft and offers **Go to stopped frame**. Opening at a video's end selects backward analysis automatically. New analyses default to 384 working pixels and zero extra feather; larger detail modes remain available.

## Smart Roto Brush with automatic connected-edge selection

The default **+ Smart Brush** now starts from a stroke inside the subject, without requiring a subject box or an initial red-background stroke. A connected-edge competition pass grows the selection along the image's color boundaries; unrelated disconnected regions need their own green stroke. **Alt** temporarily subtracts. The edge-aware brush footprint stops oversized stamps from forcing contrasting background pixels into the subject. Live selection previews, a magenta boundary and a brush-size cursor show what is being selected while drawing.

Added spatially local foreground/background edge-color estimation and 768/1024-pixel detail modes. Detail modes solve the region at up to 512 pixels and refine edges against the higher-resolution source. This is classical edge/color inference, not an Adobe or neural semantic subject detector: different-colored parts and weak/occluded boundaries may still need strokes. Saved v0.42 mattes remain compatible.

## Rotoscope Brush

Select an image or video layer and open **M → Rotoscope Brush · character cutout**. Draw a subject box, correct with green/red strokes, refine difficult edges, and propagate forward/backward through video. Review the cutout/alpha on black or white, then **Freeze & Apply**. The matte follows the layer and clip source time, intersects existing masks, works before effects, saves with the project and preserves transparency in PNG export.

Includes local graph-cut segmentation, fractional edge alpha, color-fringe cleanup, temporal edge stabilization, brush-constraint propagation, correction anchors, zoom/pan, stop/cancel, and final feather/shift/invert/bypass controls. Up to 600 samples per layer at 256–512 working pixels, subject to compressed project size. This is an AE-style assisted workflow, not Adobe's proprietary neural Roto Brush implementation; complex hair, similar colors and occlusion need corrections. See [Rotoscope Brush workflow and limits](docs/rotoscope.md).

## Selected-layer mask previews and tracking fixes

The mask editor now defaults to **Layer source / Layer cutout**, showing only the layer being masked. Its header names that layer. **Composition context** is a separate view option, used by default for adjustment layers. Other-layer masks, visibility and effects are retained; editing a text mask no longer shows an unrelated character/background layer.

Text tracking replaces the selected seed plus the layer's static Auto Trace batch with live character masks, rather than retaining old stationary traced paths. Text, video and still-layer tracking dialogs show the relevant action. Source video seeks refresh the editor; video analysis waits for frame presentation before sampling. Applying tracked results ignores unrelated UI/transform changes while still rejecting changed video/seed data.

## Animated text masks and video mask tracking

For animated text, open **M → Follow text animation**. Native masks are linked to each character's actual position, rotation, scale, tracking, stagger and loops, with counters/holes retained. Rebuild after editing text/font; use **Freeze text tracking** to turn a live path into a normal editable mask at the current frame.

For a moving character in video, draw a seed mask, choose its **Auto Track Mask** button, set direction/range/sample rate and choose Move, Move/Scale/Rotate, or Deform. Analyze, scrub the result, then Apply tracked keys. Tracking runs locally on decoded video features, generates editable Mask Path keys, and stops on low confidence. Pose changes and occlusions still need review; this is assisted tracking rather than AI subject segmentation. See [tracking workflow and limits](docs/mask-tracking.md).

## Auto Trace for text and images

Open **M → Auto Trace · text / PNG**. Flowr reads the layer's source alpha or Bright/Dark contrast and previews editable contour masks. Separate regions become Add masks; interior holes become Subtract masks. Threshold, tolerance, minimum region area and curve smoothing are adjustable. **Add traced masks to layer** appends the paths in one Undo action while retaining existing masks.

Text traces full source letters, excluding character animation, text glow and the text box; layer effects and existing owner masks are excluded. PNGs use their source transparency. Opaque high-contrast artwork can use Bright or Dark; Alpha on an opaque image traces its bounds. This is a current-source contour trace, not subject segmentation or automatic tracking. See [Auto Trace workflow](docs/auto-trace.md).

## Advanced masks on one layer

The Masks drawer has an explicit **+ Add another mask** button, a count and a new-mask mode selector. Keep separate regions with Add masks and cut holes with Subtract; all masks remain independent. Up to 32 masks can be ordered on each layer. Other-mask outlines and names are shown while editing, with individual colors and a visibility switch.

Use **Move mask** to drag an entire path. **Transform / copy path** exposes X/Y movement, independent scale and rotation around the vertex center; handles transform with the path. Changes remain a draft until Done and can create a new shape keyframe at the current time. Copy a path from a mask card or editor, paste into an editor, or use **Paste path as new mask**. Linked Feather X/Y uses the X animation for both axes while preserving the independent Y track for unlinking.

## Layer masks and character cutouts

Open **M** on the left of the preview, or **Transform → Masks**. Draw a closed Pen path around a character, drag Bézier handles to refine edges, then choose Done. Rectangle and ellipse tools are included. Use Source/Cutout view, Fit–800% zoom and Pan while editing.

Each visual layer supports ordered Add, Subtract, Intersect, Difference, Lighten, Darken and None masks, with inversion, opacity, independent X/Y feather and expansion. Animate Mask Path or numeric controls in the layer timeline; masks follow layer and parent transforms and run before effects. Masks also work on precomps and adjustment layers, in motion blur, PNG/video rendering, project saves and Undo/Redo. See [mask workflow and limits](docs/layer-masks.md).

## Text Animation library

Text animation presets now **add independent groups** instead of replacing the previous preset. Combine multiple presets, including repeated applications of the same preset, on one text layer. Each group keeps its own keys, stagger/loop settings and enable/remove controls under Timeline → Text. In and Loop begin at the playhead; Out is placed at the layer end. In/Out durations fit within 40% of the layer by default, leaving a visible middle. Groups contribute neutral values before their start. Neon and Outline styles also combine; Clean Title remains an explicit appearance reset.

Click **P** on the left edge of the preview. Its tab sizing, framed panel width and slide animation mirror the FX/Text drawers on the right. Preset cards remain scrollable.

**Presets → Text Animation** contains 92 native animations: 40 In, 32 Out and 20 Loop, plus the eight existing style/effect shortcuts in the same category. Use the In, Out, Loop or All labels; the extra category and grouping menus are removed. Sixteen new typography presets include matching kinetic stacks, editorial tracking, domino letters, rubber stamps, ribbon lines, magnetic type and cinema titles, plus Conveyor and Accordion loops. Cards provide animated previews and direction labels. Smooth native curves maintain velocity through intermediate keys and across loop seams, with controlled stops at extrema. Typewriter holds and intentional flicker/jitter remain authored styles. Existing custom curve edits are preserved. Applied animations remain editable in the text layer timeline, including stagger/order and loop settings. See [preset list and editing](docs/text-animation-presets.md).

Removed fontpack-4e2d2f92b6d6.ttf (FTY STRATEGYCIDE NCV); 250 bundled fonts remain, with old references safely falling back.

The Blood Text preset, custom Blood effects and controls are removed. Old projects open with their source text and unrelated effects retained; managed Blood maps/effects are removed.

Bundled 3D fonts now load through compressed local script assets, so opening www/index.html directly works without a local server. All 250 remaining font outlines pass local-file loading checks. The font browser changes actual 3D geometry. Empty CFF contours are removed during 3D conversion to prevent the LORD OF THE RING font from crashing shape construction. Custom 3D font import accepts TTF and OTF up to 16 MB. Browser checks cover local and hosted selection, save/reopen and PNG export. Physical Android testing remains outstanding.

## 3D Scene Setup

Tap **Add → 3D Scene**, then use Scene Setup to build a scene. **Objects** adds primitives, extruded text or local models; **Material** edits surfaces; **Transform** animates objects and five groups; **Scene** controls camera, lights and environment. Apply returns the 3D scene to the normal timeline as a rendered layer. Reopen it from Transform → Open 3D Scene Setup. See the 0.17 update and 0.16 section below for formats, limits and validation.

## Earlier updates

### MP3 export reliability
- Falls back from AudioWorklet to ScriptProcessor when module loading is rejected, unavailable or times out.
- Falls back from Worker encoding to the same bundled local MP3/WAV encoder when worker startup fails.
- Explicitly registers native Filesystem/Share plugins with the bundled official Capacitor runtime. Android exports are written to a unique cache path, then offered through the share sheet.
- Keeps completed audio in memory if saving or sharing fails. Tap **Save / share audio** to retry. You can also reopen **Export → Advanced → Save last rendered audio**. The retained audio lasts for the current app session only.
- Shows stage-specific errors and detects stalled capture. MP3 converts unsupported device sample rates to a supported encoder rate rather than rejecting them. WAV keeps the active device sample rate.

### Wave Warp and presets
Wave Warp includes **Sine, Square, Triangle, Sawtooth, Circle, Semicircle, Uncircle, Noise, and Smooth Noise**. Wave Type and Pinning use dropdowns with discrete keyframes. Phase, direction, height, wavelength, speed and noise seed are adjustable. Pinning holds the selected edges of the composition canvas in place.

Starting presets include Gentle Sine, Triangle Ripple, Square Glitch, Sawtooth Drift, Liquid Noise and Flag, plus presets for glow, displacement, RGB split, shake, vignette, grain, blur and the other visual effects. Applying a preset replaces that effect's settings and keyframes; Undo restores them. Editing a parameter clears the active preset name.

These are original Flowr settings and mathematical implementations, not Adobe `.ffx` files or pixel-identical Adobe rendering. [Adobe's documented Wave Warp controls](https://helpx.adobe.com/after-effects/desktop/apply-effects-and-animation-presets/list-of-effects/distort-effects.html) informed the Wave Type, Phase and Pinning workflow.

## Editing workspace
The editor uses a mobile layout at every screen size. Editing keeps a large composition preview, transport, compact selected-clip timeline and active tool controls visible together. The Timeline view expands all layers; the Edit view shows the selected clip above the tool controls. Desktop previews stay phone-sized; touch devices fill the available phone screen, including landscape. Layer appearance and Add effect controls collapse so the applied effect presets are easier to reach. Includes timeline resizing, layer search, searchable/collapsible effect cards, keyframes and curve controls, effect reordering/reset, and preview-only FX bypass. Exports and thumbnails always include enabled effects, regardless of preview bypass.

Visual effects include Glow, Posterize, Find Edges, Duotone, Emboss, Directional Blur, Wave Warp, Turbulent Displace, Film Grain, RGB Split, Digital Glitch, Camera Shake, Vignette and the existing basic effects.

## Import and export
- Batch import images, video and audio, stacked at the playhead or placed sequentially. Device decoders determine which codecs can be opened, including inside MOV/QuickTime and AVI files. Single-file limit: 512 MB.
- Export MP3/WAV audio, PNG/JPG frames, and MP4 video when the device supports encoding it. A single WebM fallback appears if MP4 is unavailable. MOV/AVI export is not implemented.
- Advanced export includes resolution, FPS, bitrate, time range, audio and portable `.flowr` project backup. Backups embed source media up to 100 MB; custom 2D text fonts are not embedded. Imported 3D TTF assets are embedded. Composition limit: 10 minutes.

## Build APK
Put the project contents in a GitHub repository. Run **Actions → Build APK → Run workflow**, then download the `flowr-apk` artifact when it finishes. The workflow installs dependencies, creates the Android project, generates icons, syncs plugins and builds a debug APK.

For local development:

```sh
npm install
npx cap add android
npx @capacitor/assets generate --android
npx cap sync android
npx cap open android
```

Always rebuild and install the new APK after replacing the web source. Native plugin fixes require Capacitor sync; editing an older installed APK's HTML alone will not install missing plugins.

## Validation
- Browser checks passed at desktop, portrait-phone and landscape-phone sizes.
- Real browser MP3/WAV/PNG downloads passed. MP3/WAV also passed with worker creation and AudioWorklet loading deliberately rejected.
- Mock native plugin checks confirmed registration, cache saving, MP3 retention after share failure and successful retry. These are not physical Android share-sheet tests.
- MP3 sample-rate checks pass at 22.05, 44.1, 48 and 96 kHz. Independent ffprobe inspection identifies valid stereo MP3 and WAV. MP3 has normal encoder padding; captured WAV duration matches the selected range.
- Wave types, edge pinning, zero-height neutrality, deterministic noise, preset ranges, preset application/undo and discrete waveform keyframes pass automated checks.

Optional test tools: `npm install --no-save playwright @napi-rs/canvas`. Install Playwright Chromium or set `FLOWR_CHROMIUM` to a local browser executable. Tests are in `tests/`; the browser export test starts its own local server.

## Remaining limits
Physical Android testing and APK compilation were not available. The original device failure cannot be conclusively attributed without its error text; the unsupported fallback and save paths identified in the code are now handled and tested.

Rendering is real time: keep Flowr open. The engine uses 8-bit Canvas RGBA; complex CPU effects can slow high-resolution playback and export. Video capture targets the chosen FPS but is not a frame-perfect offline renderer. The streaming MP3 resampler uses linear interpolation, not mastering-quality conversion. License notices for Capacitor and lamejs are in `www/vendor/`.


## Mobile layout correction (0.6.1)
Removed the desktop multi-panel workspace and desktop breakpoints. Phone viewport checks passed at 360×640, 390×844, 412×915 and 844×390, without horizontal overflow or a bottom navigation bar extending below the screen. The browser preview at 1366×900 remains a centered phone-width app. All 17 current visual effects have presets; this update does not remove any effects or presets.


## All-effects rendering update (0.7.0)
The mobile-only workspace remains. Effect panels show four main controls, with extra settings under **More controls**. All 17 effect types retain presets, keyframes and their existing project identifiers; there are now 55 built-in Flowr presets. Additional parameters in older projects use defaults.

| Effect | Rendering and control improvements |
| --- | --- |
| Glow | Linear-light threshold extraction, soft knee, wide halo, additive/screen blending and optional hue tint |
| Posterize | Tonal gamma, ordered dither and RGB/luminance quantization |
| Find Edges | Pre-smoothing and noise threshold before edge detection; monochrome or source-color output |
| Duotone | Tonal balance plus independent black/white levels |
| Emboss | Surface-normal lighting, light elevation and source-color relief |
| Directional Blur | Floating-point premultiplied sample averaging, weighted falloff, shutter bias and repeat-edge option |
| Wave Warp | Shared alpha-safe subpixel sampling; transparent/repeat/mirror/wrap boundaries, adjustable pin falloff and square-wave antialias width |
| Turbulent Displace | Multiple fractal-noise octaves, detail strength, axis selection and boundary modes |
| Film Grain | Gaussian grain distribution, optional color noise and luminance-dependent grain response |
| RGB Split | Fractional channel offsets, alpha-safe bilinear sampling and boundary modes |
| Digital Glitch | Seeded slices plus optional color jitter and scanlines |
| Camera Shake | Smoother quintic/fractal noise, adjustable fine vibration and high-quality transformed drawing |
| Vignette | Smooth falloff, shape roundness and highlight protection |
| Mosaic | Block aspect ratio and alpha-aware original/effect mixing |
| Zoom | Adjustable center and optional floating-point zoom-blur averaging |
| Flash | Linear-light exposure boost, warmth and shadow lift |
| Color Grade | Exposure, gamma, temperature and lookup-table grading alongside contrast, saturation, monochrome and invert |

New presets include Dream Bloom, Purple Neon, Water Surface, Horizontal Heat, Soft Shutter, Trailing Streak, Warm Film, Cool Contrast, Color Film, Wide Digital, Warm Flash, Soft Zoom Burst and Dithered Print. These are original Flowr implementations and settings.

Validation checks all 17 renders for repeatability and transparent input. Further checks cover neutral settings, exposure/monochrome behavior, constant-field blur, half-opacity blur/zoom, opaque distortion boundaries, all 55 rendered presets and all 17 mobile control panels. Nine waveform, pinning and preset-undo tests still pass. A generated effects contact sheet is included in `docs/effects-preview.png`.

Floating-point blur and multi-octave distortion favor rendering quality over processing cost. Many samples or stacked high-resolution effects can be slow on phones; reduce Samples/Detail octaves for lighter previews. Zoom blur is off by default and can be enabled when needed. Physical Android performance still requires testing.

Mobile effect previews automatically use the displayed canvas size (capped at 720 pixels on the long side). Pixel-based controls are scaled for preview, while saved values and exports stay at full composition resolution. Fine grain/dither and subpixel detail may differ in the reduced preview; exports render the full effect. Browser regression checks verify that preview scaling leaves project dimensions/parameters intact and that export bypasses it.


## Preview-first mobile workflow (0.8.0)
Reworked the layout around the supplied workflow example, without copying its artwork or adding a desktop workspace.
- Larger preview: at a 390×844 viewport, the preview area is approximately 273 px high while editing, compared with about 160 px in the preceding layout. Original composition aspect ratios are preserved.
- Compact timeline remains visible below the preview transport while using Effects, Text, Transform, Audio or Graph. It shows the selected clip and tappable keyframe diamonds. The full Timeline view retains all layers, layer search, zoom and organizing controls.
- Active tool selection and layer name share one compact header. Extra editing commands are in the transport's scissors menu; preview Fit/FX bypass are in its three-dot menu. Undo/redo and stepping remain directly available.
- Effect presets and the main sliders take priority. Layer appearance/Add effect are compact expandable controls. Reorder, Reset and Beat actions are under each effect's **Effect actions** section.
- Touchscreen landscape keeps the preview and compact timeline on the left with the active tool on the right. Desktop browser previews remain phone-width.
- Layout/interaction checks passed at 360×640, 390×844, 412×915 and 844×390: preview, timeline and tool remain visible during editing, navigation stays within the viewport, menus work and tapping a keyframe seeks to its time. All 17 effect panels and waveform/preset/keyframe checks still pass. Physical Android testing remains outstanding.


## Direct tool tabs (0.8.1)
Replaced the native Transform/Effects/Graph tool dropdown with visible touch tabs. Tabs adapt to the selected layer, highlight the active tool, and support arrow-key navigation. The large preview, compact timeline and effect controls remain in place. Mobile layout, tab switching, keyframe navigation and selected-state checks passed.


## Blending on all rendered layers (0.9.0)
**Blending mode** is now a visible control at the top of both Transform and Effects. It is available for text, images, video, shape layers, vector/freehand drawings, solids and adjustment layers. Adjustment layers previously excluded the selector and ignored blend mode while rendering; both paths are fixed. Groups and nulls are non-rendering organizers/controllers, and audio layers have no visual blend mode.

The searchable mobile picker has 38 choices, grouped into Normal, Darken, Lighten, Contrast, Difference, Color, Matte and Utility. Selecting a mode previews it immediately and records an undo step. The selected mode is saved with the layer and used in preview and export.

| Category | Choices |
| --- | --- |
| Normal | Normal, Dissolve, Dancing Dissolve |
| Darken | Darken, Multiply, Color Burn, Classic Color Burn, Linear Burn, Darker Color |
| Lighten | Add, Lighten, Screen, Color Dodge, Classic Color Dodge, Linear Dodge, Lighter Color |
| Contrast | Overlay, Soft Light, Hard Light, Linear Light, Vivid Light, Pin Light, Hard Mix |
| Difference | Difference, Classic Difference, Exclusion, Subtract, Divide |
| Color | Hue, Saturation, Color, Luminosity |
| Matte | Stencil Alpha, Stencil Luma, Silhouette Alpha, Silhouette Luma |
| Utility | Alpha Add, Luminescent Premul |

The list and behavior categories were checked against [Adobe's blending-mode documentation](https://helpx.adobe.com/after-effects/desktop/work-with-layers/work-with-layer-blending-modes/blending-modes-layer-styles.html). These are Flowr implementations, not a claim of pixel-identical Adobe compositing. **Classic Color Burn, Classic Color Dodge and Classic Difference use modern 8-bit equations and are explicitly labeled Approx.** Luminescent Premul is also labeled Approx.; Canvas loses the out-of-alpha/HDR information needed for an exact implementation. Alpha Add uses additive alpha with alpha-weighted source/backdrop color. Adobe color-management, legacy compatibility, HDR and 16/32-bit results are not replicated.

Standard modes use native Canvas blending when available; other modes have a CPU fallback with premultiplied-alpha compositing. Static Dissolve uses deterministic spatial noise; Dancing Dissolve changes with composition frames. Stencil/silhouette modes mask the complete visible stack underneath, not just one layer.

Adjustment-layer Normal blends the processed result with the original stack using opacity, preserving partial transparency rather than doubling its alpha. Other color modes blend the adjusted colors with the underlying colors. Basic blur/hue/brightness on adjustment layers now work even without an added effect.

The composition buffer is cleared to transparent instead of baking in a black layer. Preview still appears over black, PNG can preserve genuine transparency, and video frames are explicitly filled with black before compositing so matte holes cannot leave previous-frame trails.

Checks cover all 38 mode equations/render paths, comparison to native Canvas for standard modes, adjustment blending/basic filters/opacity/transparency, nine rendered layer variants, picker search and undo. Existing mobile layout, effects and export regression checks are retained. Physical Android rendering/performance still requires testing; CPU modes can be slower than native modes on large compositions.


## Visual effect drawer (0.10.0)
The old Add effect dropdown has been replaced by a visual library inspired by the supplied layout example, styled to match Flowr's dark/purple mobile workspace.

- Open **Effects → Add effect**, tap the slim handle on the preview's right side, or swipe left from that handle. The drawer slides in from the right. Close with its X, the backdrop, Escape, or a rightward swipe inside the drawer.
- Browse 17 visual effects as thumbnail cards, grouped into Color, Stylize, Distort, Blur and Motion. Search matches names, useful synonyms, settings labels and preset names; category filters combine with search. Empty results offer Clear filters.
- Tap a card to add the effect to the selected unlocked layer. The drawer closes and the new effect's controls come into view. Undo restores the prior stack; existing effects are retained and their cards collapse for focus.
- Audio uses the same library presentation for its 13 existing effects. Video layers can switch between Visual and Audio; audio-only layers open the audio library directly.
- All visual thumbnail examples are bundled WebP assets (under 200 KB total), rendered from the actual effect algorithms using original sample artwork. Browsing performs no effect renders or remote image requests. The cards show illustrative examples; adding an effect uses its default parameters. Audio cards use schematic waveform artwork, not measured audio previews.
- The large preview, compact timeline, direct tool tabs, 38 blend-mode choices and 55 visual presets are retained. The edge gesture handle stays within the preview so it does not intercept timeline or inspector controls.

Browser checks cover all 17 decoded thumbnail assets, multiword search, category filtering, empty/reset states, add/undo, drawer opening/closing, emulated swipe events, focus/Escape behavior, all 13 audio cards, and the existing editing workflow. Android system-edge gestures and physical-device behavior still need testing; the Add effect button provides the primary entry point.

Regenerate bundled previews after changing effect algorithms with `node scripts/build-effect-previews.cjs` (requires the optional `@napi-rs/canvas` test/build dependency). No image-generation or rendering dependency is needed on the installed app.

## 0.11 — Generate effects

Adds all 26 requested names in a searchable **Generate** category (43 visual effects total), with bundled rendered cards, numeric/keyframable controls, opacity and composite choices, plus Soft/Bold opacity presets. New source: `www/assets/generate.js`.

- Gradients: 4-Color Gradient (corner hues/blend), Gradient Ramp (linear/radial, endpoints).
- Shapes/patterns: Checkerboard, Grid, Circle, Ellipse, animated Cell Pattern, Mandelbrot Fractal.
- Color: Fill, Eyedropper Fill (position/averaging radius), contiguous RGBA Paint Bucket.
- Light: Advanced Lightning (seeded jitter/branches), Beam, Lens Flare, CC Light Sweep, CC Light Rays, CC Light Burst 2.5.
- Drawing: Radio Waves, Write-on (curved path/progress), Stroke, Vegas, Scribble, CC Threads, CC Glue Gun.
- Audio Spectrum performs windowed frequency measurements on imported PCM; Audio Waveform reads the same PCM. Both use the audible audio-layer mix at composition time, including timing, trim offsets, volume, mute/solo and visibility. Video sound must first be extracted. Audio DSP effects are not included in the visual analysis; no decoded audio means silence. Live preview and export use the same time-based samples.

These are original Flowr implementations, **not Adobe or Cycore plugins or exact replicas**. CC Light Rays/Burst use procedural rays, not source-pixel volumetric scattering. Glue Gun, Threads, Scribble, Stroke and Vegas use source alpha boundaries (opaque footage uses its rectangular boundary), not masks, brush dynamics or Adobe path tracing. Write-on uses an adjustable curved path rather than recorded brush strokes. Gradient colors use hue controls rather than Adobe color pickers. New controls explain relevant limitations. Position/size controls are percentages so reduced preview and full-resolution exports stay consistent. No plugin binaries are bundled.

Validation: `node tests/check-generate.cjs` checks all 26 deterministic renders, opacity bypass, presets, source boundaries and measured PCM response. Regenerate preview assets with `node scripts/build-effect-previews.cjs`. High-resolution generators, fractals and flood fills can be expensive on mobile; physical-device performance and APK build need testing.

## 0.12 — Direct effect and layer controls

- Generate effects with color parameters now offer RGB color swatches, color keyframe buttons and **Eyedropper**. Tap Eyedropper and then a visible point in the composition preview to sample its rendered color. Transparent samples leave the color unchanged. Sampling excludes editing guides. Hue sliders remain available; moving a hue slider returns that color to hue mode. Sampled RGB channels and mode persist in projects and exports.
- Effect point tools cover Gradient Ramp endpoints, four movable 4-Color Gradient points, lightning/beam endpoints, generator centers, Paint Bucket and Eyedropper Fill sample points, and drawing endpoints. Tap a point tool, then tap the preview or drag its labeled handles. Done/Escape exits. Coordinates are percentages of the composition (not layer-local or Adobe mask coordinates), and use existing keyframe tracks when animated. Four-color gradients now use weighted movable points; the advanced bilinear mode preserves a fixed corner gradient option. Point tools select movable interpolation automatically.
- Scribble, Stroke, Vegas, CC Threads and CC Glue Gun now offer a curved-path mode with editable Start/End positions, bend, Start/End trimming, width and spacing. Their point tools switch to the editable path. Source-alpha-edge mode remains available. These remain original Flowr variants, not Adobe mask-based drawing engines.
- Generate color and parameter edits integrate with Undo. Parameter sliders also have precise number fields. The existing preset reset clears color/point animation as before.
- Transform adds 50/100/200% shortcuts, ±10 percentage-point changes, Fit/Fill, Center, Flip H/V, a draggable preview Scale handle and Reset transform. Size supports up to 800%. **Independent scale & anchor** offers keyframable width/height multipliers and local-pixel anchor offsets. These render in preview and export and persist through project save/restore. New controls apply to rendered visual layers; audio, adjustments, groups and null controllers retain their existing relevant controls. Size remains the uniform multiplier; independent width/height multiply it.
- Fit/Fill supports image/video, solid, rectangle, ellipse and estimated text bounds. Parent-controlled layers and custom paths/shapes use manual scaling. Fit uses the rotated bounding box; Fill uses that bounding box ratio and can leave corner gaps on rotated artwork. Text fit is estimated, not precise glyph bounds. Anchor changes can move artwork. Existing parent inheritance continues to use its original uniform scale/rotation behavior; the new independent-axis and anchor controls affect the selected artwork only.

Checks: `check-generate.cjs` covers actual RGB output, movable gradient output and path trimming as well as all generator renders. `check-advanced-controls.cjs` covers preview sampling, point placement/drag/reopening, numeric width, flips, scale handle and one-step Undo. Existing mobile workflow and effect-browser checks pass. Physical-device gestures, performance and APK build remain unverified. New UI: `www/assets/advanced-controls.js`.

## 0.13 — Motion blur and precomposition

**Motion blur:** Select a rendered visual layer and open Transform → Motion blur. Composition/Project → Composition motion blur controls the global enable switch, shutter angle (0–720°), phase (−720–720°), export samples (2–16), preview samples (2–4) and preview-only bypass. Defaults: 180° angle, −90° phase, eight export samples and three preview samples. Per-layer switches default off. Accumulation uses premultiplied floating-point RGBA so transparent motion trails blend correctly. Preview samples are rendered at up to 720px long side; exports sample at composition resolution. Settings persist and participate in Undo.

This is **transform motion blur**, covering keyframed position, rotation, scale, independent axes, anchors and parent transforms. It does not calculate optical flow between video frames, replace Adobe's renderer or blur individual effect parameter animation on ordinary layers. Video holds its decoded current frame during shutter samples. Effects on an ordinary layer run after the transform samples. Blur on a precomp samples its rendered nested scene and can be expensive. High-resolution exports with many blurred layers need significant memory/time; actual mobile performance is unverified.

**Precompose:** Tap Precompose above the inspector, choose consecutive unlocked layers and give the new composition a name. Images, video, text, shapes, adjustment layers, audio and existing precomps can be included. Select a precomp and tap Open precomp in Transform to edit its source layers. The composition bar shows the current name; Parent returns one level. Added layers go into the open composition. Existing frame stepping, keyframes, effects, blending and layer tools work inside it.

- Precomps render into isolated transparent surfaces and can themselves be transformed, blended, animated and given effects or motion blur. They are real nested render containers, separate from organizer groups.
- Source layers and assets remain editable and persist in saved projects and portable backups. Duplicating or splitting a precomp copies its child hierarchy, IDs and media references so editing a copy does not change the original. Duplicated videos receive separate playback elements; duplicated media assets are registered for saving.
- Nested video/PCM audio retains source timing and trim offsets. Moving or trimming the containing layer remaps playback and clips the audio to its bounds. Transform also offers Mute nested audio and a nested audio volume multiplier. Audio effects remain on source audio layers. No time stretch/remapping is added in this version.
- Compositions share the project's dimensions, frame rate and duration. Source timing uses composition seconds; changing the project settings affects all precomps. This is not an independent composition bin or Adobe's full composition model. Supports up to seven nested creation levels; rendering has a depth guard.
- Choose consecutive layers to avoid changing unrelated stacking order. Include whole organizer groups and both sides of parent links. Adjustment and backdrop-dependent blend modes can change appearance when isolated into a precomp, as they now see only its internal stack. Parent selection lists are restricted to the open composition.
- Deleting a precomp deletes its contained layers from the current state; Undo restores them. Organizer groups retain their separate organizing behavior.

**Timing tools:** Transform → Timing tools adds Trim in/out to playhead and Go to in/out. Trimming precomps and media preserves source offsets. Frame stepping, Split, Fade and Graph easing tools remain available from the existing workflow.

Checks: `check-compositions.cjs` covers opaque/transparent render preservation, source isolation, navigation, transformed output, saved state, deep duplication, nested audio timing, sampled motion blur, alpha and global bypass. `check-compositions-ui.cjs` covers the mobile dialog, selection, navigation, motion settings and source-preserving trim with Undo. Existing workflow, blend, effect library, eyedropper/point/scale and actual browser MP3/WAV export tests pass, including nested audio with a non-silent PCM check and the worker/native-share fallback scenarios. Native share behavior uses mocks; an Android APK and physical-device performance/gestures have not been tested. These features are original Flowr implementations, not a claim of 100% After Effects compatibility.

## 0.14 — Rebuilt Generate controls and rendering

This revision replaces the earlier generic models for **11 effects**: 4-Color Gradient, Gradient Ramp, Fill, Eyedropper Fill, Beam, Advanced Lightning, Scribble, Stroke, CC Light Rays, CC Light Burst 2.5 and CC Light Sweep. The other 15 Generate effects remain explicitly labeled **Simplified model** in their controls. Expanded controls are not a claim of Adobe/Cycore equivalence.

Implemented controls were checked against Adobe's public [Generate effects descriptions](https://helpx.adobe.com/after-effects/using/generate-effects.html). Controls are exposed only where Flowr implements an associated behavior; unsupported mask modes and Adobe-specific capabilities are described rather than presented as working options. Numeric ranges and rendering algorithms are Flowr's own, and there is no `.aep` or `.ffx` compatibility.

- **4-Color Gradient:** four animated color/position pairs; Blend now makes transitions more gradual as it increases; Jitter; opacity; Normal/Add/Screen/Multiply/Overlay/Soft Light compositing; specific color presets.
- **Gradient Ramp:** Start/End of Ramp positions and colors, Linear/Radial Ramp, Ramp Scatter and Blend With Original. Scatter actually dithers the rendered gradient. Source/result blending interpolates premultiplied pixels.
- **Fill:** actual RGB color, an effect polygon or source alpha, Invert Mask, opacity and Blend With Original. Filling a translucent source preserves its alpha instead of stacking another copy over it.
- **Eyedropper Fill:** Sample Point, pixel radius, Skip Empty/All/All Premultiplied/Including Alpha sample modes, Maintain Original Alpha and Blend With Original.
- **Beam:** Starting/Ending Points, Starting/Ending Thickness, Length, Time, Softness, Inside/Outside Colors, 3D Perspective toggle and Composite On Original. Time moves a finite beam segment along the path; thickness really tapers along that segment. Perspective here is thickness interpolation along a 2D beam, not a 3D scene renderer. Ordinary layer transform blur does not sample Beam's animated Time parameter.
- **Advanced Lightning:** Lightning Type, Origin, contextual Direction/Outer Radius, Conductivity State, Core Radius/Opacity/Color, Glow Radius/Opacity/Color, Turbulence, Forking, Decay/Decay Main Core, Complexity, Min. Forkdistance, Alpha Obstacle and Composite On Original. It no longer self-animates: animate Conductivity State. Glow can be disabled independently. Vertical and radial types hide inapplicable direction/radius controls. Branching and alpha avoidance are Flowr heuristics, not Adobe's path/atmospheric solver. Unimplemented Adobe controls such as Core Drain and Termination Threshold are not exposed as fake settings.
- **Scribble:** fills a closed polygon or source alpha with connected angled zigzag strokes. Controls cover Fill Type, Stroke Width, Edge Width, Start/End, Angle, Curviness/Variation, Spacing/Variation, Path Overlap/Variation, Wiggle Type and frequency, End Cap, Join/Miter Limit, and composite/reveal styles. Core, edge, stroke and animation options are grouped. Static stays still; Jumpy and Smooth use deterministic time-dependent noise. Edge bands are raster approximations. Left Edge approximates the inside band; separate Right Edge, multiple-mask modes and Mask Path trimming remain unsupported.
- **Stroke:** ordered closed contours or an effect polygon, Brush Size/Hardness, brush spacing, arc-length Start/End, and On Original/On Transparent/Reveal Original paint styles. This replaces unordered edge dots, so trimming follows a continuous boundary. Raster alpha contours are traced at up to 512px long side, then mapped to render resolution; very fine details can differ from exact vector paths.
- **Light Rays / Light Burst:** thresholded source highlights are accumulated along a radial scale path, replacing decorative procedural spokes. Center, intensity, length, threshold, falloff, sample count, colorization and source blending work on the image. Floating-point premultiplied accumulation avoids low-alpha Canvas-addition quantization. These two effects share a Flowr radial-light model, not Cycore's individual engines or exact control sets.
- **Light Sweep:** pixel width, direction, sweep intensity, edge intensity/thickness, shape, color and source blending. Highlight strength follows source pixels/alpha and source-luminance edges.

**Mask editor:** Fill, Scribble and Stroke have Draw effect mask, Edit mask and Clear mask. Tap preview vertices; drag existing vertices; use Rectangle as a starting shape; Done closes and applies the polygon. Cancel/Escape discards draft changes. Applied masks persist, duplicate with their effect, and support Undo. This is one non-keyframed closed polygon per effect in composition coordinates, not AE's layer-local Bezier/multiple-mask system. It does not clip the entire layer before every effect. Scribble and Stroke can switch back to source alpha. Invert applies to Fill's effect mask.

**Control presentation:** Effect point coordinates display and accept composition pixels while storing normalized positions for preview scaling. Points can be outside the frame through numeric fields. Colors use named swatches/eyedroppers; old hue-channel sliders and generic Speed/Size fields are hidden on rebuilt effects. Binary switches are checkboxes. Lightning and Scribble use named control groups and conditional fields. Existing saved Generate values remain readable, but these new algorithms and defaults can change an existing Generate effect's appearance. Previously stored custom Scribble curve endpoints are not automatically converted to a closed polygon.

Validation: `check-generate-pro.cjs` covers all 11 engines/presets, gradient controls, manual lightning evolution, beam travel, ordered Stroke trimming, Scribble interior fill/wiggle, source-derived rays and alpha-safe Fill. `check-generate-pro-ui.cjs` covers drawing/editing/saving/Undo for polygons, pixel coordinate input, contextual lightning fields, real checkbox updates and RGB swatch editing. Existing Generate panel, effect-browser, point/eyedropper/scale and precomposition checks pass. Regenerated preview cards use the new algorithms. Mobile hardware performance, APK build and Adobe pixel-by-pixel equivalence remain unverified.

## 0.15 — Inline editing toolbar

Split, Delete, Trim start/end, Cut out, Copy, Fade in/out and Start/End now sit in an always-visible horizontal tool strip directly beneath playback. Split and Delete are first; swipe sideways for the remaining actions. The old floating editing menu and scissors launcher are removed from the visible UI. Tools retain their existing selection/lock/trim availability rules. Preview options remain in the playback three-dot button.

The strip participates in layout, above the timeline, including the mobile landscape grid, and does not overlay clips or controls. Tool operations commit distinct Undo states even when used in rapid succession. `check-toolbar.cjs` verifies phone/landscape placement, viewport overflow, ordering, Split/Delete behavior and Undo; the existing workspace check passes.

## 0.16 — Flowr 3D Scene Setup

A working, offline WebGL 2 scene-layer system, inspired by the scene-assembly workflow of Element 3D. This is an original integration using the bundled Three.js r180 renderer, **not the Element 3D plugin, an `.e3d` reader, or a claim of visual/feature parity**.

### Start a scene

1. Tap **Add → 3D Scene**. A beveled chrome title opens in Scene Setup.
2. In **Objects**, add text, boxes, spheres, toruses, cylinders or planes; import a local model; select, show/hide, duplicate or delete objects. Text has extrusion and bevel controls, three bundled font choices and custom TTF import.
3. Use **Transform** or the viewport's Move/Rotate/Scale gizmos. The target selector switches between the selected object and Groups 1–5. Assign objects to groups in Object setup. World/local gizmo space and optional snapping are available. Orbit mode supports drag to rotate and two-finger dolly/pan. Frame focuses the current target.
4. Use **Material** for surfaces and **Scene** for camera and lighting. Apply returns to the timeline. Reopen from **Transform → Open 3D Scene Setup**. Cancel restores the state from opening the editor, including the Undo history. Editor camera orbit changes the scene's render camera.

### Implemented rendering and editing

- Real geometry depth, occlusion and perspective; beveled/extruded text; independent object transforms and five group transforms. Text caps and sides are separate material slots. Layer position/scale/rotation/opacity remain the normal outer 2D layer transform.
- PBR metalness/roughness materials with studio-environment reflections, clearcoat, transmission, IOR, thickness, emissive color/strength and opacity. Chrome, Gold, Rose Gold, Plastic, Matte, Glass and Neon presets. Imported source materials stay active until overridden. Material slots support independent colors, maps and numeric surface controls; All slots applies edits across slot overrides. Wireframe and double-sided options are available. Imported transparency and side settings are preserved when override is off.
- Base-color, normal, roughness and metalness texture maps, loaded locally. Three.js uses conventional UV maps and its standard PBR channel conventions (roughness green, metalness blue). Roughness/normal/metalness data maps are linear; color maps use sRGB. This version does not include UV unwrapping, displacement sculpting or procedural texture authoring.
- Three controllable directional lights with individual colors, strengths and positions, hemisphere ambient light, exposure, a procedural room environment and reflection intensity. Optional ground plane, ground color/height and a key-light shadow map. The editor grid/gizmos are omitted from composition output. Custom HDR/EXR environments are available in 0.17; screen-space AO and ray-traced shadows/reflections are not implemented.
- Per-object repeaters: line, grid and radial distributions, spacing/radius and copies. Up to 48 copies for primitives/text, 12 for an imported model. Copies share geometry/materials within one repeated object; use Duplicate object for separately editable copies. Total limits protect mobile rendering (40 editable objects, 200 repeated instances and an estimated one-million-triangle scene budget).
- Camera orbit yaw/pitch/distance, field of view and target coordinates are keyframable. Object/group translation, rotation and independent scale, plus material numeric values, exposure and light strengths use existing Flowr keyframes. ◆ adds/removes a key at the playhead; timeline seeking and playback evaluate the same data. Animate rotation adds a turn over up to three seconds. The main Graph panel exposes named 3D tracks and existing easing tools. Text content, topology, color swatches, repeater count and light positions are static setup properties in this version.
- Imported glTF animation clips are selectable with a speed multiplier and evaluate deterministically from layer source time. The loader/mixer supports ordinary glTF node/morph/skinned animation; simultaneous mixing of multiple clips, rig authoring and physics are not exposed.
- A scene renders as a transparent composition-sized layer. It can be combined with video, masks/effects, blend modes, adjustment layers and precomps. Applying existing Glow to the 3D layer can add a bloom-like glow around emissive objects. Separate scene layers have separate depth buffers/cameras; they do not share a global 3D world. Put intersecting geometry in the same scene.

### Models, fonts and saved assets

- **GLB:** self-contained uncompressed glTF 2.0 binary models.
- **glTF:** select the `.gltf` and its local `.bin`/PNG/JPG/WebP resources together. Resources are resolved from selected files by path/name; remote URLs are not fetched. Missing resources produce an error before the object is added.
- **OBJ:** geometry and material names; select a local `.mtl` and its texture images alongside the OBJ to assign matching material slots. Use the material editor to refine surfaces/textures. glTF is preferable for original PBR materials and animation.
- Draco, Meshopt and KTX2/Basis-compressed resources require decoder integrations that are not bundled; export an uncompressed GLB first. Proprietary Element 3D/Cinema 4D models and scene files are not supported.
- Each model selection supports up to 256 MB including resource files; separate texture/resource selections are limited to 32 MB. A model is limited to 300,000 triangles and an imported texture to 4096 × 4096 pixels. Custom TTF/OTF files are limited to 16 MB. Fonts are converted to geometry locally. Variable/collection fonts and complex text shaping/RTL layout have not been implemented; characters missing from the chosen font are reported rather than silently replaced.
- Models are centered and normalized to a two-unit bounding box on import. Original internal hierarchy, UVs and materials remain available to the renderer; Scene Setup edits the imported asset as one object, with material slots and an animation clip, rather than offering a full mesh/node/rig editor.
- Binary model resources, images and fonts live in IndexedDB, referenced by compact IDs in project state. They are not stuffed into localStorage history. Saving/reopening restores them. Project backups embed the **used** 3D source resources alongside regular media, within the existing 100 MB source cap. Import remaps asset IDs to avoid overwriting another project's resources. Duplicated scene layers own independent setup data while reusing immutable asset resources.
- Removing an object stops unused source resources from blocking rendering/export or being embedded in a backup. Local unused binary records are retained for Undo and other saved projects; a storage garbage collector is not included.

### Preview and export

Preview has Draft 480, Balanced 720 and High 1080 long-side limits. Export renders the scene at composition resolution, subject to GPU size/memory limits, before the normal video/frame encoder. PNG transparency is retained; video uses the existing black output background. Existing audio routing continues to work. GPU context loss aborts a video export instead of silently delivering a blank scene. Real-time browser video export can drop frames when the device cannot render fast enough; this is not an offline frame-sequence encoder.

WebGL 2 is required for 3D; ordinary 2D editing does not initialize a GPU scene. Glass uses the renderer's screen-space transmission and can refract objects inside this 3D scene, not the 2D video layers underneath. High-resolution rendering, shadows, transmission and repeated high-poly models can be expensive. Physical Android behavior/performance and an APK build have **not** been verified.

### Source, licenses and validation

- Engine source: `src/scene3d-engine.js`; offline bundle: `www/vendor/3d/engine.js`.
- Editor/storage integration: `www/assets/scene3d.js` and `scene3d.css`.
- Rebuild the committed engine bundle with `npm install` then `node scripts/build-scene3d.cjs`. Three.js and esbuild are pinned development dependencies. No CDN or account/API is used at runtime.
- Third-party renderer, font and font-parser licenses are in `www/vendor/3d/`. The font parser is opentype.js as vendored by Three.js. User-imported files remain project source assets.
- Documentation used: [Three.js WebGLRenderer](https://threejs.org/docs/pages/WebGLRenderer.html), [TransformControls](https://threejs.org/docs/pages/TransformControls.html), [GLTFLoader](https://threejs.org/docs/pages/GLTFLoader.html).

Browser checks use Chromium with software WebGL 2, **not mocked 3D output**:

- `check-scene3d.cjs`: actual extruded geometry, reflective materials, frame-dependent animation, repeaters, 2D composition and independent scene-layer duplication.
- `check-scene3d-ui.cjs`: mobile portrait/landscape tabs, per-slot editing, keyframes, duplication, Cancel and Apply.
- `check-scene3d-gizmos.cjs`: pointer-driven object/group translation and orbit camera control.
- `check-scene3d-assets.cjs`: binary GLB, glTF plus BIN, OBJ, glTF animation, original alpha/sides, missing-resource rejection, texture/TTF import, IndexedDB, portable ID remapping and model/font restoration after reopening.
- `check-scene3d-export.cjs`: real PNG output and playable recorded video with decoded frames that differ during 3D rotation. The test browser selected MP4.
- Existing mobile workspace, inline toolbar, blend-mode, effect-library and MP3/WAV worker/native-share-fallback checks pass. Native share checks use mocks; these checks do not establish physical phone compatibility or Adobe renderer parity.


## 0.17 — Expanded mobile 3D Scene Setup

Open **Add → 3D Scene**. Model Import is in **Objects**; material imports are in **Material**; environments are in **Scene**; motion controls are in **Animation**.

- Seven text bevel presets, from Flat and Chamfer to Double Rim and Triple Stack. Adjust bevel thickness, size, offset, segments, layer growth and spacing. Layered bevels expose separate material slots.
- Seventeen PBR material presets including Copper, Brass, Brushed Steel, Ceramic and Frosted Glass. Import JSON materials, MTL with local texture maps, or portable `.flowrmat` packages. Export a material with embedded maps for reuse. These formats do not import proprietary Element 3D material packs.
- Six environment looks and local HDR, EXR or PNG/JPG/WebP equirectangular panoramas. Use 2:1 images up to 4096 × 2048. Environment rotation, intensity and optional visible background are adjustable. Assets persist with saved projects and portable backups.
- A/B object poses, transition keyframes, easing, object and instance stagger/order, seeded scatter/noise, per-axis spin, separated text letters, mesh-part dispersion and ten auxiliary transform channels per group. Playback supports manual, once, loop and ping-pong. Optional twist, bend and animated noise deform geometry. These are Flowr controls, not complete Element 3D parity.
- GLB, glTF plus local resources, and OBJ model imports remain available. Importing OBJ with MTL and image files assigns matching material slots. Existing compressed-model and geometry limits still apply.

Validation: `check-scene3d-expanded.cjs`, `check-scene3d-expanded-ui.cjs`, `check-scene3d-imports.cjs`, `check-scene3d-assets.cjs`, and `check-scene3d-export.cjs`. Run browser/GPU checks individually: concurrent software rendering can overload the recorder. The video test initially failed under concurrent test load, then passed in isolation. Video export remains real-time and can drop frames on overloaded devices. No physical Android test or APK build has been completed for this update.


### Playback toolbar placement update

Split, Delete and Copy now lead a horizontally swipeable toolbar within the same row as Play/Pause. Trim, fades, layer boundaries, Undo/Redo and frame stepping remain accessible by swiping. The separate editing-action row is removed. Phone portrait/landscape layout, Split/Delete actions and Undo checks passed.


## 0.18 — Letter animation, skew and the supplied environment library

- **Animate → Animate position, rotation, scale & skew** exposes keyframeable XYZ position, XYZ rotation, XYZ scale and X/Y skew for the selected Object, Group or Aux target. Transform contains the same channels. Skew preserves a full shear matrix and participates in A/B poses.
- **Separate text letters** now preserves the original layout and centers each glyph's pivot. The new **Letter / mesh start transforms** controls provide individual-axis starting position, rotation and scale plus X/Y skew. With **Drive parts with progress**, these start transforms blend back to assembled letters at Transition 100%; Part stagger and Order set the sequence. Existing dispersion controls still spread outward with increasing progress.
- **Rise In, Spin In, Scale In and Skew In** enable separation for text, set useful start transforms and create a Transition animation from the playhead over the configured duration (clamped to the composition end). Reset Letters restores neutral letter settings and clears transition keys. Transition is shared by the scene, so these shortcuts replace its existing keys. To preserve an existing transition, edit the numeric controls instead.
- Twelve new native bevel looks: Knife Edge, Pillow, Coin Edge, Heavy Block, Wide Chamfer, Inset Rim, Floating Face, Broad Halo, Staircase, Art Deco, Soft Stack and Deep Outline. There are now 19 bevel presets, with complete preset values so switching styles does not inherit another style's geometry settings.
- Your supplied archive's 15 Basic 2K JPG panoramas and 24 V1 PNG environments are included unchanged under `www/assets/environments`. Select them from **Scene → Environment library → Imported**. Only the selected environment is loaded while editing; the existing renderer caches a limited number of filtered reflection maps. The ZIP and installed app are larger because these image assets are bundled for offline use. The archive's DDS alternatives are omitted because equivalent JPG files are supplied.
- Proprietary `.ebf` bevel and `.emf` material preset files were inspected but are not imported or claimed to be reproduced. The new bevels are native Flowr styles. Existing JSON/MTL/flowrmat material importing is unchanged.

Validation: `check-scene3d-letters.cjs` verifies unchanged separated layout, centered pivots, staggered transforms, exact reassembly, invertible two-axis skew, A/B shear and finite geometry across all 19 bevels. `check-scene3d-letter-ui.cjs` verifies mobile controls, keyed presets, repeatable renders, all 39 environment image loads and saved state. Existing expanded engine/UI checks and the PNG/video export check also run; the export check now exercises separated letters, scale, rotation, skew and a supplied environment. Physical Android testing remains outstanding.


## 0.18.1 — Even text dispersion

Separated text now disperses horizontally by letter order. Each adjacent pair gains the same extra gap per unit of **Letter spacing / dispersion**, replacing the center-outward radial push that created a large gap between central letters such as O and W. Each line remains centered; original font spacing and explicit word spaces are preserved. Start-transform animation, stagger and model mesh dispersion retain their existing behavior. Regression checks cover FLOWR, narrow/wide glyphs, word spaces and multiple lines at several dispersion values, plus the existing mobile letter-animation checks.


## 0.18.2 — Larger model imports

Model import now accepts up to 256 MB total for a model and its selected resource files, replacing the previous 32 MB selection cap. GLB, glTF plus resources and OBJ use the same limit, and cached-project reloads enforce it in the engine as well. The existing 300,000-triangle model limit and standalone texture/font constraints remain. The limit is a file-size allowance, not a guarantee that a given phone can decode or render every model within it.

Imported model files remain in app storage and can be saved/reopened normally. Portable JSON backups still support only 100 MB of total source files; a new preflight reports this before creating large base64 copies, so larger projects can still use local Save. Failed app-storage writes release the newly decoded asset and report an actionable storage error.

Validation: `check-scene3d-large-import.cjs` imports a valid GLB with a 40 MB buffer (one visible triangle, padded unused data to exercise file size), saves its source in IndexedDB, reopens and renders it. It also checks the exact 256 MB threshold, total resource size, rejection before reading oversized selections, decoded-asset cleanup on storage failure, and backup preflight before base64 conversion. This validates the larger-file path rather than high-poly phone performance. Existing asset import/reopen/remapping checks passed.


## 0.19 — Deep Glow

Open **Effects → Add Effect → Stylize → Deep Glow**, or search for `deep glow`.

This is an original Flowr implementation inspired by the visual workflow of [Plugin Everything's Deep Glow](https://www.plugineverything.com/deep-glow), not the commercial AE plugin. It builds a long-tail bloom from multiple floating-point blur scales, with optional linear-light processing before compositing back into Flowr's 8-bit canvas. It does not provide the commercial plugin's exact renderer, HDR input/output or GPU acceleration.

- **Glow:** radius up to 2,000 composition pixels, exposure multiplier, luminance threshold and soft threshold rolloff. Exposure changes brightness after thresholding. Zero exposure restores the original at full Original opacity.
- **Falloff & color:** core/halo strengths, falloff, saturation, inner/outer RGB tints with preview eyedroppers, and per-channel chromatic blur radii.
- **Anamorphic:** aspect ratio and angle stretch the bloom horizontally, vertically or diagonally. Angle is ignored for a circular aspect ratio.
- **Input & compositing:** luminance, brightest RGB or alpha silhouette sources; Add/Screen blending; Original opacity; Composite, Glow-only and Threshold-source views. Alpha silhouette can light a black silhouette. RGB is alpha-weighted so hidden transparent colors cannot create a halo.
- **Quality:** Draft, Balanced and High use bounded working buffers and different blur-scale counts. Deterministic RGB/alpha dithering reduces visible 8-bit bands. Preview radius scales with the composition preview, while exports use composition-pixel settings. CPU processing can be expensive; Draft helps while editing on phones. Soft thresholds and fixed dither patterns avoid introducing time-dependent random flicker.
- Seven presets: Soft Bloom, Neon Core, Deep Halo, Anamorphic, Dream Mist, Purple Cyan and Alpha Neon. Numeric controls support the existing keyframe system; Beat animation drives exposure. The effect has its own bundled rendered preview card and can be reordered, bypassed, reset or saved like the other visual effects.

`check-deep-glow.cjs` verifies zero exposure, transparent input, halo extension, source color, threshold rejection, exposure response, anisotropic direction, RGB tint, alpha-only emission, chromatic spread, extreme radii/quality and all presets. `check-deep-glow-ui.cjs` verifies effect search, preset controls, RGB pickers, keyframes, saved state, PNG export and mobile sizing. The existing effect-quality regression passes. Chromium software tests measured roughly 50–100 ms for a 320 × 180 glow pass; this is not a physical-phone performance measurement. Physical Android testing remains outstanding.


## 0.20 — Supplied font library

All 251 unique TTF/OTF files from `Fonts.rar` are included offline. The archive contains 252 files; `chinese rocks rg (1).otf` is an exact byte-for-byte duplicate of `chinese rocks rg.otf`, so one shared copy is installed. Distinct TTF/OTF versions and styles remain separate entries. This is a count of font files/styles, not distinct families.

- Regular text: **Text → Font → Browse fonts** (visual drawer in v0.21). The fonts display their embedded full names; entries with matching names include their original filename to distinguish them.
- 3D text: **3D Scene → Objects → Font**. Bundled font choices sit below the three original fonts. The existing custom font importer remains available.
- Fonts load locally on demand. Regular text uses browser FontFace loading; 3D text uses generated outlines from the selected font. Saved projects keep stable font IDs. Saving and visual export wait for the needed fonts to load instead of exporting a fallback font. Built-in fonts need no IndexedDB copy or extra embedding in portable backups; reopening requires a Flowr version containing the same pack.
- Two legacy fonts (Berate the Elementary and retro) needed their character-map tables regenerated for browser compatibility. Glyph outlines are preserved. Four fonts (Blackdeath Personal, Bristone, Hamston and Mentality) have separate 3D-only derivatives with unsupported layout tables removed; their original regular-text files are preserved. This 3D engine renders outlines and does not provide full OpenType shaping. Missing characters still depend on each font's character coverage.
- The font files and manifest are under `www/assets/fonts` and `www/assets/font-manifest.json`; lazy 2D loading is in `font-library.js`. Adding the complete font pack increases the APK/download size.

Validation: `check-font-pack.cjs` parses all 251 font files into 3D outline data. `check-font-pack-ui.cjs` loads all 251 in Chromium and checks both pickers, selection, save/restore, 3D preparation, PNG export and phone sizing. Physical Android testing remains outstanding.


## 0.21 — Visual text font library

**Text → Browse fonts** opens a right-side drawer styled like the effects library. **3D Scene → Objects → Browse fonts** opens the same library for extruded text.

- All 251 supplied font files have bundled preview cards rendered from their actual font outlines. Cards use Aa or ABC according to character coverage. Browsing does not load every font file; only applying a bundled font loads it. System and custom fonts remain available alongside the pack.
- Search font names, filenames and style categories. Categories group Sans, Serif, Script, Display, Gaming, Sports, Blackletter and Horror fonts using font metadata/name hints, rather than a manual typography audit. System/Custom groups appear when relevant.
- Tap the star to keep a font in Favorites. Recent contains the last 20 applied fonts. These preferences persist on the device; the active font remains part of project state.
- Tap a card to load/apply its font and return to the text controls, with Undo/Redo support. The same action updates the selected 3D text object's font. RGB, size, spacing, animation and other existing text controls remain in the text panel.
- The preview's right-edge shortcut opens fonts while the ordinary Text tab is active; it opens effects in the other editor tabs. Swipe the font drawer right, tap outside/Close, or press Escape to dismiss it. Closing a font drawer inside Scene Setup leaves Scene Setup open. Loading selections are canceled if the drawer is dismissed before they finish.
- The built-in font dropdowns are replaced by the Browse fonts buttons. Custom font import buttons remain. Long variant labels are shortened on the cards; accessible names and search retain the full font name/filename.

Preview generation: `scripts/build-font-previews.cjs` uses Chromium and Playwright to regenerate `www/assets/font-previews/*.webp`. They are committed assets and do not need to be generated on the phone.

Validation: `check-font-browser.cjs` checks all preview cards, browsing without font-file loads, filtering/search, Favorites/Recent, apply/Undo/Redo, context shortcut, swipe close, 3D apply/Escape and portrait/landscape sizing. Updated `check-font-pack-ui.cjs` verifies all font loading and save/restore/PNG export through the new drawer. The effects-library regression passes with all 44 visual and 13 audio effects. Physical Android testing remains outstanding.


## 0.22 — Character panel

The **Text → Character settings** button opens a right-side Character drawer following the AE-style control layout. The preview's right-edge shortcut now opens Character while the Text tab is active. Other tabs retain the effects shortcut. The panel can be closed with Close, Escape, an outside tap or a rightward swipe.

- Font family opens the existing visual font library above the Character drawer; applying a font returns to Character. All 251 previews, search, categories, Favorites and Recent remain available. A live sample in Character uses the selected font.
- Font Style selects actual installed faces in the current bundled family. Font metadata supplies family/style names; legacy named-weight families are normalized when a shared base family exists. System/custom fonts use the normal/faux style choices. Faux Bold and Faux Italic have independent buttons.
- Fill and stroke colors, individual fill/stroke switches, swap colors and preview eyedroppers are available. Stroke width is shown as the full rendered stroke width, with Fill Over Stroke / Stroke Over Fill ordering. Existing projects' outline thickness is preserved.
- Font size, automatic or explicit pixel leading, Metrics/None kerning, tracking in thousandths of an em, horizontal/vertical scale, baseline shift and Tsume change the rendered text. Old pixel tracking and relative leading remain supported. Tsume compresses glyph advances without scaling the glyph outlines.
- All Caps, Small Caps, Superscript and Subscript apply without changing the stored source text. Small Caps uses scaled uppercase glyphs with baseline alignment. Super/subscript scales and shifts the text run. These are whole-layer styling controls; mixed character selections and full OpenType small-caps/script positioning are not implemented.
- Ligatures uses browser shaping for untracked text; disabling it uses individual glyph rendering. Hindi Digits maps Western digits to Arabic-Indic digits (٠١٢). Full complex-script shaping is not preserved when the individual-character animation/tracking path is required. Optical kerning is not offered.
- Header Undo/Redo restores Character changes. Font selection and style changes persist with the project and render into exported frames. Text glow remains editable in a collapsed section. Reset Character preserves the chosen font, font size, text box, text glow and existing text animation settings.

Validation: `check-character-panel.cjs` verifies all fields, actual family/style selection, nested font-picker navigation, Undo/Redo values, eyedropper sampling, render changes for spacing/scales/baseline/caps/super-subscript/stroke order/digits, saved state, PNG export and portrait/landscape sizing. Font-pack loading/export and font/effect-browser regressions also pass. Physical Android testing remains outstanding.


## 0.23 — Dedicated Text drawer below FX

The entire non-animation text panel now lives in the **T** drawer, opened from a dedicated right-edge handle directly below **FX** on the preview. Tap or swipe left on T. FX always opens the effect library, including while the bottom Text tab is active. The T handle is shown for an unlocked text layer and hidden for other layer types.

The drawer contains source text editing, non-animated Neon/Outline/Subtitle/Clean styles, family/style selection, live font preview, the visual font browser, all Character settings, text glow and text-box settings. Font import is in the drawer footer. Text edits use the normal draft/history flow. Style presets preserve existing animation type, duration and animate-out settings.

The bottom **Text** tab now contains only Quick Animation and the existing text animators. No animation selector, duration or text-animator controls are added to the drawer. The old font, Character-launch, source-text and text-box rows have been removed from the bottom inspector.

Validation: `check-text-drawer.cjs` checks separate drawer access, T below FX within the preview at 320 × 640, 390 × 844 and 844 × 390, animation separation, text/style/box editing and save/reopen. Updated Character, font-pack, font-browser and effects-browser checks pass. Physical Android testing remains outstanding.


## 0.24 — Text animation properties on the timeline layer

The separate Text/text-animator inspector tab and its preset panel are removed. The T drawer still owns source text and appearance; animation controls are now part of the layer hierarchy.

1. Expand the text layer with its arrow. In the compact editor strip this switches to the full Timeline view.
2. Expand **Text** and choose **Animate…** to add Position, Tracking, Scale, Rotation, Opacity, Blur or Fill Color.
3. The resulting **Animator** group contains editable property values and an **Add property…** menu. Use the stopwatch/diamond to create keys on the property's own timeline row; the existing timeline diamonds and Graph editor remain available.
4. **Range Selector 1** exposes keyframeable Start, End and Offset. Its Advanced section controls Characters/Words/Lines, selector shape, smoothness and deterministic random order.

Properties can be removed individually, groups can be collapsed, enabled/disabled or removed, and edits support Undo/Redo. Position adds independent X/Y fields; Scale is the existing uniform per-character scale. Tracking is the existing pixel-based animator tracking. Fill Color has an editable RGB color and a keyframeable Fill Amount percentage. New Opacity properties start at 100% to keep the layer visible until edited. Range and property inputs use their defined limits.

Existing saved animator groups and their keyframes retain the same data and render behavior. Existing quick clip animations are preserved and can be edited/disabled in saved-clip rows under the text layer; they no longer have a separate inspector. Default text creates no new animation automatically. Text appearances, font previews and the separate FX/T drawers remain available.

Validation: `check-text-layer-animation.cjs` checks permanent inspector removal, layer expansion, neutral property creation, timeline keys and frame interpolation, property addition/removal, advanced selectors, range-offset motion, fill-color strength, Undo, saved state and mobile sizing. Character, T-drawer and effects-library regressions pass. Physical Android testing remains outstanding.


## 0.30 — Font loading and preset removal

Fixed the binary font fetch that failed under file://. `scripts/build-font-data.cjs` regenerates the 251 compressed, lazy-loaded font payloads; rebuild the 3D bundle with `node scripts/build-scene3d.cjs` after source changes. The existing TTF/OTF files remain for ordinary 2D text. Blood Text is retired; migration keeps editable wording and unrelated effects while removing its managed stack and helper maps. This is a source archive; rebuild the APK to install the update.

## 0.31 — Advanced Text preset library

Removed the requested font and its preview/3D payload. Added 64 native keyed text animations, animated preview assets and animation-type filtering. Generated groups support per-character/word/line temporal staggering, direction and periodic sampling while retaining editable property tracks. Reapplying a library animation replaces the preceding generated group and preserves manual groups. The application opens their timeline properties directly. No Blood Text preset is restored.

## 0.32 — Left preset drawer layout repair

Replaced a malformed preset stylesheet, made the launch handle a visible Presets tab anchored inside the preview, and aligned the overlay drawer to the screen’s left edge. Restored card sizing and vertical scrolling. ARIA expanded state follows open/close. The 64 animations, filters and preview controls are preserved. Checked desktop, 320/390-pixel phones and landscape positioning, scrolling, keyboard close and applying a preset.

## 0.33 — Match the FX/Text drawer pattern

The preset handle is the same compact tab style as FX/T, mirrored onto the left side. Its panel uses the same editor-frame bounds and width as Effects, with matching slide timing and a mirrored border/shadow. Mobile/coarse-pointer sizing follows the other drawers. All 64 animations, filters, preview controls and keyboard close behavior are retained. Geometry checks compare the preset handle and panel directly with FX across desktop, portrait and landscape.

## 0.34 — Preset stacking

Removed automatic deletion of previous generated text animator groups. Repeated presets receive numbered group names and unique property keys. Removing one group updates the tracked group list and retains all other animation. Text-layer Fade In/Rise In now create their own animator groups, so they do not overwrite the shared layer-opacity track. Neon/Outline apply only their own appearance fields. Tested three visible simultaneous presets, preservation of earlier keys/manual groups, a later exit, individual removal, duplicate presets, Undo/Redo/save and combined appearance styles.

## 0.35 — In/Out composition

Fixed overlapping entrance/exit timing and opacity suppression before later groups begin. Added 12 exit presets and refreshed all 76 animated previews. Saved generated exits at the old default start are repaired once; custom timing is preserved. Render checks cover In + Loop + Out, visible middle and hidden endpoints, late entrance neutrality, migration, drawer filters, stacking, history and save/reopen.

## 0.36 — Smooth typography

Unified the preset drawer into Text Animation. Added 16 typography animations (92 native animations total). Smooth monotone tangents retain velocity between keys, settle at extrema and join periodic tracks continuously. Two-key tracks use a stronger ease. Original generated curves in saved projects upgrade once; hand-edited curves and holds remain intact. Tests cover intermediate velocity, loop-seam derivatives, all 92 native renders, In/Loop/Out composition, custom-curve preservation, unified drawer labels, stacking, history, save/reopen and mobile geometry.

## 0.37 — Editable layer masks

Added layer-space cubic Bézier masks, a dedicated path editor with touch targets and zoom/pan, ordered boolean modes, inversion, opacity, anisotropic feather, expansion, path morphing and native numeric keyframes. New paths remain drafts until Done; Cancel discards edits. Path animation protects vertex count. Masks follow parenting/anchors/flips and apply before effects, including sampled motion blur and isolated effect inputs. Adjustment masks mix the original and adjusted result. Static raster masks are cached, preview work scales to display density, and exports render at composition resolution within a per-mask raster budget.

Validation covers alpha/boolean math, feather/expansion, path interpolation and restore, transformed/nested/adjustment layers, image/video rendering, transparent PNG export, draft isolation, curve insertion, rotated feather, zoom/pan, topology protection, duplication/history/save and phone/landscape layout. Existing composition, blend-mode and text-preset checks pass. Physical Android testing remains outstanding.

## 0.38 — Multiple masks and advanced editing

Made mask creation explicit and independent, expanded the layer limit to 32, added new-mask modes, colored reference outlines, linked feather animation, whole-path movement and numeric transforms, and path copy/paste. Duplicated masks retain separate geometry/property tracks. Tests create three masks through the UI (main region, subtract hole and separate island), verify each rendered region, transform vertices/handles, preserve other masks and their keys, and exercise history, saves and the mask limit.

## 0.39 — Auto Trace

Added local source alpha/contrast contour extraction, hole/island detection, simplification, optional curve smoothing and a review/apply dialog. Traces use layer coordinates and remain native editable masks with normal animation controls. A capacity check prevents partial traces over the 32-mask limit. Tests cover text B/O counters, exact binary boundary reconstruction including diagonal contacts and nested regions, decoded PNGs under transformed layers, source capture independent of owner masks/animation, threshold/filter controls, apply/history/save, cancellation and mobile layouts.

## 0.40 — Mask motion tracking

Added deterministic character-linked text masks and decoded-video feature tracking with forward/backward analysis, local deformation, confidence stops, review scrubbing, cancellation and key reduction. Text motion uses the native renderer's glyph transforms and keeps overlapping counters from erasing other letters. Font/content changes safely disable stale links until rebuilding; Freeze makes paths manually editable. Video analysis uses an isolated decoder and retains the original playhead. Applying a result updates only the analyzed range and creates one Undo action. Samples are reduced to path keys within a pixel-space tolerance to control project size.

Checks cover native rise/loop masks, hidden frames, counters/overlap, persistence/freeze, optical flow for translation/rotation/scale and local limb displacement, occlusion rejection, forward/backward decoded video, Stop/Close, review, key reduction, Undo/Redo and save/reopen. Existing masks, auto-trace, 92 text presets and layer-mask integration regressions pass. Physical Android testing remains outstanding.

## 0.41 — Mask editor scope and tracking reliability

Isolated the editor preview using the layer-map renderer, named the target layer, added explicit composition context, shortened guide labels and refreshed source frames on seek. Converted static text Auto Trace batches when enabling text follow, hid irrelevant video actions for text/stills, added a still-layer follow action, and displayed text-build progress. Tracking waits for video frame presentation (with a decoded-frame fallback), validates the source/seed rather than the full project UI state, reports stale-target errors and chooses backward tracking when starting at the clip end.

New regression checks cover a text layer over a character/background, unmasked/cutout isolation without model mutation, traced-text conversion with unrelated character masks preserved, bundled-font rise animation, editor seek refresh and tracking Apply after unrelated layer-transform/UI changes. Forward/backward motion, Stop/Close, save/reopen and existing mask/text tracking checks pass.

## 0.42 — Rotoscope Brush

Added a dedicated selected-source brush workspace with worker-based foreground/background graph cuts, soft alpha refinement, editable correction anchors and motion-assisted bidirectional video propagation. Brush constraints deform with the subject; unreliable motion stops for review. Draft Undo restores matte records as well as strokes, Stop retains completed work, and Cancel leaves the project untouched. Freeze embeds compressed validated matte data and seed labels in one Undo action. Rendering respects native masks, effects, transforms, parenting and nesting. Source-time sample indexing retains cutouts across clip moves/trims; per-frame sample rates survive reopening and mixed-rate analysis. Added fringe cleanup, final feather/shift/invert, bypass/remove, source-alpha support, black/white inspection and automatic fit in portrait/landscape.

Validation covers synthetic silhouette/hole accuracy and known fractional edges, pointer brush corrections, hard constraints after feathering, Undo/Stop/Cancel, forward/backward correction propagation, frozen-save/new-page video reload, move/trim/refreeze timing, transformed/nested alpha and masks, effect maps, PNG export pixels, multiple matte layers, corrupt-data transparency and compact UI geometry. Device performance/Android native export remains unverified.

## 0.43 — Smart connected-edge brush

Changed the default workflow to a foreground stroke. Added perceptual-color geodesic competition against explicit/background-border regions, connected component validation, contrast-limited brush stamps, an Alt subtract modifier, low-resolution live draft previews, magenta alpha boundaries and a cursor reflecting working-pixel diameter. Hard-stroke sampling now preserves thin odd-coordinate marks while bounding color-model training samples. Full matte refinement uses nearby interior foreground/background colors at edges. New 768/1024 detail modes use a bounded coarse solve plus high-resolution edge reconstruction; working mattes and source-time propagation stay portable.

New tests check selection without a box/red setup, holes, same-color disconnected distractors, edge-limited footprints, hard corrections, thin seeds, 1024 detail alpha, live pointer previews, disconnected add/Alt subtract, Freeze and saved mode. Existing roto, video propagation, rendering/export and compact layout checks remain. This does not include Adobe's proprietary semantic Roto Brush engine.

## 0.44 — Mobile preview and propagation repair

Decoupled sharp source preview from matte analysis, limited preview snapshots to 2048 pixels, matched the panel to the main mobile frame, and retained alpha/fringe inspection without substituting a low-resolution source image. Added a worker-safe bounded RGB patch field with foreground-weighted global recovery, local forward/backward matching, confidence checks and local displacement interpolation. Propagation uses at least 15 internal motion steps per second when output samples are sparse, validates automatic silhouette seeds against previous appearance, handles manual anchors before estimating intervening motion, and restores the last accepted preview/strokes on Stop/failure. Scene changes require a correction; a direct review action opens the stopped frame.

Checks include exact 1600px source stripes at 384px analysis/DPR2, roto/app width equality on desktop/phones/landscape, articulated body/arm deformation and appearance occlusion, a decoded video moving 70px with an independently waving arm, no trailing body, forward/backward analysis, Stop/Cancel, saved-media reload, source-time edits, final alpha, and existing mask/export regressions. Device-specific Android performance remains unverified.

## 0.45 — Video frame / matte synchronization

Added source snapshots paired with presented video timestamps, a coalescing seek queue, decoded-frame redraws, and shared first-frame preparation for PNG/video exports. Roto mattes and generated tracked paths use the source timestamp of the pixels being painted instead of an ahead-of-decoder timeline clock. Analysis records decoded sample times, removes duplicate samples, and Freeze can retime legacy roto data without changing its alpha. Fixed a stale queued seek during playback and prevented an old playback tick from running a second clock during export. Negative RAF deltas no longer move the playhead before zero.

A 15fps moving-source/30fps-composition regression reproduces background leakage with the old clock, then verifies source advancement, non-frame-aligned seeks, rapid scrubbing, playback without repeated seeks, and independently decoded exported-video pixels for both roto and tracked paths. Existing roto timing refresh, new-page persistence, tracking, articulated movement, nested compositions, PNG alpha, effects and browser audio export checks pass. Browser export remains realtime; this fixes source/mask correspondence, not offline deterministic encoding.
