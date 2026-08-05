# uvads.github.io

Source for <https://uvads.github.io/> — a Jekyll site using the
[Just the Docs](https://just-the-docs.com/) theme.

## Layout

```
docs/               # site source (GitHub Pages publishing directory)
  _config.yml       # site + theme configuration
  index.md          # home page
  Gemfile           # local preview only
```

The theme is loaded with `remote_theme` in `docs/_config.yml`, so its CSS and JS
come from the upstream repo at build time. Upgrading is a one-line change: bump
the version tag on the `remote_theme:` line.

## Adding a page

Create a Markdown file in `docs/` with Just the Docs front matter:

```markdown
---
title: Git Basics
layout: default
nav_order: 2
---
```

`nav_order` controls the position in the left-hand nav. For a section with
child pages, see [Navigation structure](https://just-the-docs.com/docs/navigation-structure/).

## Preview locally

```bash
cd docs
bundle install
bundle exec jekyll serve --livereload
```

Then open <http://localhost:4000/>.

## Publishing

In the repository's **Settings → Pages**, set the source to the `main` branch
and the `/docs` folder. Pushes to `main` rebuild the site.
