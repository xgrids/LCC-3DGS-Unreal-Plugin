---
title: FAQ
parent: Documentation v3.3.1
nav_order: 19
description: Frequently asked questions about LCC4Unreal, covering supported data formats, engine versions, platforms, licensing, performance, and how to handle common errors.
---

# UE5 3DGS Plugin FAQ

## Which data formats are supported?

LCC2 (.lcc2), LCC (.lcc), SOG (.sog), SPZ (.spz), PLY (.ply).

## Which UE versions are supported?

UE 5.4 ~ 5.8.

## Which platforms are supported?

Windows (Win64), Linux, LinuxArm64. macOS, Android, and iOS are not supported.

## Can it be used on a custom engine version?

Released plugin packages only work with engines published by Epic. Vendor-customized branches (such as the NVIDIA RTX branch), commercial engines derived from UE (such as Pixotope), and engines with locally modified source all require a custom build, see [Custom Engine Versions](./12-integration.md#custom-engine-versions).

## What is the difference between LCC and LCC2?

The [LCC pipeline](./01-introduction.md#two-rendering-pipelines) is the older implementation and uses chunked streaming. The [LCC2 pipeline](./01-introduction.md#two-rendering-pipelines) is the current main pipeline and supports depth output, LOD octrees, normal generation, and other newer features. Single-file formats (SOG/SPZ/PLY) use the LCC2 pipeline internally.

## How do I choose an Actor?

- `.lcc2` file → ALCC2Actor
- `.lcc` file → ALCCActor
- `.sog` file → ASogActor
- `.spz` file → ASpzActor
- `.ply` file → APlyActor

## Are relative paths supported?

Yes. Relative paths are relative to the Content directory of the project. Relative paths are mandatory when distributing to other machines.

## Is the plugin open source? Can the source be modified?

No. The plugin is distributed as binaries and provides only compiled modules and a public API, with no source code.

Because it is a binary plugin, **the `Intermediate` folder under the plugin directory must not be deleted**. The `Intermediate` of the project can be regenerated, but the one of the plugin cannot once removed, and the plugin package has to be downloaded again.

## Is PLY import supported? What are the requirements for PLY?

Yes, but the format has requirements:

- Must be a standard 3DGS format PLY
- Spherical harmonics of band 0 and band 3 are supported (band 3 means 45 coefficients)
- Property types must be `float`
- Property order must match the standard 3DGS order
- With or without a Normal property is fine

PLY files that do not meet these conditions (for example non-standard variants exported by third-party tools) have to be converted to the standard format first. Regular geometric point cloud PLY cannot be loaded.

> Note: bands 1 and 2 will be supported in the next version.

PLY also has no LOD and no chunking, so converting large scenes to the LCC2 format is recommended.

## Is Android packaging supported?

The current version does not support packaging directly to the Android platform. To use it on a VR device, run on PC and stream, see [Quick Start - Quest3](./04-quickstart-quest3.md).

## Is orthographic view supported?

It depends on the render mode:

- **Point cloud mode**: supported, both orthographic and top views display normally
- **3DGS mode**: not supported, nothing is visible in orthographic view

To view the scene in orthographic view (for floor plans or top-down views), switch `RenderMode` to `Point Cloud`.

## Can a mirror material reflect 3DGS?

No. When a reflection is needed (a rear-view mirror, for example), use the `SceneCapture2D` component of the engine to capture the image instead.

## Can it be rendered together with regular Mesh models?

Yes. The typical approach is to cut away the part of the 3DGS to be replaced with the clipping tools, then place the reworked Mesh into the clipped area; the plugin handles depth and occlusion automatically.

Set the Mesh material to Opaque; translucent materials can produce depth sorting issues. Extend the clipping edge slightly beyond the Mesh bounds.

## Can I get the latitude and longitude of a point on the 3DGS?

The plugin does not provide a coordinate conversion interface directly, but the calculation can be done manually. The `.lcc2` description file contains `epsg` and `offset` information, which can be converted together with [GetRTKBaseLocation](./23-api-reference/02-ULCCComponentBase.md#getrtkbaselocation) and the conversion methods of the geographic reference system.

## Can annotations made in LCC Studio be shown in the plugin?

No. Annotations are saved in the `attrs.lcp` file, which the plugin does not parse, so the application layer has to read and render it.

## Can the skybox configured in LCC Scene Editor be brought into UE?

No. Two easily confused concepts need to be separated here, because the "sky" seen in an LCC scene actually comes from two different places:

| | Environment Data | Skybox |
|---|---|---|
| What it is | The scene environment background generated automatically during reconstruction, including the distant sky and distant views. Essentially 3DGS Gaussians marked as "environment" | The background effect replaced through a preset template in LCC Scene Editor |
| Part of the data itself | Yes, stored along with the LCC data | No, only an overlay display feature of the Editor |
| Supported by the plugin | Rendering supported | Not supported, not carried over with the data |

In other words, the plugin can render the reconstructed environment background, but the skybox template swapped in the Editor does not come along.

For sky effects in UE, use the engine solutions such as Sky Atmosphere, SkyLight, or a custom skybox material. To view only the environment data, confirm `LoadMode` is `Both` (the default value); setting it to `OnlyMain` hides the environment part, see [Load Mode](./07-visual-settings.md#load-mode).

## Are C++ calls supported?

Yes. Add the `LCC4UnrealRuntime` module dependency in build.cs.

## Is collision supported?

`.lcc` and `.lcc2` are supported and the data must ship a collision file; `.sog`, `.spz`, and `.ply` contain no collision data and have to be built manually with Blocking Volumes or an approximate mesh. See [Collision](./15-collision.md#prerequisites).

## Is nDisplay supported?

Yes. With `ALCCActor` ([LCC pipeline](./01-introduction.md#two-rendering-pipelines)), set `Default Traversal Type` to `Circle` and restart the editor; the LCC2 pipeline can stay on the default. See [Third-party and Engine Plugin Integration](./12-integration.md#ndisplay-multi-viewport).

## Is VR supported?

Stereo rendering (InstanceStereo) is supported.

## How many clipping volumes / section planes can actually be placed?

| License | Effective count |
|---------|-----------------|
| Free edition | 50 of each type |
| Pro edition | 65536 of each type |

The two types are counted independently and do not consume each other's quota. For licensing, see [Editions and Licensing](./05-pro-features.md).

## What do I do about a specific failure?

This page only answers consultation questions. When a specific symptom already occurs (nothing displayed, flickering, low frame rate, packaging errors), see [Troubleshooting](./20-troubleshooting.md) and locate it by symptom.

To view logs and use the debug tools, see [Logging and Diagnostics](./21-diagnostics.md).

## Any other questions?

See [Contact Us](./22-contact-us.md) for help through the technical support email, the developer platform, or the Discord community.
