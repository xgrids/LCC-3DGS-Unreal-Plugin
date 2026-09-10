---
title: ALCCActorBase
parent: API Reference
grand_parent: v3.3.1 (latest)
nav_order: 1
description: ALCCActorBase is the base class of every 3DGS Actor in LCC4Unreal, providing the Load, UnLoad, and Refresh loading interfaces along with debug visualization.
---

# ALCCActorBase: 3DGS Actor Base Class

Base class of every LCC Actor, responsible for placing 3DGS data into the level. `ALCCActor`, `ALCC2Actor`, `ASogActor`, `ASpzActor`, and `APlyActor` all inherit from it.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `LCCActorBase.h` |
| Parent class | `AActor` |
| Blueprint | Inheritable (`BlueprintType`) |

```cpp
#include "LCCActorBase.h"
```

The Actor itself does no rendering: it holds a `ULCCComponentBase` and forwards loading operations to it. Rendering parameters, color, performance, and collision all live on the Component, obtained through [GetLCCComponent](#getlcccomponent).

> Scaling limitation: **LCC Actors only support uniform scaling.** Do not use scales with negative values (such as `(-1, 1, 1)`) or scales that differ per axis (such as `(2, 1, 3)`), otherwise rendering artifacts appear, usually showing up as a single line on screen.

## Properties

| Property | Type | Access | Description |
|------|------|------|------|
| `DefaultSceneRoot` | `ULCCFocusRootComponent*` | Read-only | Invisible root component. Its only purpose is to give the editor viewport a small fixed bounding box, so pressing `F` to focus keeps working even in a huge scene. |
| `LCCComponent` | `ULCCComponentBase*` | Read-only | The component that does the actual work. Its concrete type is decided when the subclass is constructed. |

Both properties are `VisibleAnywhere` + `BlueprintReadOnly`: they cannot be replaced in the Details panel or in Blueprint, only read.

## Methods

### Load

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
virtual void Load(const FString& String) const;
```

Loads 3DGS data from a path. This is the main entry point for loading a scene at runtime.

Parameters:

| Parameter | Type | Description |
|------|------|------|
| `String` | `const FString&` | Data file path. Absolute paths (`D:/Data/Tower/Tower.lcc`) and relative paths are both supported. Relative paths are resolved against the `Content` directory of the project, so `Tower/Tower.lcc` points to `Content/Tower/Tower.lcc`. |

The accepted path form differs per subclass:

| Actor | What to pass |
|-------|---------|
| `ALCCActor` | Path of an LCC1 `.lcc` file. The file name is not fixed, and `data.bin` and `index.bin` must exist in the same directory |
| `ALCC2Actor` | Path of an LCC2 `.lcc2` file. The file name is not fixed |
| `ASogActor` | Path of a `.sog` file |
| `ASpzActor` | Path of a `.spz` file |
| `APlyActor` | Path of a `.ply` file; an extension other than `.ply` is rejected outright |

> Note: the file names of `.lcc` and `.lcc2` are decided by the data producer, so do not assume the file is always named `meta.lcc`. Determine the format from the extension, or with [ULCCUtilLibrary::DetermineFileFormat](./09-ULCCUtilLibrary.md#determinefileformat).

Usage notes:

- The method is `const`, because the state change happens inside `LCCComponent` and the Actor itself is not modified.
- Loading is asynchronous. The data is not ready when the call returns, and **the current version requires polling with `GetLCCComponent()->CheckIfLoaded()`**. A load completion callback will be provided in a later version, after which polling is no longer needed.
- `CheckIfLoaded()` returning `true` only means the metadata and index are established and parameters can be read and configured safely; splat data is still streaming in per view and the image keeps filling in.
- To preset a path in the level without writing code, set `DefaultLoadPath` on the Component in the Details panel and it loads automatically at runtime.
- Passing an empty path is equivalent to unloading the current data.
- When the path passed in matches the currently loaded path, the call returns immediately and nothing is reloaded.
- For a suspicious path, validate it first with [ULCCUtilLibrary::CheckLCCValid](./09-ULCCUtilLibrary.md#checklccvalid) so that a load failure does not end up only as a log message.

C++ example:

```cpp
#include "LCCActor.h"
#include "LCCComponentBase.h"

void AMyGameMode::SpawnLCCScene()
{
    // spawn the Actor
    ALCCActor* LCCActor = GetWorld()->SpawnActor<ALCCActor>(
        ALCCActor::StaticClass(),
        FVector::ZeroVector,
        FRotator::ZeroRotator);

    if (!LCCActor)
    {
        return;
    }

    // load from an absolute path
    LCCActor->Load(TEXT("D:/Data/Tower/Tower.lcc"));

    // or load relative to the Content directory
    // LCCActor->Load(TEXT("Tower/Tower.lcc"));
}
```

Wait for loading to complete before operating on the data. The current version has no callback, so polling is the only option:

```cpp
void AMyActor::LoadAndConfigure()
{
    LCCActor->Load(TEXT("D:/Data/Tower/Tower.lcc"));

    // check readiness every 0.2 seconds
    GetWorld()->GetTimerManager().SetTimer(
        LoadCheckTimer, this, &AMyActor::OnCheckLoaded, 0.2f, true);
}

void AMyActor::OnCheckLoaded()
{
    ULCCComponentBase* Component = LCCActor->GetLCCComponent();
    if (!Component || !Component->CheckIfLoaded())
    {
        return;
    }

    GetWorld()->GetTimerManager().ClearTimer(LoadCheckTimer);

    // metadata is ready, parameters can be configured safely (splat data is still streaming)
    Component->SetRenderMode(ERenderMode::Splatting);
    UE_LOG(LogTemp, Log, TEXT("Total splats: %d"), Component->GetSplatNumber());
}
```

Blueprint as text:

```text
[Event BeginPlay]
        │
        ▼
[Spawn Actor from Class]
        Class = LCCActor
        Spawn Transform = (default)
        │ Return Value ──┐
        ▼                │
[Load] ◀────────────────┘
        Target = (Return Value of the previous step)
        String = "D:/Data/Tower/Tower.lcc"
```

### UnLoad

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayName = "UnLoad", DisplayPriority = 2))
virtual void UnLoad();
```

Unloads the current data and releases resources, including GPU buffers, node caches, and collision bodies.

Usage notes:

- With `CallInEditor`, an UnLoad button appears under the Actions category of the Details panel and can be clicked directly in the editor.
- After unloading, the Actor stays in the level and simply renders nothing. Calling `Load` again loads other data.
- When switching between large scenes, call `UnLoad` before `Load` so two datasets do not occupy video memory at the same time.
- No manual call in `EndPlay` is required; the Component cleans itself up when destroyed.

C++ example, switching scenes:

```cpp
void AMyManager::SwitchScene(const FString& NewPath)
{
    // release the old data first, so the video memory peaks do not stack
    LCCActor->UnLoad();
    LCCActor->Load(NewPath);
}
```

### Refresh

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayPriority = 3))
virtual void Refresh();
```

Reloads the current data. Internally it runs `UnLoad()` followed by `Load()`, so the cost matches a full reload and this is not a lightweight operation.

Reloading the "current" data is possible because a successful load also writes the path into `DefaultLoadPath`, and `Refresh` reads that value.

Usage notes:

- Use it to read from disk again after the data files on disk have been replaced.
- Do not use it to make the rendering result recompute. Changing a property through its Setter triggers an update automatically; to push a single frame, use `ForceUpdate()` on the Component, which is the lightweight option.
- After reloading, wait for readiness again, and runtime parameters previously set on the Component may need to be set again.
- A matching button appears under the Actions category of the Details panel in the editor.

### GetLCCComponent

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids")
ULCCComponentBase* GetLCCComponent() const;
```

