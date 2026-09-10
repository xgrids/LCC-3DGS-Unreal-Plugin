---
title: ALCC2ProxyMesh
description: ALCC2ProxyMesh provides real scene depth and normals for the ProxyMesh normal mode of 3DGS, working with 3DGS in screen space through CustomDepth.
---

# ALCC2ProxyMesh: Proxy Mesh Actor

Proxy mesh Actor that provides real scene depth and normals for the `ProxyMesh` normal mode of LCC2.

| | |
|---|---|
| Module | `LCC4UnrealRuntime` |
| Header | `LCC2ProxyMesh.h` |
| Parent class | `AActor` |
| Licensing | The `ProxyMesh` normal mode requires a license, see [Editions and Licensing](../05-pro-features.md) |

```cpp
#include "LCC2ProxyMesh.h"
```

For how the proxy mesh works, the whitebox authoring requirements, the placement steps, best practices, and troubleshooting, see [Proxy Mesh](../13-proxy-mesh.md). This page only lists the interfaces.

## Properties

| Property | Type | Access | Description |
|------|------|------|------|
| `StaticMeshComponent` | `UStaticMeshComponent*` | Read-only | Root static mesh component that provides scene depth and the GBuffer. Put the whitebox in its `StaticMesh` slot |

The component itself is `BlueprintReadOnly` and cannot be replaced, but its `StaticMesh` can be set.

CustomDepth is managed by the Actor automatically: it is enabled once a valid mesh is assigned, and stays disabled with a single warning when the mesh is empty (at most one message per instance). See [Behavior](../13-proxy-mesh.md#behavior) for details.

## Usage

```cpp
#include "LCC2ProxyMesh.h"
#include "LCC2Component.h"
#include "Components/StaticMeshComponent.h"

void AMyLightingSetup::SetupProxyMesh(ALCC2Actor* LCC2Actor, UStaticMesh* ProxyAsset)
{
    // place it at the same position as the 3DGS; pairing relies on spatial overlap, no explicit reference needed
    ALCC2ProxyMesh* Proxy = GetWorld()->SpawnActor<ALCC2ProxyMesh>(
        ALCC2ProxyMesh::StaticClass(),
        LCC2Actor->GetActorLocation(),
        LCC2Actor->GetActorRotation());
    if (!Proxy)
    {
        return;
    }

    // assign the mesh, CustomDepth turns on automatically
    Proxy->StaticMeshComponent->SetStaticMesh(ProxyAsset);

    // switch to the ProxyMesh normal mode
    ULCC2Component* Comp = Cast<ULCC2Component>(LCC2Actor->GetLCCComponent());
    if (Comp)
    {
        Comp->SetLightMode(ELightMode::Lit);
        Comp->NormalMode = ELCC2NormalGenerationMode::ProxyMesh;

        // without a license it falls back to Fixed silently, so confirm it
        if (Comp->GetEffectiveNormalGenerationMode() != ELCC2NormalGenerationMode::ProxyMesh)
        {
            UE_LOG(LogTemp, Warning, TEXT("ProxyMesh requires a license"));
        }
    }
}
```

For Blueprint usage see [Proxy Mesh - Blueprint Usage](../13-proxy-mesh.md#blueprint-usage).

## See Also

- [Proxy Mesh](../13-proxy-mesh.md): full usage description
- [Normals and Lighting](../08-normals-and-lighting.md): comparison of the four normal modes
- [ULCC2Component](./04-ULCC2Component.md): `NormalMode` and the other lighting parameters
- [Enums](./10-enums.md#elcc2normalgenerationmode): normal mode values
