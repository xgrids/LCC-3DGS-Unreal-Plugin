---
title: ALCCClippingVolume
parent: API Reference
grand_parent: v3.3.1 (latest)
nav_order: 7
description: ALCCClippingVolume clips a 3DGS scene with a box or sphere volume, supports inside and outside modes, and can be created and adjusted at runtime.
---

# ALCCClippingVolume: 3DGS Clipping Volume

Clipping volume Actor that clips away part of an LCC scene with a box or sphere volume.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `Tools/LCCClippingVolume.h` |
| Parent class | `AVolume` |
| Panel display name | LCC Clipping Volume |

```cpp
#include "Tools/LCCClippingVolume.h"
```

For the usage steps, a demonstration, and the property panel description, see [Scene Editing - Clipping Volumes](../09-scene-editing.md#clipping-volumes). This page only lists the interfaces.

## Properties

| Property | Type | Description |
|------|------|------|
| `bEnabled` | `bool` | Whether it takes part in clipping |
| `Mode` | `EClipType` | `Inside` clips away the inside of the volume, `Outside` clips away the outside |
| `VolumeType` | `EClipVolumeType` | `Box` for a box, `Sphere` for a sphere |

For the meaning of the values see [EClipType](./10-enums.md#ecliptype) and [EClipVolumeType](./10-enums.md#eclipvolumetype).

None of the three properties has a Setter, and **an assignment at runtime requires a call to [Refresh](#refresh) to take effect**. The same applies to a transform change at runtime.

## Methods

### SetUpdateComponent

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetUpdateComponent(ULCCComponentBase* Component);
```

Binds the LCC Component to update.

| Parameter | Type | Description |
|------|------|------|
| `Component` | `ULCCComponentBase*` | Target Component |

`Refresh()` needs this binding to take effect, since internally it marks the bound Component as needing its render state rebuilt. Without a binding, `Refresh()` triggers no render update.

Adding a volume with `AddClippingVolume` usually establishes the association already; when a change to a clipping volume created dynamically at runtime has no effect, check whether this method was called first.

### Refresh

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void Refresh();
```

Triggers one update of the clipping result manually; internally it marks the bound LCC Component as needing its render state rebuilt.

**A call is required after changing a clipping volume at runtime for the change to take effect**, including:

- Property changes: `bEnabled`, `Mode`, `VolumeType`. These properties have no Setter, and a direct assignment does not notify the rendering side.
- Transform changes: `SetActorLocation`, `SetActorRotation`, `SetActorScale3D`.

Automatic updates only happen in the editor: property changes in the Details panel go through `PostEditChangeProperty`, and dragging the gizmo goes through functions such as `EditorApplyTranslation`. Those paths are all under `WITH_EDITOR` and do not exist at runtime.

It requires [SetUpdateComponent](#setupdatecomponent) to have been called, or the association to have been established through `AddClippingVolume`. No render update is triggered while no Component is bound.

When changing several clipping volumes at once, call `Refresh()` on each one after its properties are changed.

## Runtime Creation

```cpp
#include "Tools/LCCClippingVolume.h"
#include "LCCComponentBase.h"

ALCCClippingVolume* AMyTool::CreateBoxClip(
    ULCCComponentBase* Component, const FVector& Location, const FVector& Extent)
{
    ALCCClippingVolume* Volume = GetWorld()->SpawnActor<ALCCClippingVolume>(
        ALCCClippingVolume::StaticClass(), Location, FRotator::ZeroRotator);
    if (!Volume)
    {
        return nullptr;
    }

    Volume->VolumeType = EClipVolumeType::Box;
    Volume->Mode = EClipType::Inside;
    Volume->bEnabled = true;

    // the size of an AVolume is controlled through scaling
    Volume->SetActorScale3D(Extent / 100.0f);

    Volume->SetUpdateComponent(Component);
    Component->AddClippingVolume(Volume);

    return Volume;
}
```

To toggle the clipping effect temporarily, do not add and remove array elements; changing `bEnabled` is lighter. Remember to call `Refresh()` after the change:

```cpp
for (ALCCClippingVolume* Volume : MyVolumes)
{
    if (Volume)
    {
        Volume->bEnabled = bEnable;
        Volume->Refresh();
    }
}
```

Moving a clipping volume at runtime for an animation also requires a refresh every time:

```cpp
// sweep the clipping volume across the scene
FVector Loc = ClipVolume->GetActorLocation();
Loc.X += 200.0f * DeltaTime;
ClipVolume->SetActorLocation(Loc);
ClipVolume->Refresh();          // runtime transform changes do not update automatically
```

For the Blueprint nodes see [Scene Editing - Blueprint Methods](../09-scene-editing.md#blueprint-methods).

## Notes

- Clipping only affects rendering, not collision. A clipped-away area can still be hit by a ray and still blocks a character.
- Several clipping volumes acting at once are evaluated independently and their effects stack. Mixing `Inside` and `Outside` easily produces unexpected results.
- **The number of clipping volumes in effect at once is limited, to 50 without a license**; the extras do not take part in rendering and a warning is logged. Clipping volumes and section planes share this quota. For licensing see [Editions and Licensing](../05-pro-features.md).
- Destroying a clipping volume only triggers one render update and **does not remove the element from the `ClippingVolumes` array of the Component**, leaving a stale pointer in the array (skipped automatically during rendering). To keep the array clean, call `RemoveClippingVolume` before destroying.

## See Also

- [Scene Editing](../09-scene-editing.md): full usage description
- [ALCCSectionPlane](./08-ALCCSectionPlane.md): sectioning with a plane
- [ULCCComponentBase](./02-ULCCComponentBase.md#addclippingvolume): `AddClippingVolume` and `RemoveClippingVolume`
