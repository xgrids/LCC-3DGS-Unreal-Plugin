---
title: ULCCComponentBase
parent: API Reference
grand_parent: Documentation v3.3.1
nav_order: 2
description: ULCCComponentBase is the base class of every 3DGS Component in LCC4Unreal, providing render mode, performance parameters, color adjustment, collision, GIS, and multi-viewport interfaces.
---

# ULCCComponentBase: 3DGS Component Base Class

Base class of every LCC Component, carrying rendering, color, performance, collision, and GIS capabilities. `ULCCComponent` (LCC1) and `ULCC2Component` (LCC2) inherit from it.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `LCCComponentBase.h` |
| Parent classes | `UPrimitiveComponent`, `IInterface_CollisionDataProvider` |

```cpp
#include "LCCComponentBase.h"
```

Get the instance through the Actor:

```cpp
ULCCComponentBase* Component = LCCActor->GetLCCComponent();
```

This page is an interface index describing the signature, parameters, and call notes of every method. For how to tune a parameter and its visual and performance consequences, see the matching topic document:

| To learn about | See |
|-------|-------|
| Render mode, spherical harmonics, anti-aliasing | [Rendering](../06-rendering.md) |
| Color, alpha, splat size | [Visual Settings](../07-visual-settings.md) |
| Normal modes, lighting | [Normals and Lighting](../08-normals-and-lighting.md) |
| Recommended performance parameter values | [Performance Parameters](../10-performance-parameters.md), [Performance Guide](../11-performance-guide.md) |
| Clipping and sectioning | [Scene Editing](../09-scene-editing.md) |
| Collision | [Collision](../15-collision.md) |
| Loading animation | [Loading Animation](../14-loading-animation.md) |

## Differences Between the Two Pipelines

Some properties and methods are declared on the base class but only one pipeline actually implements them. Confirm which pipeline the target data goes through before writing code.

| Interface | LCC1 (`ULCCComponent`) | LCC2 (`ULCC2Component`) |
|------|------------------------|-------------------------|
| `bReceiveShadows`, `EnableReceiveShadows`, `DisableReceiveShadows` | Supported | Not supported, hidden in the panel |
| `bEnableMultipleLCCActorAutoSort` | Supported | Not supported, hidden in the panel |
| `bUseCustomFOV`, `OverrideMainCameraFOV` | Supported | Not supported |
| Multi-viewport family (`SetPlayerLoadMode`, `ModifyPlayerTransform`, and others) | Supported | Not supported |
| Material switching of `SetLightMode` | Switches Lit / Unlit materials | Does not switch materials, lighting is handled inside the shader |
| `Performance.LevelFactor` | Affects the mapping between distance and Level | Acts as a screen-space error scale factor, a different mechanism |
| `Performance.bUseFullLoad` | Switch disabled by default, value is true | Changed at construction to switch enabled, value false |

## Properties

For properties with `BlueprintSetter`, dragging the value in the Details panel and calling the Setter in code go through the same path. Properties with `Interp` can be keyframed in Sequencer.

### Load Path

| Property | Type | Default | Description |
|------|------|------|------|
| `DefaultLoadPath` | `FString` | Empty | Load path. Once filled in the Details panel, the data loads automatically when the level starts. Absolute paths such as `D:\lcc\Tower\Tower.lcc`, relative paths resolved against `Content` such as `Tower/Tower.lcc`. **After a successful `Load()` this value is synchronized to the path actually loaded**, and `Refresh()` depends on it; `UnLoad()` clears it |

### Rendering Properties

