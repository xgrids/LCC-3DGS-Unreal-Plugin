---
title: Collision
parent: Documentation v3.3.1
nav_order: 15
description: Enable collision data for a 3DGS scene so characters can walk and interact inside the 3DGS world in UE5, including the loading mechanism, prerequisites, and verification.
---

# Walking in a 3DGS Scene: Collision Support

LCC4Unreal supports dynamic collision data loading, so characters can move and interact freely in the 3DGS world.

## Loading Mechanism

Collision data is stored in spatial chunks just like render data. The plugin does not load all collision at once: it loads only the chunks within a certain range around the camera and swaps them in and out dynamically as the camera moves.

```text
Camera position changes
      │
      ▼
Compute the set of collision chunks that should be loaded in range
      │
      ▼
Compare with the previous set ── same ──▶ skip, do nothing
      │ different
      ▼
Load the new chunks → hand them to the physics system for baking → generate collision bodies
```

A few implementation points:

- **The load range** is controlled by `Max Load Collision Distance(m)`, and areas beyond that distance have no collision
- **The first load is synchronous**, which prevents the character from falling through the ground before collision is ready; every later update is asynchronous and does not block the main thread
- **Updates are throttled**, so collision is not rebuilt every frame while the camera moves continuously
- **Updates are skipped when the visible chunk set is unchanged**, so small movements in place do not trigger a rebuild
- **Only the main camera is used for the calculation**, and SceneCapture does not take part in collision loading

### Advantages

- Memory usage depends only on the load range, not on the total data size, so collision works even for large scenes
- The number of collision bodies the physics system has to process stays under control and does not slow physics down as the scene grows

### Costs and Mitigations

| Issue | Cause | Mitigation |
|-------|-------|-----------|
| The character falls through the ground the instant the game starts | Collision is not built yet | Raise PlayerStart a little, or enable character movement only after collision is ready |
| No collision in the distance, so line traces miss | Areas beyond the load range have no collision bodies | Raise `Max Load Collision Distance(m)`, or switch to a check that does not rely on collision |
| No collision for a short time after teleporting or moving fast | The collision chunks of the new area are still loading | Warm up the target position with `ModifyPlayerTransform` before teleporting, see [Quick Start](./02-quickstart-windows.md) |
| Stutter while moving, with a high `Update Collision` cost | Collision chunks are swapped in and out and rebaked frequently | Reduce the load range, or confirm collision is really needed |
| The navigation mesh keeps rebuilding when used with navigation | Collision changes trigger NavMesh updates | Raise the load range so the target area loads in one pass, and use Static baking, see [Navigation System Support](./16-navigation.md) |

### Loading All Collision at Once

Most of the costs above come from swapping chunks in and out at runtime. When the scene size is limited and memory allows, raise `Max Load Collision Distance(m)` high enough to cover the whole scene, so all collision is built in one pass while the data loads.

Runtime swapping of collision chunks then disappears, and none of the earlier issues occur: falling through the ground, missing collision after teleporting, stutter while moving, and repeated navigation rebuilds. This is especially recommended when using the navigation system.

The cost is a longer load time at startup and a fixed memory usage equal to the collision of the whole scene. To judge whether it is feasible, check `Collision Data Usage` in `stat xgrids` and confirm the memory usage after a full load is acceptable.

## Prerequisites

Collision relies on the collision file shipped with the data, and the requirements differ per format:

| Format | Collision data | Support |
|--------|---------------|---------|
| `.lcc` | `collision.lci` in the data directory, `collision.bin` in older versions | Supported |
| `.lcc2` | `.ply` files in `data/mesh/` under the data directory | Supported |
| `.sog` / `.spz` / `.ply` | None | Not supported |

The plugin checks in the order `collision.lci` → `collision.bin` → `data/mesh/*.ply`, and prints a warning and disables collision when none of them exist. After loading data, use `HaveValidCollisionData()` to confirm whether the current data contains usable collision data.

> Note: `collision.lci` requires export from LCC Studio v1.6.1 or above; data exported by earlier versions uses `collision.bin`.

### Alternatives for Single-file Formats

`.sog`, `.spz`, and `.ply` are single-file formats that carry no collision data, so the plugin cannot generate collision for them. Build it in the scene when collision is needed:

- **Blocking Volume**: use engine blocking volumes to build the ground, walls, and other areas that need to block. Suited to regular shapes and the cheapest option
- **Approximate mesh**: place a mesh that matches the 3DGS shape and enable its collision. Suited to scenes with complex shapes. This mesh can reuse the [proxy mesh](./13-proxy-mesh.md) by changing its collision preset to block

Neither approach involves the LCC collision loading mechanism, so they are unaffected by `CollisionLoadMaxDistance` and are not swapped in and out as the camera moves.

## Enabling

```cpp
LCCComponent->SetLCCCollisionEnable(true);
```

Or tick `bEnableCollision` in the Details panel.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/06-collision-enable-BEwBTa11.jpg" alt="Enabling LCC collision in the Details panel" style="width: 100%; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Enabling LCC collision in the Details panel</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/06-collision-demo-DChY9XUX.gif" alt="Demonstration of LCC dynamic collision" style="width: 426px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Dynamic collision of a character in an LCC scene</p>
</div>

## Verification

Use the view mode settings and select player collision or visibility collision to observe whether collision is enabled.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/08-sceneediting-viscollision-lPa4ijt9.jpg" alt="Showing collision bodies in view mode" style="width: 100%; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Verifying collision bodies with the player collision or visibility collision mode</p>
</div>

## Related Properties

| Property | Description |
|----------|-------------|
| bEnableCollision | Enables collision |
| CollisionLoadMaxDistance | Maximum load distance of collision data (300 meters by default) |

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/08-sceneediting-collisionproperties-DxtO4L7l.jpg" alt="Collision enable and maximum load distance properties" style="width: 682px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Configuring the collision enable state and the maximum load distance</p>
</div>

## Notes

- Once collision is enabled, the LineTrace of the Unreal Engine itself can be used for ray tests against collision bodies, and the test range is limited by the load range as well
- The memory used by collision data can be viewed in `Collision Data Usage` of `stat xgrids`, the update cost in `Update Collision`, and the meaning of the items in [Performance Parameters](./10-performance-parameters.md#statistics)
