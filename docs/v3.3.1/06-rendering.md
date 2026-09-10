---
title: Rendering
parent: v3.3.1 (latest)
nav_order: 6
description: The 3DGS rendering mechanism of LCC4Unreal, covering the difference between chunked rendering and full load rendering, the video memory budget and Buffer limits, the loading limits of single-file formats, and the automatic release policy when video memory runs short.
---

# 3DGS Rendering Mechanism and Video Memory Management

This document describes how data enters video memory to participate in rendering, along with the allocation rules and capacity limits of video memory. For image parameter tuning, see [Visual Settings](./07-visual-settings.md). For load operations, see [Quick Start](./02-quickstart-windows.md).

Terminology: the smallest unit of loading and rendering after the data is split is called a node (Node). The precision of each node is identified by its Level: Level 0 has the highest precision, and a larger Level means lower precision. These definitions match [Performance Parameters](./10-performance-parameters.md).

## Chunked Rendering and Full Load Rendering

Once data is loaded, there are two ways to render it: load visible nodes on demand (chunked rendering), or load the entire dataset at once (full load rendering). The default behavior differs per pipeline:

| Pipeline | Formats | Default mode |
|----------|---------|--------------|
| LCC2 | `.lcc2` / `.ply` / `.spz` / `.sog` | Chunked rendering |
| LCC | `.lcc` | Full load rendering; falls back to chunked rendering when the data exceeds the threshold |

### Chunked Rendering

The data is split into spatial nodes, and each node contains several Levels. Each frame is processed as follows:

```text
Traversal (filter visible nodes by camera position and orientation, pick a Level for each node)
      │
      ▼
Load (read the data of the selected nodes into memory)
      │
      ▼
Upload (submit the data to video memory)
      │
      ▼
Render
```

Near nodes use a low Level, far nodes use a high Level, and invisible nodes are not loaded. A node that leaves the view is not released immediately; it stays in the cache so it can be reused directly when it enters the view again. Only when video memory runs short are the least recently used nodes evicted, based on conditions such as the last render time. Video memory usage depends mainly on the currently visible range, which is why large scenes can be supported.

The two pipelines submit draws differently:

| Pipeline | Draw submission |
|----------|-----------------|
| LCC | Each node is submitted as a separate draw, so draw calls grow with the node count |
| LCC2 | The data of all visible nodes is merged into a single Buffer and submitted in one draw |

Inherent costs of chunked rendering:

- Adjacent nodes may be assigned different Levels, so density and sharpness differ at the boundary and node borders become visible
- Levels keep switching while the camera moves, which can produce detail popping

### Full Load Rendering

The camera-based node filtering step is skipped. The entire dataset is loaded at once and stays resident in video memory, so moving the camera no longer triggers traversal or Level selection.

Advantages:

- The whole scene uses a single Level, so there is no Level difference between nodes and no visible node borders
- The data stays resident in video memory, so moving the camera causes no loading, releasing, or detail popping
- Per-frame traversal and upload overhead is eliminated, giving a more stable frame rate

The cost is that all data stays resident in video memory, and usage grows linearly with scene size, so this only suits small scenes.

### Full Load Rendering Criteria

Whether full load rendering is used is determined at load time by the Use Full Load switch and the Level 0 splat count:

```text
Load(data file)
      │
      ▼
Read the Level 0 splat count
      │
      ▼
Use Full Load enabled? ── No ──▶ Chunked rendering
      │ Yes
      ▼
Splat count ≤ Full Load Splat Number? ── No ──▶ Chunked rendering
      │ Yes
      ▼
   Full load rendering
```

Both properties live under the Performance category in the Details panel of the Actor:

| Property | Description |
|----------|-------------|
| Use Full Load | Whether full load rendering is allowed |
| Full Load Splat Number | Upper limit of the Level 0 splat count that allows full load rendering (unit: 10,000, default 1500) |

Default values differ by Actor type:

| Actor | Use Full Load default | Reason |
|-------|-----------------------|--------|
| ALCCActor | Enabled | Level differences between nodes are obvious and node borders are visible, and every node costs one draw call, so full load rendering looks better in small scenes |
| ALCC2Actor / APlyActor / ASpzActor / ASogActor | Disabled | Level switching transitions continuously by screen-space error and node borders are subtle, and only one draw call is needed, so on-demand loading saves more video memory by default |

### When to Enable Full Load Rendering Manually on LCC2