| Property | Type | Default | Setter | Description |
|------|------|------|--------|------|
| `RenderMode` | `ERenderMode` | `Splatting` | `SetRenderMode` | Render as 3DGS or as point cloud |
| `LoadMode` | `ELoadMode` | `Both` | `SetLoadMode` | Render the main part, the environment, both, or neither |
| `LightMode` | `ELightMode` | `Unlit` | `SetLightMode` | Whether to take part in scene lighting |
| `SplatScale` | `float` | 1.0 | `SetSplatScale` | Splat quad size, from 0.001 to 1.0. 1.0 is already the upper limit |
| `GlobalAlpha` | `float` | 1.0 | `SetGlobalAlpha` | Overall 3DGS opacity, from 0 to 1 |
| `GlobalAlpha_PointCloud` | `float` | 0.2 | `SetGlobalAlpha_PointCloud` | Overall point cloud opacity, from 0 to 1 |
| `bUseShcoef` | `bool` | true | `SetUseShcoef` | Whether spherical harmonics are enabled. Not editable when the data carries no spherical harmonics |
| `bUseMipFilter` | `bool` | true | `SetUseMipFilter` | Anti-flicker filtering |
| `bCanSetShcoef` | `bool` | — | — | Criterion for whether the data carries spherical harmonics, `EditDefaultsOnly`, not accessible from Blueprint. For a runtime check use [CanSetShcoef()](#cansetshcoef) |
| `bAffectAntiAliasingMethod` | `bool` | true | — | When enabled, switches the anti-aliasing method automatically per project settings, see [Anti-aliasing](../07-visual-settings.md#anti-aliasing) |
| `bReceiveShadows` | `bool` | false | — | Shadow receiving, experimental. **LCC1 only**, and only in 3DGS mode |

### Performance Properties

The `Performance` field is of type `FRenderInfo`. Every value comes with an enable checkbox, and the built-in default is used while the checkbox is off. For the field list see [Structs](./11-structs.md#frenderinfo), and for recommended values see [Performance Parameters](../10-performance-parameters.md).

### Color Adjustment Properties

| Property | Type | Default | Slider range | Setter |
|------|------|------|------|--------|
| `Saturation` | `FVector4` | (1,1,1) | 0~2 | `SetSaturation` |
| `Contrast` | `FVector4` | (1,1,1) | 0~2 | `SetContrast` |
| `Gamma` | `FVector4` | (1,1,1) | 0~2 | `SetGamma` |
| `Offset` | `FVector4` | (0,0,0) | -1~1 | `SetOffset` |
| `ColorTint` | `FLinearColor` | White | — | `SetColorTint` |

The four components correspond to R, G, B, and overall, in that order. For a description of the color adjustment effects see [Visual Settings](../07-visual-settings.md).

Note that these ranges are only the slider ranges of the panel (`UIMin` / `UIMax`) and **there is no Clamp**. Values outside the range passed to a Setter in code are not truncated, and the result is on the caller. `SplatScale` and `GlobalAlpha`, by contrast, are really clamped.

### Point Cloud Coloring Properties

| Property | Type | Default | Setter |
|------|------|------|--------|
| `ElevationColorBottom` | `FLinearColor` | Blue | `SetElevationColorBottom` |
| `ElevationColorTop` | `FLinearColor` | Red | `SetElevationColorTop` |

### Collision Properties

| Property | Type | Default | Setter | Description |
|------|------|------|--------|------|
| `bEnableCollision` | `bool` | false | `SetLCCCollisionEnable` | Whether collision data is loaded. Requires the data itself to carry a collision file |

See [Collision](../15-collision.md) for details.

### Camera Properties

The following three items **only take effect on the LCC1 pipeline**, and the Details panel of an LCC2 component hides the latter two.

| Property | Type | Default | Description |
|------|------|------|------|
| `bUseCustomFOV` | `bool` | false | Whether to override the FOV of the first camera |
| `OverrideMainCameraFOV` | `float` | 90.0 | FOV value used for the override, from 5 to 180 |
| `bEnableMultipleLCCActorAutoSort` | `bool` | true | Sorts several LCC Actors in the same scene by distance and sets translucency priority, see [Multi-Actor Translucency Sorting](../07-visual-settings.md#multi-actor-translucency-sorting) |

### GIS Properties

| Property | Type | Default | Description |
|------|------|------|------|
| `bEnableGeoPlace` | `bool` | false | Whether the scene is placed by latitude and longitude |
| `GeoLocationOffset` | `FVector` | (0,0,0) | Position offset |
| `GeoMultiply` | `FVector` | (1,1,1) | Scale multiplier, an advanced item |

For the setup steps used together with Cesium, see [Third-party and Engine Plugin Integration](../12-integration.md#gis-cesium-integration).

### Clipping and Section Properties

| Property | Type | Description |
|------|------|------|
| `ClippingVolumes` | `TArray<TObjectPtr<ALCCClippingVolume>>` | Array of clipping volumes |
| `SectionPlanes` | `TArray<TObjectPtr<ALCCSectionPlane>>` | Array of section planes |

Both arrays are `BlueprintReadOnly`. Do not modify the elements directly; maintain them with [AddClippingVolume](#addclippingvolume) and the related methods.

### Animation Properties

The animation has two stages: the first stage waits for `FirstStageDelay` and scales from 0 to `AnimationMinScale`, and the second stage waits for `SecondStageDelay` and scales from `AnimationMinScale` to `SplatScale`.

| Property | Type | Default | Description |
|------|------|------|------|
| `bEnableAnimation` | `bool` | false | Enables the animation. The `SetEnableAnimation` setter also resets the timeline |
| `bInverseAnimation` | `bool` | false | Plays in reverse, contracting from far to the center, which is the disappearing effect |
| `InverseMaxRangeTime` | `float` | 30.0 | Time in seconds matching the initially visible radius of the reverse animation. Multiplied by `AnimationSpeed` to get the actual radius |
| `AnimationSpeed` | `float` | 100.0 | Animation speed |
| `AnimationMinScale` | `float` | 0.2 | Target scale of the first stage, from 0.0001 to 1.0 |
| `FirstStageDelay` | `float` | 0.0 | Delay of the first stage, in seconds |
| `SecondStageDelay` | `float` | 5.0 | Delay of the second stage, in seconds |
| `EnvironmentDelay` | `float` | 10.0 | Delay of the environment data, in seconds |
| `AnimationOriginOffset` | `FVector3f` | (0,0,0) | Offset of the animation origin |
| `FirstStageColor` | `FLinearColor` | Gold | Scan line color of the first stage, requires `bUseFirstStageColor` (on by default) |
| `SecondStageColor` | `FLinearColor` | Gold | Scan line color of the second stage, requires `bUseSecondStageColor` (on by default) |
| `ScanLineThickness` | `float` | 5.0 | Scan line width |

For the effect of each parameter and how to tune it, see [Loading Animation](../14-loading-animation.md).

## Loading and State

### Load

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
bool Load(FString LCCPath);
```

Loads data. This is what `ALCCActorBase::Load` eventually calls.

| Parameter | Type | Description |
|------|------|------|
| `LCCPath` | `FString` | Data file path, absolute or relative to `Content` |

Returns `bool`: `true` when path validation passes and the load process starts. Passing an empty path performs an unload and also returns `true`.

Usage notes:

- `true` only means the process started, not that the data is ready. The current version requires polling with [CheckIfLoaded](#checkifloaded); a load completion callback will be provided in a later version.
- When the path passed in matches the currently loaded path, the call returns immediately and nothing is loaded again.
- A successful load writes the path into `DefaultLoadPath`.

```cpp
ULCCComponentBase* Component = LCCActor->GetLCCComponent();
if (Component && Component->Load(TEXT("D:/Data/Tower/Tower.lcc")))
{
    UE_LOG(LogTemp, Log, TEXT("Load started"));
}
```

### UnLoad

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
virtual void UnLoad();
```

Unloads the data, releases node caches, GPU buffers, and collision bodies, and clears `DefaultLoadPath`.

Two things to keep in mind:

- Because `DefaultLoadPath` is cleared, calling `Refresh()` right after `UnLoad()` loads nothing.
- The metadata is **not** cleared. After unloading, `GetMetaInfo()` and `GetSplatNumber()` still return the data of the previous load, so do not use them to check whether the data is unloaded; use [CheckIfLoaded](#checkifloaded).

### Refresh

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void Refresh();
```

Reloads the current data. The implementation is `UnLoad()` plus `Load(DefaultLoadPath)`, so the cost matches a full reload.

Use it to read from disk again after the data files on disk have been replaced. To update the rendering for a single frame, use [ForceUpdate](#forceupdate).

### ForceUpdate

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void ForceUpdate();
```

Marks the next frame to force a scene update once, at very low cost.

The plugin skips node updates while neither the camera nor the rendering parameters change. When something affecting visibility was changed externally and the image did not follow, use this to push a frame. Do not call it every frame, which defeats the skip optimization.

### CheckIfLoaded

```cpp
UFUNCTION(BlueprintPure, Category = "XGrids")
virtual bool CheckIfLoaded() const;
```

Whether the metadata and index structure are established. `BlueprintPure`, so it is a pure node without execution pins in Blueprint.

Exact meaning: LCC1 checks whether the node manager is created, LCC2 checks whether the tree is created. `true` means **metadata parsing is complete and the metadata can be read and parameters configured safely**, but splat data is still streaming in per view and the image keeps filling in.

In other words, it does not mean "the image is complete". For cases that need to wait until the image is stable, this function gives no answer.

```cpp
if (Component->CheckIfLoaded())
{
    // safe to read metadata and configure parameters
}
```

Blueprint as text, polling every 0.2 seconds:

```text
[Event BeginPlay]
        │
        ▼
[Set Timer by Event]
        Time = 0.2
        Looping = true
        Event ──▶ [Custom Event: CheckLoaded]

[Custom Event: CheckLoaded]
        │
        ▼
[Get LCC Component] ──▶ [Check If Loaded]
                              │ Return Value ──┐
                              ▼                │
                        [Branch] ◀─────────────┘
                              │ True
                              ▼
                        [Clear and Invalidate Timer by Handle]
                              │
                              ▼
                        (configuration after loading completes)
```

### GetSplatNumber

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
int GetSplatNumber() const;
```

Returns the **total splat count** of the data, taken from the `TotalSplats` field of the metadata.

Usage notes:

- This is the inherent total of the data. It does not change as the camera moves and is **not the amount actually rendered in the current frame**.
- Returns 0 before the first load. But since `UnLoad()` does not clear the metadata, it still returns the previous value after unloading.
- To know the render load of the current frame, open the statistics panel with `Stats()` on the Actor and read the live data.

```cpp
if (Component->CheckIfLoaded())
{
    UE_LOG(LogTemp, Log, TEXT("Total splats in dataset: %d"),
        Component->GetSplatNumber());
}
```

### HaveValidSplatData

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
virtual bool HaveValidSplatData();
```

Whether valid splat data is available for rendering. Returns `false` when loading failed or the data is empty.

### HaveValidCollisionData

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
bool HaveValidCollisionData();
```

Whether the data carries a collision file.

Note that it **only checks the newer `collision.lci`**, not the legacy `collision.bin`, and not point cloud `.ply` collision. Collision loading itself supports all three formats, so a dataset carrying only `collision.bin` can make this function return `false` while `SetLCCCollisionEnable(true)` still loads successfully.

To check all three formats accurately, use [ULCCUtilLibrary::DetermineCollisionType](./09-ULCCUtilLibrary.md#determinecollisiontype) (pass the directory containing the data).

```cpp
if (Component->HaveValidCollisionData())
{
    Component->SetLCCCollisionEnable(true);
}
```

### CanRender

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
virtual bool CanRender() const;
```

Whether the conditions for rendering are currently met. The criteria are the visibility state of the component plus whether the data structure of the subclass is established (LCC1 checks the node manager, LCC2 checks the tree). Metadata validity is not involved.

### CanSetShcoef

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
bool CanSetShcoef() const;
```

Whether the data carries spherical harmonics coefficients. The implementation is equivalent to checking that the file type is `EFileType::Quality`.

Check it before calling [SetUseShcoef](#setuseshcoef--getuseshcoef), and use it to decide whether the spherical harmonics switch is greyed out in a UI.

### GetMetaInfo

```cpp
UFUNCTION(BlueprintPure, Category = "XGrids")
virtual FMetaInfoBase GetMetaInfo() const;
```

Returns the data metadata, including name, version, coordinate system, total Level count, total splat count, and more. For the fields see [Structs](./11-structs.md#fmetainfobase).

Read it after loading completes; the fields are zero values while the data is not ready. The return value is a base class slice, so use the dedicated methods on the subclasses for the complete LCC1/LCC2 fields.

```cpp
if (Component->CheckIfLoaded())
{
    const FMetaInfoBase Meta = Component->GetMetaInfo();
    UE_LOG(LogTemp, Log, TEXT("Name=%s Levels=%d RTK=%s"),
        *Meta.Name, Meta.TotalLevel, Meta.IsRTK() ? TEXT("yes") : TEXT("no"));
}
```

### GetLocalVisibleBounds

```cpp
UFUNCTION(BlueprintPure, Category = "XGrids")
virtual FBox GetLocalVisibleBounds() const;
```

Returns the visible bounding box of the model in **component local space**. Returns an invalid box (`FBox(ForceInit)`) when the data is unavailable, so check `IsValid` before use.

For world space, transform it with `GetComponentTransform()`. A common use is placing the camera automatically so the whole scene fits in view.

```cpp
const FBox LocalBounds = Component->GetLocalVisibleBounds();
if (LocalBounds.IsValid)
{
    const FBox WorldBounds =
        LocalBounds.TransformBy(Component->GetComponentTransform());

    const FVector Center = WorldBounds.GetCenter();
    const float Radius = WorldBounds.GetExtent().Size();
    // use Center and Radius to compute the viewing position
}
```

### GetLccVersion

```cpp
virtual ELCCVersion GetLccVersion() const;
```

Returns the data version, `ELCCVersion::LCC` or `ELCCVersion::LCC2`. C++ only.

## Rendering

For the visual effect and trade-offs of each parameter, see [Rendering](../06-rendering.md) and [Visual Settings](../07-visual-settings.md); this section only covers the interfaces.

### SetRenderMode / GetRenderMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetRenderMode(ERenderMode InRenderMode);

UFUNCTION(BlueprintPure, Category = "XGrids")
ERenderMode GetRenderMode() const;
```

Switches between 3DGS (`ERenderMode::Splatting`) and point cloud (`ERenderMode::PointCloud`).

In point cloud mode the 3DGS-specific items have no effect, and opacity comes from `GlobalAlpha_PointCloud` instead.

Matching panel parameter: [Render Mode](../07-visual-settings.md#render-mode).

```cpp
Component->SetRenderMode(ERenderMode::PointCloud);
```

Blueprint as text:

```text
[Input Action: ToggleView]
        │ Pressed
        ▼
[Get LCC Component]
        Target = LCCActor
        │ Return Value ──┐
        ▼                │
[Set Render Mode] ◀──────┘
        Target = (Return Value)
        In Render Mode = Point Cloud
```

### ToggleRenderMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void ToggleRenderMode();
```

Switches back and forth between 3DGS and point cloud without checking the current state.

### SetLoadMode / GetLoadMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetLoadMode(ELoadMode Mode);

UFUNCTION(BlueprintPure, Category = "XGrids")
ELoadMode GetLoadMode() const;
```

Controls whether the main data, the environment data, or both are rendered. For the values see [ELoadMode](./10-enums.md#eloadmode), and for the panel description see [Load Mode](../07-visual-settings.md#load-mode).

`ELoadMode::None` amounts to a temporary hide while the already loaded data stays in memory, so recovery is faster than with `UnLoad`.

```cpp
Component->SetLoadMode(ELoadMode::OnlyMain);
```

### SetLightMode / GetLightMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
virtual void SetLightMode(ELightMode InLightMode);

UFUNCTION(BlueprintPure, Category = "XGrids")
ELightMode GetLightMode() const;
```

Controls whether the data takes part in scene lighting, `ELightMode::Unlit` or `ELightMode::Lit`.

The two pipelines implement it differently:

- LCC1 does it by switching between Lit and Unlit materials.
- LCC2 does not switch materials; lighting is handled inside the shader. LCC2 therefore also needs `NormalMode` configured to get reasonable shading, see [ULCC2Component](./04-ULCC2Component.md#normalmode) and [Normals and Lighting](../08-normals-and-lighting.md).

Captured data already has the on-site lighting baked in, so switching to `Lit` easily overexposes. LCC2 can lower the original brightness with `LightingScale`.

**This Setter has no effect in point cloud mode**: the assignment is skipped internally and a warning is logged. To change the light mode, switch back to 3DGS first.

Matching panel parameter: [Light Mode](../07-visual-settings.md#light-mode).

### ToggleLightMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void ToggleLightMode();
```

Switches between `Unlit` and `Lit`.

### SetUseShcoef / GetUseShcoef

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetUseShcoef(bool InUseShcoef);

UFUNCTION(BlueprintPure, Category = "XGrids")
bool GetUseShcoef() const;
```

Toggles spherical harmonics. Spherical harmonics provide view-dependent color variation; with them off the color is fixed.

Usage notes:

- It requires the data to carry spherical harmonics, **checked with [CanSetShcoef](#cansetshcoef)**. `Portable` data has no spherical harmonics available and this item is not editable in the panel.
- **This Setter is silently ignored in point cloud mode.** Internally it requires the data to be of type `Quality` and the current mode not to be point cloud; the assignment is skipped when either condition fails.
- LCC2 can lower the band count instead of turning it off completely, see [SetSHBands](./04-ULCC2Component.md#setshbands--getshbands).
- For the panel description see [Spherical Harmonics (SH)](../07-visual-settings.md#spherical-harmonics-sh).

```cpp
if (Component->CanSetShcoef())
{
    Component->SetUseShcoef(false);
}
```

### ToggleShcoef

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void ToggleShcoef();
```

Toggles spherical harmonics, useful for comparing the effect.

### SetSplatScale / GetSplatScale

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetSplatScale(float InSplatScale);

UFUNCTION(BlueprintPure, Category = "XGrids")
float GetSplatScale() const;
```

Splat quad size, from 0.001 to 1.0, default 1.0.

Usage notes:

- **The default of 1.0 is the upper limit, so it can only be lowered.**
- Lowering it reduces overdraw and raises the frame rate, at the price of possible holes in the image once the quads get smaller.
- Only affects 3DGS mode.
- For the panel description see [SplatScale](../07-visual-settings.md#splatscale).

```cpp
// trade overdraw for frame rate, weighing the holes case by case
Component->SetSplatScale(0.8f);
```

### SetGlobalAlpha / GetGlobalAlpha

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetGlobalAlpha(float InGlobalAlpha);

UFUNCTION(BlueprintPure, Category = "XGrids")
float GetGlobalAlpha() const;
```

Overall 3DGS opacity, from 0 to 1. Marked `Interp`, so it can be keyframed in Sequencer for fades.

For point cloud mode use `SetGlobalAlpha_PointCloud`; the two values are independent. For the panel description see [Global Alpha](../07-visual-settings.md#global-alpha).

```cpp
// fade out frame by frame
const float Next = FMath::FInterpTo(
    Component->GetGlobalAlpha(), 0.0f, DeltaTime, 2.0f);
Component->SetGlobalAlpha(Next);
```

Blueprint as text, fading out with a Timeline:

```text
[Timeline: FadeOut]
        Length = 2.0
        Float Track "Alpha" = 1.0 → 0.0
        │ Update
        ▼
[Set Global Alpha]
        Target = (LCC Component)
        In Global Alpha = (Alpha output of the Timeline)
```

### SetGlobalAlpha_PointCloud / GetGlobalAlpha_PointCloud

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetGlobalAlpha_PointCloud(float InGlobalAlpha);

UFUNCTION(BlueprintPure, Category = "XGrids")
float GetGlobalAlpha_PointCloud() const;
```

Overall opacity in point cloud mode, from 0 to 1, default 0.2.

### SetUseMipFilter / GetUseMipFilter

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetUseMipFilter(bool InUseMipFilter);

UFUNCTION(BlueprintPure, Category = "XGrids")
bool GetUseMipFilter() const;
```

Toggles anti-flicker filtering, enabled by default. When enabled it uses a low-pass filter with opacity compensation and is more stable at different scales; when disabled the image is sharper but may alias and flicker. Only effective in 3DGS mode.

For the panel description see [Mip Filter](../07-visual-settings.md#mip-filter).

### EnableReceiveShadows / DisableReceiveShadows

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void EnableReceiveShadows();

UFUNCTION(BlueprintCallable, Category = "XGrids")
void DisableReceiveShadows();
```

Toggles shadow receiving, an experimental feature.

Usage notes:

- **Supported on the LCC1 pipeline only**; the Details panel of an LCC2 component hides the property.
- Only effective in 3DGS mode, with a significant performance impact.
- Implemented internally by switching to a dedicated material, so material setup runs again.
- For the panel description see [Shadow Receiving](../07-visual-settings.md#shadow-receiving).

## Performance

This group of Setters shares one behavior: **calling one automatically sets the matching enable checkbox to `true`**. The Getters return the effective value, and while the checkbox is off they return the built-in default rather than the value entered earlier.

`GetPreloadDistance` is the only exception: it returns the raw field value, not the effective value.

For recommended values see [Performance Parameters](../10-performance-parameters.md) and [Performance Guide](../11-performance-guide.md).

### SetMaxDistance / GetMaxDistance

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetMaxDistance(const int32 InDistance);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetMaxDistance() const;
```

Maximum render distance in meters, built-in default 300. Nodes beyond that distance are not rendered.

Matching panel parameter: [Max Distance](../10-performance-parameters.md#max-distance).

```cpp
Component->SetMaxDistance(80);
```

### SetMaxSplatNum / GetMaxSplatNum

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetMaxSplatNum(const int32 InSplatNum);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetMaxSplatNum() const;
```

Maximum splat count per frame, **in units of ten thousand**, upper limit 10000. Passing 1500 means 15 million.

Values beyond what the GPU can handle in a single frame are clamped automatically. Matching panel parameter: [Max Splat Num](../10-performance-parameters.md#max-splat-num).

```cpp
Component->SetMaxSplatNum(1500);   // 15 million
```

### SetLevelFactor / GetLevelFactor

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetLevelFactor(const float InLevelFactor);

UFUNCTION(BlueprintPure, Category = "XGrids")
float GetLevelFactor() const;
```

LOD scale factor, from 0.01 to 20, default 1. A larger value means less detail and better performance.

The mechanism differs per pipeline:

- LCC1: scales `RangeForLevel` from the project settings, changing the mapping between distance and Level.
- LCC2: acts as a scale factor of the screen-space error during node selection.

The visual effect of the same value therefore cannot be compared directly between the two pipelines; measure each one.

Matching panel parameter: [Level Factor](../10-performance-parameters.md#level-factor).

```cpp
Component->SetLevelFactor(1.5f);
```

### SetStartLevel / GetStartLevel

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetStartLevel(const int32 InStartLevel);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetStartLevel() const;
```

Start Level, from 0 to 20, default 0. Level 0 has the highest detail; raising it skips the finest levels and noticeably reduces video memory usage and load amount.

Matching panel parameter: [Start Level](../10-performance-parameters.md#start-level).

### SetEndLevel / GetEndLevel

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetEndLevel(const int32 InEndLevel);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetEndLevel() const;
```

End Level, from 0 to 20, default 20. It limits the coarsest Level and normally needs no change.

Matching panel parameter: [End Level](../10-performance-parameters.md#end-level).

### SetMaxCollisionDistance / GetMaxCollisionDistance

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetMaxCollisionDistance(const int32 InMaxCollisionDistance);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetMaxCollisionDistance() const;
```

Maximum collision load distance in meters, built-in default 300.

For long-range ray tests, this value has to cover the test range, otherwise nothing is hit.

Matching panel parameter: [Max Load Collision Distance](../10-performance-parameters.md#max-load-collision-distance).

### SetPreloadDistance / GetPreloadDistance

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetPreloadDistance(const int32 InPreloadDistance);

UFUNCTION(BlueprintPure, Category = "XGrids")
int32 GetPreloadDistance() const;
```

Preload distance in meters.

Note: in the current version the effective logic is fixed at `MaxDistance + 35`, the value set through the Setter does not take part in the calculation, and the item is not exposed in the panel. The interface is kept for compatibility and needs no calls in normal use.

## Color Adjustment

The color adjustment properties are all `FVector4`, with the four components corresponding to R, G, B, and overall, in that order. Every color adjustment Setter takes effect immediately. For a description of the effects see [Visual Settings](../07-visual-settings.md).

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Color")
void SetSaturation(const FVector4 InSaturation);   // saturation, 0~2
void SetContrast(const FVector4 InContrast);       // contrast, 0~2
void SetGamma(const FVector4 InGamma);             // gamma, 0~2
void SetOffset(const FVector4 InOffset);           // additive offset, -1~1
void SetColorTint(const FLinearColor InColor);     // multiplicative tint

// matching Getters
FVector4 GetSaturation() const;
FVector4 GetContrast() const;
FVector4 GetGamma() const;
FVector4 GetOffset() const;
FLinearColor GetColorTint() const;
```

`Offset` is additive and lifts the dark areas along with everything else; `ColorTint` is multiplicative and keeps the dark areas dark. For the effect of each item see [Color Adjustment](../07-visual-settings.md#color-adjustment).

```cpp
// lower saturation and brighten slightly
Component->SetSaturation(FVector4(0.6f, 0.6f, 0.6f, 0.6f));
Component->SetOffset(FVector4(0.05f, 0.05f, 0.05f, 0.f));
```

### Point Cloud Elevation Coloring

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|PointCloud")
void SetElevationColorBottom(FLinearColor InElevationColorBottom);

UFUNCTION(BlueprintCallable, Category = "XGrids|PointCloud")
void SetElevationColorTop(FLinearColor InElevationColorTop);

UFUNCTION(BlueprintPure, Category = "XGrids|PointCloud")
FLinearColor GetElevationColorBottom() const;

UFUNCTION(BlueprintPure, Category = "XGrids|PointCloud")
FLinearColor GetElevationColorTop() const;
```

Bottom and top colors of the elevation gradient, blue to red by default.

```cpp
Component->SetRenderMode(ERenderMode::PointCloud);
Component->SetElevationColorBottom(FLinearColor(0.0f, 0.2f, 1.0f, 1.0f));
Component->SetElevationColorTop(FLinearColor(1.0f, 0.1f, 0.0f, 1.0f));
```

## Collision

### SetLCCCollisionEnable

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetLCCCollisionEnable(const bool InEnable);
```

Toggles collision data loading.

| Parameter | Type | Description |
|------|------|------|
| `InEnable` | `bool` | `true` loads collision, `false` unloads the loaded collision bodies |

Usage notes:

- It requires the data to carry a collision file, confirmed with [HaveValidCollisionData](#havevalidcollisiondata) first.
- Collision streams in by distance, limited by `Performance.CollisionLoadMaxDistance`.
- Once enabled, engine features such as `LineTraceByChannel`, character movement, and physics simulation work directly.
- The first activation carries a one-off baking cost that may cause a brief stutter. Enable it during the loading stage where possible.
- `ShowCollision()` on the Actor shows the collision wireframe to confirm whether it was loaded.

For the full description see [Collision](../15-collision.md).

```cpp
if (Component->HaveValidCollisionData())
{
    Component->SetMaxCollisionDistance(50);
    Component->SetLCCCollisionEnable(true);
}
```

## GIS

Geo placement is usually used together with Cesium to place an LCC scene at real earth coordinates. For the setup steps, plugin dependencies, and caveats, see [Third-party and Engine Plugin Integration](../12-integration.md#gis-cesium-integration).

### SetGeoPlacement

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetGeoPlacement(bool InEnable);
```

Toggles placing the scene by latitude and longitude.

Once enabled, the scene is positioned automatically by real geographic coordinates, the manually set Actor location is overridden, and fine adjustment goes through `GeoLocationOffset`. Several LCC scenes with it enabled align automatically at their real relative positions.

Usage notes:

- **A `Refresh()` runs internally** (unload plus reload), so a call takes effect at any time and no manual `Load()` is needed. The cost matches a full reload, so do not toggle it frequently.
- It requires the data to carry RTK information. **Check with `GetMetaInfo().IsRTK()`, not `CanUseGeoPlace()`**: the latter requires geo placement to be enabled already and is always `false` before that.

```cpp
if (Component->CheckIfLoaded() && Component->GetMetaInfo().IsRTK())
{
    Component->SetGeoPlacement(true);   // reloads automatically
}
```

### CanUseGeoPlace

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
bool CanUseGeoPlace() const;
```

Whether geo placement is currently in a usable state. Three conditions must hold at the same time: `bEnableGeoPlace` is enabled, the data carries RTK information, and the geo referencing system is created.

**This is not a criterion for "whether the data supports geo placement".** Because it requires `bEnableGeoPlace` to be `true` already, a call before enabling always returns `false`. To check whether the data supports it before enabling, use `GetMetaInfo().IsRTK()`.

Its actual purpose is confirming after enabling that it really took effect.

### GetRTKBaseLocation

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
FVector GetRTKBaseLocation() const;
```

Returns the RTK base station position converted into engine space, which is the geographic origin of the data. It is the reference point when external geographic coordinates need to be converted into engine coordinates.

> Note: internally it uses the geo referencing system directly for the coordinate conversion **without a null check**. Calling it while the geo referencing system is not created crashes. Confirm [CanUseGeoPlace](#canusegeoplace) is `true`, or that [GetGeoReferencingSystem](#getgeoreferencingsystem) returns non-null, before calling.

### GetGeoReferencingSystem

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
ALCCGeoReferencingSystem* GetGeoReferencingSystem() const;
```

Returns the geo referencing system Actor in the scene, created automatically by the plugin when geo placement is enabled.

```cpp
ALCCGeoReferencingSystem* GeoSystem = Component->GetGeoReferencingSystem();
if (GeoSystem)
{
    // projected coordinates to engine coordinates
    FVector EngineLocation;
    GeoSystem->ProjectedToEngine(ProjectedCoord, EngineLocation);

    // east/north/up directions at that point
    FVector East, North, Up;
    GeoSystem->GetENUVectorsAtEngineLocation(EngineLocation, East, North, Up);
}
```

## Multi-Viewport

The same dataset can use different rendering strategies for different cameras. A typical use is high-quality 3DGS in the main viewport and a point cloud top-down view rendered through SceneCapture for a minimap.

> Note: this group of interfaces **applies to the LCC1 pipeline only**. Calling them on the LCC2 pipeline (`ALCC2Actor` and `.sog` / `.spz` / `.ply`) raises no error but does not produce the expected result. Use LCC1 data when differentiated multi-viewport rendering or teleport preloading is needed.

They take object pointers, not integer IDs.

For the overall approach to multiple cameras and multiple screen outputs (nDisplay, Aximmetry, Pixotope, and others), see [Third-party and Engine Plugin Integration](../12-integration.md#ndisplay-multi-viewport).

### SetPlayerLoadMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetPlayerLoadMode(class APlayerController* PlayerController, ELoadMode InLoadMode);
```

Sets a separate LoadMode for the given player controller, effective on the next frame.

```cpp
APlayerController* PC2 = UGameplayStatics::GetPlayerController(GetWorld(), 1);
Component->SetPlayerLoadMode(PC2, ELoadMode::OnlyMain);
```

### SetPlayerRenderMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetPlayerRenderMode(class APlayerController* PlayerController, ERenderMode InRenderMode);
```

Sets a separate RenderMode for the given player controller, effective on the next frame.

### SetSceneCaptureLoadMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetSceneCaptureLoadMode(class USceneCaptureComponent2D* InCapture2D, ELoadMode InLoadMode);
```

Sets a separate LoadMode for the given SceneCapture.

[SceneCaptureComponent Support](../10-performance-parameters.md#scenecapturecomponent-support) has to be enabled in the project settings first, otherwise LCC content is not rendered in SceneCapture.

### SetSceneCaptureRenderMode

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetSceneCaptureRenderMode(class USceneCaptureComponent2D* InCapture2D, ERenderMode InRenderMode);
```

Sets a separate RenderMode for the given SceneCapture.

```cpp
// minimap uses point cloud and the main part only, the cheapest option
Component->SetSceneCaptureRenderMode(MinimapCapture, ERenderMode::PointCloud);
Component->SetSceneCaptureLoadMode(MinimapCapture, ELoadMode::OnlyMain);
```

### ModifyPlayerTransform

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void ModifyPlayerTransform(class APlayerController* PlayerController, FTransform InTransform);
```

Overrides the position of the given player in this scene, used to preload the data of a destination before teleporting.

The problem it solves: teleporting straight to a distant place leaves the destination nodes unloaded, so the player sees a blank main part that gradually fills in.

Correct sequence:

```text
ModifyPlayerTransform(destination)
        ↓ wait around 0.2 seconds so the destination nodes start loading
teleport the player for real
        ↓ wait around another 0.2 seconds
CancelModifyPlayerTransform
```

```cpp
void AMyTeleporter::TeleportWithPreload(APlayerController* PC, const FTransform& Destination)
{
    Component->ModifyPlayerTransform(PC, Destination);

    FTimerHandle Handle;
    GetWorld()->GetTimerManager().SetTimer(Handle,
        [this, PC, Destination]()
        {
            PC->GetPawn()->SetActorTransform(Destination);

            FTimerHandle CancelHandle;
            GetWorld()->GetTimerManager().SetTimer(CancelHandle,
                [this, PC]()
                {
                    Component->CancelModifyPlayerTransform(PC);
                },
                0.2f, false);
        },
        0.2f, false);
}
```

Blueprint as text:

```text
[Custom Event: TeleportTo]
        Destination (Transform)
        │
        ▼
[Modify Player Transform]
        Target = (LCC Component)
        Player Controller = Get Player Controller
        In Transform = Destination
        │
        ▼
[Delay]  Duration = 0.2
        │
        ▼
[Set Actor Transform]
        Target = Get Player Pawn
        New Transform = Destination
        │
        ▼
[Delay]  Duration = 0.2
        │
        ▼
[Cancel Modify Player Transform]
        Target = (LCC Component)
        Player Controller = Get Player Controller
```

### CancelModifyPlayerTransform

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void CancelModifyPlayerTransform(class APlayerController* PlayerController);
```

Cancels the position override and returns to using the live player position.

It has to be called, otherwise node scheduling for that player stays stuck at the overridden position and no new nodes load as the player moves.

## Clipping and Section

For how to use clipping and sectioning, see [Scene Editing](../09-scene-editing.md), along with [ALCCClippingVolume](./07-ALCCClippingVolume.md) and [ALCCSectionPlane](./08-ALCCSectionPlane.md).

### AddClippingVolume

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void AddClippingVolume(ALCCClippingVolume* InClippingVolume);
```

Adds a clipping volume.

```cpp
ALCCClippingVolume* Volume = GetWorld()->SpawnActor<ALCCClippingVolume>(
    ALCCClippingVolume::StaticClass(), Location, FRotator::ZeroRotator);

Volume->VolumeType = EClipVolumeType::Box;
Volume->Mode = EClipType::Inside;
Volume->bEnabled = true;

Component->AddClippingVolume(Volume);
```

### RemoveClippingVolume

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void RemoveClippingVolume(ALCCClippingVolume* InClippingVolume);
```

Removes a clipping volume. For a temporary deactivation, changing `bEnabled` on the clipping volume is lighter.

### AddSectionPlane

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void AddSectionPlane(ALCCSectionPlane* InSectionPlane);
```

Adds a section plane.

### RemoveSectionPlane

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void RemoveSectionPlane(ALCCSectionPlane* InSectionPlane);
```

Removes a section plane.

## Loading Animation

### SetEnableAnimation / GetEnableAnimation

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Animation")
void SetEnableAnimation(bool bInEnableAnimation);

UFUNCTION(BlueprintPure, Category = "XGrids|Animation")
bool GetEnableAnimation() const;
```

Toggles the animation and **resets the animation timer origin to the current time**. It is therefore not limited to the loading stage: a call at any moment replays the animation from the start.

Parameters of each stage are set directly on the properties; there are no separate Setters. To make the parameters apply to the whole animation, set them before calling this function. Changing a parameter during playback also takes effect, as a mid-flight adjustment.

```cpp
Component->AnimationSpeed = 30.0f;
Component->SecondStageDelay = 1.0f;
Component->SetEnableAnimation(true);
```

For the reverse (disappearing) animation, call order, parameter values, and practical notes, see [Loading Animation](../14-loading-animation.md#controlling-the-animation-at-runtime).

## See Also

- [ALCCActorBase](./01-ALCCActorBase.md): the Actor-side interfaces
- [ULCCComponent](./03-ULCCComponent.md): the LCC1-only point cloud ray test
- [ULCC2Component](./04-ULCC2Component.md): the LCC2-only spherical harmonics and normal modes
- [Enums](./10-enums.md): values of `ERenderMode`, `ELoadMode`, `ELightMode`, and others
- [Structs](./11-structs.md): fields of `FRenderInfo` and `FMetaInfoBase`
