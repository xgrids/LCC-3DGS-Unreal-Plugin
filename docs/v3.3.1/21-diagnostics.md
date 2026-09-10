---
title: Logging and Diagnostics
parent: Documentation v3.3.1
nav_order: 21
description: How to read LCC4Unreal logs and use its debug tools, covering the meaning of the stat XGrids items, node bound and collision visualization, and a reference table of common log messages.
---

# 3DGS Rendering Logs and Debug Tools

This page covers how to obtain diagnostic information and how to read each tool. To locate a problem by symptom, see [Troubleshooting](./20-troubleshooting.md).

## Viewing the Plugin Log

The plugin log has two categories:

| Category | Source |
|----------|--------|
| `LogLCC` | LCC pipeline and common features (loading, collision, licensing, utility functions) |
| `LogLCC2` | Specific to the LCC2 pipeline |

### Viewing It in the Editor

Open `Window > Output Log`. While investigating, type `LogLCC` in the filter box to find the plugin messages quickly, or run the commands below in the console to raise the log level:

```text
log LogLCC Verbose
log LogLCC2 Verbose
```

The editor log file is located at:

```text
<project directory>/Saved/Logs/<project name>.log
```

### Viewing It in a Packaged Build

Launch a Development build with the `-log` parameter to bring up a console window. The log file is located at:

```text
<package directory>/<project name>/Saved/Logs/<project name>.log
```

Shipping builds print no log by default, so use a Development build for investigation.

> When reporting an issue, attach **the complete log file of the run where the problem occurred**, not just a few filtered plugin lines. Locating a problem often requires the context outside the plugin.

## Render Statistics: stat XGrids

Run `stat XGrids` in the console, or use the `Stats` button under the Actions category of the Actor Details panel.

Read it together with `stat unit` to first judge whether the bottleneck is the 3DGS.

### Count Items

The most frequently read items:

| Metric | Meaning |
|--------|---------|
| `Total Splats` | Total splat count of the dataset |
| `Level0 Splats` | Splat count of Level 0 (the highest precision) |
| `Current Render Splats` | **Splat count actually rendered in the current frame**, the main item for judging render load |
| `Current Render Main Splats` | Amount of main data rendered in the current frame |
| `Current Render Environment Splats` | Amount of environment data rendered in the current frame |
| `Total Nodes` | Total node count |
| `Current Render Nodes` | Node count rendered in the current frame |
| `Visible Node Num` | Visible node count |
| `LCC Draw Call` | Draw call count produced by the plugin |
| `LCC Sort Num` | Number of sorts |
| `Camera Num` | Number of cameras taking part in rendering |

