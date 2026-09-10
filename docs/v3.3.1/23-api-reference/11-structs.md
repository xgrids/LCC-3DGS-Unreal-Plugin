---
title: Structs
parent: API Reference
grand_parent: Documentation v3.3.1
nav_order: 11
description: Reference for the public struct types of LCC4Unreal, covering the fields of the FRenderInfo performance parameters, the FMetaInfoBase metadata, the FLCCSplat ray hit result, and others.
---

# Structs: Struct Type Reference

Struct types the `LCC4UnrealRuntime` module exposes.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `Core/LCCStructure.h` |

```cpp
#include "Core/LCCStructure.h"
```

Structs marked `BlueprintType` can be split and assembled in Blueprint with Break / Make nodes.

## Performance

### FRenderInfo

Common performance parameters. Used by `ULCCComponentBase::Performance`.

The design characteristic of this struct is that **every value comes with an enable checkbox**. While the checkbox is off the built-in plugin default is used, and the entered value only applies once it is on. The matching Setter on the Component sets the checkbox to `true` automatically.

| Field | Type | Default | Matching checkbox | Description |
|------|------|------|---------|------|
| `MaxDistance` | `uint32` | 300 | `bDistanceLimit` | Maximum render distance, in meters |
| `MaxSplatNum` | `uint32` | 3000 | `bMaxSplatNumLimit` | Maximum splat count per frame, in units of ten thousand, upper limit 10000 |
| `LevelFactor` | `float` | 1 | `bCanSetLevelFactor` | LOD distance scale factor, from 0.01 to 20 |
| `PreloadDistance` | `uint32` | 200 | `bPreloadDistanceLimit` | Preload distance. The actual value is `MaxDistance + 35` |
| `StartLevel` | `uint32` | 0 | `bCanSetStartLevel` | Start Level, from 0 to 20 |
| `EndLevel` | `uint32` | 20 | `bCanSetEndLevel` | End Level, from 0 to 20 |
| `CollisionLoadMaxDistance` | `uint32` | 300 | `bCollisionDistanceLimit` | Maximum collision load distance, in meters |
| `bUseFullLoad` | `bool` | true | `bCanSetUseFullLoad` | Full load switch for small scenes. **Rewritten to checkbox enabled, value `false` when `ULCC2Component` is constructed** |
| `FullLoadSplatNum` | `uint32` | 1500 | `bCanSetFullLoadNum` | Full load threshold, in units of ten thousand |

Helper methods:

```cpp
bool HasMaxDistanceLimit() const;      // whether a distance limit exists
bool HasMaxNumLimit() const;           // whether a count limit exists
bool NeedPreload() const;              // whether preloading is needed
uint32 GetMaxDistance() const;         // effective maximum distance
uint32 GetMaxSplatNum() const;         // effective maximum count
float GetLevelFactor() const;          // effective LOD factor
uint32 GetStartLevel() const;          // effective start Level
uint32 GetEndLevel() const;            // effective end Level
uint32 GetMaxCollisionDistance() const;
uint32 GetPreloadDistance() const;     // returns GetMaxDistance() + 35
bool GetIfUseFullLoad(uint32 SplatNum) const;   // whether the given count uses full load
void Reset();                          // resets the value fields only, not the enable checkboxes
```

Two easy pitfalls:

- `Reset()` resets the value fields only (distance, count, Level, factor) and **resets none of the `b*` enable checkboxes**, nor `bUseFullLoad` and `FullLoadSplatNum`.
- `FRenderInfo::GetPreloadDistance()` returns `GetMaxDistance() + 35`, while the function of the same name on the component, `ULCCComponentBase::GetPreloadDistance()`, returns the raw field value. The two results differ.

The Getters return the **effective value**: while the checkbox is off they return the built-in default, not the value entered earlier. This is easy to confuse, and reading the field directly may differ from calling the Getter.

Usage notes:

- Modifying through the Setter on the Component is the easiest route and needs no manual checkbox handling.
- When changing struct fields directly, remember to set the checkbox to `true` as well, otherwise nothing takes effect.

```cpp
// recommended: use the Setter, the checkbox is handled automatically
Component->SetMaxDistance(200);

// changing the field directly requires setting the checkbox yourself
Component->Performance.bDistanceLimit = true;
Component->Performance.MaxDistance = 200;
```

### FLCCRenderInfo

LCC1-specific performance parameters. Used by `ULCCComponent::LCCPerformance`.

| Field | Type | Default | Matching checkbox | Description |
|------|------|------|---------|------|
| `SortFactor` | `float` | 1 | `bCanSetSortFactor` | Sort frequency scale factor, from 0.2 to 5. A larger value means fewer sorts and better performance |
| `bAddExtraPreNodes` | `bool` | Decided by a macro | `bEnableExtraPreNodes` | Adds extra preload nodes, mitigating holes at the screen edge while turning the view quickly |

