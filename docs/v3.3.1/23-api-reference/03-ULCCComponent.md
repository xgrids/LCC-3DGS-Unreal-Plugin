---
title: ULCCComponent
description: ULCCComponent is the Component specific to the LCC pipeline, providing point cloud ray tests, differentiated multi-viewport rendering, and seam cutting.
---

# ULCCComponent: LCC Pipeline Component

Component specific to the LCC1 pipeline, inheriting [ULCCComponentBase](./02-ULCCComponentBase.md). On top of the base class capabilities it adds ray tests based on splat positions, plus seam cutting.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `LCCComponent.h` |
| Parent class | `ULCCComponentBase` |
| Held by | `ALCCActor` |

```cpp
#include "LCCComponent.h"
```

Get the instance:

```cpp
ULCCComponent* Comp = Cast<ULCCComponent>(LCCActor->GetLCCComponent());
```

## Properties

| Property | Type | Default | Description |
|------|------|------|------|
| `bEnableSeamCutting` | `bool` | false | Enables seam cutting manually, see [Seam Cutting](../07-visual-settings.md#seam-cutting). LCC file version 5.0 and above needs no manual activation and enables it automatically |
| `LCCPerformance` | `FLCCRenderInfo` | — | LCC1-specific performance parameters, see the table below |

`MetaInfo` (of type `FLCCMetaInfo`) is a protected member, not accessible from outside on the C++ side and only visible to Blueprint (read-only in the Details panel). In C++ use `GetMetaInfo()` on the base class to read the basic fields.

### LCCPerformance

Also in the form of a checkbox plus a value, using the default from the project settings while the checkbox is off.

| Field | Type | Default | Checkbox | Description |
|------|------|------|------|------|
| `SortFactor` | `float` | 1 | `bCanSetSortFactor` | Sort frequency scale factor, from 0.2 to 5, see [Sort Factor](../10-performance-parameters.md#sort-factor) |
| `bAddExtraPreNodes` | `bool` | true | `bEnableExtraPreNodes` | Adds extra preload nodes, see [Add Extra Preload Nodes](../10-performance-parameters.md#add-extra-preload-nodes) |

```cpp
ULCCComponent* Comp = Cast<ULCCComponent>(LCCActor->GetLCCComponent());
if (Comp)
{
    // sort less often to save performance
    Comp->LCCPerformance.bCanSetSortFactor = true;
    Comp->LCCPerformance.SortFactor = 2.0f;

    // head turns are fast in VR, add preload nodes
    Comp->LCCPerformance.bEnableExtraPreNodes = true;
    Comp->LCCPerformance.bAddExtraPreNodes = true;
}
```

## Methods

### GetUseSeamCut

```cpp
bool GetUseSeamCut() const;
```

Whether the shader has seam cutting enabled. Returns `true` when either condition holds: the LCC file version is 5.0 or above, or `bEnableSeamCutting` was enabled manually. C++ only.

## Multi-Viewport

The multi-viewport interfaces are declared on the base class [ULCCComponentBase](./02-ULCCComponentBase.md#multi-viewport) but **only take effect on the LCC1 pipeline**. `SetPlayerLoadMode`, `SetPlayerRenderMode`, `SetSceneCaptureLoadMode`, `SetSceneCaptureRenderMode`, `ModifyPlayerTransform`, and `CancelModifyPlayerTransform` are only meaningful when called on a `ULCCComponent`.

The reason is that the per-view rendering strategy and node scheduling are consumed by the LCC1 rendering path, and LCC2 is not wired up yet. Calling them on `ULCC2Component` (including `ALCC2Actor` and `.sog` / `.spz` / `.ply`) raises no error but does not produce the expected result.

Differentiated rendering such as high quality in the main viewport and low quality in a minimap, or preloading before a teleport, currently requires LCC1 data.

## Raycast

> ⚠️ **This group of interfaces is not sufficiently tested and may not work as expected.** In production, use the "enable collision plus engine ray test" approach instead: load the collision data with [SetLCCCollisionEnable](./02-ULCCComponentBase.md#setlcccollisionenable) first, then test with engine interfaces such as `LineTraceByChannel`. See [Collision](../15-collision.md) for details.

LCC1 provides four ray test functions targeting splat positions. The test target is the point cloud data itself and no collision file is required.

The four functions fall into two groups, differing along two dimensions:

| | Single hit | Multiple hits |
|---|---|---|
| Default radius (10 cm) | `LineTraceSingle` | `LineTraceMulti` |
| Custom radius | `LineTraceSingleWithRadius` | `LineTraceMultiWithRadius` |

Debug parameters shared by all four:

| Parameter | Type | Description |
|------|------|------|
| `Start` | `const FVector&` | Ray start, in world coordinates |
| `End` | `const FVector&` | Ray end, in world coordinates |
| `bDrawDebugLine` | `bool` | Whether to draw the debug line |
| `Duration` | `float` | Debug line duration, in seconds |
| `Thickness` | `float` | Debug line thickness |

> Note: these four functions are `protected` on the C++ side and can be called as nodes directly on the Blueprint side. In C++ use them inside a subclass of `ULCCComponent`, or switch to the ray tests provided by the engine.

### Recommendation

**Prefer collision bodies plus engine ray tests.** From v0.6.0 on, once the data carries a collision file and collision is enabled, `LineTraceByChannel` returns a complete `FHitResult` (normal, material, hit component, and more), works seamlessly with the engine physics, AI, and navigation systems, and that path is thoroughly validated.

The dedicated interfaces here are only worth considering in the following cases, and require thorough testing of your own:

- The data has no collision file and switching to data with collision is not possible.
- The node information of the hit point is required (`FLCCSplat::NodeVector` gives X, Y, and Level).

### LineTraceSingle

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Raycast")
bool LineTraceSingle(const FVector& Start, const FVector& End, FLCCSplat& OutHit,
                     bool bDrawDebugLine, float Duration, float Thickness) const;
```

Single ray test, returning the nearest hit point. The test radius is fixed at 10 cm.

| Parameter | Type | Description |
|------|------|------|
| `OutHit` | `FLCCSplat&` | Output hit information, containing the world position `Location` and the node information `NodeVector` |

Returns `bool`: `true` on a hit.

```cpp
// inside a subclass of ULCCComponent
void UMyLCCComponent::PickFromCamera(const FVector& CamLocation, const FVector& CamForward)
{
    const FVector Start = CamLocation;
    const FVector End = Start + CamForward * 10000.f;

    FLCCSplat Hit;
    if (LineTraceSingle(Start, End, Hit, true, 2.0f, 2.0f))
    {
        UE_LOG(LogTemp, Log, TEXT("Hit at %s, node %s"),
            *Hit.Location.ToString(), *Hit.NodeVector.ToString());
    }
}
```

Blueprint as text, picking with a mouse click:

```text
[Input Action: Pick]
        │ Pressed
        ▼
[Get Player Camera Manager] ──▶ [Get Camera Location]  → Start
                            └─▶ [Get Actor Forward Vector]
                                        │
                                        ▼
                                [Multiply] (× 10000)
                                        │
                                        ▼
                                [Add] (Start + previous step) → End
        │
        ▼
[Line Trace Single]
        Target = (LCC Component)
        Start = Start
        End = End
        Draw Debug Line = true
        Duration = 2.0
        Thickness = 2.0
        │ Return Value ──┐
        ▼                │
[Branch] ◀───────────────┘
        │ True
        ▼
[Break LCCSplat]  ← wired to Out Hit
        Location → used to place a marker
```

### LineTraceSingleWithRadius

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Raycast")
bool LineTraceSingleWithRadius(const FVector& Start, const FVector& End, float Radius,
                               FLCCSplat& OutHit, bool bDrawDebugLine,
                               float Duration, float Thickness) const;
```

Single ray test with a configurable test radius.

| Parameter | Type | Description |
|------|------|------|
| `Radius` | `float` | Test radius, in centimeters |

Usage notes:

- A point cloud is discrete, and a radius that is too small easily passes through the gaps between points, so nothing is hit even while pointing straight at an object. Increase the radius in that case.
- A larger radius hits more easily but lowers precision, and the returned point may deviate from the position actually intended.
- Start from 20~50 cm for sparse data; 5~10 cm is enough for dense data.

```cpp
FLCCSplat Hit;
// large radius for a sparse point cloud, raising the hit rate
if (LineTraceSingleWithRadius(Start, End, 30.0f, Hit, false, 0.f, 0.f))
{
    SpawnMarkerAt(Hit.Location);
}
```

### LineTraceMulti

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Raycast")
bool LineTraceMulti(const FVector& Start, const FVector& End,
                    bool bDrawDebugLine, float Duration, float Thickness,
                    TArray<FLCCSplat>& OutHits) const;
```

Multi ray test, returning every hit point along the path. The test radius is fixed at 10 cm.

| Parameter | Type | Description |
|------|------|------|
| `OutHits` | `TArray<FLCCSplat>&` | Output of every hit point |

Usage notes:

- Use it for penetrating tests, for example measuring how many layers of structure a line passes through, or doing cross-section analysis.
- The number of hit points can be large, so avoid calling it every frame.

```cpp
TArray<FLCCSplat> Hits;
if (LineTraceMulti(Start, End, true, 3.0f, 1.0f, Hits))
{
    UE_LOG(LogTemp, Log, TEXT("Passed through %d splats"), Hits.Num());

    for (const FLCCSplat& Hit : Hits)
    {
        DrawDebugPoint(GetWorld(), Hit.Location, 8.f, FColor::Yellow, false, 3.f);
    }
}
```

### LineTraceMultiWithRadius

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Raycast")
bool LineTraceMultiWithRadius(const FVector& Start, const FVector& End, float Radius,
                              bool bDrawDebugLine, float Duration, float Thickness,
                              TArray<FLCCSplat>& OutHits) const;
```

Multi ray test with a configurable test radius. The parameters mean the same as above.

A larger radius increases the number of hit points noticeably, which suits statistical needs such as estimating the point density inside a cylindrical range.

## FLCCSplat

```cpp
USTRUCT(BlueprintType)
struct FLCCSplat
{
    FVector Location;         // world coordinates of the hit point
    FLCCVector NodeVector;    // node the hit point belongs to
};
```

The `X`, `Y`, and `Level` fields of `FLCCVector` locate the node in the octree. `Level` reflects the detail level currently loaded at that spot, which tells whether the data being looked at is fine or coarse.

For the complete fields see [Structs](./11-structs.md#flccsplat).

## See Also

- [ULCCComponentBase](./02-ULCCComponentBase.md): common rendering and performance interfaces
- [ULCC2Component](./04-ULCC2Component.md): the matching component of the LCC2 pipeline
- [Structs](./11-structs.md): fields of `FLCCSplat`, `FLCCVector`, and `FLCCRenderInfo`
