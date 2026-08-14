# uvads.github.io

Source for <https://uvads.github.io/> — a Jekyll site using the
[Just the Docs](https://just-the-docs.com/) theme.

## Layout

Jekyll builds from the repository root; page content lives in `docs/`.

```
_config.yml         # site + theme configuration
docs/               # page source, one file per left-nav entry
  index.md          # home page (permalink: /)
  storage.md
  databases.md
  *.md              # the rest are redirect stubs to guides in other repos
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

`nav_order` controls the position in the left-hand nav. The nav mirrors the
Systems Essentials list on the home page, so the numbers are already taken:
Home 1, Managing Your Environment 2, Git Basics 3, Storage 5, Working with
Databases 6, Containers 7, Workflow Orchestration 8, Consuming Streaming Data
9. Slot 4 is deliberately empty and reserved for Compute Resources, which the
home page lists but nobody has written yet. For a section with child pages, see
[Navigation structure](https://just-the-docs.com/docs/navigation-structure/).

Topics whose guide lives in another repo get a stub page here that holds the
nav position and forwards to the real site. The theme can only render external
links after all local pages, so a stub is the only way to keep one topic order
across the nav and the home page.

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
