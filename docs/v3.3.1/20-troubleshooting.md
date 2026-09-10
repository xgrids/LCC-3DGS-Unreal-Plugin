---
title: Troubleshooting
parent: Documentation v3.3.1
nav_order: 20
description: Locate common LCC4Unreal failures by symptom, covering data loading failures, nothing displayed, flickering and ghosting, lighting problems, frame rate and video memory issues, packaging errors, and integration problems.
---

# Locating Common UE5 3DGS Failures by Symptom

This page is organized by **the symptom you see**, and each entry gives the possible causes, how to confirm them, and how to fix them.

For how to view logs and use the debug tools, see [Logging and Diagnostics](./21-diagnostics.md). For consultation questions (which formats are supported, the difference between the two pipelines), see [FAQ](./19-faq.md).

## Contents

| Category | Symptoms covered |
|----------|-----------------|
| [Do These Two Things First](#do-these-two-things-first) | Recommended for any problem |
| [Data Loading Failures](#data-loading-failures) | Nothing happens after Load, invisible after packaging, Blueprint project packaging, wrong GIS position, crash on huge data |
| [Nothing Displayed or Partially Displayed](#nothing-displayed-or-partially-displayed) | Nothing visible at all, distant content missing, holes at the edge, blank SceneCapture, occluded by water |
| [Image Quality Issues](#image-quality-issues) | Ghosting, flickering, holes, grayish colors, seams, a line across the scene |
| [Lighting Problems](#lighting-problems) | Overexposure, no shape shading, ProxyMesh has no effect, shadows cut off, effects hidden |
| [Parameter Changes Have No Effect](#parameter-changes-have-no-effect) | Checkbox unticked, full load, clipping and section, spherical harmonics, license quota |
| [Performance Problems](#performance-problems) | Low frame rate, high video memory, load stutter, wrong occlusion |
| [Collision and Navigation](#collision-and-navigation) | Line traces miss, character falls or clips through, NavMesh not generated |
| [Crashes](#crashes) | ArraySliceIndex assertion error |
| [Licensing Problems](#licensing-problems) | Status is not a green check, various licensing errors |
| [Building and Packaging](#building-and-packaging) | Missing binaries, missing precompiled manifest, packaging failure, Android |
| [Still Not Solved](#still-not-solved) | What to collect before reporting an issue |

## Do These Two Things First

Most problems can be located within these two steps, so run through them for any symptom.

1. **Open the Output Log and read the plugin log.** Load failures, path errors, and licensing problems all leave clear information there. See [Viewing the Plugin Log](./21-diagnostics.md#viewing-the-plugin-log).
2. **Confirm the data loaded successfully.** Select the Actor and check whether MetaInfo in the Details panel has content (Total Splats greater than 0). Empty means the data did not come in, so go straight to [Data Loading Failures](#data-loading-failures).

## Data Loading Failures

### Symptom: nothing appears after clicking Load

Check in order:

| Possible cause | How to confirm | Fix |
|----------------|----------------|-----|
| The path does not exist or is misspelled | The log shows `LCC file :<path> does not exist.` | Verify the path. Relative paths are based on `Content` |
| LCC1 data is missing files | The log shows `Load Meta.lcc error` or `meta.lcc file :<path> load error` | `data.bin` and `index.bin` must sit next to the `.lcc` file; missing either one fails |
| The wrong Actor is used | No explicit error, but a blank image | `.lcc2` uses `ALCC2Actor`, `.lcc` uses `ALCCActor`, and single-file formats each have their own Actor. See [FAQ](./19-faq.md#how-do-i-choose-an-actor) |
| The format is not supported | The log shows `LCC4Unreal do not support this file format!` | Confirm the extension is within the supported range, see [Introduction](./01-introduction.md#supported-formats) |
| The `.ply` is not a 3DGS format | The log states the PLY was rejected | The plugin only supports `.ply` with 3DGS properties; regular geometric point clouds cannot be loaded |

### Symptom: fine in the editor, invisible after packaging

Check two things in order.

**One, whether an absolute path was used.** Absolute paths only work on the local machine and no longer exist on another one. Switch to a relative path, relative to the project `Content` directory, for example `Scenes/Tower/meta.lcc2`.

**Two, whether the data directory is configured in the packaging settings.** The two settings serve different purposes, so pick as needed:

| Setting | Purpose |
|---------|---------|
| `Additional Non-Asset Directories To Copy` | The data is copied into the package output as regular files |
| `Additional Non-Asset Directories To Package` | The data is packed into the pak file |

Use the latter to pack LCC data into the pak, and the former when the data only needs to ship alongside the package output. Both live under `ProjectSettings > Packaging`.

Repackage after configuring them and check whether the data really is in the package output. For the detailed configuration, see [Quick Start](./02-quickstart-windows.md#release-packaging).

### Symptom: the plugin does not work after packaging a Blueprint project

Blueprint-only projects are supported in the editor only, not for packaging. A C++ project is required for packaging.

### Symptom: the geographic position is wrong, GIS mode has no effect

Check these items:

- Whether the data contains RTK information. Use `GetMetaInfo().IsRTK()` to check; the log showing `This lcc does not have RTK information!` means the data has no geographic information
- When enabling from code, call `SetGeoPlacement(true)`, which reloads automatically so the setting takes effect. Assigning `bEnableGeoPlace` directly does not trigger a reload
- Use `GeoLocationOffset` for fine adjustment when the position is offset

For the setup steps with Cesium, see [Third-party and Engine Plugin Integration](./12-integration.md#gis-cesium-integration).

### Symptom: crash while navigating very large LCC2 data

On v1.0.0 this is a known defect of that version (a GPU Buffer overrun). Upgrade to v2.x or above.

### Symptom: array related error when loading a large PLY

A known defect of v3.0.0, fixed in v3.3.0 and above; upgrade the version.

Also, when a PLY has no spherical harmonics, files above 2 GB may fail to load, so converting to the LCC2 format is recommended.

## Nothing Displayed or Partially Displayed

### Symptom: the Actor is in the scene but no content is visible

| Possible cause | How to confirm | Fix |
|----------------|----------------|-----|
| The data did not load | See the previous section | Solve the loading problem first |
| [`LoadMode`](./07-visual-settings.md#load-mode) is set to `None` | Check the Details panel | Change it back to `Both` |
| The camera is beyond the render distance | Move closer and see whether it appears | Raise [Max Distance](./10-performance-parameters.md#max-distance) |
| A clipping volume cut the content away | Temporarily disable `bEnabled` on the clipping volume | Check the clip mode, since `Inside` and `Outside` do the opposite, see [EClipType](./23-api-reference/10-enums.md#ecliptype) |
| A section plane cut the content away | Temporarily disable the section plane | Check `Mode` and the plane orientation, see [ESectionType](./23-api-reference/10-enums.md#esectiontype) |
| [`GlobalAlpha`](./07-visual-settings.md#global-alpha) is 0 | Check the Details panel | Change it back to 1.0 |
| Everything is in the environment data but `OnlyMain` is set | Switch `LoadMode` and compare | Choose according to the actual data, see [ELoadMode](./23-api-reference/10-enums.md#eloadmode) |

### Symptom: distant content is missing and appears only when getting closer

This is the normal result of LOD and the distance limit, not a failure. To display distant content as well:

- Raise [Max Distance](./10-performance-parameters.md#max-distance), at the price of lower performance
- Lower [Level Factor](./10-performance-parameters.md#level-factor) so higher precision is used at the same distance
- Fog can help hide the distance boundary

### Symptom: holes at the screen edge while turning the view quickly

Node preloading cannot keep up with the view change. The LCC pipeline can enable [Add Extra Preload Nodes](./10-performance-parameters.md#add-extra-preload-nodes), which appends extra preload nodes at the price of more nodes to render.

### Symptom: SceneCapture or the minimap is blank

Enable [SceneCaptureComponent Support](./10-performance-parameters.md#scenecapturecomponent-support) in the project settings first. It is disabled by default and has a slight performance cost.

To set the render strategy for a SceneCapture separately from code, see [SetSceneCaptureRenderMode](./23-api-reference/02-ULCCComponentBase.md#setscenecapturerendermode); note this group of interfaces only works on the LCC pipeline.

### Symptom: 3DGS is occluded by the water surface

Caused by the depth handling of the single layer water material. Enable SingleLayerWater Support, see [Single Layer Water Support](./17-single-layer-water.md).

## Image Quality Issues

### Symptom: ghosting and trailing while moving

Caused by the anti-aliasing method. TSR and TAA rely on history frames for temporal accumulation, so a moving 3DGS easily leaves the previous frame behind.

Try them in order and find the balance between quality and ghosting:

```text
None → FXAA → MSAA → TAA → TSR
```

When the scene contains 3DGS only, `None` can be set directly, which removes ghosting and saves the anti-aliasing cost. For the default value of each pipeline and where to set it, see [Performance Parameters](./10-performance-parameters.md#anti-aliasing-methods).

### Symptom: the image flickers and edges jitter

Try these in order of benefit:

1. Check the anti-aliasing method. This is the most common cause, and TSR is recommended for the LCC2 pipeline, see [Performance Parameters](./10-performance-parameters.md#anti-aliasing-methods).
2. LCC pipeline: lower [Sort Factor](./10-performance-parameters.md#sort-factor) to sort more often. When the sort frequency is too low, the front-to-back order of translucency only updates every few frames, showing up as slight jitter.
3. Check [Small Splat Threshold](./10-performance-parameters.md#small-splat-threshold); a value that is too large makes the background grainy.
4. For fine structures flickering while the camera moves in and out, try enabling [Mip Filter](./07-visual-settings.md#mip-filter), which applies a low-pass filter with opacity compensation and is more stable at different scales. `.ply` / `.spz` / `.sog` have it disabled by default.

### Symptom: the image has holes and looks sparse

[SplatScale](./07-visual-settings.md#splatscale) is set too low. The default 1.0 is the upper limit; lowering it reduces overdraw and raises the frame rate, but smaller quads expose gaps. Raise it back somewhat.

### Symptom: colors look gray and flat

Handle it with the color parameters, see [Visual Settings](./07-visual-settings.md#color-adjustment). The common approach is raising `Contrast` slightly, or brightening the dark areas with `Gamma`. For the code interface, see [Color Adjustment](./23-api-reference/02-ULCCComponentBase.md#color-adjustment).

### Symptom: obvious seams in the image (LCC pipeline)

LCC file version 5.0 and above handles seams automatically. For data from older versions, enable [Seam Cutting](./07-visual-settings.md#seam-cutting) manually, which corresponds to the property [bEnableSeamCutting](./23-api-reference/03-ULCCComponent.md#properties).

### Symptom: a line appears across the scene

Check the scale of the Actor. **The LCC family of Actors (`ALCCActor`, `ALCC2Actor`, `ASogActor`, `ASpzActor`, `APlyActor`) supports uniform scaling only.**

Do not use non-uniform scaling like this:

- With negative values, for example `(-1, 1, 1)`
- With unequal axes, for example `(2, 1, 3)`

All three axes must stay equal, for example `(1, 1, 1)` or `(2, 2, 2)`. Non-uniform scaling causes rendering anomalies that show up as a line in the image.

## Lighting Problems

### Symptom: overexposure after switching to Lit

The colors of captured data already have the lighting of the capture site baked in, and scene lights add another layer on top.

- LCC2 pipeline: lower the original brightness with `LightingScale`, see [Normals and Lighting](./08-normals-and-lighting.md#lcc2-lighting-scale)
- Check whether the scene lighting intensity is too high

### Symptom: no shape shading in Lit mode, the image looks flat

This is expected behavior. 3DGS data has no geometric normals, and `Fixed`, `ViewFacing`, and `Hemispherical` all construct normals by approximation, so they can only produce overall brightness changes and **none of them can produce shading that follows the shape**. For the definition of each mode, see [ELCC2NormalGenerationMode](./23-api-reference/10-enums.md#elcc2normalgenerationmode).

Only `ProxyMesh` can produce real shape shading. It requires authoring and placing a proxy mesh and requires a license. See [Proxy Mesh](./13-proxy-mesh.md) and [Normals and Lighting](./08-normals-and-lighting.md#choosing-a-mode).

The difference between the three approximate modes is how overall brightness changes with lighting and view, not whether shape shading exists:

- `Fixed`: the whole area shares one fixed normal, completely stable while the camera moves
- `ViewFacing`: the normals follow the camera, so overall brightness changes while turning the view
- `Hemispherical`: normals map from screen position onto a fixed hemisphere, so overall brightness transitions more smoothly than the other two while a directional light rotates

### Symptom: ProxyMesh mode is set but has no effect

| Item to check | How to confirm |
|---------------|----------------|
| Whether NormalMode is ProxyMesh | Check the Details panel |
| Whether the license is valid | Check Status in the plugin panel; a green check means the license is fine. From code, `GetEffectiveNormalGenerationMode()` returning `Fixed` means it was downgraded |
| Whether the proxy mesh has a StaticMesh | The log shows a `has a null StaticMesh` warning |
| Whether the proxy mesh overlaps the 3DGS in space | Pairing is judged by position, so no overlap means no effect |

See [Proxy Mesh](./13-proxy-mesh.md#troubleshooting).

### Symptom: shading appears in the wrong place

The proxy mesh deviates too much from the actual 3DGS surface. Either improve how closely the proxy mesh matches, or switch to an approximate normal mode, which has no shape shading but is more stable.

### Symptom: ProxyMesh shadows appear only nearby and disappear in the distance

Shadows are cut off by distance. This is an engine issue, not a plugin defect.

Fix: select the ProxyMesh Actor and turn `Far Shadow` **off and back on** to restore complete shadows. The property is under the Lighting category of StaticMeshComponent.

### Symptom: huge abnormal shadows in Lit mode

Approximate normal modes produce anomalies at certain lighting angles. Try in order:

1. Switch [NormalMode](./08-normals-and-lighting.md#choosing-a-mode) to find which mode behaves correctly
2. Adjust the angle of the directional light
3. Switch to [ProxyMesh](./13-proxy-mesh.md) mode with a well-matched proxy mesh, which gives the best result

### Symptom: colors changed after enabling shadows

This is expected behavior. Once 3DGS receives external lighting, its colors change with the light source, so adjust the color and intensity of the directional light.

### Symptom: brightness cannot be adjusted, the whole scene is dark

When the project uses a compositing plugin such as Composure, disable the post-processing related options on the Component and control exposure through a Post Process Volume instead.

For exposure related settings, see [Visual Settings](./07-visual-settings.md#post-processing).

### Symptom: Niagara effects are invisible on the 3DGS

**The LCC2 pipeline outputs depth, so effect occlusion is correct** and this problem normally does not occur.

Only the LCC pipeline (`.lcc` data) can hit it, because it outputs no depth and the front-to-back order of translucency has to be decided by sort priority. The fix is raising `Translucent Sort Priority` on the Niagara System so it renders above the 3DGS.

### Symptom: messy content around the outside of the scene

That is the environment data. Change [LoadMode](./07-visual-settings.md#load-mode) from `Both` to `OnlyMain` to render only the main part.

### Symptom: switching the light mode does nothing

**[SetLightMode](./23-api-reference/02-ULCCComponentBase.md#setlightmode--getlightmode) has no effect in point cloud mode**: it skips the assignment and prints a warning. Switch back to 3DGS mode first.

## Parameter Changes Have No Effect

### Symptom: values under Performance were changed but nothing changed

**Every parameter has a checkbox on its left, and while it is unticked the plugin uses its built-in default and the entered value has no effect.** This is the single most common pitfall.

For the built-in default of each parameter, see [Performance Parameters](./10-performance-parameters.md#actor-panel-parameters).

### Symptom: changing the full load parameters does nothing

[Use Full Load](./10-performance-parameters.md#use-full-load) and [Full Load Splat Number](./10-performance-parameters.md#full-load-splat-number) are evaluated at load time, so the data has to be reloaded after changing them.

For the difference between the two loading methods, see [Rendering](./06-rendering.md#chunked-rendering-and-full-load-rendering).

### Symptom: changing clipping volume or section plane properties at runtime does nothing

`bEnabled`, `Mode`, and `VolumeType` have no setter, so **`Refresh()` must be called on that Actor** after assigning them at runtime. The same applies to changing the transform (position, rotation) at runtime.

Changing properties in the Details panel in the editor updates automatically and needs no manual call.

See [ALCCClippingVolume](./23-api-reference/07-ALCCClippingVolume.md#refresh).

### Symptom: spherical harmonics is enabled but the image did not change

- The data may be of the `Portable` type, which contains no spherical harmonics. Confirm with [CanSetShcoef()](./23-api-reference/02-ULCCComponentBase.md#cansetshcoef); for the type definitions, see [EFileType](./23-api-reference/10-enums.md#efiletype)
- [SetUseShcoef](./23-api-reference/02-ULCCComponentBase.md#setuseshcoef--getuseshcoef) fails silently in point cloud mode, so switch back to 3DGS first
- LCC2 can lower the band count instead of disabling it completely, see [SH Bands](./07-visual-settings.md#sh-bands)

### Symptom: many clipping volumes were added but only some take effect

The free edition limits each type to 50. The log states it explicitly: `Unlicensed: enabled clipping volumes limited to 50 ...`. For licensing, see [Editions and Licensing](./05-pro-features.md).

## Performance Problems

For the complete tuning workflow for a low frame rate, see [Performance Guide](./11-performance-guide.md); this section only lists quick checks.

### Symptom: low frame rate

First confirm whether the bottleneck is the 3DGS. Use `stat unit` to read Game / Draw / GPU, then `stat XGrids` to read the cost of LCC itself. When the LCC share is low, the problem is elsewhere in the scene (lighting, post-processing, Blueprint logic) and tuning LCC parameters brings no improvement.

Once the 3DGS is confirmed as the cause, adjust in order of benefit:

1. Raise [Level Factor](./10-performance-parameters.md#level-factor) (the most noticeable benefit)
2. Lower [Max Distance](./10-performance-parameters.md#max-distance)
3. Lower [Max Splat Num](./10-performance-parameters.md#max-splat-num)
4. Raise [Start Level](./10-performance-parameters.md#start-level) to skip the finest Level
5. Disable spherical harmonics or lower [SH Bands](./07-visual-settings.md#sh-bands)
6. Switch to [point cloud mode](./07-visual-settings.md#render-mode) if necessary

### Symptom: video memory usage is too high

- Raise [Level Factor](./10-performance-parameters.md#level-factor)
- Lower [Max Splat Num](./10-performance-parameters.md#max-splat-num)
- Adjust [LCC2 GPU Memory Budget](./10-performance-parameters.md#lcc2-gpu-memory-budget) on the LCC2 pipeline
- Adjust the [Max GPU Usage Percetage For Free](./10-performance-parameters.md#max-gpu-usage-percetage-for-free) release threshold

For the video memory allocation rules and the automatic release mechanism, see [Rendering](./06-rendering.md#video-memory-management-and-tuning). Single-file formats (`.sog` / `.spz` / `.ply`) load fully at once, so their video memory usage is constant and does not change with the view, see [Loading Limits of Single-file Formats](./06-rendering.md#loading-limits-of-single-file-formats).

### Symptom: stutter while loading

- Enabling collision for the first time has a one-off baking cost, so enable it during the loading stage rather than in the middle of player interaction
- Lower [Max Load Collision Distance](./10-performance-parameters.md#max-load-collision-distance) to load only the range needed
- Adjust the thread configuration, see [Performance Parameters](./10-performance-parameters.md#thread-pool-settings)

### Symptom: wrong occlusion when several 3DGS interpenetrate

- LCC pipeline: enable [multi-Actor translucency sorting](./07-visual-settings.md#multi-actor-translucency-sorting)
- LCC2 pipeline: adjust the [depth threshold](./07-visual-settings.md#depth-threshold) to change where depth is written

## Collision and Navigation

### Symptom: line traces miss the 3DGS

| Item to check | Action |
|---------------|--------|
| Whether the data contains collision | Single-file formats contain no collision data, see [Prerequisites](./15-collision.md#prerequisites) |
| Whether collision is enabled | Tick `bEnableCollision`, see [Enabling](./15-collision.md#enabling) |
| Whether collision is loaded at that location | Use `ShowCollision()` to see the wireframe, see [Collision Visualization](./21-diagnostics.md#collision-visualization) |
| Whether the trace distance exceeds the collision load range | Raise [Max Load Collision Distance](./10-performance-parameters.md#max-load-collision-distance) |

The log showing `There is neither collision.bin nor collision.lci in the folder` means there is no collision file in the data directory.

LCC1 has another set of ray test interfaces against point cloud positions, but they are **not sufficiently tested**, so using collision plus engine ray tests is recommended, see [ULCCComponent Raycast](./23-api-reference/03-ULCCComponent.md#raycast).

### Symptom: the character falls down right at the start

Collision is loaded dynamically in chunks, so the collision data may not be built yet when the game starts, and the character falls when there is no collision underfoot.

How to handle it:

- Place PlayerStart slightly above the ground
- Or delay character movement by a few seconds
- Enable collision during the loading stage rather than after the player starts interacting

### Symptom: the character clips through or falls after moving a certain distance away

Collision streams in by distance, so areas beyond the load range have no collision bodies. Raise `Max Load Collision Distance(m)` to cover the activity range of the character.

Collision can also fail to keep up when the character moves too fast, which is likewise mitigated by raising the load distance.

### Symptom: the NavMesh is not generated at all

| Item to check | Action |
|---------------|--------|
| The data contains no collision | Confirm `.lcc` or `.lcc2` is used; single-file formats have no collision data |
| Collision is not enabled | Tick `bEnableCollision` and confirm `CanEverAffectNavigation = true` |
| Collision is not loaded yet | Select player collision or visibility collision in view mode and confirm collision bodies appeared in the target area |
| The collision load range is insufficient | Raise `Max Load Collision Distance(m)` to cover the whole AI activity area |
| NavMeshBoundsVolume is missing or does not cover the area | Place and scale it to cover the target area |
| Navigation was not rebuilt | Run `Build → Build Paths` and save the level |

For the complete workflow, see [Navigation System Support](./16-navigation.md).

## Crashes

### Symptom: crash after an ArraySliceIndex assertion error

The error looks like this:

```text
Assertion failed: ArraySliceIndex >= 0
```

The cause is that the current version **does not support the Adaptive GBuffer format of Substrate**.

Fix:

1. Open `ProjectSettings > Rendering` and find **Substrate GBuffer Format (Project)**.
2. Change the value to **BlendableGBuffer**. This is the engine default.
3. Restart the engine.

`AdaptiveGBuffer` is a known incompatible format, and `BlendableGBuffer` works correctly.

## Licensing Problems

**Check Status in the plugin panel first**: a green check means the license is fine and Pro edition features are available. Anything other than a green check means the license did not take effect, so read the log to confirm the reason.

The licensing information in the log is fairly explicit, so act on what it states:

| Log message | Meaning and action |
|-------------|-------------------|
| `ProjectID is invalid; generate one in Project Settings` | The project has no Project ID. Generate one under `Project Settings > Project > Description` |
| `Failed to decode AppKey, please check.` | The AppKey content is incomplete or was copied incorrectly, copy it again |
| `Invalid AppKey, please check.` | The AppKey format is wrong, confirm it is the complete string from the developer platform |
| `Authorization has expired, please check.` | The license expired, regenerate it on the developer platform |
| `AppKey has expired. Please generate a new one.` | Same as above |
| `HTTP request failed` / `HTTP error! Status: <code>` | A network problem or the licensing server is unreachable, check the network and the firewall |
| `Signature Verification Failed` | Signature verification failed, contact technical support |

For the licensing workflow, see [Editions and Licensing](./05-pro-features.md).

## Building and Packaging

### Symptom: missing binaries or module build failure

When any of the errors below appears, regenerate and rebuild the project following the steps in this section:

```text
Missing UnrealGame binary. You may have to build the UE project with your IDE.
Alternatively, build using UnrealBuildTool with the commandline:
UnrealGame <Platform> <Configuration>
```

```text
*** could not be compiled. Try rebuilding from source manually
```

The cause is that after the plugin is added to a C++ project, the engine detects the new modules but the matching build output is missing. Handle it as follows:

1. Close the project.
2. Locate the `*.uproject` file of the project.
3. Right-click `*.uproject` and select **Generate Visual Studio project files**.
4. Wait for the VS project to finish regenerating.
5. Double-click `*.sln` to open Visual Studio.
6. Right-click the project in Solution Explorer and select **Set as Startup Project** to make sure it is the startup project.
7. Confirm the configuration is **Development Editor** and **Win64**.
8. Click **Debug > Start Without Debugging** to launch the project.
9. Once the build passes, the project opens normally. Afterwards double-clicking `*.uproject` is enough and this workflow is not needed every time.

When it still fails after the steps above, delete the `Intermediate` directory of the project first, then run again from step 3.

For the plugin installation steps, see [Quick Start](./02-quickstart-windows.md#step-4-install-the-plugin).

### Symptom: packaging fails

Check these three items first:

- **Full Rebuild under `ProjectSettings > Packaging` must stay disabled**, and do not run Rebuild in VS. The plugin does not support either, see [Quick Start](./02-quickstart-windows.md#notes)
- Confirm a C++ project is used, since Blueprint projects cannot be packaged
- Confirm the engine version is within the supported range (UE 5.4 ~ 5.8)

For specific errors, see the two sections below.

### Symptom: missing precompiled manifest during packaging

The error looks like this:

```text
Missing precompiled manifest for 'LCC4UnrealRuntime',
'\Shipping\LCC4UnrealRuntime\LCC4UnrealRuntime.precompiled'.
This module was most likely not flagged for being included in a precompiled build
- set 'PrecompileForTargets = PrecompileTargetsType.Any;' in LCC4UnrealRuntime.build.cs
to override. If part of a plugin, also check if its 'Type' is correct.
```

**Why it happens:** the files mentioned in the error ship with the plugin and live in its `Intermediate` directory. Running Full Rebuild under `ProjectSettings > Packaging`, or Rebuild in VS, makes the engine clean the `Intermediate` directory and delete those precompiled outputs along with it.

LCC4Unreal is a binary plugin with no source code, so the deleted outputs **cannot be rebuilt** and can only be restored from the plugin package. Avoid both operations for that reason.

**How to fix it:**

1. Confirm **Full Rebuild** under `ProjectSettings > Packaging` is disabled, and do not run Rebuild in VS.
2. Copy the contents of the plugin directory `lcc4unreal/Intermediate/Build/Win64/UnrealGame` into `<project directory>/Intermediate/Build/Win64/<project name>`.
3. Copy the contents of the plugin directory `lcc4unreal/Intermediate/Build/Win64/x64` into `<project directory>/Intermediate/Build/Win64/x64`.
4. Restart the engine and package again.

> The target directory name in step 2 is the **project name**, not `UnrealGame`. For a project called `MyProject`, the target path is `MyProject/Intermediate/Build/Win64/MyProject`.

**If the error points to other files:** the two directories above cover the common cases. When the error mentions another file, handle it the same way: find that file under the `Intermediate` directory of the plugin at the same relative path and copy it to the matching location in the project.

**If the Intermediate directory of the plugin was cleaned too:** there is nothing left to copy from, so download the plugin package again and extract it over the existing files to restore them.

### Symptom: build fails on a custom engine

Released plugin packages only work with engines published by Epic. Vendor-customized branches, commercial engines derived from UE, and engines with locally modified source all require a custom build, see [Custom Engine Versions](./12-integration.md#custom-engine-versions).

### Symptom: errors when packaging for Android

The current version does not support packaging directly to the Android platform. This is a platform compatibility limitation and cannot be solved by changing the packaging configuration.

To use it on a VR device, run on PC and stream, see [Quick Start - Quest3](./04-quickstart-quest3.md).

## Still Not Solved

Collect the following information and contact us, see [Contact Us](./22-contact-us.md):

- Plugin version and engine version
- Data format and approximate size
- **The complete log file of the run where the problem occurred**, unfiltered and not just the error lines
- Reproduction steps
- Graphics card model and driver version

For the log file location and other details, see [Logging and Diagnostics](./21-diagnostics.md#what-to-attach-when-reporting-an-issue).
