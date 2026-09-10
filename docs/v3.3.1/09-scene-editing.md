---
title: Scene Editing
parent: Documentation v3.3.1
nav_order: 9
description: Use clipping volumes and section planes to clip 3DGS scenes spatially, cutting holes, opening up building shells, or keeping only a local area, with property descriptions and Blueprint methods.
---

# 3DGS Scene Clipping and Sectioning

## Clipping Volumes

A clipping volume can be scaled, moved, and rotated freely, which helps developers clip an LCC scene spatially.

- A single Actor supports up to 65536 clipping volumes, limited to 50 in the free edition; see [Editions and Licensing](./05-pro-features.md)
- Box and sphere shapes are supported, and both can be scaled, moved, and rotated

> Note: clipping only affects the final rendering. Clipped-away areas still take part in node traversal, load, and video memory upload as usual. Clipping therefore cannot reduce the load amount or video memory usage; use the methods in the [Performance Guide](./11-performance-guide.md) to control those costs.

### Steps

1. Place an LCCActor/LCC2Actor and load the scene.
2. Drag `LCC Clipping Volume` into the scene.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/06-clipping-volume-add-DVTdDOyg.jpg" alt="Placing an LCC Clipping Volume into the scene" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Dragging an LCC Clipping Volume into the scene</p>
</div>

3. Move the clipping volume to the target position.
4. Add the Clipping Volume to the `ClippingVolumes` array in the Details panel of the Actor.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/06-clipping-inside-kq5UVxvk.jpg" alt="Inside clipping result of a clipping volume" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The Inside clipping result of a clipping volume</p>
</div>

### Demo

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/08-section-anilipping-53jJnP6G.gif" alt="Moving a clipping volume and the resulting clipping" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Moving a clipping volume and observing the clipping in real time</p>
</div>

### Properties

| Property | Description |
|----------|-------------|
| bEnabled | Whether it is enabled |
| Mode | Inside (clip inside) or Outside (clip outside) |
| VolumeType | Box or Sphere |

### Blueprint Methods

```text
LCCComponent → AddClippingVolume
LCCComponent → RemoveClippingVolume
ClippingVolume → Refresh()
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/08-sceneediting-bpclipping-DjsUchUL.jpg" alt="Blueprint nodes for adding, removing, and refreshing a clipping volume" style="width: 701px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Adding, removing, and refreshing a clipping volume with Blueprint nodes</p>
</div>

---

## Section Planes

A section plane can be scaled, moved, and rotated freely to section the scene along a plane.

- A single Actor supports up to 65536 section planes, limited to 50 in the free edition; see [Editions and Licensing](./05-pro-features.md)
- Sectioning upward or downward is supported, and the count is tracked separately from clipping volumes

> Note: sectioning only affects the final rendering. Sectioned-away areas still take part in node traversal, load, and video memory upload as usual. Sectioning therefore cannot reduce the load amount or video memory usage.

### Steps

1. Place an LCCActor/LCC2Actor and load the scene.
2. Drag `LCC Section Plane` into the scene.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/06-section-plane-add-BCFPSpuZ.jpg" alt="Placing an LCC Section Plane into the scene" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Dragging an LCC Section Plane into the scene</p>
</div>

3. Move it to the target position.
4. Add the Section Plane to the `SectionPlanes` array in the Details panel of the Actor.

### Demo

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/08-section-anisection-DIFXVF4O.gif" alt="Moving a section plane and the resulting sectioning" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Moving a section plane and observing the sectioning in real time</p>
</div>

### Properties

| Property | Description |
|----------|-------------|
| bEnabled | Whether it is enabled |
| Mode | Upside (section upward) or Downside (section downward) |

### Blueprint Methods

```text
LCCComponent → AddSectionPlane
LCCComponent → RemoveSectionPlane
SectionPlane → Refresh()
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/08-sceneediting-bpsectionplane-DwqhILqT.jpg" alt="Blueprint nodes for adding, removing, and refreshing a section plane" style="width: 704px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Adding, removing, and refreshing a section plane with Blueprint nodes</p>
</div>

---
