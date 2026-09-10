---
title: Editions and Licensing
description: Feature differences between the LCC4Unreal free edition and Pro edition, plus how to register an App Key for a Pro license and verify the license status.
---

# Editions and Licensing: Free Edition vs Pro Edition

The plugin comes in a free edition and a Pro edition. The free edition loads and renders all supported data formats normally. Some features require registering an App Key to license the Pro edition before they can be used.

<div style="text-align: left; margin: 1.5em 0 0.5em;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-Pro-GCo1gAFd.jpg" alt="Feature differences between the free edition and the Pro edition" style="width: 100%; max-width: 100%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Feature differences between the free edition and the Pro edition</p>
</div>

## Licensed Features

| Feature | Free edition | Pro edition |
|------|-------|-------|
| [ProxyMesh normal mode](./13-proxy-mesh.md) | Unavailable, falls back to Fixed mode automatically | Available |
| [Working color space conversion](./07-visual-settings.md#post-processing) | Unavailable | Available |
| [Clipping volumes and section planes](./09-scene-editing.md) | Up to 50 of each type | Up to 65536 of each type |

Using a restricted feature without a license does not cause a load failure or an error. The plugin falls back to free edition behavior automatically.

### Working Color Space Conversion

Before 3DGS colors are composited back into the image, they go through the `SRGB → Linear → InverseACES → working color space` conversion chain. Converting to the engine Working Color Space is a Pro edition feature, and both pipelines are subject to this restriction.

Without a license, this step is skipped and the colors are composited without conversion. The engine default working color space is sRGB, where the results before and after conversion are identical, so the free edition and the Pro edition look the same. After the working color space is changed to a gamut other than sRGB (for example to match an ACES workflow or OpenColorIO), unlicensed 3DGS colors deviate from regular objects in the scene.

## Register an App Key

The free edition requires no registration, so this section can be skipped.

1. Click the LCC button on the editor toolbar to open the LCC4Unreal Panel.
2. Click to copy the AppId in the Pro area.
3. Go to the [XGrids developer platform](https://developer.xgrids.com) to request an AppKey.
4. Paste the received key in the App Key area.
5. Click Apply.

<div style="text-align: left; margin: 0.5em 0;">
  <img src="https://cdn-docs.xgrids.cloud/assets/02-get-appkey-DBc4zVOR.jpg" alt="Register the App Key in the LCC4Unreal panel" style="max-width: 50%; width: 50%;" />
  <p style="color: #666; font-size: 0.9em; margin-top: 0.5em;">Enter and apply the App Key in the LCC4Unreal panel</p>
</div>

The App Key is bound to the AppId and is saved in the project configuration after registration. The same project uses it on Windows, Linux, and Quest3, with no need to register again per platform.

> Note: confirm the license is still valid before packaging for release, otherwise the restricted features in the packaged output fall back to free edition behavior.

## Verify the License Status

Once the license takes effect, the Pro area of the LCC4Unreal Panel shows a licensed state. Whether a specific feature is available can also be confirmed at runtime through the API:

```cpp
// Confirm whether the ProxyMesh mode actually takes effect
const ELCC2NormalGenerationMode Mode = LCC2Component->GetEffectiveNormalGenerationMode();
```

This API returns the actual mode after the license check: `ProxyMesh` when licensed, `Fixed` when not. The property value itself is not modified.