Helper methods:

```cpp
float GetSortFactor() const;
bool GetIfAddExtraPreNodes() const;
void Reset();
```

Related page: [ULCCComponent](./03-ULCCComponent.md#lccperformance)

## Metadata

### FMetaInfoBase

Metadata base struct, inherited by both `FLCCMetaInfo` and `FLCC2MetaInfo`. Return type of `ULCCComponentBase::GetMetaInfo`.

| Field | Type | Description |
|------|------|------|
| `Name` | `FString` | Data name |
| `Version` | `FString` | Data version string |
| `Description` | `FString` | Description |
| `Source` | `FString` | Source |
| `DataType` | `FString` | Data type |
| `FileType` | `FString` | File type, corresponding to `EFileType` |
| `Offset` | `TArray<float>` | Coordinate offset, 3 elements |
| `Shift` | `TArray<float>` | Translation amount |
| `Epsg` | `int32` | EPSG coordinate system number, greater than 0 means geographic coordinate information is present |
| `Scale` | `TArray<int>` | Scale, 3 elements |
| `GUID` | `FString` | Unique identifier of the data |
| `TotalLevel` | `int` | Total Level count |
| `TotalSplats` | `int` | Total splat count |
| `Splats` | `TArray<int>` | Splat count of each Level |

Helper methods:

```cpp
bool IsValid() const;              // whether the metadata is valid
void Reset();
bool IsRTK() const;                // whether RTK geographic information is present
FVector GetOffset() const;         // offset as FVector, returns a zero vector with a warning when fewer than 3 elements
FVector3f GetScale();              // note: not const, and does not check the element count
EFileType GetFileType() const;     // file type enum
float GetVersion() const;          // version as float
uint32 GetLevel0SplatNum() const;  // splat count of Level 0, in units of ten thousand
```

`GetScale()` and `GetOffset()` differ in robustness: `GetOffset()` checks the element count and returns a zero vector with a warning when there are fewer than 3, while `GetScale()` indexes the first three elements directly and goes out of bounds when `Scale` has fewer than 3 elements.

`IsRTK()` decides by `Epsg > 0` plus a non-zero offset and looks at the data only. It is not equivalent to `CanUseGeoPlace()`, which additionally requires `bEnableGeoPlace` to be enabled and the geo referencing system to be created.

Usage notes:

- Read it after loading completes; the fields are all zero values while the data is not ready.
- `TotalSplats` is the total amount of the data. `ULCCComponentBase::GetSplatNumber()` returns this very field, so the two are the same value.

```cpp
if (Component->CheckIfLoaded())
{
    const FMetaInfoBase Meta = Component->GetMetaInfo();

    UE_LOG(LogTemp, Log, TEXT("Name: %s"), *Meta.Name);
    UE_LOG(LogTemp, Log, TEXT("Levels: %d, total splats: %d"),
        Meta.TotalLevel, Meta.TotalSplats);
    UE_LOG(LogTemp, Log, TEXT("Level 0 splats: %u (ten thousand)"),
        Meta.GetLevel0SplatNum());
    UE_LOG(LogTemp, Log, TEXT("Has SH: %s"),
        Meta.GetFileType() == EFileType::Quality ? TEXT("yes") : TEXT("no"));

    if (Meta.IsRTK())
    {
        UE_LOG(LogTemp, Log, TEXT("EPSG: %d, offset: %s"),
            Meta.Epsg, *Meta.GetOffset().ToString());
    }
}
```

Blueprint as text:

```text
[Get Meta Info]
        Target = (LCC Component)
        │ Return Value ──┐
        ▼                │
[Break FMetaInfoBase] ◀──┘
        Name → [Print String]
        Total Splats → [Print String]
        Total Level → [Print String]
```

### FLCCMetaInfo

Complete LCC1 metadata, inheriting `FMetaInfoBase`.

| Additional field | Type | Description |
|---------|------|------|
| `CellLengthX` | `float` | Grid size along X |
| `CellLengthY` | `float` | Grid size along Y |
| `Attributes` | `TArray<FBoundingBoxInfoWithName>` | Bounding box of each attribute, indexed by name |
| `IndexDataSize` | `int` | Index data size |
| `Encoding` | `FString` | Encoding method |
| `BoundingBox` | `FBoundingBoxInfo` | Overall bounding box |

The condition of `IsValid()` is stricter than in the base class: `TotalLevel`, `TotalSplats`, `CellLengthX`, `CellLengthY`, and `IndexDataSize` must all be greater than 0.

Bounding box methods (all returning `FBox`, C++ only):

```cpp
FVector2f GetCellLength() const;
int32 GetXNum() const;                    // grid count along X
int32 GetYNum() const;                    // grid count along Y
FBox GetVisibleBoundingBox() const;       // visible bounding box
FBox GetScaleBox() const;
FBox GetEnvironmentScaleBox() const;
FBox GetEnvironmentVisibleBox() const;
FBox GetShcoef() const;                   // value range of the spherical harmonics
FBox GetEnvShcoef() const;
FBox GetNormal() const;
FBox GetEnvNormal() const;
FBox GetColor() const;
FVector2f GetOpacity() const;
```

### FLCC2MetaInfo

Complete LCC2 metadata, inheriting `FMetaInfoBase`. Return type of `ULCC2Component::GetMetaInfo2`.

| Additional field | Type | Description |
|---------|------|------|
| `WorkPath` | `FString` | Path of the LCC2 directory |
| `SplatType` | `EDataSourceType` | Data source format, `SOG` / `SPZ` / `PLY` |
| `SHBands` | `int32` | Highest spherical harmonics band in this tree, 0 means none, values 1/2/3 |
| `EnvName` | `int32` | Environment file index, -1 means none |
| `Root` | `FTreeNode` | Octree root node |
| `Files` | `TArray<FString>` | List of splat data files |
| `MeshFiles` | `TArray<FString>` | List of mesh files |
| `BVHFiles` | `TArray<FString>` | List of BVH files |

The condition of `IsValid()` is that `Files` is not empty.

Helper method:

```cpp
FBox GetBounds() const;    // bounding box of the root node, already converted to centimeters
```

About `SHBands`: it decides the stride of the spherical harmonics slots, so every data source inside the same spherical harmonics buffer has to agree. A tree mixing different band counts takes the widest one. Single-file formats (`.sog` / `.spz` / `.ply`) read the exact band count from the decoder header; LCC2 has no per-file band concept and `FileType` decides it, with `Portable` having no spherical harmonics and `Quality` having 3 bands.

## Raycast Result

### FLCCSplat

Hit result of a ray test. Used by the ray test functions of `ULCCComponent`.

| Field | Type | Description |
|------|------|------|
| `Location` | `FVector` | World coordinates of the hit point |
| `NodeVector` | `FLCCVector` | Node information of the hit point |

```cpp
FLCCSplat Hit;
if (LineTraceSingle(Start, End, Hit, false, 0.f, 0.f))
{
    // hit position
    const FVector HitPos = Hit.Location;
    // detail level currently loaded at that spot
    const int32 Level = Hit.NodeVector.Level;
}
```

Related page: [ULCCComponent ray tests](./03-ULCCComponent.md#raycast)

### FLCCVector

Node coordinates, locating a node in the octree.

| Field | Type | Description |
|------|------|------|
| `X` | `int32` | Node X index |
| `Y` | `int32` | Node Y index |
| `Level` | `int32` | Level the node belongs to |

The parameterized constructor and the copy constructor clamp the three fields to non-negative values, but a direct assignment (writing the fields after default construction, or using `operator=`) does not clamp.

Helper methods:

```cpp
FString ToString() const;     // format is "X:%i,Y:%i,Level:%i"
void Reset();
static double Dist2D(const FLCCVector& A, const FLCCVector& B);
```

Equality compares `X`, `Y`, and `Level` only, not the internal camera information and other runtime fields.

`Level` tells whether the data currently visible is fine or coarse; a smaller value is finer.

## Bounding Box

### FBoundingBoxInfo

Bounding box information.

| Field | Type | Description |
|------|------|------|
| `Min` | `TArray<float>` | Coordinates of the minimum point |
| `Max` | `TArray<float>` | Coordinates of the maximum point |

### FBoundingBoxInfoWithName

Named bounding box, used by the attribute list in the metadata.

| Field | Type | Description |
|------|------|------|
| `Name` | `FString` | Attribute name |
| `BoundingBoxInfo` | `FBoundingBoxInfo` | Bounding box |

Equality compares `Name` only.

## SOG Metadata

This group of structs describes the metadata of a `.sog` file and is output by the SOG parsing functions of `ULCCUtilLibrary`.

### FLCC2SogMeta

Main SOG metadata struct.

| Field | Type | Description |
|------|------|------|
| `Version` | `int32` | Format version |
| `Count` | `int32` | Splat count |
| `bAntialias` | `bool` | Whether anti-aliasing is enabled |
| `Instances` | `FLCC2SogInstances` | List of instance files |
| `Means` | `FLCC2SogMeans` | Position data, with minimum, maximum, and file list |
| `Scales` | `FLCC2SogScales` | Scale data, with a 256-entry codebook and file list |
| `Quats` | `FLCC2SogQuats` | File list of the rotation data |
| `Semantics` | `FLCC2SogSemantics` | File list of the semantic data |
| `Sh0` | `FLCC2SogSh0` | Band 0 spherical harmonics, with a 256-entry codebook and file list |
| `ShN` | `FLCC2SogShN` | High-order spherical harmonics, with band count, codebook, count, and file list |

Helper method:

```cpp
bool HasShN() const;    // whether high-order spherical harmonics are present, equivalent to ShN.Files being non-empty
```

Usage notes:

- Read it with `ParseSogMetaFromFile` without loading the render data, at very low cost.
- It reveals the point count and the spherical harmonics situation before loading, so the quality configuration can be decided in advance.

```cpp
FLCC2SogMeta Meta;
if (ULCCUtilLibrary::ParseSogMetaFromFile(Path, Meta))
{
    UE_LOG(LogTemp, Log, TEXT("Version: %d, count: %d, high-order SH: %s"),
        Meta.Version, Meta.Count, Meta.HasShN() ? TEXT("yes") : TEXT("no"));
}
```

### FLCC2SogShN

High-order spherical harmonics data.

| Field | Type | Description |
|------|------|------|
| `Bands` | `int32` | Spherical harmonics bands |
| `Count` | `int32` | Count |
| `Files` | `TArray<FString>` | File list |
| `Codebook` | `TStaticArray<float, 256>` | Quantization codebook, not visible to Blueprint |

Helper method: `bool HasShN() const`.

### FLCC2SogImageData

SOG image data.

| Field | Type | Description |
|------|------|------|
| `PixelData` | `TArray<uint8>` | Pixel data, RGBA format, 4 bytes per pixel |
| `Width` | `int32` | Width |
| `Height` | `int32` | Height |

### Other Sub-structs

The following structs contain only a file list, some with a quantization codebook:

| Struct | Fields |
|-------|------|
| `FLCC2SogInstances` | `Files` |
| `FLCC2SogSemantics` | `Files` |
| `FLCC2SogQuats` | `Files` |
| `FLCC2SogMeans` | `Mins`, `Maxs`, `Files` |
| `FLCC2SogScales` | `Codebook` (256 entries), `Files` |
| `FLCC2SogSh0` | `Codebook` (256 entries), `Files` |

`Codebook` is a `TStaticArray<float, 256>`, not a `UPROPERTY`, and is not visible in Blueprint.

## Octree Nodes

This group of structs describes the LCC2 octree. It is low-level data that regular development does not need to touch directly.

### FTreeNode

Octree node.

| Field | Type | Description |
|------|------|------|
| `Id` | `FString` | Node identifier |
| `XMin` / `YMin` / `ZMin` | `double` | Minimum point of the AABB |
| `XMax` / `YMax` / `ZMax` | `double` | Maximum point of the AABB |
| `ChildNum` | `int32` | Number of child nodes |
| `Children` | `TArray<FTreeNode>` | Array of child nodes, not a `UPROPERTY` |
| `Data` | `FTreeNodeData` | Node data |

Helper method:

```cpp
void ClearIterative();
```

It collects every node breadth-first and cleans up in reverse order, avoiding a stack overflow from recursive destruction in a very deep tree. Call it before an `FTreeNode` held manually leaves scope.

### FTreeNodeData

Node data container, marking with booleans whether each kind of data is present.

| Field | Type | Description |
|------|------|------|
| `bHas3DGS` | `bool` | Whether 3DGS data is present |
| `_3DGS` | `FTreeNodeData3DGS` | 3DGS data |
| `bHasMesh` | `bool` | Whether mesh data is present |
| `Mesh` | `FTreeNodeDataMesh` | Mesh data |
| `bHasBVH` | `bool` | Whether BVH data is present |
| `BVH` | `FTreeNodeDataBVH` | BVH data |

### FTreeNodeData3DGS

| Field | Type | Description |
|------|------|------|
| `Name` | `int32` | File index |
| `Start` | `int32` | Start offset |
| `Count` | `int32` | Count |

### FTreeNodeDataMesh

| Field | Type | Description |
|------|------|------|
| `Name` | `int32` | File index |
| `Vertex` | `int32` | Vertex count |
| `Face` | `int32` | Face count |

### FTreeNodeDataBVH

| Field | Type | Description |
|------|------|------|
| `Name` | `int32` | File index |

## See Also

- [ULCCComponentBase](./02-ULCCComponentBase.md): usage of `FRenderInfo` and `FMetaInfoBase`
- [ULCCComponent](./03-ULCCComponent.md): usage of `FLCCSplat` and `FLCCRenderInfo`
- [ULCCUtilLibrary](./09-ULCCUtilLibrary.md): the SOG metadata parsing functions
- [Enums](./10-enums.md): the enum types used in these structs