The LCC2 pipeline disables full load rendering by default. Enable it manually in these cases:

- The scene is limited in size, video memory is sufficient, and both image quality and frame rate must stay stable
- The camera moves fast or teleports frequently, node loading cannot keep up, and holes or detail popping appear
- Product-level presentations, screen recordings, sequence rendering, and other cases where image changes during loading are unacceptable
- Density differences at node boundaries are still visible

If video memory becomes tight or the frame rate drops after enabling it, the scene has exceeded the range full load rendering suits, so switch back to chunked rendering.

> Note: full load rendering is still subject to the maximum render distance. Nothing is displayed when the distance between the camera and the data exceeds that value.

## Video Memory Budget and Capacity Limits

The LCC2 pipeline merges resident splat data into a single Buffer for rendering, so the amount of data a single model can load depends on the capacity limits of these Buffers.

### Buffer Structure and Per-splat Footprint

The data is stored in two Buffers:

| Buffer | Per-splat footprint | Content |
|--------|---------------------|---------|
| Gaussian base data Buffer | 40 bytes | Position, rotation, color, scale |
| Spherical harmonics Buffer | 48 bytes (up to band 3) | Spherical harmonics coefficients, allocated only for data that carries them |

### Hard Limit of a Single Buffer

Unreal records Buffer sizes as 32-bit integers, giving a theoretical limit of 4 GB. The plugin actually uses 4095 MB (1 MB less than 4 GB), and the reserved headroom keeps byte calculations from overflowing during upload. This limit applies to each Buffer separately and cannot be raised through settings.

### Video Memory Budget Setting

The video memory available to a single model is controlled by **LCC2 GPU Memory Budget (MB)** in ProjectSettings > Plugins > LCC4Unreal:

| Item | Value |
|------|-------|
| Default | 2048 MB |
| Adjustable range | 2048 ~ 8192 MB |
| Takes effect | Restart the editor after changing |

The budget is the total for both Buffers, not the limit of a single one. Data with spherical harmonics therefore counts as 88 bytes per splat (40 + 48), and data without them counts as 40 bytes.

### Additional Cost of the Sorting Buffer

Before 3DGS is rendered, splats are sorted by their distance to the camera. The sorting result is view-dependent, so each view needs its own set of sorting Buffers. This part is not counted in the budget above: each resident splat costs an extra 16 bytes per view (two copies each of the sort key and index, used for alternating reads and writes during sorting).

With the 2048 MB budget and data carrying spherical harmonics, roughly 24.4 million splats can stay resident, and the sorting Buffer for a single view is about 372 MB.

The left and right eyes of stereo rendering, each player in split screen, and each SceneCapture all count as one view. Sorting Buffers grow as the view count grows, and they do not shrink back after the view count drops, so reserve video memory for the largest view count that has occurred.

## Loading Limits of Single-file Formats

Single-file formats are `.ply` / `.spz` / `.sog`, meaning formats that hold the entire dataset in one file with no node subdivision or Level information. Such data cannot be loaded in parts: the plugin can only decode it as a single node in one pass and keep all of it resident in video memory, so there is a definite splat count limit. `.lcc` and `.lcc2` already complete node subdivision at export time and are not single-file formats.

### How the Limit Is Calculated

The plugin computes the limit before loading and rejects data that exceeds it:

| Budget | Without spherical harmonics | With spherical harmonics |
|--------|-----------------------------|--------------------------|
| 2048 MB (default) | About 53 million | About 24.4 million |
| 4096 MB | About 107 million | About 44.7 million |
| 8192 MB | About 107 million | About 44.7 million |

The limit is the smallest of the following three conditions, and the video memory budget is only one of them:

| Condition | Description |
|-----------|-------------|
| Video memory budget | The budget divided by the bytes per splat (40 B without spherical harmonics, 88 B with them) |
| Single Buffer limit | 4095 MB divided by the maximum stride (40 B without spherical harmonics, 48 B with them) |
| Spherical harmonics array capacity | Applies only with spherical harmonics; the decoded result uses 32-bit indices, giving a limit of about 44.7 million |

That is why the number stops growing past 4096 MB: without spherical harmonics the single Buffer limit applies, and with them the spherical harmonics array capacity applies. In addition, the video memory actually available on the graphics card lowers the loadable amount further.

