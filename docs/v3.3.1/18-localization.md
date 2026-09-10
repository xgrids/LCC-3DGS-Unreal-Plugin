---
title: Localization
description: Language settings for the LCC4Unreal plugin interface, which can follow the editor language or force English.
---

# Plugin Interface Language Settings

> Localization

## Overview

The LCC4Unreal plugin UI supports **6 languages**:

| Language | Localization directory |
|----------|-----------------------|
| English | `Content/Localization/XGrids/en/` |
| Simplified Chinese | `Content/Localization/XGrids/zh-Hans/` |
| Japanese | `Content/Localization/XGrids/ja/` |
| Korean | `Content/Localization/XGrids/ko/` |
| French | `Content/Localization/XGrids/fr/` |
| German | `Content/Localization/XGrids/de/` |

Localization covers the plugin panels (Quick Add, licensing, Helper, About), the DisplayName and ToolTip in the property panel, and the description text of console commands.

## Configuration

Select the language preference in the Lcc4Unreal panel of the editor:

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/16-Localization-Language-5lL04o_8.jpg" alt="LCC4Unreal language preference setting" style="width: 382px; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Selecting the language preference in the LCC4Unreal panel</p>
</div>

| Option | Enum value | Description |
|--------|-----------|-------------|
| **Follow Editor** (default) | `ELanguagePreference::FollowEditor` | Follows the current editor language setting. Falls back to English when the editor language is not in the supported list |
| **Always English** | `ELanguagePreference::AlwaysEnglish` | The plugin UI always shows English regardless of the editor language |

### Result of Follow Editor Mode

With the editor language set to Chinese, the plugin panel shows Chinese automatically:

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/16-Localization-Follow-Editor-DWyRMK6r.jpg" alt="Chinese interface following the editor language" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The plugin interface shown in Chinese under Follow Editor mode</p>
</div>

### Result of Always English Mode

Even with the editor in Chinese, the plugin panel still shows English:

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/16-Localization-AlwaysEnglish-BXkK9Rje.jpg" alt="Plugin interface always in English" style="max-width: 100%; width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">The plugin interface always shown in English under Always English mode</p>
</div>