Returns the pointer to the Component inside the Actor. This is the entry point to every rendering capability.

Return value: `ULCCComponentBase*`. Non-null under normal conditions, since it is created when the Actor is constructed.

Usage notes:

- Downcast when a subclass-specific capability is needed. `ALCCActor` gives a `ULCCComponent`, while `ALCC2Actor` and the three single-file Actors give a `ULCC2Component`.
- Always downcast with `Cast<>`, never a C-style cast. `Cast` returns `nullptr` on a type mismatch, which makes an early return easy.

C++ example, changing common base class parameters:

```cpp
ULCCComponentBase* Component = LCCActor->GetLCCComponent();
if (Component)
{
    Component->SetSplatScale(0.8f);
    Component->SetGlobalAlpha(1.0f);
}
```

Downcasting to reach LCC2-specific parameters:

```cpp
#include "LCC2Component.h"

ULCC2Component* LCC2Comp = Cast<ULCC2Component>(LCC2Actor->GetLCCComponent());
if (LCC2Comp)
{
    LCC2Comp->SetSHBands(2);
    LCC2Comp->NormalMode = ELCC2NormalGenerationMode::Hemispherical;
}
```

Blueprint as text; casting in Blueprint uses a Cast To node:

```text
[Get LCC Component]
        Target = LCCActor
        │ Return Value ──┐
        ▼                │
[Cast To LCC2Component] ◀┘
        Object = (Return Value of the previous step)
        │ Cast Succeeded
        ▼
[Set SH Bands]
        Target = (As LCC2 Component output of the Cast)
        In SH Bands = 2
```

