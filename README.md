# uvads.github.io

Source for <https://uvads.github.io/> — a Jekyll site using the
[Just the Docs](https://just-the-docs.com/) theme.

## Layout

Jekyll builds from the repository root; page content lives in `docs/`.

```
_config.yml         # site + theme configuration
Gemfile             # local preview only
docs/               # page source
  index.md          # home page (permalink: /)
  storage.md
.github/workflows/
  jekyll-gh-pages.yml   # builds and deploys on every push to main
```

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

## Preview locally

From the repository root:

```bash
bundle install
bundle exec jekyll serve --livereload
```

Then open <http://localhost:4000/>. Pages under `docs/` are served at
`/docs/<name>.html`. The home page is the exception, since its `permalink: /`
front matter moves it to the site root.

## Publishing

The `.github/workflows/jekyll-gh-pages.yml` workflow builds and deploys the site
on every push to `main`. The repository's **Settings → Pages** source must be set
to **GitHub Actions** (not a branch and folder) for that workflow to publish.
