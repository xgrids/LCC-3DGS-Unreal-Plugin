---
title: API Reference
parent: v3.3.1 (latest)
nav_order: 23
has_children: true
permalink: /v3.3.1/23-api-reference/
description: C++ and Blueprint interface reference for LCC4Unreal, covering Actors, Components, clipping and sectioning tools, the utility function library, enums, and structs, with call examples.
---

# API Reference: C++ and Blueprint Interfaces

This chapter covers the types the `LCC4UnrealRuntime` module exposes, including Actors, Components, utility classes, enums, and structs. Every page gives method signatures, parameter meanings, return values, when to call them, and call examples in both C++ and Blueprint.

## Getting Started

Add the module dependency in the `*.Build.cs` file of the project:

```csharp
public class MyProject : ModuleRules
{
    public MyProject(ReadOnlyTargetRules Target) : base(Target)
    {
        PublicDependencyModuleNames.AddRange(new string[]
        {
            "Core", "CoreUObject", "Engine",
            "LCC4UnrealRuntime"   // add this line
        });
    }
}
```

Include the headers:

```cpp
#include "LCCActorBase.h"        // Actor base class
#include "LCCComponentBase.h"    // Component base class
#include "Core/LCCEnum.h"        // enums
#include "Core/LCCStructure.h"   // structs
#include "Tools/LCCUtilLibrary.h"
```

`LCCDefinition.h` is an aggregate header that pulls in enums, structs, and macros at once:

```cpp
#include "LCCDefinition.h"
```

## Class Hierarchy

```text
AActor
└── ALCCActorBase                       LCC carrier in the scene, forwards operations to the internal Component
    ├── ALCCActor                       LCC1 data (.lcc + data.bin + index.bin)
    ├── ALCC2Actor                      LCC2 data (.lcc2 + chunk files)
    ├── ASogActor                       .sog file
    ├── ASpzActor                       .spz file
    └── APlyActor                       .ply file

UPrimitiveComponent
├── ULCCComponentBase                   rendering, color, performance, collision, and GIS capabilities all live here
│   ├── ULCCComponent                   LCC1 only: point cloud ray test, multiple viewports, seam cutting
│   └── ULCC2Component                  LCC2 only: spherical harmonics bands, lighting normals, depth threshold
└── ULCCFocusRootComponent              invisible root of the Actor, used only for editor focus

AVolume
├── ALCCClippingVolume                  clipping volume (box/sphere, inside or outside)
└── ALCCSectionPlane                    section plane (keeps the upper or the lower side)

AActor
└── ALCC2ProxyMesh                      proxy mesh, provides real normals and depth for LCC2

UBlueprintFunctionLibrary
└── ULCCUtilLibrary                     static functions for path validation, format detection, clipboard, version, and more
```

Key convention: **the Actor is only a shell, the capabilities live on the Component**. `Load`, `UnLoad`, and `Refresh` on `ALCCActorBase` all forward to `LCCComponent` internally. To adjust rendering parameters, get the Component with `GetLCCComponent()` first.

## Choosing an Actor

| Data form | Actor | Actual Component |
|---------|-------|---------------|
| `.lcc` file (with `data.bin` and `index.bin` in the same directory) | `ALCCActor` | `ULCCComponent` |
| `.lcc2` file | `ALCC2Actor` | `ULCC2Component` |
| Single `.sog` file | `ASogActor` | `ULCC2Component` |
| Single `.spz` file | `ASpzActor` | `ULCC2Component` |
| Single `.ply` file | `APlyActor` | `ULCC2Component` |

The `.sog`, `.spz`, and `.ply` formats reuse the LCC2 rendering pipeline internally, so the Component obtained is always `ULCC2Component`.

For the difference between the two pipelines, see [Introduction - Two Rendering Pipelines](../01-introduction.md#two-rendering-pipelines).

## Relationship to the Feature Documentation

This chapter is the interface listing: signatures, parameters, and call notes. For how to tune a parameter and what the consequences are, see the matching feature document:

| Topic | Feature document |
|------|---------|
| Render mode, video memory, full load rendering | [Rendering](../06-rendering.md) |
| Image parameter values | [Visual Settings](../07-visual-settings.md) |
| Normals and lighting | [Normals and Lighting](../08-normals-and-lighting.md) |
| Clipping and sectioning | [Scene Editing](../09-scene-editing.md) |
| Performance parameters, statistics | [Performance Parameters](../10-performance-parameters.md), [Performance Guide](../11-performance-guide.md) |
| Integration with Cesium, nDisplay, and others | [Third-party and Engine Plugin Integration](../12-integration.md) |
| Proxy mesh | [Proxy Mesh](../13-proxy-mesh.md) |
| Loading animation | [Loading Animation](../14-loading-animation.md) |
| Collision | [Collision](../15-collision.md) |
| License scope | [Editions and Licensing](../05-pro-features.md) |

## Page Index

Scene objects:

| Page | Content |
|------|------|
| [ALCCActorBase](./01-ALCCActorBase.md) | Actor base class, load/unload/refresh/debug visualization |
| [ULCCComponentBase](./02-ULCCComponentBase.md) | Component base class, rendering, performance, color, collision, GIS |
| [ULCCComponent](./03-ULCCComponent.md) | LCC1 only, point cloud ray test, multiple viewports |
| [ULCC2Component](./04-ULCC2Component.md) | LCC2 only, spherical harmonics bands, lighting normal modes |
| [SOG / SPZ / PLY Actors](./05-sog-spz-ply-actors.md) | `ASogActor` / `ASpzActor` / `APlyActor` |
| [ALCC2ProxyMesh](./06-ALCC2ProxyMesh.md) | Proxy mesh, source of real normals and depth |

Editing tools:

| Page | Content |
|------|------|
| [ALCCClippingVolume](./07-ALCCClippingVolume.md) | Clipping volume |
| [ALCCSectionPlane](./08-ALCCSectionPlane.md) | Section plane |

Helper types:

| Page | Content |
|------|------|
| [ULCCUtilLibrary](./09-ULCCUtilLibrary.md) | Static utility functions |
| [Enums](./10-enums.md) | Public enums and the meaning of their values |
| [Structs](./11-structs.md) | Public struct fields |

## Blueprint Notation

This chapter uses no screenshots; Blueprint wiring is described as text. The conventions are:

- `[Node Name]` denotes a Blueprint node.
- `│` with `▼` denotes the execution flow (white wire) going from the previous node to the next one.
- The label next to an arrow is the name of a data pin, for example `Return Value`.
- An indented `pin = value` below a node is an input pin setting of that node.

Example, meaning "switch the render mode to point cloud when the game starts":

```text
[Event BeginPlay]
        │
        ▼
[Get LCC Component]
        Target = Self
        │ Return Value ──┐
        ▼                │
[Set Render Mode] ◀──────┘
        Target = (Return Value of the previous step)
        In Render Mode = Point Cloud
```

## How to Read Method Entries

Every method entry follows the same structure: signature, description, parameters and return value, usage notes, examples.

The `UFUNCTION` markup determines availability on the Blueprint side:

| Markup | Blueprint behavior |
|------|---------|
| `BlueprintCallable` | Regular node with execution pins |
| `BlueprintPure` | Pure getter node without execution pins |
| `BlueprintSetter` | The function also runs when the property is changed in the Details panel |
| `CallInEditor` | A button appears directly in the Details panel |
| No markup | C++ only |

[← Back to documentation index](../)