### SelectFile

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayName = "Load", DisplayPriority = 1))
virtual void SelectFile();
```

Opens the system file dialog and loads the selection immediately.

Usage notes:

- Shown as a Load button in the Details panel, the most common way to load data in the editor.
- Every subclass overrides it to filter the matching extension: `ASogActor` lists only `.sog`, `ASpzActor` only `.spz`, `APlyActor` only `.ply`.
- It depends on editor dialog capabilities, so do not use it in a packaged runtime; use [Load](#load) at runtime instead.

### DebugNodeBound

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayName = "Debug Node Bound", DisplayPriority = 4))
void DebugNodeBound();
```

Toggles visualization of octree node bounds, drawing the currently loaded nodes as wireframe boxes.

Box color corresponds to the node Level: red, orange, yellow, green, blue, and purple in order, where red is the lowest Level (highest detail) and white is the highest Level (lowest detail).

Usage notes:

- Use it to investigate LOD problems. Red boxes still rendered far away mean `LevelFactor` is too small or `StartLevel` is set too low, and performance is being wasted.
- Conversely, cool-colored high-Level boxes up close mean detail is suppressed too aggressively and the image looks blurry.
- Call it again to turn it off; it is a toggle.
- Only effective in the editor and in development builds.

### Stats

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayPriority = 5))
void Stats();
```

Toggles the render statistics panel, equivalent to running `stat XGrids` in the console.

The panel gives the currently rendered splat count, the number of nodes being loaded, and the time spent in each stage. Watching it while tuning performance parameters is more reliable than tuning by feel.

### ShowCollision

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayName = "Show Collision", DisplayPriority = 6))
void ShowCollision();
```

Toggles collision wireframe visualization, equivalent to running `r.xgrids.DrawCollision` in the console.

Usage notes:

- It requires the data itself to carry collision and `bEnableCollision` on the Component to be enabled, otherwise nothing is shown.
- Turn it on first when a character clips through geometry or a ray test misses, to confirm whether the collision bodies were loaded at all.
- Collision streams in by distance, so the absence of wireframe far away is expected and is controlled by `Performance.CollisionLoadMaxDistance`.

### ShowFPS

```cpp
UFUNCTION(BlueprintCallable, CallInEditor, Category = "Actions",
    meta = (DisplayName = "Show FPS", DisplayPriority = 7))
void ShowFPS();
```

Toggles the frame rate display, equivalent to running `stat fps` in the console. Use it together with `Stats` to watch the frame rate and the splat count side by side.

## Complete Example

Load an LCC scene at runtime and configure rendering parameters once it is ready:

```cpp
// MyLCCLoader.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyLCCLoader.generated.h"

UCLASS()
class AMyLCCLoader : public AActor
{
    GENERATED_BODY()

public:
    /** Data path to load, absolute or relative to the Content directory */
    UPROPERTY(EditAnywhere, Category = "MyLCC")
    FString ScenePath = TEXT("Tower/Tower.lcc");

protected:
    virtual void BeginPlay() override;

private:
    void OnLoadCheck();

    UPROPERTY()
    TObjectPtr<class ALCCActor> LCCActor;

    FTimerHandle LoadCheckTimer;
};
```

```cpp
// MyLCCLoader.cpp
#include "MyLCCLoader.h"
#include "LCCActor.h"
#include "LCCComponentBase.h"

void AMyLCCLoader::BeginPlay()
{
    Super::BeginPlay();

    LCCActor = GetWorld()->SpawnActor<ALCCActor>(
        ALCCActor::StaticClass(), GetActorTransform());
    if (!LCCActor)
    {
        return;
    }

    LCCActor->Load(ScenePath);

    GetWorld()->GetTimerManager().SetTimer(
        LoadCheckTimer, this, &AMyLCCLoader::OnLoadCheck, 0.2f, true);
}

void AMyLCCLoader::OnLoadCheck()
{
    ULCCComponentBase* Component = LCCActor ? LCCActor->GetLCCComponent() : nullptr;
    if (!Component || !Component->CheckIfLoaded())
    {
        return;
    }

    GetWorld()->GetTimerManager().ClearTimer(LoadCheckTimer);

    // metadata is ready, configure as needed
    Component->SetRenderMode(ERenderMode::Splatting);
    Component->SetSplatScale(0.9f);
    Component->SetMaxDistance(200);        // render up to 200 meters
    Component->SetMaxSplatNum(1500);       // at most 15 million splats per frame
    Component->SetLCCCollisionEnable(true);

    UE_LOG(LogTemp, Log, TEXT("LCC ready, splat number: %d"),
        Component->GetSplatNumber());
}
```

## See Also

- [ULCCComponentBase](./02-ULCCComponentBase.md): rendering and performance parameters live here
- [SOG / SPZ / PLY Actors](./05-sog-spz-ply-actors.md): dedicated Actors for the three single-file formats
- [ULCCUtilLibrary](./09-ULCCUtilLibrary.md): path and format validation before loading
