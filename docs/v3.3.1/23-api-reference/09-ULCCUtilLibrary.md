---
title: ULCCUtilLibrary
description: ULCCUtilLibrary provides the Blueprint static utility functions of LCC4Unreal, covering data path validation, 3DGS format detection, collision type detection, clipboard access, and version queries.
---

# ULCCUtilLibrary: Blueprint Utility Function Library

Static utility function library providing path validation, format detection, clipboard operations, version queries, and other helpers.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `Tools/LCCUtilLibrary.h` |
| Parent class | `UBlueprintFunctionLibrary` |

```cpp
#include "Tools/LCCUtilLibrary.h"
```

Everything is a static function, called through the class name without an instance:

```cpp
const bool bValid = ULCCUtilLibrary::CheckPathValid(Path);
```

These nodes need no Target pin in Blueprint; search for the function name directly.

## Path and Format Validation

Validating before loading separates "the path is wrong" from "the data itself has a problem" and saves a lot of investigation time.

### CheckPathValid

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool CheckPathValid(FString Path);
```

Checks whether a **file** exists.

| Parameter | Type | Description |
|------|------|------|
| `Path` | `FString` | File path to check |

Returns `bool`: `true` when the file exists.

Usage notes:

- **It only checks files, and passing a directory path returns `false`.** Internally it uses a file existence check.
- It only checks existence, not whether the content is valid LCC data.
- It is the first check of the loading process, ruling out a mistyped path or a moved file.

```cpp
if (!ULCCUtilLibrary::CheckPathValid(Path))
{
    UE_LOG(LogTemp, Error, TEXT("Path does not exist: %s"), *Path);
    return;
}
```

### CheckLCCValid

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool CheckLCCValid(FString Path, FString& OutWorkPath);
```

Checks whether the data is valid LCC1 data.

| Parameter | Type | Description |
|------|------|------|
| `Path` | `FString` | Path of the `.lcc` file, whose name is not fixed |
| `OutWorkPath` | `FString&` | Output work directory, that is, the directory containing the data files |

Returns `bool`: `true` when valid.

Usage notes:

- LCC1 needs the `.lcc` file itself plus `data.bin` and `index.bin` in the same directory, and a missing one returns `false`. The name of the `.lcc` file is not fixed.
- The output `OutWorkPath` is the directory with the file name removed, useful when other files in the same directory need to be composed.
- Relative paths are supported: when the path passed in does not exist, `Content/<given path>` is tried once more, matching the rule of `Load()`.

```cpp
FString WorkPath;
if (ULCCUtilLibrary::CheckLCCValid(Path, WorkPath))
{
    UE_LOG(LogTemp, Log, TEXT("Valid LCC1 data, work path: %s"), *WorkPath);
}
```

### CheckLCC2Valid

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool CheckLCC2Valid(FString Path, FString& OutWorkPath);
```

Checks an LCC2 path and extracts the work directory. The parameters mean the same as above.

Usage notes:

- LCC2 does not need `data.bin` and `index.bin`, so this function only confirms the path exists and then takes the parent directory as `OutWorkPath`; it does not validate the integrity of the data content. The real content validation happens during loading.
- When it is unclear whether a path is LCC1 or LCC2, checking the extension (`.lcc` or `.lcc2`) is the most direct approach. Trying both validation functions works as well:

```cpp
FString WorkPath;

if (Path.EndsWith(TEXT(".lcc2"), ESearchCase::IgnoreCase))
{
    if (ULCCUtilLibrary::CheckLCC2Valid(Path, WorkPath))
    {
        // use ALCC2Actor
    }
}
else if (Path.EndsWith(TEXT(".lcc"), ESearchCase::IgnoreCase))
{
    if (ULCCUtilLibrary::CheckLCCValid(Path, WorkPath))
    {
        // use ALCCActor
    }
}
else
{
    UE_LOG(LogTemp, Error, TEXT("Not an LCC dataset: %s"), *Path);
}
```

> Note: do not treat "`CheckLCC2Valid` returned true" as proof of LCC2. Its validation is very loose and a `.lcc` path returns `true` as well, which would mistake LCC1 data for LCC2. Check the extension first.

### DetermineFileFormat

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static EFileFormat DetermineFileFormat(const FString& Path);
```

Determines the file format.

