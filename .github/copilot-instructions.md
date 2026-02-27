# Copilot Instructions - tildemark.github.io

## Project Overview

This is a **Jekyll blog** powered by the **Chirpy theme**, deployed to GitHub Pages at `blog.sanchez.ph`. The site is built from Markdown posts and deployed via GitHub Actions.

### Key Architecture
- **Content**: Posts in `_posts/` (YAML frontmatter + Markdown), drafts in `_drafts/`
- **Theme**: `jekyll-theme-chirpy` (Ruby gem) - manages layouts, styles, and components
- **Build**: `bundle exec jekyll build` (production) or `bundle exec jekyll s -l` (dev)
- **Deployment**: GitHub Actions → GitHub Pages (see `.github/workflows/pages-deploy.yml`)
- **Testing**: HTML validation via `html-proofer` in test.sh
- **Custom Plugin**: `_plugins/posts-lastmod-hook.rb` - automatically captures `last_modified_at` from git commits

## Essential Workflows

### Development
```bash
# Run dev server with live reload (localhost:4000)
./tools/run.sh

# Run with custom host
./tools/run.sh -H 0.0.0.0

# Production mode (minified assets, optimized)
./tools/run.sh -p
```

### Testing & Deployment
```bash
# Build and validate HTML (runs html-proofer)
./tools/test.sh

# Build with multiple config files
./tools/test.sh -c "_config.yml,_config.production.yml"
```

Both scripts are in `tools/` with bash utilities for config parsing and error handling.

## Post Structure & Conventions

### File Naming
Posts follow pattern: `_posts/YYYY-MM-DD-slug-title.md` (e.g., `2025-12-02-from-c-to-typeScript.md`)

### YAML Frontmatter (Required Fields)
```yaml
---
title: "Post Title"
date: 2025-12-02 08:00:00 +0800
categories: [Category1, Category2]
tags: [tag1, tag2, tag3]
toc: true                    # Table of contents
comments: true
math: false                  # Enable KaTeX math
mermaid: false              # Enable Mermaid diagrams
pin: false                  # Pin to top
image:
  path: https://cdn.sanchez.ph/blog/image.webp
  alt: "Alt text"
---
```

### Content Patterns
- **Images**: Use absolute CDN URLs (https://cdn.sanchez.ph/blog/*)
- **Math**: Wrapped in `$...$` (inline) or `$$...$$` (block) when `math: true`
- **Links**: Relative paths for internal pages (e.g., `/about`)

## Chirpy Theme Specifics

- **Layouts** (from gem): Inherited from `jekyll-theme-chirpy` - only override if absolutely needed in `_layouts/`
- **Data files** (`_data/contact.yml`, `share.yml`): Configure social links and sharing options
- **Tabs** (`_tabs/about.md`, `categories.md`, etc.): Navigation structure—these are **mandatory** for theme nav
- **Assets**: Keep JS/CSS minimal; theme handles 90% via gem. Custom CSS goes in `assets/css/`

## Important Quirks & Gotchas

1. **Last Modified Dates**: The plugin (`posts-lastmod-hook.rb`) requires git history. Push posts before they show accurate `last_modified_at` timestamps in the Chirpy sidebar.

2. **Config Override**: When running `test.sh`, multiple configs are merged in reverse order. Later configs override earlier ones.

3. **JEKYLL_ENV**: Must be `production` for minified assets and optimizations. Dev mode (default) keeps assets unminified.

4. **Image CDN**: All images reference `https://cdn.sanchez.ph/blog/`. If images aren't rendering, verify they're uploaded to that CDN.

5. **Drafts**: Posts in `_drafts/` won't build unless `--drafts` flag is used. For draft previews, use `./tools/run.sh` locally.

## Common Tasks

- **Add a new post**: Create `_posts/YYYY-MM-DD-slug.md` with frontmatter, run dev server to preview
- **Edit a published post**: Change content + date auto-captures via git hook
- **Change site title/config**: Edit `_config.yml` top section (title, tagline, url, social links)
- **Update navigation tabs**: Modify files in `_tabs/` directory
- **Fix broken links**: `./tools/test.sh` will catch internal link issues via html-proofer
- **Custom CSS**: Add to `assets/css/` (will be included by Chirpy theme)

## Key Files to Know

- `_config.yml` - All site configuration (read lines 1-100 for overrides)
- `Gemfile` - Ruby dependencies (jekyll-theme-chirpy, html-proofer)
- `tools/run.sh` - Dev server launch with live reload
- `tools/test.sh` - Production build + HTML validation
- `_plugins/posts-lastmod-hook.rb` - Git-based last-modified dates
- `.github/workflows/pages-deploy.yml` - CI/CD pipeline to GitHub Pages

## When to Ask for Help

- Theme customization beyond CSS → Check Chirpy wiki
- Jekyll/Liquid syntax → Jekyll documentation
- GitHub Actions → GitHub's CI/CD docs
- Ruby version issues → `ruby --version` and Gemfile ruby constraint
