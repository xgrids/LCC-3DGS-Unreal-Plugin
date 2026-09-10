---
title: Introduction
parent: Documentation v3.3.1
nav_order: 1
description: LCC4Unreal is a 3DGS (3D Gaussian Splatting) rendering plugin developed by XGRIDS on Unreal Engine 5. It supports LCC2, SOG, SPZ, and PLY formats, works with UE 5.4 through 5.8, and packages for Windows and Linux.
---

# LCC Unreal Plugin Introduction

<div style="text-align: left; margin: 0.5em 0;"><a href="https://xgrids.com/intl" target="_blank" rel="noopener"><img src="https://img.shields.io/badge/Website-XGRIDS-3364FF?style=flat-square" alt="Website" /></a> <a href="https://developer.xgrids.com" target="_blank" rel="noopener"><img src="https://img.shields.io/badge/Download-Developer%20Platform-1E9E63?style=flat-square" alt="Download" /></a> <a href="https://github.com/xgrids/LCC-3DGS-Unreal-Plugin" target="_blank" rel="noopener"><img src="https://img.shields.io/badge/GitHub-Repo%20%26%20Issues-181717?style=flat-square&logo=github&logoColor=white" alt="GitHub" /></a> <a href="https://discord.gg/b99H8wjRaH" target="_blank" rel="noopener"><img src="https://img.shields.io/badge/Discord-Developer%20Community-5865F2?style=flat-square&logo=discord&logoColor=white" alt="Discord" /></a></div>

LCC Unreal Plugin (LCC4Unreal) is a 3D Gaussian Splatting (3DGS) rendering plugin developed by XGRIDS on Unreal Engine 5. It imports, renders, and supports interaction with multiple 3DGS data formats inside UE5.

## Supported Versions

| Version | Supported Engine |
|---------|------------------|
| v1.0.0 and above | Unreal Engine **5.4 ~ 5.8** |
| Below v1.0.0 | Unreal Engine 5.1 ~ 5.7 |

Supported platforms:

- Development on Windows: packages Win64, and cross-compiles to Linux x86 and Linux arm64
- Development on native Linux: packages Linux x86

> Note: released plugin packages are built against engines published by Epic, including the prebuilt version installed through the Epic Games Launcher and source builds compiled from unmodified official source. Vendor-customized branches (such as the NVIDIA RTX branch), commercial engines derived from UE (such as Pixotope), and engines with locally modified source cannot use the official plugin packages directly and require a custom build. See [Custom Engine Versions](./12-integration.md#custom-engine-versions).

## Supported Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| LCC2 | `.lcc2` | XGRIDS proprietary format and the current primary format. Supports depth, LOD, and an octree structure |
| LCC | `.lcc` | XGRIDS proprietary format, superseded by LCC2 |
| SOG | `.sog` | Introduced by PlayCanvas. Uses the LCC2 pipeline, supports depth, no LOD |
| SPZ | `.spz` | Introduced by Niantic Labs, compatible with v2/v3/v4. Uses the LCC2 pipeline, supports depth, no LOD |
| PLY | `.ply` | Standard 3DGS point cloud format. Uses the LCC2 pipeline, supports depth, no LOD |

## Two Rendering Pipelines

The plugin ships two rendering implementations, referred to as the LCC pipeline and the LCC2 pipeline. The format determines which pipeline the data goes through:

| Pipeline | Formats | Characteristics |
|----------|---------|-----------------|
| LCC2 | `.lcc2` / `.sog` / `.spz` / `.ply` | Current primary pipeline. Outputs depth and participates in occlusion tests; all visible data is merged into a single buffer and submitted in one draw call |
| LCC | `.lcc` | Earlier implementation, used only for loading `.lcc` data. Does not output depth, and submits one draw call per node |

Feature coverage differs between the two pipelines: some parameters are available on only one pipeline and are hidden in the Details panel of the other. Each document marks the pipeline a parameter belongs to.

## Corresponding Actors

| Actor | Pipeline | Purpose |
|-------|----------|---------|
| ALCC2Actor | LCC2 | Loads the .lcc2 format |
| ALCCActor | LCC | Loads the .lcc format |
| ASogActor | LCC2 | Loads a single .sog file |
| ASpzActor | LCC2 | Loads a single .spz file |
| APlyActor | LCC2 | Loads a single .ply file |

## Getting the Plugin

[LCC Unreal Plugin download](https://developer.xgrids.com/#/download?page=LCC_UNREAL_SDK_UE54)

## Plugin Directory Structure

<div style="text-align: left; margin: 0.5em 0;">
  <img src="../images/01-plugin-folder-structure.jpg" alt="LCC4Unreal plugin directory structure" style="width: 240px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">LCC4Unreal plugin directory structure</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/01-plugin-files-ScLMBnj2.jpg" alt="LCC4Unreal plugin files and subdirectories" style="width: 612px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">LCC4Unreal plugin files and subdirectories</p>
</div>

| Directory / File | Description |
|------------------|-------------|
| Binaries | Binary libraries |
| Content | Plugin assets (materials and others) |
| Intermediate | Build intermediates (UHT-generated headers and code, do not delete) |
| Resources | Icon assets |
| Shaders | Rendering shaders |
| Source | Module source definitions |
| LCC4Unreal.uplugin | Plugin descriptor file |

## Coordinate System

LCC data uses a Z-up right-handed coordinate system and UE uses a Z-up left-handed system. No extra handling is normally required.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/01-coordinate-system-wdT46nYL.jpg" alt="LCC and Unreal Engine coordinate systems compared" style="width: 766px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">LCC and Unreal Engine coordinate systems compared</p>
</div>

For Y-up PLY files, rotate the Actor 90 degrees around the X axis.

## Feature Overview

- Multi-format 3DGS rendering (LCC/LCC2/PLY/SOG/SPZ)
- Dual rendering modes: 3DGS and point cloud
- Multiple normal generation modes (Fixed/ViewFacing/Hemispherical/ProxyMesh)
- Scene lighting interaction (Lit/Unlit)
- Color adjustment (saturation/contrast/gamma/offset/tint)
- Shadow receiving (experimental)
- Clipping volumes and section planes (up to 65536 each, 50 each in the free edition)
- Dynamic collision data loading
- GIS geographic placement (Cesium integration)
- Loading animation
- nDisplay support
- VR stereo rendering
- DLSS support
- Navigation System support
- Single layer water support
- OpenColorIO color management
- Multi-camera / SceneCapture support
- Full load rendering mode (automatic merge for small scenes)
