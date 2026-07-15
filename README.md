# LCC Plugin for UE — 3D Gaussian Splatting in Unreal Engine

> 3DGS / Gaussian Splatting / UE5 Plugin / Point Cloud / Radiance Field / Virtual Production / Digital Twin / NeRF Alternative / Real-time Rendering / VR / XR

[English](#) | [中文](./README_zh.md)

[![Discord](https://img.shields.io/badge/Discord-Join%20Community-5865F2?logo=discord&logoColor=white)](https://discord.gg/R8ysxTfDj)
[![Forum](https://img.shields.io/badge/Forum-Developer%20Community-orange)](https://developer.xgrids.com/#/forum)
[![Website](https://img.shields.io/badge/Website-xgrids.com-blue)](https://xgrids.com/intl/lccUE)
[![UE5](https://img.shields.io/badge/Unreal%20Engine-5.1--5.8-black?logo=unrealengine)](https://xgrids.com/intl/lccUE)
[![Platform](https://img.shields.io/badge/Platform-Windows%20|%20Linux-lightgrey)]()
[![License](https://img.shields.io/badge/License-Free%20%2B%20Pro-green)]()

## What is this?

**LCC Plugin for UE** is a production-ready Unreal Engine plugin for loading, rendering, editing, and relighting **3D Gaussian Splatting (3DGS)** scenes at billion-point scale — the same technology behind recent breakthroughs in neural radiance fields and photorealistic scene reconstruction.

If you work with **3DGS, Gaussian Splats, point clouds, NeRF-style captures, or photogrammetry** and need to bring them into **Unreal Engine 5** for real-time visualization, virtual production, simulation, or VR, this plugin is built for you.

## Why LCC Plugin for UE?

| | LCC Plugin for UE |
|---|---|
| Scale | Billion-point 3DGS scenes loaded in one piece |
| Formats | LCC2, LCC, PLY, SOG, SPZ (v3/v4 compatible) |
| Performance | Streaming LoD for stable high frame rates |
| Relighting | Proxy Mesh based relighting |
| Depth | Native depth buffer for true cinematic DOF |
| VR | High-fidelity streaming to PICO, Meta Quest, etc. |
| Virtual Production | nDisplay, disguise, Pixotope, Aximmetry |
| Integration | C++ API + Blueprint, no code required for basic use |
| Engine | UE 5.1 / 5.2 / 5.3 / 5.4 / 5.5 / 5.6 / 5.7 / 5.8 |
| Platform | Windows, Linux (cross-compile) |
| Graphics API | DirectX 11, DirectX 12, Vulkan |

## Introduction

3D Gaussian Splatting (3DGS) is a next-generation real-time rendering technique that represents scenes as millions of 3D Gaussian primitives, enabling photo-realistic reconstruction and real-time viewing of real-world environments captured from photos or LiDAR scans. It offers faster training and rendering than NeRF while achieving comparable or better visual quality.

LCC Plugin for UE brings billion-scale Gaussian Splatting worlds into Unreal Engine. Import, edit, relight, and render massive 3DGS scenes in-engine — for film, digital twins, simulation, games, and VR.

Powered by LCC2's extreme compression, the plugin loads billion-point 3D Gaussian scenes into Unreal Engine in one piece while preserving high fidelity, covering hundreds of thousands of square meters of real-world space.

## Use Cases

- **Film & Virtual Production** — Turn real-world 3D Gaussian Splatting scenes into production-ready virtual sets for LED walls. Relight, edit, and add cinematic effects.
- **Digital Twins** — Create interactive spatial digital twins from captured environments for visualization, design review, and operations.
- **Simulation** — Build high-fidelity sim environments from real-world 3DGS captures for robotics, autonomous driving, CARLA, and training.
- **Games & VR** — Convert photogrammetry and Gaussian Splat captures into immersive game levels and VR experiences on PICO, Meta Quest, and other XR devices.
- **Architecture & BIM** — Combine 3DGS captures with Cesium for geospatial context, or use alongside mesh assets for design review.

## Key Features

### Data & Format
- Multi-format 3D Gaussian Splat support: LCC2, LCC, PLY, SOG, SPZ (v3/v4 compatible)
- Standard 3DGS PLY file import (works with gsplat, 3DGS original, Nerfstudio, Postshot, Luma, Polycam, etc.)
- No point-count limit — load billions of Gaussians
- LCC2 extreme compression for massive real-world scenes

### Rendering & Performance
- Streaming Level-of-Detail (LoD) for stable high frame rates
- NVIDIA DLSS support
- DirectX 11 / DirectX 12 / Vulkan
- Native depth rendering
- Anti-aliasing toggle

### Lighting & Post-Processing
- Proxy Mesh based relighting (Pro)
- Self-shadowing for Gaussian Splats (Pro)
- Depth of field with native depth buffer
- Exposure control, color correction regions
- Post Process Volume support
- Professional color management: OCIO / ACES workflow (Pro)

### Scene Editing
- Seamless 3DGS + mesh asset fusion and co-editing
- Mixed rendering of multiple 3DGS scenes, meshes, and effects
- Clipping and sectioning (unlimited in Pro)
- Translation, rotation, scaling
- Global alpha control
- Splat scale adjustment
- Elevation coloring
- Node Bound visualization

### XR & Virtual Production
- High-fidelity VR streaming (PICO, Meta Quest, and mainstream headsets)
- Native nDisplay support
- disguise, Pixotope, Aximmetry integration
- LED wall virtual production workflows

### Simulation & Geospatial
- Cesium for Unreal Engine support
- CARLA Simulator support
- Collision detection (single & multi ray cast)

### Developer Experience
- Code-free Blueprint visual scripting
- C++ native API for runtime control
- Console commands for performance profiling and debugging
- Configurable thread pools (traversal, loading, sorting)
- LCC file selection dialog
- Load animation support
- Dynamic load / unload of scene data
- UE 5.1 – 5.8 support

## Download

Please visit [developer.xgrids.com](https://developer.xgrids.com/#/download?page=LCC_UNREAL_SDK_UE54) to get the latest plugin.

## Pricing

| | Free | Pro (Limited Time Free) |
|---|---|---|
| Multi-format 3DGS loading | ✅ | ✅ |
| Rendering & LoD performance | ✅ | ✅ |
| Depth rendering & DOF | ✅ | ✅ |
| NVIDIA DLSS | ✅ | ✅ |
| Collision | ✅ | ✅ |
| VR streaming | ✅ | ✅ |
| nDisplay | ✅ | ✅ |
| Cesium & CARLA | ✅ | ✅ |
| Basic relighting | ✅ | ✅ |
| Limited clipping | ✅ | ✅ |
| Proxy Mesh relighting | — | ✅ |
| Self-shadowing | — | ✅ |
| ACES / OCIO color pipeline | — | ✅ |
| Unlimited clipping & sectioning | — | ✅ |

## Documentation

- [Developer Docs](https://developer.xgrids.com/#/document?titleId=en-1720509312452)
- [Tutorials](https://developer.xgrids.com/#/tutorial?page=UE_SDK)
- [Release Notes](https://developer.xgrids.com/#/document?titleId=en-1720509717058)

## Compatibility

| Engine | UE 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 5.8 |
|---|---|
| Platform | Windows, Linux (cross-compile) |
| Graphics API | DirectX 11, DirectX 12, Vulkan |
| VR Headsets | PICO, Meta Quest, and others |
| VP Systems | nDisplay, disguise, Pixotope, Aximmetry |

## About XGRIDS

XGRIDS is building the world's leading spatial intelligence platform, bridging the physical and digital worlds to provide the foundational infrastructure for AI to operate in real-world environments.

100+ invention patents power the AI-driven spatial-intelligence engine, multi-sensor SLAM, and 3D Gaussian Splatting workflows. 2000+ clients across surveying, construction, heritage preservation, and digital entertainment.

## Contact

- Email: enterprise@xgrids.com
- Website: [xgrids.com](https://xgrids.com/intl/lccUE)
- Discord: [discord.gg/R8ysxTfDj](https://discord.gg/R8ysxTfDj)

---

**Keywords:** 3D Gaussian Splatting, 3DGS, Gaussian Splat, Unreal Engine Plugin, UE5 Plugin, UE5 3DGS, point cloud rendering, radiance field, NeRF Unreal Engine, photogrammetry Unreal, virtual production, digital twin, real-time rendering, VR visualization, nDisplay, Gaussian Splat viewer, large-scale point cloud, billion-point rendering, XGRIDS, LCC, spatial computing
