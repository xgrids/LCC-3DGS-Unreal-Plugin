---
title: SOG / SPZ / PLY Actors
description: ASogActor, ASpzActor, and APlyActor load a single .sog, .spz, or .ply 3DGS file in UE5, reusing the LCC2 rendering pipeline internally.
---

# SOG / SPZ / PLY Actors: Single-file 3DGS Loading

The `ASogActor`, `ASpzActor`, and `APlyActor` Actors correspond to the `.sog`, `.spz`, and `.ply` formats respectively and load a single 3DGS file without an LCC directory structure. All three inherit [ALCCActorBase](./01-ALCCActorBase.md).

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Headers | `SogActor.h`, `SpzActor.h`, `PlyActor.h` |
| Parent class | `ALCCActorBase` |
| Internal Component | `ULCC2Component` |

```cpp
#include "SogActor.h"
#include "SpzActor.h"
#include "PlyActor.h"
```

All three reuse the complete LCC2 rendering pipeline internally by constructing a virtual `FLCC2MetaInfo`. In other words, **the LCC2 capabilities are equally available to these three formats**, including spherical harmonics band control, lighting normal modes, clipping and sectioning, and color adjustment. By the same token, the limitations of the LCC2 pipeline are inherited as well, for example the multi-viewport interfaces having no effect.

## Supported Formats

| Actor | Extension | Use case |
|-------|-------|---------|
| `ASogActor` | `.sog` | Compressed format, small size, fast to load |
| `ASpzActor` | `.spz` | Compressed format |
| `APlyActor` | `.ply` | Common point cloud / 3DGS exchange format, the native output of most training tools |

