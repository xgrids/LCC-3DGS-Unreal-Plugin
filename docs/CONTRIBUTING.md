---
title: Maintaining these docs
nav_order: 99
description: How the documentation site is organised and how to add a new version.
---

# Maintaining these docs

The site is built by GitHub Pages from the `docs/` folder on `main`, using the [just-the-docs](https://just-the-docs.com/) theme via `remote_theme`. There is no build step to run locally — pushing to `main` triggers a rebuild.

## Layout

```
docs/
├── _config.yml           site config, theme, search, aux links
├── README.md             site home (nav_order: 1)
├── CONTRIBUTING.md       this page (nav_order: 99)
└── v3.3.1/
    ├── README.md         version hub, has_children: true
    ├── 01-introduction.md … 22-contact-us.md
    ├── 23-api-reference/
    │   ├── README.md     has_children: true
    │   └── 01-… … 11-structs.md
    └── 24-changelog/
        ├── README.md     has_children: true
        └── 49-v3.3.1.md … 65-v0.0.1.md
```

Sidebar structure comes entirely from front matter, not from folder nesting. Every page needs it.

## Front matter rules

Top-level page inside a version:

```yaml
---
title: Rendering
parent: v3.3.1 (latest)
nav_order: 6
---
```

Page inside a subsection:

```yaml
---
title: ALCCActorBase
parent: API Reference
grand_parent: v3.3.1 (latest)
nav_order: 1
---
```

Section index (a page with children):

```yaml
---
title: API Reference
parent: v3.3.1 (latest)
nav_order: 23
has_children: true
permalink: /v3.3.1/23-api-reference/
---
```

Two things to watch:

- `parent` and `grand_parent` match against the **`title` string**, not the filename or path. If a title changes, every page referencing it must change too.
- Titles must be unique across the site. A changelog page titled `v3.3.1` under a hub also titled `v3.3.1` breaks the tree — that is why the hub is `v3.3.1 (latest)`.

## Adding a new version

Say v3.4.0 ships:

1. Copy the folder: `docs/v3.3.1` → `docs/v3.4.0`
2. In `docs/v3.4.0/README.md` set:
   ```yaml
   title: v3.4.0 (latest)
   nav_order: 9          # lower than the previous latest, so it sorts above
   has_children: true
   permalink: /v3.4.0/
   ```
3. In every page under `docs/v3.4.0/`, change `parent:` / `grand_parent:` from `v3.3.1 (latest)` to `v3.4.0 (latest)`
4. In the old `docs/v3.3.1/README.md`, drop `(latest)` from the title (→ `v3.3.1`) and raise `nav_order` to `11`, then update `parent:` / `grand_parent:` in its pages to match
5. Add a row to the Versions table in `docs/README.md`
6. Edit the new version's content, then push

Step 3 and 4 are bulk edits across ~50 files each; a find-and-replace across the folder is enough since the strings are exact.

To retire a version from the sidebar without deleting it, add `nav_exclude: true` to its hub — the pages stay reachable by URL and in search.

## Images

Images are not stored in this repo. They are referenced from the docs CDN:

```html
<img src="https://cdn-docs.xgrids.cloud/assets/01-plugin-files-ScLMBnj2.jpg" alt="…" />
```

The `-ScLMBnj2` suffix is a content hash generated when the official docs site is built, so it cannot be derived from the filename. To add an image, publish it on the official docs site first and copy the resulting CDN URL from the rendered page.

## Local preview

Optional, needs Ruby:

```bash
cd docs
bundle init
bundle add jekyll just-the-docs jekyll-remote-theme jekyll-relative-links jekyll-seo-tag jekyll-sitemap
bundle exec jekyll serve
```

`remote_theme` requires network access on the first run.
