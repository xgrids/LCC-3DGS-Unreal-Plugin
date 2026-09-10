---
title: Quick Start - Linux
parent: Documentation v3.3.1
nav_order: 3
description: Install the LCC4Unreal plugin on Linux and render 3DGS scenes with Unreal Engine 5, covering source build compilation, C++ project creation, data loading, and Linux application packaging.
---

# Render a 3DGS Scene on Linux

This document starts from installing Unreal Engine and covers how to install LCC4Unreal, create a C++ project, load LCC data, compile, run, and package a Linux application.

## 1. Recommended Hardware and Software Environment

| Item | Recommended |
| --- | --- |
| Operating system | Ubuntu 22.04 |
| CPU | Intel/AMD quad-core 2.5 GHz or higher; more cores recommended for source builds |
| Memory | 32 GB; 64 GB or more recommended for source builds |
| Graphics card | NVIDIA GeForce RTX 2080 or a higher-performance discrete card |
| Video memory | 8 GB or more |
| IDE | VS Code or Rider |
| Disk | 300 GB and above |

## 2. Choose an Unreal Engine Installation Method

Two Unreal Engine installation methods are available on Linux:

| Method | Characteristics | Suitable for |
| --- | --- | --- |
| Prebuilt version (Installed Build) | Ready to use after download and full extraction; creating a C++ project is more straightforward | **Preferred for beginners** |
| Source build | Engine source can be modified; the first build takes a long time | Developers who need to modify the engine or plugin internals |

Prefer the prebuilt version when only LCC4Unreal is needed and the Unreal Engine source does not have to be modified.

> Note: LCC4Unreal ships in prebuilt form and matches the official engine binaries. Do not modify the engine source, as a modified engine may be incompatible with the plugin, causing load failures or abnormal behavior.

---

### 2.1 Method A: Install the Unreal Engine Linux Prebuilt Version (recommended)

