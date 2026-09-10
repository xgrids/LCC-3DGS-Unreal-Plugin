---
title: ULCC2Component
parent: API Reference
grand_parent: Documentation v3.3.1
nav_order: 4
description: ULCC2Component is the Component specific to the LCC2 pipeline, providing spherical harmonics band control, lighting normal modes, and depth threshold settings, and serving the .lcc2, .sog, .spz, and .ply formats.
---

# ULCC2Component: LCC2 Pipeline Component

Component specific to the LCC2 pipeline, inheriting [ULCCComponentBase](./02-ULCCComponentBase.md). It adds spherical harmonics band control, a lighting normal system, and depth threshold control.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `LCC2Component.h` |
| Parent class | `ULCCComponentBase` |
| Held by | `ALCC2Actor`, `ASogActor`, `ASpzActor`, `APlyActor` |

```cpp
#include "LCC2Component.h"
```

Get the instance:

```cpp
ULCC2Component* Comp = Cast<ULCC2Component>(LCC2Actor->GetLCCComponent());
```

This component serves four data formats at once: the LCC2 directory, `.sog`, `.spz`, and `.ply`. The latter three reuse the same rendering pipeline by constructing virtual metadata, so the Component obtained is always `ULCC2Component`.

> Note: the multi-viewport interfaces on the base class (`SetPlayerLoadMode`, `SetPlayerRenderMode`, `SetSceneCaptureLoadMode`, `SetSceneCaptureRenderMode`, `ModifyPlayerTransform`, `CancelModifyPlayerTransform`) **do not apply to the LCC2 pipeline**; calls raise no error but do not produce the expected result. See [ULCCComponent](./03-ULCCComponent.md#multi-viewport) for details.

## Properties

| Property | Type | Default | Range | Description |
|------|------|------|------|------|
| `SHBands` | `int32` | 3 | 1~3 | Spherical harmonics bands. Defaults to the band count stored in the data, which is also the upper limit |
| `LightingScale` | `float` | 0.0 | 0~1 | Brightness of the original 3DGS color before scene lighting is applied |
| `AlphaThreshold` | `float` | 0.5 | 0.01~0.99 | Accumulated opacity threshold for the median depth decision, an advanced item |
| `NormalMode` | `ELCC2NormalGenerationMode` | `Fixed` | — | Normal generation mode, deciding how 3DGS responds to scene light |
| `FixedNormal` | `FVector` | (0, 0, 1) | — | World normal shared by the whole 3DGS in `Fixed` mode |
| `NormalPole` | `FVector` | (1, 0, 1) | — | Surface orientation in `Hemispherical` mode |
| `NormalHemiSpread` | `float` | 0.5 | 0~4 | Strength of the shading variation in `Hemispherical` mode |

The normal properties carry `EditConditionHides`, so the Details panel only shows the items the current `NormalMode` uses.

Two more differences from the base class:

- `MetaInfo` (of type `FLCC2MetaInfo`) is a private member, not accessible from outside on the C++ side and only visible to Blueprint (read-only in the Details panel). In C++ use [GetMetaInfo2](#getmetainfo2) to read the complete fields.
- Construction rewrites the full load setting of `Performance` to "checkbox enabled, value `false`", the opposite of the `FRenderInfo` default. In other words, **LCC2 does not use full load rendering by default** and it has to be enabled explicitly when needed.

## Spherical Harmonics

### SetSHBands / GetSHBands

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetSHBands(int32 InSHBands);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetSHBands() const;
```

Sets how many spherical harmonics bands take part in the calculation, from 1 to 3. It applies to every format this component renders, including LCC2, SOG, SPZ, and PLY.

| Parameter | Type | Description |
|------|------|------|
| `InSHBands` | `int32` | Band count, 1 is cheapest, 3 is the most complete. Values beyond what the data stores have no effect |

Usage notes:

- The default comes from the loaded data and is also the upper limit. Setting 3 on data that stores only 2 bands adds nothing.
- Compared with turning spherical harmonics off entirely through `SetUseShcoef(false)`, lowering the band count is a finer trade-off.
- It requires `bUseShcoef` to be enabled and the render mode to be 3DGS, otherwise the value does not take part in the calculation.
- For the trade-off between quality and cost see [Visual Settings - SH Bands](../07-visual-settings.md#sh-bands).

```cpp
ULCC2Component* Comp = Cast<ULCC2Component>(Actor->GetLCCComponent());
if (Comp)
{
    // drop to 2 bands on mid-range devices
    Comp->SetSHBands(2);
}
```

Configuring by quality tier:

```cpp
void ConfigureSH(ULCC2Component* Comp, EQualityTier Tier)
{
    if (!Comp || !Comp->CanSetShcoef())
    {
        return;   // the data itself has no spherical harmonics, nothing to do
    }

    switch (Tier)
    {
    case EQualityTier::High:
        Comp->SetUseShcoef(true);
        Comp->SetSHBands(3);
        break;
    case EQualityTier::Medium:
        Comp->SetUseShcoef(true);
        Comp->SetSHBands(2);
        break;
    case EQualityTier::Low:
        Comp->SetUseShcoef(false);   // turn it off entirely
        break;
    }
}
```

## Lighting

### LightingScale

```cpp
UPROPERTY(Interp, EditAnywhere, BlueprintReadWrite, Category = "XGrids|Lighting",
    meta = (UIMin = "0.0", UIMax = "1.0", ClampMin = "0.0", ClampMax = "1.0"))
float LightingScale = 0.0f;
```

Base color brightness of 3DGS before scene lighting is applied, from 0 to 1, default 0. 0 means fully driven by scene lighting, 1 means the full original brightness is kept.

For the effect of the value and how to tune it, see [Normals and Lighting - LCC2 Lighting Scale](../08-normals-and-lighting.md#lcc2-lighting-scale).

Usage notes:

- Only meaningful while `LightMode` is `Lit`; it has no effect under `Unlit`.
- It is a `BlueprintReadWrite` property, assigned directly, with no Setter function.
- Marked `Interp`, so it can be keyframed in Sequencer for day-night changes.

```cpp
Comp->SetLightMode(ELightMode::Lit);
Comp->LightingScale = 0.3f;    // keep 30% of the original brightness, the rest from scene light
```

### NormalMode

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "XGrids|Lighting")
ELCC2NormalGenerationMode NormalMode = ELCC2NormalGenerationMode::Fixed;
```

Controls how the 3DGS surface responds to scene lighting. 3DGS data itself has no geometric normals, and the first three modes construct normals through approximation; only `ProxyMesh` provides real normals.

For a comparison of the four modes, their use cases, and how to tune them, see [Normals and Lighting](../08-normals-and-lighting.md#choosing-a-mode); for the value definitions see [Enums](./10-enums.md#elcc2normalgenerationmode).

```cpp
// Fixed mode, normal pointing up
Comp->NormalMode = ELCC2NormalGenerationMode::Fixed;
Comp->FixedNormal = FVector(0.0, 0.0, 1.0);

// Hemispherical mode, adjusting the direction the light sweeps and the contrast
Comp->NormalMode = ELCC2NormalGenerationMode::Hemispherical;
Comp->NormalPole = FVector(1.0, 0.0, 1.0);
Comp->NormalHemiSpread = 1.2f;
```

### FixedNormal

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "XGrids|Lighting",
    meta = (EditCondition = "NormalMode == ELCC2NormalGenerationMode::Fixed", EditConditionHides))
FVector FixedNormal = FVector(0.0, 0.0, 1.0);
```

`Fixed` mode only. World-space normal shared by the whole 3DGS, pointing up by default. Normalization is not required.

Usage notes:

- One normal shared by everything means the brightness of everything is a single value, equal to `dot(normal, light direction)`.
- Light arriving from behind the normal darkens everything, possibly to black. In an outdoor scene with the normal pointing up and the light source above, this is usually fine.
- When the main subject of the scene is a wall, setting the normal to the wall orientation is more reasonable.

### NormalPole

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "XGrids|Lighting",
    meta = (EditCondition = "NormalMode == ELCC2NormalGenerationMode::Hemispherical", EditConditionHides))
FVector NormalPole = FVector(1.0, 0.0, 1.0);
```

`Hemispherical` mode only. The direction the surface is treated as facing, which decides how the shading boundary moves across the 3DGS as the directional light rotates. Normalization is not required.

How to tune it: place the directional light first, then adjust this vector and watch whether the direction of the shading boundary matches expectations.

### NormalHemiSpread

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "XGrids|Lighting",
    meta = (UIMin = "0.0", UIMax = "4.0", ClampMin = "0.0",
        EditCondition = "NormalMode == ELCC2NormalGenerationMode::Hemispherical", EditConditionHides))
float NormalHemiSpread = 0.5f;
```

`Hemispherical` mode only. Strength of the shading variation across the surface, from 0 to 4, default 0.5.

A smaller value makes the shading flatter and more even; a larger value makes the contrast between the lit and shadowed areas stronger.

### AlphaThreshold

```cpp
UPROPERTY(Interp, EditAnywhere, BlueprintReadWrite, Category = "XGrids", AdvancedDisplay,
    meta = (UIMin = "0.01", UIMax = "0.99", ClampMin = "0.01", ClampMax = "0.99"))
float AlphaThreshold = 0.5f;
```

Accumulated opacity threshold for the median depth decision, from 0.01 to 0.99, default 0.5. An advanced item.

Opacity accumulates from near to far, and when the accumulated value reaches this threshold the current depth is recorded as the representative depth of that pixel. A smaller value biases the depth toward the camera side, a larger value toward the far side.

It affects occlusion between 3DGS and traditional meshes, shadow casting, and post-processing. The default suits most cases; for a description of the values see [Visual Settings - Depth Threshold](../07-visual-settings.md#depth-threshold).

### GetEffectiveNormalGenerationMode

```cpp
UFUNCTION(BlueprintPure, Category = "XGrids")
ELCC2NormalGenerationMode GetEffectiveNormalGenerationMode() const;
```

Returns the normal mode actually in effect after license validation.

Return value: the original value of `NormalMode` when licensed; `Fixed` when the license-restricted `ProxyMesh` is selected without a license.

Usage notes:

- The stored `NormalMode` property is not modified; only rendering follows the downgraded mode. Reading `NormalMode` and reading this function may therefore disagree.
- Use it for UI messaging. When the user selected `ProxyMesh` but `Fixed` is what actually runs, the interface should say so.

```cpp
const ELCC2NormalGenerationMode Requested = Comp->NormalMode;
const ELCC2NormalGenerationMode Effective = Comp->GetEffectiveNormalGenerationMode();

if (Requested != Effective)
{
    UE_LOG(LogTemp, Warning,
        TEXT("ProxyMesh mode requires a license, fell back to Fixed"));
}
```

Blueprint as text:

```text
[Get Effective Normal Generation Mode]
        Target = (LCC2 Component)
        │ Return Value ──┐
        ▼                │
[Equal (Enum)] ◀─────────┘
        A = (Return Value)
        B = Proxy Mesh
        │
        ▼
[Branch]
        │ False
        ▼
[Print String]  In String = "ProxyMesh requires a license, fell back to Fixed"
```

## Other Methods

### GetMetaInfo2

```cpp
FLCC2MetaInfo GetMetaInfo2();
```

Returns the complete LCC2 metadata. C++ only.

Compared with `GetMetaInfo()` on the base class, it adds the work path, data source type, spherical harmonics bands, octree root node, file list, and more. For the fields see [Structs](./11-structs.md#flcc2metainfo).

### GetLccVersion

```cpp
virtual ELCCVersion GetLccVersion() const override;
```

Always returns `ELCCVersion::LCC2`.

## See Also

- [Normals and Lighting](../08-normals-and-lighting.md): full description of the normal modes and lighting scale
- [Visual Settings](../07-visual-settings.md): values for SH bands, depth threshold, and other parameters
- [ULCCComponentBase](./02-ULCCComponentBase.md): common rendering and performance interfaces
- [ALCC2ProxyMesh](./06-ALCC2ProxyMesh.md): the companion Actor of the `ProxyMesh` normal mode
- [SOG / SPZ / PLY Actors](./05-sog-spz-ply-actors.md): these three formats use this component too
- [Enums](./10-enums.md#elcc2normalgenerationmode): normal mode values
