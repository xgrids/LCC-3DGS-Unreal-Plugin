---
title: Quick Start - Quest3
parent: v3.3.1 (latest)
nav_order: 4
description: View Unreal Engine 5 3DGS scenes on Meta Quest 3 through Quest Link streaming, using PC compute power for high-quality VR rendering.
---

# View 3DGS Scenes on Meta Quest 3

Render the Unreal project on the PC and stream it to a Meta Quest 3 device through Meta Quest Link, using PC compute power for high-quality rendering.

## 1. Set Up the VR Project

1. Start the UE5 engine

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-createproject-startengine-CXPZvoHu.jpg" alt="Start UE5" style="max-width: 100%; width: 100%;" />
</div>

2. Create a project from the VR template

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-create-project2-CJSShlfB.jpg" alt="VR template" style="max-width: 100%; width: 100%;" />
</div>

> Note: select the Virtual Reality template and set the project type to C++.

3. The VR template project opens with the following interface

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/04-quest-vr-template-ui-BHGnVEMr.jpg" alt="VR template interface" style="max-width: 100%; width: 100%;" />
</div>

## 2. Download Quest Link

1. Visit the Meta official website to download Meta Quest Link: [www.meta.com](https://www.meta.com/zh-cn/help/quest/pcvr/)

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/04-quest-link-download-DYU9Xpwz.jpg" alt="Quest Link download" style="max-width: 100%; width: 100%;" />
</div>

2. After installation, confirm the device is connected

## 3. Integrate the LCC4Unreal Plugin

For the full steps of plugin installation, Actor placement, and data loading, see the quick start guide for the matching platform:

- [Quick Start - Windows](./02-quickstart-windows.md)
- [Quick Start - Linux](./03-quickstart-linux.md)

After plugin integration and data loading are complete, continue with the streaming steps below.

1. Enable the Link feature on the VR device

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/04-quest-enable-link-BPnLmfNd.jpg" alt="Enable Link" style="max-width: 100%; width: 100%;" />
</div>

2. Click VR Preview to start the streaming preview, and the image displays normally:

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/04-quest-normal-view-CiHIcldv.jpg" alt="Normal image" style="max-width: 100%; width: 100%;" />
</div>