Returns `EFileFormat`. It actually judges by extension and requires the path to exist:

| Extension | Return value |
|-------|-------|
| `.lcc` | `LCC` |
| `.splats` | `Splats` |
| `.las` | `LAS` |
| `.ply` | `PLY` |
| Anything else, or a path that does not exist | `None` |

Usage notes:

- **This function does not recognize `.lcc2`, `.sog`, or `.spz`; all of them return `None`.** Check the extension manually when these formats need covering.
- A path that does not exist also returns `None`, so `None` has two possible meanings: the format is unsupported, or the file is simply not there. To separate them, call [CheckPathValid](#checkpathvalid) first.

Because the coverage is incomplete, checking the extension directly is a sturdier approach for a general loader:

```cpp
UClass* PickActorClass(const FString& Path)
{
    const FString Ext = FPaths::GetExtension(Path).ToLower();

    if (Ext == TEXT("lcc"))  return ALCCActor::StaticClass();
    if (Ext == TEXT("lcc2")) return ALCC2Actor::StaticClass();
    if (Ext == TEXT("sog"))  return ASogActor::StaticClass();
    if (Ext == TEXT("spz"))  return ASpzActor::StaticClass();
    if (Ext == TEXT("ply"))  return APlyActor::StaticClass();

    return nullptr;
}
```

For the full example see [SOG / SPZ / PLY Actors](./05-sog-spz-ply-actors.md#choosing-the-actor-automatically-by-extension).

### DetermineSourceType

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static ELCCSourceType DetermineSourceType(const FString& Path);
```

Determines the data source type.

Returns `ELCCSourceType`: `Local` for a local file, `Http` for a network address.

Use it to branch the handling logic, for example when network data needs downloading first or streaming.

### DetermineCollisionType

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static ECollisionType DetermineCollisionType(const FString& Path);
```

Determines the collision data format.

| Parameter | Type | Description |
|------|------|------|
| `Path` | `const FString&` | **Directory containing the data**, not the path of the `.lcc` file. The function looks for `collision.lci`, `collision.bin`, and similar files in that directory |

Returns `ECollisionType`:

| Value | Meaning |
|------|------|
| `None` | No collision data |
| `Bin` | Legacy `.bin` format |
| `Lci` | Newer `.lci` format |
| `Ply` | Point cloud `.ply` collision |

Usage notes:

- **Pass a directory, not a file path.** Passing the path of a `.lcc` file always returns `None`. The directory is available from `OutWorkPath` of `CheckLCCValid`.
- `None` means the data carries no collision, in which case enabling `bEnableCollision` has no effect.
- Once the data is loaded, `HaveValidCollisionData()` on the Component is easier and avoids composing the directory manually.

## Version and Environment

### GetLCC4UnrealVersion

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static FString GetLCC4UnrealVersion();
```

Returns the plugin version string.

Uses: showing it in an about screen, writing it to a log, attaching it to an issue report.

```cpp
UE_LOG(LogTemp, Log, TEXT("LCC4Unreal version: %s"),
    *ULCCUtilLibrary::GetLCC4UnrealVersion());
```

### GetProjectId

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static FString GetProjectId();
```

Returns the identifier of the current project. Required when requesting a license or investigating a licensing problem.

### GetLCCConfigPath

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static FString GetLCCConfigPath();
```

Returns the path of the plugin configuration file.

### GetLCC4UnrealRootPath

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static FString GetLCC4UnrealRootPath();
```

Returns the root directory path of the plugin. Use it to compose paths when the resources shipped with the plugin need accessing.

### GetLocale

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static ELocale GetLocale();
```

Returns the current locale, `ELocale::EN_US` or `ELocale::ZH_CN`.

Order of the decision: the Language item in the project settings comes first, and `EN_US` is returned directly when it is Always English; otherwise the current editor language decides, and Chinese returns `ZH_CN`.

Uses: making your own UI text follow the plugin language setting, or picking the website domain by region.

```cpp
const FString DownloadUrl = (ULCCUtilLibrary::GetLocale() == ELocale::ZH_CN)
    ? TEXT("https://xgrids.cn/support/download")
    : TEXT("https://xgrids.com/intl/support/download");
```

## Viewport Identifiers

### GetPlayerUniqueID

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool GetPlayerUniqueID(class APlayerController* PlayerController, int32& OutUniqueID);
```

Gets the unique identifier of a player controller.

| Parameter | Type | Description |
|------|------|------|
| `PlayerController` | `APlayerController*` | Target player controller |
| `OutUniqueID` | `int32&` | Output unique identifier |

Returns `bool`: `true` on success.

Usage notes:

- Use it as a key when maintaining a mapping table of "one configuration per viewport" yourself.
- Regular multi-viewport render configuration only needs the controller pointer passed to `SetPlayerLoadMode` and `SetPlayerRenderMode`, with no manual ID retrieval.

### GetSceneCaptureUniqueID

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool GetSceneCaptureUniqueID(class USceneCaptureComponent2D* SceneCaptureComponent2D,
                                    int32& OutUniqueID);
```

Gets the unique identifier of a SceneCapture component. The parameters and return value mean the same as above.

## Clipboard

### CopyToClipboard

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static void CopyToClipboard(FString CopyString);
```

Writes a string to the system clipboard.

| Parameter | Type | Description |
|------|------|------|
| `CopyString` | `FString` | Content to copy |

Typical use: a "copy diagnostic information" button that makes it easier for users to report problems.

```cpp
void AMyDebugUI::CopyDiagnostics()
{
    const FString Info = FString::Printf(
        TEXT("Plugin: %s\nProject: %s\nSplats: %d"),
        *ULCCUtilLibrary::GetLCC4UnrealVersion(),
        *ULCCUtilLibrary::GetProjectId(),
        Component->GetSplatNumber());

    ULCCUtilLibrary::CopyToClipboard(Info);
}
```

### GetClipboardString

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static FString GetClipboardString();
```

Reads the content of the system clipboard.

Typical use: a "paste the path from the clipboard" button that avoids typing a long path.

```cpp
void AMyLoader::LoadFromClipboard()
{
    const FString Path = ULCCUtilLibrary::GetClipboardString();
    if (ULCCUtilLibrary::CheckPathValid(Path))
    {
        LCCActor->Load(Path);
    }
}
```

Blueprint as text:

```text
[Button Clicked: PasteAndLoad]
        │
        ▼
[Get Clipboard String]
        │ Return Value ──┐
        ▼                │
[Check Path Valid] ◀─────┘
        Path = (Return Value)
        │ Return Value ──┐
        ▼                │
[Branch] ◀───────────────┘
        │ True
        ▼
[Load]
        Target = LCCActor
        String = (clipboard content)
```

## Texture

### GetTextureFromBase64

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static UTexture2D* GetTextureFromBase64(const FString& Base64String);
```

Converts Base64-encoded image data into a runtime texture.

| Parameter | Type | Description |
|------|------|------|
| `Base64String` | `const FString&` | Base64-encoded image data |

Returns `UTexture2D*`: `nullptr` when the conversion fails.

Use: turning a Base64 image obtained from a network interface or a configuration file into a texture for UI directly, without writing it to disk and importing it.

```cpp
UTexture2D* Texture = ULCCUtilLibrary::GetTextureFromBase64(Base64Data);
if (Texture)
{
    MyImageWidget->SetBrushFromTexture(Texture);
}
```

## SOG Metadata Parsing

These functions read the metadata of a `.sog` file without loading the render data, which suits data previews and list pages.

### ParseSogMetaFromFile

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool ParseSogMetaFromFile(const FString& FilePath, FLCC2SogMeta& OutMeta);
```

Parses SOG metadata from a file.

| Parameter | Type | Description |
|------|------|------|
| `FilePath` | `const FString&` | Path of the `.sog` file |
| `OutMeta` | `FLCC2SogMeta&` | Output metadata |

Returns `bool`: `true` when parsing succeeds.

Usage notes:

- It reads metadata only and does not load splat data, at very low cost.
- It reveals the point count and whether high-order spherical harmonics are present before loading, which helps decide whether a downgraded configuration is needed.

```cpp
FLCC2SogMeta Meta;
if (ULCCUtilLibrary::ParseSogMetaFromFile(TEXT("D:/Data/scene.sog"), Meta))
{
    UE_LOG(LogTemp, Log, TEXT("Splat count: %d, has high-order SH: %s"),
        Meta.Count, Meta.HasShN() ? TEXT("yes") : TEXT("no"));

    // lower the configuration in advance when the point count is too large
    if (Meta.Count > 5000000)
    {
        Component->SetMaxSplatNum(1000);
    }
}
```

### ParseSogMetaFromData

```cpp
UFUNCTION(BlueprintCallable, Category = "XGrids|Util")
static bool ParseSogMetaFromData(const TArray<uint8>& Data, FLCC2SogMeta& OutMeta);
```

Parses SOG metadata from a byte array in memory.

Use this version when the data came from a network download and has not been written to disk yet.

For the fields of `FLCC2SogMeta` see [Structs](./11-structs.md#flcc2sogmeta).

## C++ Only

The following functions have no `UFUNCTION` markup and can only be called from C++.

### ConvertStrToMetaInfo

```cpp
static FLCCMetaInfo ConvertStrToMetaInfo(const FString& JsonStr);
```

Converts the JSON content of a `.lcc` file into an `FLCCMetaInfo` struct.

Use it to read and parse metadata files yourself, for example scanning datasets in bulk while building a data management tool.

### ConvertStrToLCC2MetaInfo

```cpp
static FLCC2MetaInfo ConvertStrToLCC2MetaInfo(const FString& JsonStr);
```

Converts the JSON content of a `.lcc2` file into an `FLCC2MetaInfo` struct.

### SelectFile

```cpp
static FString SelectFile(ELCCVersion LCCVersion);
```

Opens the file dialog with the extension filtered by version. Returns the selected path, or an empty string on cancel.

Editor only. `SelectFile()` on the Actor calls this internally.

> Note: `ULCCUtilLibrary` also contains a number of static functions meant for internal plugin use only (frustum calculation, the internal implementation of SOG parsing, subdirectory extension checks, and others). They are visible to the compiler but are not part of the public API, their behavior may change between versions, and they should not be relied on.

## Complete Example: Validate Before Load

```cpp
#include "Tools/LCCUtilLibrary.h"
#include "LCCActor.h"
#include "LCC2Actor.h"

bool AMyLoader::ValidateAndLoad(const FString& Path)
{
    // 1. path existence
    if (!ULCCUtilLibrary::CheckPathValid(Path))
    {
        UE_LOG(LogTemp, Error, TEXT("Path does not exist: %s"), *Path);
        return false;
    }

    // 2. tell the format apart by extension
    const FString Ext = FPaths::GetExtension(Path).ToLower();
    const bool bIsLCC1 = (Ext == TEXT("lcc"));
    const bool bIsLCC2 = (Ext == TEXT("lcc2"));

    if (!bIsLCC1 && !bIsLCC2)
    {
        // for dispatching .sog / .spz / .ply see the SOG / SPZ / PLY Actors page
        UE_LOG(LogTemp, Error, TEXT("Not an LCC dataset: %s"), *Path);
        return false;
    }

    // 3. validate data integrity and take the work directory at the same time
    FString WorkPath;
    const bool bValid = bIsLCC2
        ? ULCCUtilLibrary::CheckLCC2Valid(Path, WorkPath)
        : ULCCUtilLibrary::CheckLCCValid(Path, WorkPath);

    if (!bValid)
    {
        UE_LOG(LogTemp, Error, TEXT("Incomplete LCC dataset: %s"), *Path);
        return false;
    }

    // 4. spawn the matching Actor and load
    UClass* ActorClass = bIsLCC2 ? ALCC2Actor::StaticClass() : ALCCActor::StaticClass();
    ALCCActorBase* Actor = GetWorld()->SpawnActor<ALCCActorBase>(ActorClass);
    if (!Actor)
    {
        return false;
    }
    Actor->Load(Path);

    // 5. confirm whether collision data exists while at it. Note the work directory is passed, not the file path
    const ECollisionType CollisionType =
        ULCCUtilLibrary::DetermineCollisionType(WorkPath);
    UE_LOG(LogTemp, Log, TEXT("Collision type: %d"),
        static_cast<int32>(CollisionType));

    return true;
}
```

## See Also

- [SOG / SPZ / PLY Actors](./05-sog-spz-ply-actors.md): the full example of dispatching by format
- [Enums](./10-enums.md): values of `EFileFormat`, `ECollisionType`, `ELocale`, and others
- [Structs](./11-structs.md): fields of `FLCCMetaInfo` and `FLCC2SogMeta`
