# uvads.github.io

Source for <https://uvads.github.io/> — a Jekyll site using the
[Just the Docs](https://just-the-docs.com/) theme.

## Layout

Jekyll builds from the repository root; page content lives in `docs/`.

```
_config.yml         # site + theme configuration
docs/               # page source
  index.md          # home page (permalink: /)
  storage.md
.github/workflows/
  jekyll-gh-pages.yml   # builds and deploys on every push to main
```

There is no Gemfile and nothing to install. GitHub builds the site with its own
pinned `github-pages` gem bundle, which already includes every plugin listed in
`_config.yml`.

The theme is loaded with `remote_theme` in `_config.yml`, so its CSS and JS
come from the upstream repo at build time. Upgrading is a one-line change: bump
the version tag on the `remote_theme:` line.

## Adding a page

Create a Markdown file in `docs/` with Just the Docs front matter:

```markdown
---
title: Git Basics
layout: default
nav_order: 3
---
```

`nav_order` controls the position in the left-hand nav (Home is 1, Storage is
2). For a section with child pages, see
[Navigation structure](https://just-the-docs.com/docs/navigation-structure/).

Pages under `docs/` are published at `/docs/<name>.html`. The home page is the
exception, since its `permalink: /` front matter moves it to the site root.

## Publishing

Push to `main`. The `.github/workflows/jekyll-gh-pages.yml` workflow builds and
deploys on every push, and the repository's **Settings → Pages** source must be
set to **GitHub Actions** (not a branch and folder) for it to publish.

There is no local preview step, so the Actions run is the build. To watch it:

```bash
gh run watch
```

A push usually takes two to three minutes to appear on the live site.
