---
title: v3.3.1 (latest)
nav_order: 10
has_children: true
permalink: /v3.3.1/
description: Documentation index for LCC4Unreal v3.3.1 — quickstart guides, feature reference, API reference and changelog.
---

# LCC Unreal Plugin v3.3.1
{: .no_toc }

**Latest release.** Applies to plugin v3.3.1 on UE 5.4 – 5.8. For other versions see [all versions](../#versions).
{: .fs-5 .fw-300 }

3D Gaussian Splatting rendering plugin for Unreal Engine 5. Supports LCC2, LCC, SOG, SPZ and PLY, works on UE 5.4 – 5.8, packages for Windows and Linux.

New here? Start with [Introduction](./01-introduction.md) for an overview, then follow the quickstart for your platform.

## Getting Started

| Page | What it covers |
| --- | --- |
| [Introduction](./01-introduction.md) | Supported formats, the two render pipelines, actor types, coordinate system, feature overview |
| [Quick Start - Windows](./02-quickstart-windows.md) | Install the plugin, load your first 3DGS scene, package for Win64 |
| [Quick Start - Linux](./03-quickstart-linux.md) | Native Linux development and cross-compiling from Windows |
| [Quick Start - Quest3](./04-quickstart-quest3.md) | VR setup and streaming to Meta Quest 3 |
| [Editions and Licensing](./05-pro-features.md) | Free vs Pro feature comparison and activation |

## Rendering and Appearance

| Page | What it covers |
| --- | --- |
| [Rendering](./06-rendering.md) | Render modes, point cloud mode, depth output, anti-aliasing, DLSS |
| [Visual Settings](./07-visual-settings.md) | Saturation, contrast, gamma, offset, hue, elevation coloring |
| [Normals and Lighting](./08-normals-and-lighting.md) | Normal generation modes, Lit/Unlit, shadow reception |
| [Proxy Mesh](./13-proxy-mesh.md) | Proxy-mesh based relighting and self-shadowing (Pro) |
| [Single Layer Water](./17-single-layer-water.md) | Water surface rendering with 3DGS scenes |

## Scene and Interaction

| Page | What it covers |
| --- | --- |
| [Scene Editing](./09-scene-editing.md) | Clipping volumes, section planes, transforms, global alpha, splat scale |
| [Collision](./15-collision.md) | Collision data, single and multi ray casts |
| [Navigation System Support](./16-navigation.md) | NavMesh generation over 3DGS scenes for AI pathfinding |
| [Loading Animation](./14-loading-animation.md) | Forward / reverse load animation and dynamic load / unload |

## Performance

| Page | What it covers |
| --- | --- |
| [Performance Parameters](./10-performance-parameters.md) | Every performance-related property and what it does |
| [Performance Guide](./11-performance-guide.md) | Tuning workflow, LoD strategy, thread pools, profiling |

## Integration

| Page | What it covers |
| --- | --- |
| [Third-party and Engine Plugin Integration](./12-integration.md) | nDisplay, Cesium, CARLA, Sequencer, SceneCapture, custom engine builds |
| [Localization](./18-localization.md) | Editor language settings |

## Reference

| Page | What it covers |
| --- | --- |
| [API Reference](./23-api-reference/) | Actors, components, Blueprint library, enums and structs |
| [Changelog](./24-changelog/) | Release notes for every version |

## Help

| Page | What it covers |
| --- | --- |
| [FAQ](./19-faq.md) | Common questions about formats, licensing and behavior |
| [Troubleshooting](./20-troubleshooting.md) | Concrete symptoms and how to fix them |
| [Logging and Diagnostics](./21-diagnostics.md) | Console commands, log categories, performance stats |
| [Contact Us](./22-contact-us.md) | Support channels |

## Example Project

[3dgs-unreal-example](https://github.com/xgrids/3dgs-unreal-example) is a ready-to-open UE project built on this plugin, split into 13 standalone levels that each demonstrate one feature.