How to use it: when `Current Render Splats` keeps sitting close to the limit of [Max Splat Num](./10-performance-parameters.md#max-splat-num), the count limit is already cutting things off and the image detail is reduced. Either raise the limit (performance for quality), or adjust [Level Factor](./10-performance-parameters.md#level-factor) to reduce nodes at the source.

The total data size can also be read from code, see [GetSplatNumber](./23-api-reference/02-ULCCComponentBase.md#getsplatnumber).

### Timing Items

Costs broken down by stage, used to locate which step is slow:

| Metric | Corresponding stage |
|--------|--------------------|
| `Render LCC` | Total plugin render cost |
| `Traversal Time` | Node traversal, which decides the nodes rendered this frame |
| `Determine Nodes Level` | Computing the precision Level of each node |
| `Load Data To CPU` | Reading data into memory |
| `Upload To GPU` | Uploading data to video memory |
| `Splat Sort` | Splat sorting (required for translucent rendering) |
| `Node Sort` | Node sorting |
| `Build Mesh Batch` | Building the draw batches |
| `Wait Node Ready` | Waiting for node data to be ready |
| `Component Update` | Component updates |
| `Update Camera Info` | Camera information updates |
| `Load Meta File` | Loading the meta information file |
| `Load Index Data` | Loading the index data (LCC pipeline) |
| `Load Collision Data` | Loading collision data |
| `Load Environment Data` | Loading environment data |
| `Update Collision` | Collision updates |
| `Get Physics Trimesh Data` | Generating the physics collision mesh |
| `Release Memory` | Memory release |
| `Preload Node` | Node preloading |
| `Get From Cache` | Fetching data from the cache |
| `Create Thread` | Thread creation |
| `Draw Node Box` | Node bound drawing (only while debug visualization is enabled) |

How to use it:

- When `Splat Sort` is high, the LCC pipeline can raise [Sort Factor](./10-performance-parameters.md#sort-factor) to sort less often
- When `Wait Node Ready` is high, data loading cannot keep up, possibly a slow disk or too few [threads](./10-performance-parameters.md#thread-pool-settings)
- When `Upload To GPU` is high, the amount uploaded per frame is large, so adjust [Level Factor](./10-performance-parameters.md#level-factor) or [Max Splat Num](./10-performance-parameters.md#max-splat-num)
- When `Update Collision` and `Get Physics Trimesh Data` are high, lower [Max Load Collision Distance](./10-performance-parameters.md#max-load-collision-distance)

For the complete tuning workflow, see [Performance Guide](./11-performance-guide.md).

### Memory and Video Memory Items

| Metric | Meaning |
|--------|---------|
| `CPU Usage` | Memory used by the plugin |
| `GPU Usage` | Video memory used by the plugin |
| `Position Data(For Raycast) Usage` | Usage of the position data for ray tests |
| `Collision Data Usage` | Usage of collision data |
| `CPU Occupy Percentage` | Percentage of memory used |
| `GPU Occupy Percentage` | Percentage of video memory used |

The percentage items correspond to the release thresholds in the project settings. Usage sitting close to [Max GPU Usage Percetage For Free](./10-performance-parameters.md#max-gpu-usage-percetage-for-free) for long periods means resource reclamation triggers frequently, which can cause stutter.

For the video memory allocation rules and budget limits, see [Rendering](./06-rendering.md#video-memory-budget-and-capacity-limits).

### Thread Items

| Metric | Meaning |
|--------|---------|
| `Collision Loader Thread Num` | Number of collision loader threads |
| `Exporter Thread Num` | Number of exporter threads |

## Node Bound Visualization

Toggle it with the `Debug Node Bound` button under the Actions category of the Actor Details panel, or run in the console:

```text
r.Xgrids.DrawNodeBox 1
```

This draws the currently loaded nodes as wireframe boxes. **The box color corresponds to the Level of the node**: red, orange, yellow, green, blue, and purple progress in order, where red is the lowest Level (the highest precision) and white is the highest Level (the lowest precision).

For the node and Level concepts, see [Rendering](./06-rendering.md#chunked-rendering-and-full-load-rendering).

How to read it:

- Red boxes still rendered in the distance mean the precision is too high and performance is wasted. Raise [Level Factor](./10-performance-parameters.md#level-factor)
- Cool-colored boxes everywhere up close mean the precision is pushed down too hard and the image looks blurry. Lower Level Factor or check [Start Level](./10-performance-parameters.md#start-level)
- A box count far above expectation means [Max Distance](./10-performance-parameters.md#max-distance) may be set too large

It only works in the editor and Development builds. The matching code interface is [DebugNodeBound](./23-api-reference/01-ALCCActorBase.md#debugnodebound).

## Collision Visualization

Toggle it with the `Show Collision` button under the Actions category of the Actor Details panel, or run in the console:

```text
r.xgrids.DrawCollision 1
```

This draws the wireframe of the loaded collision bodies.

How to read it:

- No wireframe at all: the data contains no collision file, or `bEnableCollision` is off, see [Collision Prerequisites](./15-collision.md#prerequisites)
- Wireframe only nearby: normal behavior, since collision streams in by distance and is limited by [Max Load Collision Distance](./10-performance-parameters.md#max-load-collision-distance)
- The wireframe is offset from the image content: the collision data does not match the render data, contact technical support

Enable this first when investigating line traces that miss or characters clipping through, to confirm whether collision was actually loaded. For the loading mechanism, see [Collision](./15-collision.md#loading-mechanism); the code interface is [ShowCollision](./23-api-reference/01-ALCCActorBase.md#showcollision).

## Frame Rate Display

The `Show FPS` button under the Actions category of the Actor Details panel is equivalent to running `stat fps` in the console. Read it together with `stat unit` to see the cost distribution across threads.

## Common Log Message Reference

Look up the meaning and action by message content. All of the following are the exact text the plugin outputs.

### Loading

| Log message | Meaning and action |
|-------------|-------------------|
| `LCC file :<path> does not exist.` | The path does not exist. Verify the path; relative paths are based on `Content` |
| `meta.lcc file :<path> load error,Please check.` | Parsing the LCC1 meta information file failed, the file may be corrupted |
| `Load Meta.lcc error,Please check your file!` | Same as above |
| `Read index.bin error,path:<path>.` | Reading the index file failed. Confirm `index.bin` exists and is not corrupted |
| `LCC4Unreal do not support this file format!` | The format is not supported, confirm the extension is within the supported range |
| `The data file:<path> does not exist,Please check your file!` | A data chunk file is missing, the data directory may be incomplete |

### Collision

| Log message | Meaning and action |
|-------------|-------------------|
| `There is neither collision.bin nor collision.lci in the folder:<path>, please check.` | There is no collision file in the data directory, so collision cannot be enabled for this data |
| `Failed to open file <path>` | Opening the collision file failed, check file permissions and integrity |
| `Read collision data error,path:<path>.` | Reading the collision data failed, the file may be corrupted |
| `Invalid indices in collision data!` | The collision data content is abnormal, contact technical support |

### Rendering

| Log message | Meaning and action |
|-------------|-------------------|
| `r.PostProcessing.PropagateAlpha is 0. LCC4Unreal requires this to be enabled for correct rendering.` | This option must be enabled, otherwise alpha blending is incorrect. The plugin also shows a prompt |
| `Unlicensed: enabled clipping volumes limited to <N> ...` | The free edition clipping volume count exceeds the quota and the excess is not rendered. See [Editions and Licensing](./05-pro-features.md) |
| `Unlicensed: enabled section planes limited to <N> ...` | Same as above, for the section plane count |

### GIS

| Log message | Meaning and action |
|-------------|-------------------|
| `This lcc does not have RTK information!` | The data contains no RTK geographic information, so geographic placement cannot be used |
| `MetaInfo's Offset has no 3 elements!` | The offset field in the meta information is abnormal, the data may have a problem |

### Licensing

| Log message | Meaning and action |
|-------------|-------------------|
| `ProjectID is invalid; generate one in Project Settings (Project/Description). Refusing authentication.` | The project has no Project ID, generate one in the project settings |
| `Failed to decode AppKey, please check.` | The AppKey content is incomplete, copy it again |
| `Invalid AppKey, please check.` | The AppKey format is wrong |
| `Authorization has expired, please check.` | The license expired, regenerate it |
| `AppKey has expired. Please generate a new one.` | Same as above |
| `Authentication Failed: <message>` | The failure reason returned by the server, act on the message content |
| `HTTP request failed` | The licensing server is unreachable, check the network |
| `HTTP error! Status: <code>` | The server returned an error status code |
| `Signature Verification Failed` | Signature verification failed, contact technical support |

### Proxy Mesh

| Log message | Meaning and action |
|-------------|-------------------|
| `has a null StaticMesh` | The proxy mesh Actor has no StaticMesh assigned and has no effect. Reported once per instance |

## What to Attach When Reporting an Issue

Collecting the items on this checklist speeds up diagnosis noticeably.

1. **Plugin version and engine version.** The plugin version is visible in the bottom-right corner of the plugin panel, and can also be found by searching LCC4Unreal in the `Edit > Plugins` window of the engine.
2. **The complete log file from when the problem occurred.** Provide the `.log` file itself, unfiltered and not just the single error line. The context around the log often contains the key information needed for diagnosis, and filtering loses those clues. For the log location, see [Viewing the Plugin Log](#viewing-the-plugin-log).
3. **Data information.** Format, approximate size (Total Splats), and whether it contains collision.
4. **Reproduction steps.** A minimal reproduction path starting from a new project is the most valuable.
5. **Hardware environment.** Graphics card model, driver version, and video memory size.
6. **Project ID.** Required for licensing issues; it is visible under `Project Settings > Project > Description`.

For contact details, see [Contact Us](./22-contact-us.md).