1. Open the official Epic [Unreal Engine for Linux](https://www.unrealengine.com/linux) page;
2. Sign in with an Epic Games account;
3. Download the `.zip` archive of the required Unreal Engine version.

   <div style="text-align: left; margin: 0.5em 0;">
     <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-download-Epic-Dfa_AAYh.jpg" alt="Download the Unreal Engine Linux prebuilt version from the Epic Games page" style="max-width: 100%; width: 100%;" />
     <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select and download the required Unreal Engine Linux prebuilt package from the official Epic Games page</p>
   </div>

4. Create an engine directory in the file manager, for example:

   ```text
   /home/[USER]/Unreal/Engines/UnrealEngine
   ```

5. Extract all files and directories from the archive into that engine root directory. Do not copy only the single `UnrealEditor` file, otherwise the engine is missing the other content required to run and build;
6. Enter the engine root directory in the file manager, right-click an empty area, and select **Open in Terminal**. Subsequent commands must run in the correct engine directory.

   <div style="text-align: left; margin: 0.5em 0;">
     <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-setlocation-U0-ZWm7K.jpg" alt="Open a Linux terminal in the Unreal Engine root directory" style="max-width: 100%; width: 100%;" />
     <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Enter the fully extracted Unreal Engine root directory and open a terminal from there</p>
   </div>

7. Start the Unreal Engine editor.

General form:

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor"
</pre>

If a missing execute permission is reported:

```bash
chmod +x "/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
chmod +x "/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor"
</pre>

#### 2.1.1 Prepare the Toolchain for C++ Development

Following the Epic Games [Linux development quick start](https://dev.epicgames.com/documentation/unreal-engine/linux-development-quickstart-for-unreal-engine?application_version=5.7), run the following in the Unreal Engine directory:

```bash
cd "/[UE_ROOT]/Engine/Build/BatchFiles/Linux"
./SetupToolchain.sh
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
cd "/home/alice/Unreal/Engines/UnrealEngine/Engine/Build/BatchFiles/Linux"
./SetupToolchain.sh
</pre>

### 2.2 Method B: Build Unreal Engine from Source

Skip to section 4 if a source build of Unreal Engine already runs.

#### 2.2.1 Get the Source

Get it from GitHub: [https://github.com/EpicGames/UnrealEngine](https://github.com/EpicGames/UnrealEngine)

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-download-source-DIjGyJIl.jpg" alt="Get the Unreal Engine source from the Epic Games GitHub repository" style="width: 818px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Visit the Epic Games Unreal Engine GitHub repository and get the source of the required version</p>
</div>

#### 2.2.2 Download Dependencies and the Native Toolchain

```bash
cd "/[UE_ROOT]"
./Setup.sh
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
cd "/home/alice/Unreal/Engines/UnrealEngine"
./Setup.sh
</pre>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-download-deps-ChMEUMbM.jpg" alt="Run Setup.sh to download Unreal Engine source dependencies and the Linux toolchain" style="width: 692px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Use Setup.sh to download the dependencies required for a source build and the native Linux toolchain</p>
</div>

Success criteria:

- The command returns normally at the end;
- No `fatal` messages and no unhandled download failures;
- The matching Unreal Engine toolchain exists in `Engine/Extras/ThirdPartyNotUE/SDKs/HostLinux`.

If the following appears:

```text
fatal: not a git repository
```

the source origin, the current directory, or the Git metadata has a problem. Do not ignore it and continue compiling. Confirm the current terminal is in the correct Unreal Engine root directory first.

#### 2.2.3 Generate Project Files

```bash
cd "/[UE_ROOT]"
./GenerateProjectFiles.sh
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
cd "/home/alice/Unreal/Engines/UnrealEngine"
./GenerateProjectFiles.sh
</pre>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-generate-make-60XEUAo7.jpg" alt="Run GenerateProjectFiles.sh to generate Unreal Engine project files and Makefile" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Generate project files and the Linux Makefile for the Unreal Engine source</p>
</div>

#### 2.2.4 Compile UnrealEditor

Limit the parallel job count on the first build to avoid running out of memory. `make -j4 UnrealEditor` works by default; with only 32 GB of memory, use `make -j2 UnrealEditor` instead; if the engine Makefile provides no separate target, use `make -j4`:

```bash
cd "/[UE_ROOT]"
make -j4 UnrealEditor

# Use this with only 32 GB of memory:
make -j2 UnrealEditor

# Use this when the Makefile provides no separate target:
make -j4
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-compile-B6Tp_4_4.jpg" alt="Compile the UnrealEditor source target with make" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Limit the parallel job count and compile the source build of UnrealEditor</p>
</div>

Hundreds or even thousands of actions on the first source build are normal. They belong to the engine itself, not to the source files of the blank project. Build time depends on CPU, memory, and disk.

Check whether it succeeded:

```bash
test -x "/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor" \
  && echo "Source build UnrealEditor is ready" \
  || echo "Build not finished"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
test -x "/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor" \
  &amp;&amp; echo "Source build UnrealEditor is ready" \
  || echo "Build not finished"
</pre>

Start the project browser:

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor"
</pre>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-startue-BJGrNmhw.jpg" alt="Start the source build of UnrealEditor and open the Unreal Project Browser" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Start the Unreal Project Browser after the source build of UnrealEditor is compiled</p>
</div>

---

## 3. Install and Configure VS Code

### 3.1 Install VS Code and C++ Development Tools

On Ubuntu, download and install from the official [Visual Studio Code](https://code.visualstudio.com/) site, or use the package method supported by the distribution.

Before compiling C++ projects with VS Code, complete the Unreal Engine Linux toolchain installation from section 2 and prepare the basic C++ compilation and debugging tools. Install common tools such as `build-essential`, `clang`, `lldb`, and `cmake` according to the Ubuntu version and the team environment; the actual compiler version follows the requirements of the current Unreal Engine version and the toolchain installed by `SetupToolchain.sh`/`Setup.sh`.

The following VS Code extensions are recommended:

- **C/C++**: code browsing, completion, and debugging;
- **C/C++ Extension Pack**: additional common C++ development capabilities;
- **CodeLLDB**: LLDB debugging on Linux;
- **Makefile Tools**: view and run Makefile targets (optional).

The official Epic Games documentation can also be used as a reference: [Setting up VS Code for Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/setting-up-visual-studio-code-for-unreal-engine?application_version=5.7).

### 3.2 Generate and Open the VS Code Workspace

After a C++ project is created, the project directory normally contains `.vscode/` and `<ProjectName>.code-workspace`. If they are missing, run:

```bash
"/[UE_ROOT]/GenerateProjectFiles.sh" \
  -vscode \
  -project="/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject" \
  -game \
  -engine
```

In VS Code, select **File → Open Workspace from File...** and open `<ProjectName>.code-workspace`. Do not open a single `.cpp` file only.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-vscode-CwZLsBd3.jpg" alt="Open the code-workspace of an Unreal Engine project in VS Code" style="width: 386px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Open the generated .code-workspace to load the Unreal Engine C++ project configuration</p>
</div>

## 4. Create a C++ Project

### 4.1 Start Unreal Engine

Run from the terminal:

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor"
</pre>

### 4.2 Project Browser Settings

In the Unreal Project Browser:

1. Select **Games**;
2. Select **Blank**;
3. Set Project Defaults to **C++**;
4. Set Target Platform to **Desktop**;
5. For Quality Preset, select **Scalable** for a first test, or **Maximum** on a high-performance workstation;
6. Select Starter Content as needed;
7. Enter the project location:

   ```text
   /home/[USER]/Unreal/Projects
   ```

8. Enter the project name:

   ```text
   [PROJECT_NAME]
   ```

9. Click **Create**.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-create-project-FvC0q3tP.jpg" alt="Select Games and the Blank C++ template in the Unreal Project Browser" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select the Blank template in the Games category and set the project type to C++</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-create-project1-D1hjf0fZ.jpg" alt="Set the save location and project name of the Unreal Engine C++ project" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Set the project location, name, and platform options, then create the C++ project</p>
</div>

For the project name and path, use only:

- English letters;
- Digits;
- Underscores.

Do not use Chinese characters, spaces, `&`, brackets, or other special characters.

### 4.3 Prebuilt and Source Builds Behave Differently Here

#### Case A: Prebuilt Version (Installed Build)

The usual flow is:

```text
create project → auto-compile <ProjectName>Editor → open VS Code → open the new project editor
```

The old project browser window closes briefly, because Unreal Engine starts an editor process that loads the new `.uproject`.

#### Case B: Source Build

The usual flow is:

```text
create project → generate Source/, Makefile, .vscode/ → close project browser → open VS Code
```

A source build of Unreal Engine may not build and open the new project automatically, and instead expects the developer to compile it first in the IDE or terminal. **This is not a VS Code configuration error and does not mean project creation failed.**

At this point the project normally contains only:

```text
.vscode/
Config/
Content/
Intermediate/
Saved/
Source/
Makefile
[PROJECT_NAME].code-workspace
[PROJECT_NAME].uproject
```

Not yet present:

```text
Binaries/Linux/
```

This means the project modules have not been compiled.

## 5. First Compile Without the Plugin

Compile the blank project once without LCC4Unreal installed to confirm Unreal Engine, the toolchain, and the project itself work.

### 5.1 Compile with VS Code (recommended)

1. Generate and open `<ProjectName>.code-workspace` as described in section 3.2;
2. Open **Run and Debug** in the VS Code sidebar;
3. Select **`<ProjectName>Editor (Development)`** in the configuration list at the top;
4. Click the launch button to run the first compile;
5. Wait for the build to finish and confirm there are no C++ compilation errors. A long first build is normal.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-build-lBhplMP-.jpg" alt="Compile the Editor Development configuration of the project in VS Code Run and Debug" style="width: 856px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Select the &lt;ProjectName&gt;Editor (Development) configuration and complete the first C++ compile of the project</p>
</div>

After a successful build, VS Code can launch the project editor directly.

### 5.2 Optional: Use the Project Makefile

```bash
cd "/[UE_PROJECTS_DIR]/[PROJECT_NAME]"
make "[PROJECT_NAME]Editor"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
cd "/home/alice/Unreal/Projects/Test2"
make "Test2Editor"
</pre>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-compile-project-BzzOzj-_.jpg" alt="Compile the Editor target with the project Makefile in a Linux terminal" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Compile the &lt;ProjectName&gt;Editor target through the project Makefile</p>
</div>

### 5.3 Open the Project After a Successful Compile

Graphical method: find `<ProjectName>.uproject` in the file manager, right-click it, and select **Open With Unreal Engine Editor**. This method applies when the system already associates the target Unreal Engine version correctly.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-start-project-jbiIx-JN.jpg" alt="Right-click to open a uproject with Unreal Engine Editor in the Linux file manager" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Right-click the project .uproject file and select to open it with Unreal Engine Editor</p>
</div>

Command line method: specify the full paths of both UnrealEditor and the `.uproject`. This is the more reliable way to open a project when multiple Unreal Engine versions coexist:

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor" "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor" "/home/alice/Unreal/Projects/Test2/Test2.uproject"
</pre>

Once the project opens in the editor, close the editor and continue with the plugin installation.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-open-editor-CAsbH1Ek.jpg" alt="Unreal Engine opening the specified C++ project" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Load the project with the specified Unreal Engine version</p>
</div>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-editor-ui--hXTuGpq.jpg" alt="C++ project successfully opened in the Unreal Engine editor main interface" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The blank C++ project compiles successfully and opens in the Unreal Engine editor</p>
</div>

## 6. Install LCC4Unreal

Download the plugin from the XGRIDS developer platform:

- Plugin download entry: [developer.xgrids.com](https://developer.xgrids.com)

### 6.1 Close Related Programs

Before copying the plugin:

1. Save the project;
2. Close the Unreal Engine editor;
3. Wait for the UnrealEditor background process to exit;
4. VS Code can stay open, but do not build the project at the same time.

Check whether UnrealEditor is still running:

```bash
pgrep -a UnrealEditor
```

No output means the editor has exited.

### 6.2 Copy the Complete Plugin into the Project

1. Create a `Plugins` directory in the project root:

   ```bash
   mkdir -p "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/Plugins"
   ```

   <pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
   mkdir -p "/home/alice/Unreal/Projects/Test2/Plugins"
   </pre>

2. Extract the downloaded plugin package and copy the **complete `LCC4Unreal` folder** into the project. The final directory must be:

   ```text
   [PROJECT_ROOT]/Plugins/LCC4Unreal/
   ```

   Do not copy only `Binaries`, `Content`, or individual files, and do not create a duplicated nested directory such as `Plugins/LCC4Unreal/LCC4Unreal/`.

3. Confirm the plugin descriptor file exists. The directory must contain:

   ```text
   [PROJECT_ROOT]/Plugins/LCC4Unreal/LCC4Unreal.uplugin
   ```

   If the actual `.uplugin` file name changes between release packages, confirm it sits directly under `Plugins/LCC4Unreal/` and not one level deeper.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-plugins-copy-BF2n5xuE.jpg" alt="Copy the complete LCC4Unreal plugin folder into the project Plugins directory" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Copy the complete LCC4Unreal folder to [PROJECT_ROOT]/Plugins/LCC4Unreal and check the .uplugin file</p>
</div>

### 6.3 Restart the Project

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor" "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor" "/home/alice/Unreal/Projects/Test2/Test2.uproject"
</pre>

This command specifies both which Unreal Engine to use and which project to open. When multiple Unreal Engine versions coexist, it is more reliable than double-clicking the `.uproject` and does not require registering one version as the global default.

### 6.4 What to Do When a Rebuild Prompt Appears

The prompt looks like:

```text
The following modules are missing or built with a different engine version.
Would you like to rebuild them now?
```

It means the project dynamic libraries do not exist or do not match the current Unreal Engine version.

- Select **Yes/Rebuild** when the full Unreal Engine root path in the command is confirmed correct;
- Finishing quickly and opening the editor normally indicates an incremental build of the project modules;
- If the prompt appears every time, check whether a different engine is used each time, whether `Binaries/Linux` is writable, and whether the system time is correct;
- In a multi-engine environment, dismiss the prompt first and then build explicitly with the `Build.sh` command in section 13.

## 7. Choose the Right Actor for the Data Format

Different data formats require the matching Actor:

| Data format | Actor to search for in Unreal Engine | C++ class name |
| --- | --- | --- |
| `.lcc2` | `LCC2Actor` | `ALCC2Actor` |
| `.lcc` | `LCC Actor` | `ALCCActor` |
| `.sog` | `SogActor` | `ASogActor` |
| `.spz` | `SpzActor` | `ASpzActor` |
| `.ply` | `PlyActor` | `APlyActor` |

The steps below use `.lcc2` data and `LCC2Actor` as the example. For other formats, use the matching Actor from the table.

### 7.1 Add LCC2Actor and Load Data

1. Click to add `LCC2Actor` in the LCC4Unreal panel.

   <div style="text-align: left; margin: 0.5em 0;">
     <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-add-lccactor-IAqAwIct.jpg" alt="Add LCC2Actor to the scene in the LCC4Unreal panel" style="width: 624px; max-width: 100%;" />
     <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Add LCC2Actor in the LCC4Unreal panel to match the .lcc2 data format</p>
   </div>

2. Select `LCC2Actor` in the World Outliner;
3. Find **Actions** in the Details panel;
4. Click **Load**;
5. Select the `.lcc2` data file.

   <div style="text-align: left; margin: 0.5em 0;">
     <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-loadlcc2-D8QksE9C.jpg" alt="Select and load lcc2 data in the Actions panel of LCC2Actor" style="max-width: 100%; width: 100%;" />
     <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Load the .lcc2 data file from the Details panel of LCC2Actor</p>
   </div>

6. Wait for the data to finish loading.

   <div style="text-align: left; margin: 0.5em 0;">
     <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-loadlcc2-scene-DsPU0yvs.jpg" alt="lcc2 data loaded successfully and displayed in the Unreal Engine scene" style="max-width: 100%; width: 100%;" />
     <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The .lcc2 data appears in the Unreal Engine viewport once loading finishes</p>
   </div>

The editor may become temporarily unresponsive while loading large data. Watch memory, video memory, and the Output Log instead of force-closing it immediately.

## 8. Include Non-Asset LCC Data in Packaging

`.lcc2`, `.lcc`, `.sog`, `.spz`, and `.ply` are normally not Unreal Engine `.uasset` files. Placing the files in `Content` alone does not include them in the installation package.

Recommended directory:

```text
/[UE_PROJECTS_DIR]/[PROJECT_NAME]/Content/LCCData/
```

Open:

```text
Edit → Project Settings → Packaging
```

Find:

```text
Additional Non-Asset Directories To Copy
```

Add:

```text
LCCData
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-setcontent-lcc2data-DqAXGlPJ.jpg" alt="Add LCCData to Additional Non-Asset Directories To Copy in Packaging" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Configure Content/LCCData as a non-asset directory copied during packaging</p>
</div>

Then make sure the relative path loaded by the LCC Actor or LCC2Actor matches the packaged directory.

### 8.1 Set the Startup Map

Open:

```text
Edit → Project Settings → Maps & Modes
```

Under **Default Maps**, set **Game Default Map** to the target level that contains the LCC Actor/LCC2Actor and the configured data path. Save the level and the project configuration afterwards, so the packaged application does not start in a blank map.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-start-level-BGRhPGKL.jpg" alt="Set the Game Default Map of the Unreal Engine project in Maps and Modes" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Set the Game Default Map used at startup of the packaged application under Default Maps in Maps &amp; Modes</p>
</div>

## 9. Pre-Packaging Checklist

Confirm each item before packaging:

- [ ] The correct Unreal Engine version is used;
- [ ] The LCC4Unreal plugin package matches the Unreal Engine version and the Linux architecture;
- [ ] `<ProjectName>Editor Linux Development` compiles successfully;
- [ ] The plugin is enabled;
- [ ] The Pro license is valid (required only when Pro edition features are used, see [Editions and Licensing](./05-pro-features.md));
- [ ] The LCC data is visible in editor Play;
- [ ] `Game Default Map` is set;
- [ ] The data to package is added to Additional Non-Asset Directories To Copy;
- [ ] The disk has enough space;
- [ ] The output directory is writable;
- [ ] All levels and assets are saved;
- [ ] Build a Development package first, then try Shipping after it succeeds.

---

## 10. Package for Linux with Project Launcher

### 10.1 Open Project Launcher

In the Unreal Engine editor, select:

```text
Platforms → Project Launcher
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-project-launcher-DeOOEMJy.jpg" alt="Open Project Launcher from the Unreal Engine Platforms menu" style="width: 591px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Open Project Launcher from the Platforms menu</p>
</div>

Click:

```text
Add → Create Custom Profile
```

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-launch-profile-1-Cx1J2_Lf.jpg" alt="Create a custom launch profile in Project Launcher" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Create the Linux packaging profile through Add → Create Custom Profile</p>
</div>

### 10.2 Configure the Profile

1. In the custom profile, select **Linux** as the target platform and set the content packaging scheme to **Pak** file;
2. Enable archiving of the build result and select a writable archive directory. That directory is where the distributable version is actually saved after packaging.

   <div style="text-align: left; margin: 0.5em 0;">
     <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-launch-profile-2-CBfGO3Td.jpg" alt="Select Linux and Pak and set the archive directory in the Project Launcher custom profile" style="max-width: 100%; width: 100%;" />
     <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Set the target platform to Linux, select Pak file, enable archiving, and specify the packaging output directory</p>
   </div>

3. Click the launch icon to the right of the custom profile.

Packaging normally includes:

```text
Build → Cook → Stage → Package → Archive
```

The first packaging run is slower than an incremental one. Do not force-stop it because it stays at one percentage for a short while. On success, output similar to the following appears:

```text
BUILD SUCCESSFUL
```

### 10.3 If It Keeps Showing "Querying"

Check the following in order:

1. Wait for Unreal Engine to finish platform and SDK detection;
2. Open **Window → Developer Tools → Output Log**;
3. Search for `Turnkey`, `SDK`, `Linux`, `Error`;
4. Confirm the current host is Linux and the target platform is also Linux;
5. Confirm the dependencies and toolchain of a source build of Unreal Engine were installed by `Setup.sh`;
6. Confirm `SetupToolchain.sh` was run for the prebuilt version;
7. Close duplicated Project Launcher windows and reopen it;
8. Do not build the same project in another terminal at the same time.

## 11. Package from the Command Line (optional)

Once graphical packaging is stable, RunUAT can be used:

```bash
mkdir -p "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/Packaged"

"/[UE_ROOT]/Engine/Build/BatchFiles/RunUAT.sh" BuildCookRun \
  -project="/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject" \
  -noP4 \
  -targetplatform=Linux \
  -clientconfig=Development \
  -build \
  -cook \
  -stage \
  -pak \
  -archive \
  -archivedirectory="/[UE_PROJECTS_DIR]/[PROJECT_NAME]/Packaged"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
mkdir -p "/home/alice/Unreal/Projects/Test2/Packaged"

"/home/alice/Unreal/Engines/UnrealEngine/Engine/Build/BatchFiles/RunUAT.sh" BuildCookRun \
  -project="/home/alice/Unreal/Projects/Test2/Test2.uproject" \
  -noP4 \
  -targetplatform=Linux \
  -clientconfig=Development \
  -build \
  -cook \
  -stage \
  -pak \
  -archive \
  -archivedirectory="/home/alice/Unreal/Projects/Test2/Packaged"
</pre>

With `-archive` and `-archivedirectory`, look for the final runnable version in the specified archive directory instead of assuming it sits in `Saved/StagedBuilds/Linux`.

If the project uses specific maps, add map restrictions according to the project settings and the command parameters of the current Unreal Engine. Beginners should use Project Launcher first, to avoid incomplete packages caused by missing command parameters.

## 12. Run the Packaged Result

### 12.1 Run the Archived Result

If archiving was enabled in Project Launcher, or RunUAT used `-archive -archivedirectory="..."`, go to the archive directory that was actually configured. Different Unreal Engine versions may create an additional `Linux` subdirectory there, so locate the project launch script first and then run it:

```bash
cd "/[ARCHIVE_DIR]"
find . -maxdepth 3 -type f -name "*.sh" -print

chmod +x "./[SUBDIR]/[PROJECT_NAME].sh"
"./[SUBDIR]/[PROJECT_NAME].sh"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example (archived result in Packaged/Linux):
cd "/home/alice/Unreal/Projects/Test2/Packaged"
find . -maxdepth 3 -type f -name "*.sh" -print
chmod +x "./Linux/Test2.sh"
"./Linux/Test2.sh"
</pre>

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-package-done1-BrC0LRZT.jpg" alt="View and launch the archived packaging result in the Linux file manager" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Enter the archive directory configured in Project Launcher or RunUAT and launch the project script</p>
</div>

### 12.2 Run the Unarchived Staged Output

Run from the following directory only when archiving was not performed and the staged result needs to be checked directly:

```bash
cd "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/Saved/StagedBuilds/Linux"
chmod +x "./[PROJECT_NAME].sh"
"./[PROJECT_NAME].sh"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
cd "/home/alice/Unreal/Projects/Test2/Saved/StagedBuilds/Linux"
chmod +x "./Test2.sh"
"./Test2.sh"
</pre>

Check after running:

- It starts;
- The default map is correct;
- The LCC data is visible;
- No lingering processes remain after the application is closed.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/03-linux-run-program-D0VesxuP.jpg" alt="Run the packaged Linux application showing the loaded LCC scene" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Launch the packaged Linux application and verify the default map, the LCC data, and the licensed features</p>
</div>

## 13. Command Reference

`[UE_ROOT]`, `[UE_PROJECTS_DIR]`, `[PROJECT_NAME]`, and `[ARCHIVE_DIR]` in the commands below are placeholders. Replace them with the actual full paths and names before running.

Start the project browser:

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor"
</pre>

Generate VS Code project files:

```bash
"/[UE_ROOT]/GenerateProjectFiles.sh" \
  -vscode \
  -project="/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject" \
  -game \
  -engine
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/GenerateProjectFiles.sh" \
  -vscode \
  -project="/home/alice/Unreal/Projects/Test2/Test2.uproject" \
  -game \
  -engine
</pre>

Compile the project Editor target:

```bash
"/[UE_ROOT]/Engine/Build/BatchFiles/Linux/Build.sh" \
  "[PROJECT_NAME]Editor" Linux Development \
  -Project="/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject" \
  -WaitMutex
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Build/BatchFiles/Linux/Build.sh" \
  "Test2Editor" Linux Development \
  -Project="/home/alice/Unreal/Projects/Test2/Test2.uproject" \
  -WaitMutex
</pre>

Open a specific project with a specific Unreal Engine:

```bash
"/[UE_ROOT]/Engine/Binaries/Linux/UnrealEditor" "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/[PROJECT_NAME].uproject"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example:
"/home/alice/Unreal/Engines/UnrealEngine/Engine/Binaries/Linux/UnrealEditor" "/home/alice/Unreal/Projects/Test2/Test2.uproject"
</pre>

Run the archived result:

```bash
cd "/[ARCHIVE_DIR]"
find . -maxdepth 3 -type f -name "*.sh" -print
chmod +x "./[SUBDIR]/[PROJECT_NAME].sh"
"./[SUBDIR]/[PROJECT_NAME].sh"
```

Run the unarchived staged output:

```bash
cd "/[UE_PROJECTS_DIR]/[PROJECT_NAME]/Saved/StagedBuilds/Linux"
chmod +x "./[PROJECT_NAME].sh"
"./[PROJECT_NAME].sh"
```

<pre style="color: #666; font-family: inherit; font-size: 0.9em; line-height: 1.6; margin: 0.5em 0; white-space: pre-wrap; overflow: visible; background: transparent; border: 0; padding: 0;">Example (unarchived staged output):
cd "/home/alice/Unreal/Projects/Test2/Saved/StagedBuilds/Linux"
chmod +x "./Test2.sh"
"./Test2.sh"
</pre>