> Note: known issue. The budget setting accepts values up to 8192 MB, but the loadable amount for single-file formats already reaches its limit at 4096 MB, so raising it further has no effect and produces no message. When data still exceeds the limit at a budget of 4096 MB, convert it to `.lcc2` or switch to data without spherical harmonics. This limitation will be improved in a later version.

### What Happens Beyond the Limit

When single-file data exceeds the limit, loading is rejected, and the Output Log prints the actual splat count, the current limit, and the splat count supported at the maximum budget. Handle it in this order of preference:

1. Convert to `.lcc2` so the plugin loads on demand instead of keeping everything resident.
2. Raise LCC2 GPU Memory Budget to 4096 MB and retry after a restart. This only helps when the budget is the current bottleneck; raising it to 8192 MB has no effect on single-file formats.
3. Switch to data without spherical harmonics, which drops the bytes counted against the budget from 88 to 40 per splat.

Switching to `.sog` or `.spz` does not help exceed the limit: the compression of these formats only applies to the file on disk, and every splat occupies exactly the same amount of video memory after decoding.

### The LCC2 Format Is Not Subject to This Limit

`.lcc2` splits data into multiple nodes, and each node is decoded and uploaded separately. Any single node stays far below the limits above, so the total amount of data has no hard limit and can significantly exceed the video memory budget. What is constrained are two runtime quantities:

| Constrained quantity | Source of the limit | Behavior when exceeded |
|----------------------|---------------------|------------------------|
| Splats resident at once | Video memory budget and single Buffer limit | Least recently used nodes are evicted and swapped in and out |
| Splats rendered in one frame | About 89 million (4095 MB ÷ 48 B) | Distant nodes are dropped by distance |

The per-frame render amount is also limited by Max Splat Num on the Actor, and the smaller of the two applies.

## Video Memory Management and Tuning

### Automatic Release When Video Memory Runs Short

At runtime the plugin keeps monitoring the actual video memory usage of the graphics card. When usage exceeds the percentage set by **Max GPU Usage Percentage For Release** in ProjectSettings, nodes are ranked by a combination of last render time, access frequency, data size, and Level, and the least recently used ones are released first. High-Level nodes are protected to avoid large holes when the view is turned quickly.

Related settings:

| Setting | Description |
|---------|-------------|
| Max GPU Usage Percentage For Release | Video memory usage percentage that triggers automatic release (50 ~ 100%) |
| GPU Release Percentage | Percentage of data released on each trigger (10 ~ 100%) |
| LCC2 GPU Memory Budget (MB) | Video memory budget for the splat data of a single model |

### Diagnosing and Adjusting an Insufficient Budget

A budget that is too small does not cause a load failure; instead the data is repeatedly swapped in and out. Common symptoms:

- Holes appear while the camera moves or turns, and fill in gradually after it stops
- The same area repeatedly sharpens and blurs while the view swings back and forth
- A local area stays blurry and does not sharpen even when approached
- The frame rate fluctuates with the amount of content in view and drops noticeably in open views
- Behavior is normal when the scene is static, but stutters when moving

Local blurriness is easily mistaken for poor data quality. When low-Level data cannot be loaded in time, the plugin fills the gap with the high-Level data already in video memory so no part of the image is missing, then replaces it once the low-Level data is ready. When insufficient video memory causes low-Level data to be evicted repeatedly, that area stays at a high Level. The test is whether it sharpens once approached: normally one or two seconds is enough to fill in, and if it never changes, the low-Level data cannot get into video memory.

Confirmation: check the predicted cost of the model printed in the Output Log at load time and compare it with the current budget. If the predicted cost of a single model approaches or exceeds the budget, the resident space is insufficient. The release count can also be observed with `stat LCC`; a value that keeps growing indicates frequent eviction.

Raising the budget only works if the graphics card has headroom. The budget is only the upper limit the plugin requests; once it exceeds the video memory actually available on the card, the driver swaps data to system memory, and the frame rate drop is worse than swapping in and out. Work through the following order:

1. Confirm the available video memory on the card. After adding the sorting Buffer cost, the budget should leave headroom for the engine itself and other resources.
2. Raise the budget step by step (2048 → 4096 → 8192) and check after each restart whether holes and frame rate improve.
3. If the improvement is small, the bottleneck is not the budget, so reduce the data volume instead: lower Max Splat Num, enable the Max Distance limit, or switch to data without spherical harmonics.

> Note: the budget is calculated independently per model. When several LCC Actors are placed in a scene, video memory usage accumulates with the number of models, so lower the budget per model accordingly.
