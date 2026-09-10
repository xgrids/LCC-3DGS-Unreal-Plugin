---
title: Enums
parent: API Reference
grand_parent: v3.3.1 (latest)
nav_order: 10
description: Reference for the public enum types of LCC4Unreal, covering render mode, load mode, light mode, normal generation mode, clipping and section direction, file format, and other values.
---

# Enums: Enum Type Reference

Enum types the `LCC4UnrealRuntime` module exposes.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `Core/LCCEnum.h` |

```cpp
#include "Core/LCCEnum.h"
```

Enums marked `BlueprintType` are usable directly in Blueprint, can be wired to a Switch node, and can serve as a variable type.

## Rendering

### ERenderMode

Render mode. Used by `ULCCComponentBase::RenderMode`.

| Value | Panel display | Description |
|---|---------|------|
| `Splatting` | 3DGS | Renders as 3D Gaussian Splatting, high quality, high cost |
| `PointCloud` | Point Cloud | Renders as a point cloud, low quality, low cost |

Switching to point cloud lowers the cost substantially and suits top-down views, minimaps, and distant views where detail is not needed. In point cloud mode the 3DGS-specific parameters (spherical harmonics, `SplatScale`, `GlobalAlpha`) have no effect.

Related interfaces: [SetRenderMode](./02-ULCCComponentBase.md#setrendermode--getrendermode), [ToggleRenderMode](./02-ULCCComponentBase.md#togglerendermode)

### ELoadMode

Load filter mode. Used by `ULCCComponentBase::LoadMode`.

| Value | Panel display | Description |
|---|---------|------|
| `Both` | Both | Renders the main part and the environment |
| `OnlyMain` | OnlyMain | Renders the main part only |
| `OnlyEnvironment` | OnlyEnvironment | Renders the environment only |
| `None` | None | Renders nothing |

The main part is the core captured area, and the environment is supplementary surrounding data. `None` amounts to a temporary hide while the data stays in memory, so recovery is far faster than a reload.

Related interface: [SetLoadMode](./02-ULCCComponentBase.md#setloadmode--getloadmode)

### ELightMode

Light mode. Used by `ULCCComponentBase::LightMode`.

| Value | Panel display | Description |
|---|---------|------|
| `Unlit` | Unlit | Unaffected by scene lighting, shows the original colors as captured |
| `Lit` | Lit | Takes part in scene lighting and can interact with dynamic lights |

Captured data already has the on-site lighting baked in, so switching to `Lit` adds another layer and easily overexposes. LCC2 needs `LightingScale` to lower the original brightness. Only effective in 3DGS mode.

Related interfaces: [SetLightMode](./02-ULCCComponentBase.md#setlightmode--getlightmode), [ToggleLightMode](./02-ULCCComponentBase.md#togglelightmode)

### EShadowMode

Shadow mode.

| Value | Panel display | Description |
|---|---------|------|
| `None` | None | Receives no shadows |
| `Static` | Static | Receives static shadows |
| `Dynamic` | Dynamic | Receives dynamic shadows |

### EColorationMode

Coloration mode.

| Value | Panel display | Description |
|---|---------|------|
| `None` | None | Uses the tint only |
| `Data` | Data | Uses the imported RGB |
| `Elevation` | Evelation | Colors by elevation, overriding the original colors |
| `Position` | Position | Colors by relative position, overriding the original colors |

`Elevation` works with `ElevationColorBottom` and `ElevationColorTop` for terrain height visualization.

### ELCC2NormalGenerationMode

LCC2 normal generation mode. Used by `ULCC2Component::NormalMode`.

3DGS data itself has no geometric normals, while lighting needs them. The first three modes are approximate constructions, and only `ProxyMesh` provides real normals.

| Value | Panel display | Real shading | Description |
|---|---------|---------|------|
| `Fixed` | Fixed | No | The whole 3DGS shares one user-specified world normal, pointing up by default. The brightness of everything equals `dot(normal, light direction)` |
| `ViewFacing` | View Facing | No | Everything faces the camera and is lit evenly |
| `Hemispherical` | Hemispherical | No | Camera-independent normals, mapped from screen position onto a fixed world hemisphere |
| `ProxyMesh` | Proxy Mesh | Yes | Real depth and normals come from an `ALCC2ProxyMesh` in the scene, and 3DGS contributes color only |

Trade-offs of each mode:

| Mode | Advantages | Disadvantages |
|------|------|------|
| `Fixed` | No diagonal light band, fully stable as the camera moves, lowest cost | No shape shading at all; everything darkens when light comes from behind the normal |
| `ViewFacing` | No configuration needed, no diagonal light band | No shape shading; overall brightness drifts as the camera turns |
| `Hemispherical` | No diagonal light band, stable as the camera moves, smoothest transition as sunlight sweeps across | Still not real shading; requires tuning `NormalPole` and `NormalHemiSpread` |
| `ProxyMesh` | Real geometric normals and depth | Requires authoring and placing a matching proxy mesh; requires a license and falls back to `Fixed` without one |

Recommendation: `Fixed` by default; `Hemispherical` for smooth sunlight movement; `ProxyMesh` for real shape shading; `ViewFacing` only for a quick look at the result.

Related pages: [ULCC2Component](./04-ULCC2Component.md#normalmode), [ALCC2ProxyMesh](./06-ALCC2ProxyMesh.md)

## Clipping and Section

### EClipType

Clipping direction. Used by `ALCCClippingVolume::Mode`.

| Value | Panel display | Description |
|---|---------|------|
| `Inside` | Inside | The inside of the volume is clipped away |
| `Outside` | Outside | The outside of the volume is clipped away |

`Inside` is for cutting holes and opening up a shell; `Outside` is for keeping only the area of interest.

### EClipVolumeType

Clipping volume shape. Used by `ALCCClippingVolume::VolumeType`.

| Value | Description |
|---|------|
| `Box` | Box volume |
| `Sphere` | Sphere volume |

### ESectionType

Section direction. Used by `ALCCSectionPlane::Mode`.

| Value | Panel display | Description |
|---|---------|------|
| `Upside` | Upside | What is above the plane is cut away |
| `Downside` | Downside | What is below the plane is cut away |

Up and down are relative to the orientation of the plane itself, not to world coordinates. Rotating the section plane gives a section in any direction.

Related pages: [ALCCClippingVolume](./07-ALCCClippingVolume.md), [ALCCSectionPlane](./08-ALCCSectionPlane.md)

## Data and Format

### ELCCVersion

Data version.

| Value | Description |
|---|------|
| `LCC` | LCC1 format |
| `LCC2` | LCC2 format |

Related interface: `ULCCComponentBase::GetLccVersion`

### EFileFormat

File format. Return type of `ULCCUtilLibrary::DetermineFileFormat`.

| Value | Matching extension |
|---|------|
| `None` | Unrecognized, or the path does not exist |
| `LCC` | `.lcc` |
| `Splats` | `.splats` |
| `LAS` | `.las` |
| `PLY` | `.ply` |

Note that the detection scope of `DetermineFileFormat` is limited: **`.lcc2`, `.sog`, and `.spz` all return `None`**. Check the extension directly for those formats, see [DetermineFileFormat](./09-ULCCUtilLibrary.md#determinefileformat) for details.

### EDataSourceType

Data source format of an LCC2 tree. Used by `FLCC2MetaInfo::SplatType`.

| Value | Description |
|---|------|
| `SOG` | `.sog` data source |
| `SPZ` | `.spz` data source |
| `PLY` | `.ply` data source |

This is a plain C++ enum without `UENUM` markup and is not visible in Blueprint.

### EFileType

Data content type.

| Value | Panel display | Description |
|---|---------|------|
| `Portable` | Portable | RGB only, no spherical harmonics |
| `Quality` | Quality | Both RGB and spherical harmonics |

`Portable` data has no spherical harmonics available, and `bUseShcoef` is not editable then. `CanSetShcoef()` actually just checks whether the type is `Quality`.

### ELCCSourceType

Data source. Return type of `ULCCUtilLibrary::DetermineSourceType`.

| Value | Description |
|---|------|
| `Local` | Local file |
| `Http` | Network address |

### ECollisionType

Collision data format. Return type of `ULCCUtilLibrary::DetermineCollisionType`.

| Value | Description |
|---|------|
| `None` | No collision data |
| `Bin` | Legacy `.bin` format |
| `Lci` | Newer `.lci` format |
| `Ply` | Point cloud `.ply` collision |

When `None` is returned, enabling `bEnableCollision` has no effect.

### ELoadStatus

Load status.

| Value | Description |
|---|------|
| `Success` | Succeeded |
| `Failure` | Failed |
| `Progress` | In progress |

## Performance and Scheduling

### ETraversalType

Node traversal method. Used by Default Traversal Type in the project settings (Project Settings → Plugins → LCC4Unreal).

| Value | Panel display | Description |
|---|---------|------|
| `Circle` | Circle | Circular traversal centered on the camera with the maximum render distance as the radius |
| `Sector` | Frustum | Traversal based on the camera frustum |

`Frustum` only processes nodes inside the view and is usually cheaper, but nodes outside the view are not preloaded, so a fill-in delay can appear while turning the view quickly.

Note that the enum name is `Sector` while the panel shows Frustum.

### ESortMethod

Translucency sorting method. Used by Default Sort Method in the project settings.

| Value | Description |
|---|------|
| `GPU` | Sorts on the GPU |
| `CPU` | Sorts on the CPU |

GPU is the normal choice.

### ESortMode

Sort order.

| Value | Description |
|---|------|
| `Ascending` | Ascending |
| `Descending` | Descending |

### EComputeType

Compute pipeline type.

| Value | Description |
|---|------|
| `Compute` | Uses Compute on the graphics pipeline |
| `AsyncCompute` | Uses the async Compute pipeline |

The current implementation uses `Compute`; `AsyncCompute` has rendering issues and is not enabled yet.

## Localization

### ELocale

Locale. Return type of `ULCCUtilLibrary::GetLocale`.

| Value | Description |
|---|------|
| `EN_US` | English |
| `ZH_CN` | Simplified Chinese |

### ELanguagePreference

Language preference of the plugin interface. Used by Language in the project settings.

| Value | Panel display | Description |
|---|---------|------|
| `FollowEditor` | Follow Editor | Follows the current editor language setting |
| `AlwaysEnglish` | Always English | Forces English regardless of the editor language |

## Using Enums in Blueprint

A Switch node can branch on an enum in Blueprint:

```text
[Get Render Mode]
        Target = (LCC Component)
        │ Return Value ──┐
        ▼                │
[Switch on ERenderMode] ◀┘
        │ Splatting  ──▶ (3DGS branch logic)
        │ Point Cloud ─▶ (point cloud branch logic)
```

A direct comparison works as well:

```text
[Get Effective Normal Generation Mode]
        │ Return Value ──┐
        ▼                │
[Equal (Enum)] ◀─────────┘
        A = (Return Value)
        B = Proxy Mesh
        │
        ▼
[Branch]
```

## See Also

- [ULCCComponentBase](./02-ULCCComponentBase.md): usage of `ERenderMode`, `ELoadMode`, and `ELightMode`
- [ULCC2Component](./04-ULCC2Component.md): usage of `ELCC2NormalGenerationMode`
- [ULCCUtilLibrary](./09-ULCCUtilLibrary.md): utility functions returning the format enums
- [Structs](./11-structs.md): the structs that use these enums
