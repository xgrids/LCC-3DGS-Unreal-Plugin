---
title: Loading Animation
parent: Documentation v3.3.1
nav_order: 14
description: Configure appear and disappear animations for a 3DGS scene, including two-stage animation parameters, inverse animation, scan line effects, and runtime control.
---

# 3DGS Appear and Disappear Animations

## Overview

The loading animation plays while LCC data loads. In the editor, click the **Refresh** button to trigger a reload and preview the animation.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/10-anim-refresh-btn-CnmyetuJ.jpg" alt="Previewing the loading animation with the Refresh button" style="width: 533px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Clicking the Refresh button to preview the loading animation</p>
</div>

- 3DGS mode: two-stage animation (nothing → point cloud → 3DGS)

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/10-anim-3dgs-stages-BnSoQB1h.jpg" alt="The two stages of the 3DGS loading animation" style="width: 526px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The 3DGS loading animation transitions from point cloud to 3DGS</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/10-anim-3dgs-demo-CLkQussD.gif" alt="Demonstration of the two-stage 3DGS loading animation" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Demonstration of the two-stage 3DGS loading animation</p>
</div>

- Point cloud mode: one-stage animation (nothing → point cloud)

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/10-anim-pointcloud-stage-ZPp5-gWL.jpg" alt="Point cloud loading animation stage" style="width: 511px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The one-stage loading animation of point cloud mode</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/10-anim-pointcloud-demo-Dy3JZBR7.gif" alt="Demonstration of the point cloud loading animation" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Demonstration of the point cloud mode loading animation</p>
</div>

## Parameters

The control parameters live under the Animation category of LCCComponent:

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/10-anim-params-panel-pBWTyvSO.jpg" alt="LCC loading animation parameter panel" style="width: 520px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Configuring the loading animation parameters in the Animation category</p>
</div>

| Parameter | Default | Description |
|-----------|---------|-------------|
| bEnableAnimation | false | Enables the loading animation |
| AnimationSpeed | 100 | Animation speed |
| AnimationMinScale | 0.001 | Point size of the first stage (3DGS mode only) |
| FirstStageDelay | 0 | First stage delay in seconds. The data is not displayed before it starts |
| SecondStageDelay | - | Second stage delay (point cloud → 3DGS). Equal to FirstStageDelay skips the first stage |
| EnvironmentDelay | - | Environment node delay. The environment does not take part in the animation and appears directly when the time is reached |
| AnimationOriginOffset | (0,0,0) | Offset of the scan origin |
| FirstStageColor | Gold | Scan line color of the first stage (an alpha of 0 hides the scan line) |
| SecondStageColor | - | Scan line color of the second stage |
| ScanLineThickness | 5 | Scan line width |

## Inverse Animation

With inverse animation enabled, the animation contracts from the distance toward the center and disappears.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/13-anim-inverse-CjHBrU2J.jpg" alt="LCC inverse animation parameter panel" style="width: 520px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Configuring the inverse animation parameters in the Animation category</p>
</div>

| Parameter | Default | Description |
|-----------|---------|-------------|
| Inverse Animation | false | Enables inverse animation. Once enabled, the animation contracts from the distance toward the center and disappears |
| Inverse Max Range Time | 30 | Sets the initial visible range of the inverse animation. The value needs to cover the whole scene |

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/13-inverse-ani-3dgs-U7jNmtnu.gif" alt="Demonstration of the two-stage 3DGS inverse animation" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Demonstration of the two-stage 3DGS inverse animation</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/13-inverse-ani-point-D5cYbPm6.gif" alt="Demonstration of the point cloud inverse animation" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Demonstration of the point cloud mode inverse animation</p>
</div>

## Controlling the Animation at Runtime

Besides configuring it in the Details panel, the animation can also be controlled at runtime from code or Blueprint.

### Replaying the Animation

Besides switching the animation on and off, `SetEnableAnimation` also resets the animation timing origin to the current time. It is therefore not limited to the loading stage: calling it at any moment replays the animation from the beginning.

```cpp
// Replay the animation on an already loaded scene without reloading the data
Component->SetEnableAnimation(true);
```

The matching Blueprint node is Set Enable Animation.

### Order of Parameters and the Switch

Once the animation starts, it follows the current parameters. Parameters can still be changed after it starts and the change takes effect on the next frame, but that is a mid-play adjustment and the animation does not restart from the beginning.

To make parameters apply to the whole animation, **set the parameters first and call `SetEnableAnimation` last**:

```cpp
// Set the parameters first
Component->AnimationSpeed = 30.0f;
Component->AnimationMinScale = 0.1f;
Component->SecondStageDelay = 1.0f;
Component->bUseFirstStageColor = true;
Component->FirstStageColor = FLinearColor(0.f, 0.8f, 1.f, 1.f);

// Turn on the switch last; the animation starts at this moment with the parameters above
Component->SetEnableAnimation(true);
```

To change a parameter during playback (adjusting the speed in real time, for example), change the property directly without calling `SetEnableAnimation` again. Conversely, calling `SetEnableAnimation(true)` during playback restarts the animation, so avoid repeated calls in normal mode.

### Playing the Disappear Animation

Set `bInverseAnimation` to `true`, then call `SetEnableAnimation(true)` to reset the timeline:

```cpp
// Disappear
Component->bInverseAnimation = true;
Component->SetEnableAnimation(true);

// Appear again
Component->bInverseAnimation = false;   // remember to reset
Component->SetEnableAnimation(true);
```

Text Blueprint:

```text
[Custom Event: PlayDisappear]
        │
        ▼
[Set Inverse Animation]
        Target = (LCC Component)
        bInverse Animation = true
        │
        ▼
[Set Enable Animation]
        Target = (LCC Component)
        b In Enable Animation = true
```

### Practical Notes

- **Set `bInverseAnimation` before `SetEnableAnimation`.** In the reverse order the timeline is reset before the direction changes, and the animation plays once in the old direction.
- **Reset `bInverseAnimation` to `false` after each appear animation finishes.** Otherwise the same object stays in inverse mode the next time it is used.
- **The default 30 seconds of `InverseMaxRangeTime` is too large for fast interaction and needs lowering.** This value multiplied by `AnimationSpeed` gives the initial visible radius, which must be large enough to cover the whole scene, but too large a value makes the disappear process drag.
- **Avoid repeated calls to `SetEnableAnimation(true)` in normal mode.** Every call resets the timeline and the animation keeps restarting from the beginning.