For the support scope of each format see [Introduction - Supported Formats](../01-introduction.md#supported-formats).

## Comparison with LCC Directory Formats

| | `.sog` / `.spz` / `.ply` | LCC / LCC2 directory |
|---|--------|----------------|
| File organization | One file | Metadata file plus several data chunks |
| LOD levels | None | Present, streamed by distance |
| Suitable scale | Small and medium scenes | Large scenes |
| Collision data | None | Possible |
| Ease of distribution | High, a single file | Requires keeping the directory intact |

How to choose: use these three formats when the data is small, when verifying the result quickly, or when single-file distribution is needed. Large scenes rely on LOD to control performance and require the LCC / LCC2 directory format.

## Methods

The public interfaces of all three Actors are inherited from `ALCCActorBase`; only the methods below are overridden.

### SelectFile

```cpp
// ASogActor
virtual void SelectFile() override;   // the dialog lists only .sog

// ASpzActor
virtual void SelectFile() override;   // the dialog lists only .spz

// APlyActor
virtual void SelectFile() override;   // the dialog lists only .ply
```

Opens the file dialog, each filtering its own extension. Shown as a Load button in the Details panel.

Editor only; use `Load` in a packaged runtime.

### Load (overridden by APlyActor only)

```cpp
// APlyActor
virtual void Load(const FString& String) const override;
```

Loads a `.ply` file. **A path whose extension is not `.ply` is rejected outright** and no parsing is attempted.

`ASogActor` and `ASpzActor` do not override `Load` and use the base class implementation.

## Examples

### Loading a single .sog file

```cpp
#include "SogActor.h"
#include "LCC2Component.h"

void AMyLoader::LoadSogFile()
{
    ASogActor* SogActor = GetWorld()->SpawnActor<ASogActor>(
        ASogActor::StaticClass(), FVector::ZeroVector, FRotator::ZeroRotator);
    if (!SogActor)
    {
        return;
    }

    SogActor->Load(TEXT("D:/Data/scene.sog"));

    // internally a ULCC2Component, so all LCC2 capabilities are available
    ULCC2Component* Comp = Cast<ULCC2Component>(SogActor->GetLCCComponent());
    if (Comp)
    {
        Comp->SetSHBands(2);
        Comp->SetSplatScale(0.9f);
    }
}
```

### Choosing the Actor automatically by extension

Every format has its own dedicated Actor, and a real project usually needs to dispatch by extension.

Do not rely on [DetermineFileFormat](./09-ULCCUtilLibrary.md#determinefileformat) for this: it only recognizes `.lcc`, `.splats`, `.las`, and `.ply`, and returns `None` for `.lcc2`, `.sog`, and `.spz`. Checking the extension directly is more reliable:

```cpp
#include "Tools/LCCUtilLibrary.h"
#include "LCCActor.h"
#include "LCC2Actor.h"
#include "SogActor.h"
#include "SpzActor.h"
#include "PlyActor.h"

ALCCActorBase* AMyLoader::LoadAnyFormat(const FString& Path)
{
    if (!ULCCUtilLibrary::CheckPathValid(Path))
    {
        UE_LOG(LogTemp, Error, TEXT("Path does not exist: %s"), *Path);
        return nullptr;
    }

    const FString Ext = FPaths::GetExtension(Path).ToLower();

    UClass* ActorClass = nullptr;
    if (Ext == TEXT("lcc"))       ActorClass = ALCCActor::StaticClass();
    else if (Ext == TEXT("lcc2")) ActorClass = ALCC2Actor::StaticClass();
    else if (Ext == TEXT("sog"))  ActorClass = ASogActor::StaticClass();
    else if (Ext == TEXT("spz"))  ActorClass = ASpzActor::StaticClass();
    else if (Ext == TEXT("ply"))  ActorClass = APlyActor::StaticClass();

    if (!ActorClass)
    {
        UE_LOG(LogTemp, Error, TEXT("Unsupported format: %s"), *Path);
        return nullptr;
    }

    ALCCActorBase* Actor = GetWorld()->SpawnActor<ALCCActorBase>(
        ActorClass, FVector::ZeroVector, FRotator::ZeroRotator);
    if (Actor)
    {
        Actor->Load(Path);
    }
    return Actor;
}
```

Blueprint has no node that takes the extension directly, so use a string check:

```text
[Custom Event: LoadAnyFormat]
        Path (String)
        │
        ▼
[Check Path Valid]
        Path = Path
        │ Return Value ──┐
        ▼                │
[Branch] ◀───────────────┘
        │ True
        ▼
[Ends With]
        Source String = Path
        In Suffix = ".sog"
        Search Case = Ignore Case
        │ Return Value ──┐
        ▼                │
[Branch] ◀───────────────┘
        │ True
        ▼
[Spawn Actor from Class]
        Class = SogActor
        │ Return Value ──┐
        ▼                │
[Load] ◀─────────────────┘
        Target = (Return Value)
        String = Path

(the False branch keeps checking ".spz", ".ply", ".lcc2", ".lcc" with Ends With)
```

## Notes

- These three formats have no LOD levels. Loading constructs a single-level, single-node tree (total level count 1, root node without children), so LOD parameters such as `StartLevel`, `EndLevel`, and `LevelFactor` are still read but have no levels to choose from and produce no effect. Performance is controlled mainly through `MaxSplatNum` and `SplatScale`.
- These three formats carry no collision data; for alternatives see [Collision - Alternatives for Single-file Formats](../15-collision.md#alternatives-for-single-file-formats).
- A large file is loaded in full at once, and video memory usage stays constant and does not change with the view. For the loading limit and how it is calculated, see [Rendering - Loading Limits of Single-file Formats](../06-rendering.md#loading-limits-of-single-file-formats).
- They go through the LCC2 rendering pipeline, so the multi-viewport interfaces on the base class do not apply.
- All three set `bUseMipFilter` on the Component to `false` at construction (the base class default is `true`) and set the Actor rotation to `(0, 0, 90)`.
- `.ply` has several sub-formats, and the plugin supports the variant carrying 3DGS properties. Loading a regular geometry mesh `.ply` fails.

## See Also

- [ALCCActorBase](./01-ALCCActorBase.md): all the inherited interfaces
- [ULCC2Component](./04-ULCC2Component.md): the component these three Actors use internally
- [ULCCUtilLibrary](./09-ULCCUtilLibrary.md): format detection and path validation
