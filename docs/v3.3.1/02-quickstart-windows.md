---
title: Quick Start - Windows
parent: v3.3.1 (latest)
nav_order: 2
description: Complete steps for installing the LCC4Unreal plugin on Windows and rendering a 3DGS scene in Unreal Engine 5, covering engine installation, plugin deployment, data loading, and project packaging.
---

# Render a 3DGS Scene on Windows

Render a 3DGS scene in Unreal Engine 5, starting from scratch.

## Environment Setup

### Step 1: Install Unreal Engine

1. Download and install the [Epic Games Launcher](https://www.unrealengine.com/download)

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-download-epic-DiSEf6FX.jpg" alt="Epic Games Launcher download page" style="width: 100%; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Download and install the Epic Games Launcher from the official site</p>
</div>

2. Start Epic Games, select Unreal Engine, and click Library

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-download-Select-C1Dqrx9k.jpg" alt="Open the Unreal Engine library" style="width: 100%; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Open the Unreal Engine library in the Epic Games Launcher</p>
</div>

3. Click "+", select an engine version, and click Install to start the installation

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-download-ue-CQVJBg08.gif" alt="Select a version and install Unreal Engine" style="width: 100%; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select an engine version and install Unreal Engine</p>
</div>

### Step 2: Install Visual Studio (required for C++ projects)

Install Visual Studio when creating a C++ project or building a Shipping package:

1. Download [Visual Studio 2022](https://visualstudio.microsoft.com/)
2. [Unreal Engine Visual Studio setup documentation](https://dev.epicgames.com/documentation/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine?application_version=5.7)

> Blueprint-only projects do not require Visual Studio, so this step can be skipped. Packaging still requires a C++ project.

### Step 3: Create a Project

1. Start Unreal Engine and click new project

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-createproject-startengine-CXPZvoHu.jpg" alt="Start Unreal Engine and create a new project" style="max-width: 70%; width: 70%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Start Unreal Engine and create a new project</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-create-project-HfJPT__x.jpg" alt="Create a new project in the project browser" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Create a new project in the Unreal Engine project browser</p>
</div>

2. Select a template

3. Select the **C++** or **Blueprint** project type

4. Select the project directory and click Create

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-create-project2-CJSShlfB.jpg" alt="Configure project type, name, and save directory" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Configure project type, name, and save directory</p>
</div>

> - Some templates do not support C++ projects.
> - With the LCC Unreal Plugin, Blueprint projects work in the editor only and cannot be packaged.
> - Use English characters for the C++ project path and name.

5. For a C++ project, wait until the Visual Studio project files are generated

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-project-directory-CHaRQL4J.jpg" alt="Project directory after C++ project generation" style="width: 696px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Project directory structure after C++ project generation</p>
</div>

### Step 4: Install the Plugin

1. Close the editor
2. Create a `Plugins` folder in the project root (skip if it already exists)
3. Copy the entire downloaded LCC4Unreal plugin folder into `Plugins/`

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-plugins-folder-CI6nOilG.jpg" alt="LCC4Unreal plugin in the project Plugins directory" style="width: 639px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Copy the LCC4Unreal plugin into the project Plugins directory</p>
</div>

4. Restart the editor
5. Confirm the plugin is enabled: Edit > Plugins > search for "LCC4Unreal"

> - **C++ project note**: on the first startup after adding the plugin, the engine detects the new module and prompts for a rebuild. If startup fails, do the following:
> - In Visual Studio, right-click the project → Generate Project Files → rebuild

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-delete-intermediate-Djnv363F.jpg" alt="Prepare to regenerate project files after deleting Intermediate" style="max-width: 70%; width: 70%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Regenerate project files after deleting the Intermediate directory</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-regenerate-project-B4XM8baE.jpg" alt="Regenerate the Visual Studio project files" style="max-width: 70%; width: 70%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Regenerate the Visual Studio project files</p>
</div>

## Hardware Requirements

| Hardware | Recommended |
|------|----------|
| CPU | Intel i7 6th generation or AMD Ryzen 5 1600 and above |
| GPU | NVIDIA GTX 2060 and above (DX11/SM5 support required) |
| Memory | 16GB DDR4 3200MHz and above |
| Video memory | 4GB and above (8GB+ recommended for large scenes) |
| Disk | SSD recommended |

## Register an App Key

Register an App Key before using Pro edition features. Skip this step when using only free edition features. See [Editions and Licensing](./05-pro-features.md).

## Workflow

### Step 1: Place the Actor

**Quick Add panel:**

1. Click the LCC button on the toolbar to open the panel
2. Select "LCC2Actor" in the Quick Add area

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-quickadd-lccactor-wMiiPmtt.jpg" alt="Add LCC2Actor through Quick Add" style="max-width: 50%; width: 50%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Add LCC2Actor through the Quick Add panel</p>
</div>

### Step 2: Load Data

1. Select the Actor
2. Click **Load** under the **Actions** category in the Details panel, then select the data file in the file dialog
3. `DefaultLoadPath` can also be set directly:
   - Absolute path: `D:\data\scene\meta.lcc2`
   - Relative path (relative to the Content directory): `Scenes/Tower/meta.lcc2`

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-load-lcc-path-DUzq539v.jpg" alt="Set the LCC data load path in the Details panel" style="max-width: 50%; width: 50%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Set the LCC data load path in the Details panel</p>
</div>

#### Load Through Blueprint

```text
LCC2Actor->Load("D:/data/scene/meta.lcc2")
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-load-bplcc-path-BMmgzjWi.jpg" alt="Load LCC data with the Blueprint Load node" style="max-width: 50%; width: 50%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Load LCC data with the Blueprint Load node</p>
</div>

#### Load in C++

Add the module dependency in build.cs:

```csharp
PublicDependencyModuleNames.Add("LCC4UnrealRuntime");
```

```cpp
#include "LCCActorBase.h"
#include "LCC2Component.h"

MyActor->Load(TEXT("D:/data/scene/meta.lcc2"));
```

#### Unload Data

```cpp
MyActor->UnLoad();
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/11-bp-unload-BPncgr9D.jpg" alt="Unload LCC data with the Blueprint UnLoad node" style="width: 489px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Unload LCC data through the Blueprint UnLoad node</p>
</div>

Unloading releases all memory and video memory resources held by the Actor.

### Step 3: Adjust Display

| Property | Default | Description |
|------|--------|------|
| RenderMode | `3DGS` | `3DGS` (default) or `Point Cloud` |
| LightMode | `Unlit` | `Unlit` (default) or `Lit` |
| SplatScale | 1.0 | Splat size, 0.001~1.0 |
| GlobalAlpha | 1.0 | 3DGS opacity, 0.0~1.0 |

### Step 4: Run

Press Play (Alt+P) to render in real time.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-Play-Rander-CmDwNyvE.jpg" alt="3DGS scene rendering after running the project" style="max-width: 50%; width: 50%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Real-time 3DGS scene rendering after running the project</p>
</div>

## Release Packaging

### Configure Packaging

1. **Set the startup level**

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-set-startup-level-C3M19CfV.jpg" alt="Open the default map settings of the project" style="width: 727px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Open the default map settings in the project settings</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-select-level-DsT44ovY.jpg" alt="Select the editor and game startup levels" style="max-width: 70%; width: 70%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select the startup levels used by the editor and the game</p>
</div>

2. **Select a build configuration**

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-build-config-DjSIiK8e.jpg" alt="Select the Development or Shipping build configuration" style="max-width: 70%; width: 70%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select the Development or Shipping build configuration</p>
</div>

| Configuration | Use case |
|------|----------|
| Development | Development and debugging, keeps logs and statistics |
| Shipping | Final delivery, best size and performance |

3. **Configure the data directory** (when using relative paths)

Add the LCC data directory under `ProjectSettings > Packaging`. Two settings are available, chosen by distribution method:

| Setting | Data form | Use case |
|-------|---------|---------|
| `Additional Non-Asset Directories To Copy` | Copied into the packaged output as plain files | Data stays visible and replaceable in the output directory |
| `Additional Non-Asset Directories To Package` | Packed into the pak file | Data ships inside the pak instead of being exposed as separate files |

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-package-directory-Bcc_QGPC.jpg" alt="Add the non-asset data directory to copy" style="max-width: 80%; width: 80%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Add the LCC data directory to the non-asset copy list</p>
</div>

> When neither is configured, the packaged output contains no LCC data and runtime loading fails.

4. **Run packaging**

`Platforms > Windows > Package Project`

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-package-build-Dqlp2RpX.jpg" alt="Run Windows packaging from the Platforms menu" style="width: 596px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Run Windows project packaging from the Platforms menu</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-package-target-CE3oaa0E.jpg" alt="Select the Windows packaging output directory" style="width: 284px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select the packaging output directory for the Windows project</p>
</div>


## Notes

Full Rebuild in the project settings and Rebuild in Visual Studio are not supported for packaging.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-build-ProjectRebuild-CPLv_Pn8.jpg" alt="Do not enable Full Rebuild when packaging" style="width: 589px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Do not enable Full Rebuild or run Rebuild when packaging</p>
</div>



## Linux Platform

See [Quick Start - Linux](./03-quickstart-linux.md).

## VR (Quest3 Streaming)

See [Quick Start - Quest3](./04-quickstart-quest3.md).
