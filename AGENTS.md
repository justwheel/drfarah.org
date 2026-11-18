# AGENTS.md (drfarah.org)

This file provides guidance to LLM-based coding agents (e.g., Claude Code, Copilot, Cursor, Gemini CLI) when working with code in this repository.

## Project Overview

Personal website for Farah Aldali at https://drfarah.org/, built with [Hugo](https://gohugo.io) using the custom **Toph** theme. Licensed MPL-2.0.

## Build & Development

```bash
# Local dev server
hugo server

# Build for production
hugo --minify

# Update theme submodule
git submodule update --remote --rebase

# If themes/toph/ is empty or missing, initialize the submodule
git submodule update --init
```

Hugo 0.166.0 or later (the theme's floor, see `themes/toph/theme.toml`).
Development and CI use 0.170.1, kept in step so a green build means the compiler the site was developed against. Since 0.170.0 the default Hugo binary covers everything this site needs, so there is no edition to select. The site compiles no Sass, but CI still installs Dart Sass: `.github/workflows/hugo.yaml` is kept in sync with the jwheel.org workflow it is copied from, and that workflow pins it.

## Two-Repository Architecture

This repo contains the **site content and configuration**. The theme lives in a separate Git repository included as a submodule:

- **Site repo** (this): content, config, static assets
- **Theme repo** (`themes/toph/`): layouts, CSS, partials, data, JS — tracked at `git@github.com:justwheel/toph-hugo-theme.git`

Changes to layouts, CSS (`assets/css/main.css`), partials, or data files require working in the theme repo (checked out at `themes/toph/` as a submodule). The submodule pointer in this repo is updated separately via `git submodule update --remote --rebase`.

If `themes/toph/` does not exist or is empty, the git submodule has not been cloned correctly. Check the `.gitmodules` file in the repository root and run `git submodule update --init` to resolve.

## Content Structure

- `content/about.adoc` — Biography / about page (AsciiDoc).
- `content/blog/` — Blog posts (Markdown), organized by `YYYY/MM/slug.md`. Front matter: `title`, `date`, `categories`, `tags`.
- `content/projects/` — Project profiles with numeric prefix ordering (e.g., `01-project.en.md`). Front matter requires: `title`, `date`, `slug`, `icon`, `hide_sitemap: true`, `categories: ["projects"]`. Translations use `.<lang>.md` suffix.
- `content/footer/` — Dynamic footer badges. Front matter requires: `categories: ["footer"]`, `hide_sitemap: true`.
- `content/categories/` — Category term `_index.md` files with human-readable `title` and optional `hide_sitemap: true` to hide from listings.
- `content/tags/` — Tag term `_index.md` files. Tags with `hide_sitemap: true` are hidden from the word cloud and taxonomy listings.
- `static/img/` — Images; `static/archive/` — archived assets.

Structural categories (`footer`, `projects`) are filtered from taxonomy pages via `params.taxonomy_exclude` in config. Individual categories and tags can also be hidden via `hide_sitemap: true` in their `_index.md` front matter.

## AsciiDoc Conventions

AsciiDoc pages use YAML front matter for metadata, just like Markdown pages. **Do NOT use AsciiDoc H1 headers (`= Title`) for the page title** — use `title:` in the front matter instead:

```asciidoc
---
title: Who is Farah Aldali?
---

Content here in AsciiDoc format...
```

**AsciiDoc detection:** Use `.File.Ext == "adoc"` to detect AsciiDoc content in templates. Do NOT use `.Markup` — it returns an object (not a string) in Hugo 0.157+ and string comparison will silently fail.

## URL Preservation (NEVER BREAK)

**File names in `content/blog/` MUST NEVER BE CHANGED UNDER ANY CIRCUMSTANCES.** Once a blog post is "published", its URL is permanent. Renaming, moving, or restructuring any existing blog file would break permalinks to this content.

## Configuration

`config.yaml` (YAML, not TOML). Key sections:

- `params.biography` — name, gender, tagline, email, knows_about
- `params.social` — social media profiles (see Social Media Platform Registry below)
- `params.colors.light` / `params.colors.dark` — brand palette. **The nesting is mandatory**: the theme reads only these subtrees, and a flat `colors.primary` key is silently ignored (this site rendered uncolored for weeks because of it). Keys are `primary`, `secondary`, `accent`, `background`; `dark` falls back to `light` per key, and `background` falls back to `secondary`. See "Brand palette and dark mode" below.
- `params.color_mode` — `auto` (default, OS preference plus navbar toggle), `light`, or `dark`
- `params.fonts` — default, title, header (with weights)
- `params.hero.shape` — `circle` (default), `square`, or `none`
- `params.legal.license` — `name`, `url`, `title` for the footer copyright line
- `params.date_format` — Go time layout; this site uses `"02 January 2006"`
- `params.favicon` — defaults to `/favicon.ico`
- `params.images` — **triple-purpose**: `[0]` is the hero photo, the OG/Twitter fallback, and the Person JSON-LD image. Do not put a 1200×630 social card here; set page-level `images:` front matter instead.
- `params.taxonomy_exclude` — categories hidden from taxonomy listings
- `disableKinds` — suppresses `taxonomy` and `term` output because there is no blog yet; `categories` front matter still drives projects and footer content
- `menu.nav` — site navigation entries (home, about)
- `languages` — two languages: en (default), ar (RTL), both active, with explicit `weight` values so `hreflang` and the switcher stay deterministically ordered

## Brand palette and dark mode

Light-mode brand colors are baked into the compiled stylesheet; dark-mode values are emitted as an inline `<style>` block, so the dark set can change without a rebuild.

The navbar is `--primary` with `--secondary` text, which constrains the dark palette: brightening the primary for dark mode drops cream nav text below AA. This site deepens the primary instead (`#7A2E4A`), and headings recover contrast through the theme's `color-mix(in srgb, var(--primary) 60%, white)`.

`--accent-color` doubles as a surface behind `--secondary` text (brand chip, active nav item, theme toggle, dropdown hover, tag badges) **and** as the nav hover underline. That is why the dark accent stays deep (`#5B1A33`): a bright accent would make cream text on those surfaces fail. The hover underline is consequently low-contrast; the hover state also changes the background, so it stays perceivable. Tracked upstream in the theme repo as the accent-token split.

Re-check contrast against the lightest surface a text role can land on, not just the page background, and verify both modes with a `params.color_mode: dark` overlay build.

## Machine-readable index

`data/llms.yaml` enables `/llms.txt` and publishes unmodified sources under `/source/en/` and `/source/ar/`. The index itself is written in exactly one language (`index.language`); the `sources.languages: all` setting is what makes it bilingual. Delete the file to turn the feature off.

## Social Media Platform Registry

Social media configuration uses a single source of truth:

1. **`data/social.yaml`** (theme) — Platform registry mapping keys to display names, URL templates, and Bootstrap Icons classes. Entries are alphabetized.
2. **`params.social`** (site config) — Simple key-value map of platform key → username/handle.

```yaml
# Example in config.yaml
params:
  social:
    instagram: drfarahaldali
    threads: drfarahaldali
```

All consumers read from `params.social` + `data/social.yaml`:
- `hero.html` — renders icon buttons on the homepage
- `nav.html` — populates the social dropdown in the navbar
- `seo-meta.html` — builds JSON-LD `sameAs` array

**`menu.social` is NOT used.** Social links derive entirely from `params.social`. Adding a new platform to the theme requires only adding an entry to `data/social.yaml`.

## Homepage Architecture

The homepage (`index.html`) renders in order:
1. **Hero partial** — profile photo, tagline (`params.biography.tagline`), social media icon buttons, about page link
2. **For-hire banner** — only if `params.hire_me` is true
3. **Homepage content** — from `_index.md`/`_index.adoc` if it exists and has content
4. **Recent posts** — latest blog posts grid
5. **Projects** — carousel + list, only rendered if pages with `categories: ["projects"]` exist

The h1 page title is rendered by `header.html`, not the hero partial.

## Taxonomy Templates

The theme provides taxonomy-specific layouts:
- `layouts/categories/terms.html` — three-column magazine-style cards with images, descriptions, and recent posts
- `layouts/tags/terms.html` — word cloud with font-size/opacity scaling by post count, pill-shaped buttons, default sort by most-used
- `layouts/_default/terms.html` — fallback list with sort toggle and exclusion filtering
- `layouts/_default/term.html` — single term page with post list and plaintext excerpts

Tags are always lowercase. Categories use Title Case from their `_index.md` title field.

Content filtering: `hide_sitemap: true` in a category/tag `_index.md` hides it from taxonomy listing pages and the tag word cloud. This reuses the same front matter field used by structural content (projects, footer).

Date display format: `02 January 2006` (e.g., "09 May 2023") across all templates. `datetime` HTML attributes use ISO `2006-01-02`.

Excerpts use `.Plain | htmlUnescape | truncate 250` for safe plaintext without HTML entity artifacts.

## Heading Anchors

Section headings display a clickable 🔗 anchor on hover for sharing direct links to sections.

- **Markdown:** Uses a render hook at `layouts/_default/_markup/render-heading.html`
- **AsciiDoc:** Uses `replaceRE` on `.Content` in `single.html` since Asciidoctor bypasses Hugo render hooks
- Both produce identical `hanchor` class markup and share the same CSS

## GitHub API Access (CRITICAL)

**ALWAYS** get explicit user consent before posting any content to GitHub under their account. This includes comments, replies, issue creation, PR creation, and any other action that publishes content visible to others.

The user and the agent work as a team. Communication on GitHub must be effective, genuine, and honest. This requires a human-in-the-loop check before every public-facing action.

Workflow:
1. Draft the proposed comment/reply in the terminal
2. Present it to the user for review
3. Wait for explicit approval (e.g., "post it")
4. Only then execute the GitHub API call

Never skip this step, even if the user has approved similar comments before. Each comment is a separate approval.

## Git Conventions (CRITICAL)

- **Gitmoji** prefix on all commit subject lines (e.g., `🍱 content: Import blog posts`)
    - Gitmoji source: https://github.com/carloscuesta/gitmoji/blob/master/packages/gitmojis/src/gitmojis.json
- **`Assisted-by:`** trailer citing exact AI model name (as a general best-practice convention)
- Use `git commit --signoff` to add the `Signed-off-by` trailer — do not write it manually
- Commit messages emphasize WHY, not just WHAT
- Write commit messages to `/tmp/` with unique, descriptive filenames (e.g., `/tmp/commit-msg-add-footer-badge.txt`) — multiple sessions may run concurrently, so never use a generic name like `/tmp/commit-msg.txt`. User runs the commit manually with these flags:
    - `--file /tmp/<filename>.txt` — use the prepared commit message
    - `--signoff` — add `Signed-off-by` trailer
    - `--edit` — allow the user to review/edit the message before committing
- **NEVER** run `git push` or create PRs — user does these manually
- **NEVER** use `--no-gpg-sign` or skip hooks
- **NEVER** reply to GitHub PR review comments until AFTER the fix is committed and pushed to the remote
- User creates branches and approves all changes

## JavaScript Conventions

- Vanilla JS only (no dependencies beyond Bootstrap)
- Use `const` for variables that are not reassigned; `var` only when reassignment is needed
- Always provide explicit radix to `parseInt()` (e.g., `parseInt(value, 10)`)

## Style Guide for Agents

When demonstrating or suggesting bash commands, always use the fully-expanded form of flags and parameters (e.g., `--signoff` instead of `-s`, `--file` instead of `-F`, `--init` instead of `-i`). This promotes learning for the user.
