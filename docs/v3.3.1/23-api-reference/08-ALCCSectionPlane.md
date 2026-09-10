---
title: ALCCSectionPlane
parent: API Reference
grand_parent: Documentation v3.3.1
nav_order: 8
description: ALCCSectionPlane sections a 3DGS scene with a plane and keeps one side, suitable for viewing the interior of a building or producing a layer-by-layer reveal effect.
---

# ALCCSectionPlane: 3DGS Section Plane

Section plane Actor that cuts an LCC scene with a plane and keeps one side of it.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `Tools/LCCSectionPlane.h` |
| Parent class | `AVolume` |
| Panel display name | LCC Section Plane |

```cpp
#include "Tools/LCCSectionPlane.h"
```

For the usage steps, a demonstration, and the property panel description, see [Scene Editing - Section Planes](../09-scene-editing.md#section-planes). This page only lists the interfaces.

The editor shows an arrow component indicating the plane orientation, and the side the arrow points to is `Upside`.

## Properties

| Property | Type | Description |
|------|------|------|
| `bEnabled` | `bool` | Whether it takes part in sectioning |
| `Mode` | `ESectionType` | `Upside` cuts away what is above the plane, `Downside` cuts away what is below |

Up and down are relative to the orientation of the plane itself, not to world coordinates. Rotating the section plane gives a section in any direction. For the meaning of the values see [ESectionType](./10-enums.md#esectiontype).

Neither property has a Setter, and **an assignment at runtime requires a call to [Refresh](#refresh) to take effect**. The same applies to a transform change at runtime.

## Methods

### SetUpdateComponent

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void SetUpdateComponent(ULCCComponentBase* Component);
```

Binds the LCC Component to update.

`Refresh()` needs this binding to take effect, since internally it marks the bound Component as needing its render state rebuilt. Without a binding, no render update is triggered.

### Refresh

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
void Refresh();
```

Triggers one update of the sectioning result manually; internally it marks the bound LCC Component as needing its render state rebuilt.

**A call is required after changing a section plane at runtime for the change to take effect**, including:

- Property changes: `bEnabled`, `Mode`. These properties have no Setter, and a direct assignment does not notify the rendering side.
- Transform changes: `SetActorLocation`, `SetActorRotation`.

Automatic updates only happen in the editor: property changes in the Details panel go through `PostEditChangeProperty`, and dragging the gizmo goes through functions such as `EditorApplyTranslation`. Those paths are all under `WITH_EDITOR` and do not exist at runtime.

It requires [SetUpdateComponent](#setupdatecomponent) to have been called, or the association to have been established through `AddSectionPlane`. No render update is triggered while no Component is bound.

When changing several section planes at once, call `Refresh()` on each one after its properties are changed.

## Runtime Creation

```cpp
#include "Tools/LCCSectionPlane.h"
#include "LCCComponentBase.h"

ALCCSectionPlane* AMyTool::CreateSectionPlane(
    ULCCComponentBase* Component, const FVector& Location, const FRotator& Rotation)
{
    ALCCSectionPlane* Plane = GetWorld()->SpawnActor<ALCCSectionPlane>(
        ALCCSectionPlane::StaticClass(), Location, Rotation);
    if (!Plane)
    {
        return nullptr;
    }

    Plane->Mode = ESectionType::Upside;   // cut away what is above the plane
    Plane->bEnabled = true;

    Plane->SetUpdateComponent(Component);
    Component->AddSectionPlane(Plane);

    return Plane;
}
```

A few common configurations:

| Effect | Location and rotation | Mode |
|------|-----------|------|
| Top-down view of a building interior | A given height, `FRotator::ZeroRotator` | `Upside` |
| Vertical section to view a facade | `FRotator(0, 0, 90)` to make the plane vertical | `Upside` |
| Keep only the middle layer | One plane above and one below | `Downside` for the lower bound, `Upside` for the upper bound |

Moving the section plane produces a layer-by-layer reveal animation. Call `Refresh()` after every move at runtime:

```cpp
FVector Loc = SectionPlane->GetActorLocation();
Loc.Z = FMath::FInterpTo(Loc.Z, TargetHeight, DeltaTime, 1.0f);
SectionPlane->SetActorLocation(Loc);
SectionPlane->Refresh();        // runtime transform changes do not update automatically
```

Property changes need a refresh as well:

```cpp
SectionPlane->bEnabled = false;
SectionPlane->Refresh();
```

For the Blueprint nodes see [Scene Editing - Blueprint Methods](../09-scene-editing.md#blueprint-methods-1).

## Notes

- Sectioning only affects rendering, not collision. A sectioned-away part can still be hit by a ray and still blocks a character.
- `Upside` and `Downside` are relative to the orientation of the plane itself. After a rotation, confirm which side is "up" again; looking at the arrow in the editor is the most direct way.
- The Z scale is set to 0.01 at construction (a thin slice), and changing the scale in the editor is forced back to that value. A runtime scale change is not subject to this constraint, but normally needs no change.
- **The number of section planes in effect at once is limited, to 50 without a license**; the extras do not take part in rendering and a warning is logged. Section planes and clipping volumes share this quota. For licensing see [Editions and Licensing](../05-pro-features.md).
- Destroying a section plane only triggers one render update and **does not remove the element from the `SectionPlanes` array of the Component**, leaving a stale pointer in the array (skipped automatically during rendering). To keep the array clean, call `RemoveSectionPlane` before destroying.

## See Also

- [Scene Editing](../09-scene-editing.md): full usage description
- [ALCCClippingVolume](./07-ALCCClippingVolume.md): clipping with a bounded volume
- [ULCCComponentBase](./02-ULCCComponentBase.md#addsectionplane): `AddSectionPlane` and `RemoveSectionPlane`
