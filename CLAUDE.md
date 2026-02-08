# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Jekyll-based technical blog ("AuruTus's Forget Not") hosted on GitHub Pages at https://aurutus.github.io/. Content focuses on systems programming topics: concurrency, optimization, compiler semantics.

## Development Commands

```bash
# Local development server (with live reload)
jekyll serve

# Build static site to _site/
jekyll build
```

No package manager or additional tooling is needed. Deployment is automatic via GitHub Pages on push to `main`.

## Architecture

- **Theme**: Minima (GitHub Pages built-in)
- **Syntax highlighting**: highlight.js (v10.5.0) with custom language packs (PowerShell, T-SQL, plaintext) — Kramdown's built-in highlighter is disabled in `_config.yml`
- **Plugins**: jekyll-feed (RSS), jekyll-sitemap (SEO)

### Key directories

- `_posts/` — Blog posts in Markdown with YAML front matter
- `_layouts/` — Custom `post.html` extending minima's default layout
- `_includes/` — Reusable components: `head.html` (scripts/styles), `sharelinks.html` (social sharing), `navlinks.html` (prev/next navigation)
- `css/override.css` — Theme style overrides
- `js/highlightjs/` — Highlight.js library, language packs, and themes

## Blog Post Conventions

Posts use standard Jekyll naming: `YYYY-MM-DD-slug.md` in `_posts/`.

Front matter format:
```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
tags: [tag1, tag2]
---
```

- All posts default to tag `Other` if no tags are specified (configured in `_config.yml` defaults)
- The archive page (`archive.md`) groups posts by tag using Liquid templates

## Commit Message Convention

- `docs:` for content additions (new posts, content updates)
- `chore:` for maintenance and configuration changes
