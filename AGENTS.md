# AGENTS.md (drfarah.org)

This file provides guidance to LLM-based coding agents working in this repository.
It covers site content, configuration, and delivery.
Behavior that holds for any downstream site — layouts, CSS, partials, Hugo API quirks — belongs in `themes/toph/AGENTS.md`, not here.
Write every Markdown and AsciiDoc file in this repo using one sentence per line.

## Project Overview

Personal website for Farah Aldali at https://drfarah.org/, built with [Hugo](https://gohugo.io) using the custom **Toph** theme.
Licensed MPL-2.0.
The site is a bilingual (English and Arabic) biography: a homepage, an about page, and project profiles.
There is no blog yet, which is why no blog scaffolding exists anywhere in the content tree.
The theme is a Git submodule pinned at `v1.3.0-4-gd0381ef` as of the last upgrade.

## Build & Development

```bash
# Local dev server
hugo server

# Build for production, with the flags CI uses
hugo --gc --minify

# Update the theme submodule
git submodule update --remote --rebase

# If themes/toph/ is empty or missing, initialize the submodule
git submodule update --init
```

Hugo 0.166.0 or later is required; that is the theme's floor, recorded in `themes/toph/theme.toml`.
Development and CI use 0.170.1, kept in step so that a green build means the compiler the site was developed against.
Since 0.170.0 the default Hugo binary covers everything this site needs, so there is no edition to select.
Asciidoctor is required for `.adoc` content: `gem install asciidoctor`.
The site compiles no Sass, but CI still installs Dart Sass because `.github/workflows/hugo.yaml` is kept in sync with the jwheel.org workflow it is copied from, and that workflow pins it.

## CI and deployment

`.github/workflows/hugo.yaml` builds the site and deploys it to GitHub Pages.
It runs on every push and pull request, on manual dispatch, and twice a day at 08:15 and 20:15 UTC.
The cron rebuilds and redeploys identical bytes while nothing is date-gated; it earns its keep once scheduled content exists.
Provenance matters here: the workflow is a copy of jwheel.org's, kept in sync rather than hand-trimmed, so upstream fixes arrive by re-copying the file.
That is why Dart Sass `1.106.1` is installed even though no Sass is compiled here.
Tool caches are keyed on the pinned version strings, so bumping `HUGO_VERSION` or `DART_SASS_VERSION` busts the cache instead of silently reusing a stale binary.
The build runs `hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"` with `HUGO_ENVIRONMENT=production` and `TZ: Europe/Berlin`; match those flags locally when reproducing a CI-only failure.
The `Verify /llms.txt and source publishing` step runs the theme's own `.github/scripts/check_llms.py` against the artifact that ships, with `--source-prefix source` matching `data/llms.yaml`, so a silent regression fails the build rather than going live.
Deployment runs only for pushes to `main`; pull requests stop after the build.
When you bump a pin, update `.github/workflows/hugo.yaml`, `README.md`, and this file in the same commit.

## Two-Repository Architecture

This repo contains the **site content and configuration**.
The theme lives in a separate Git repository included as a submodule:

- **Site repo** (this): content, config, static assets
- **Theme repo** (`themes/toph/`): layouts, CSS, partials, data, JS — tracked at `git@github.com:justwheel/toph-hugo-theme.git`

Changes to layouts, CSS (`assets/css/`), partials, or theme data require working in the theme repo.
The submodule pointer in this repo is updated separately with `git submodule update --remote --rebase` and committed as its own change.
Do not edit files under `themes/toph/` as part of a site commit: that work belongs upstream, and a local edit is discarded the next time the submodule moves.
If `themes/toph/` does not exist or is empty, the submodule has not been cloned; check `.gitmodules` and run `git submodule update --init`.

## Content Structure

- `content/_index.adoc` — homepage body and hero-page metadata.
- `content/_index.ar.adoc` — Arabic homepage.
- `content/about.adoc` — biography / about page.
- `content/about.ar.adoc` — Arabic about page.
- `content/projects/50-freiburg-international-academy.en.md` — project profile.
- `content/projects/50-freiburg-international-academy.ar.md` — its Arabic translation.
- `static/img/` — images: `logo.jpg`, `og-default.jpg`, `fia-logo.svg`, `apple-touch-icon.png`.
- `static/favicon.ico` — site icon.

There is no `content/blog/`, `content/categories/`, `content/tags/`, `content/footer/`, `static/archive/`, `layouts/`, `archetypes/`, or `i18n/` in this repo.
Do not document, create, or scaffold those paths speculatively; add them when the content that needs them exists.

Project profiles use a numeric filename prefix to order the homepage list, and `slug` to pin the URL independently of the filename.
Front matter requires `title`, `date`, `slug`, `icon`, `hide_sitemap: true`, and `categories: ["projects"]`.
Translations use the `.en.md` / `.ar.md` suffix pair so Hugo links them as translations.
Structural categories (`projects` today, and `footer` if footer badges are ever added) are filtered from listings by `params.taxonomy_exclude`.

## Brand assets

`static/favicon.ico` carries 16, 32, and 48 pixel frames.
`static/img/og-default.jpg` is the 1200×630 social card, wired per page through front-matter `images:` rather than through `params.images`.
`static/img/logo.jpg` is `params.images[0]`, which serves three roles: hero photo, OpenGraph/Twitter fallback, and the Person JSON-LD image.
`static/img/apple-touch-icon.png` is generated but unwired: the theme exposes only one favicon param, so nothing references it.
Nothing in the repository regenerates these files, so a palette change means re-exporting them by hand and committing new binaries.
Keep them in step with `params.colors`: the favicon and social card should sample the same two or three brand colors as the site.

## AsciiDoc Conventions

AsciiDoc pages use YAML front matter for metadata, exactly like Markdown pages.
**Do NOT use an AsciiDoc H1 header (`= Title`) for the page title**; put it in `title:` front matter instead:

```asciidoc
---
title: Who is Farah Aldali?
images: ["/img/og-default.jpg"]
---

Content here in AsciiDoc format...
```

External links use `https://example.com[label]`; internal links use `link:/about/[label]` so the source keeps a site-relative URL.
Asciidoctor bypasses Hugo's render hooks, so the theme rebases root-relative links and injects heading anchors at render time in `partials/render-page-content.html`; writing a bare `/about/` is still wrong in a subpath deployment.
**AsciiDoc detection:** use `.File.Ext == "adoc"` to detect AsciiDoc in templates.
Do not use `.Markup`, which returns an object rather than a string in Hugo 0.157+, so string comparison fails silently.

`markup.asciidocExt.preserveTOC: true` is set in `config.yaml` so an AsciiDoc `:toc:` document attribute survives into `.Content`.

## URL Preservation (NEVER BREAK)

File names under `content/` MUST NEVER be renamed or moved once the page is published.
The homepage and about page derive their URLs from their filenames (`/` and `/about/`), so renaming either breaks the permalink and every existing `link:` reference.
Project pages protect their URL with `slug:`, but the filename still encodes homepage ordering and the `.en`/`.ar` translation pairing, so treat it as fixed too.
If a URL genuinely has to change, add an alias in front matter rather than editing the old path in place.

## Configuration

`config.yaml` is YAML, not TOML.
Its sections follow jwheel.org's layout and ordering so the two files can be compared without translating between them.
Groups appear in this order: base settings, security, markup, privacy, navigation menu, `params`, internationalization.
Separate groups with two blank lines, and place each `# --- … ---` banner comment directly on the group it labels.
Inside `params`, keep related settings adjacent rather than strictly alphabetical, matching jwheel.org's order wherever a key exists in both files.

- `params.biography` — `name`, `tagline`, `gender`, `email`, `first_online`, `knows_about`.
- `params.colors.light` / `params.colors.dark` — brand palette. **The nesting is mandatory**: the theme reads only these subtrees, and a flat `colors.primary` key is silently ignored, which left this site uncolored for weeks. Keys are `primary`, `secondary`, `accent`, `background`; `dark` falls back to `light` per key, and `background` falls back to `secondary`. See "Brand palette and dark mode".
- `params.color_mode` — `auto` (default: OS preference plus navbar toggle), `light`, or `dark`.
- `params.description` — meta description; overridden per language.
- `params.fonts` — `default`, `title`, `header`, each with a weight.
- `params.images` — **triple-purpose** `[0]`, as described under "Brand assets". Do not put a 1200×630 social card here; use page-level `images:` front matter.
- `params.hero.shape` — `circle` (default), `square`, or `none`.
- `params.favicon` — `/favicon.ico`.
- `params.git_repo` — repository link in the footer box.
- `params.taxonomy_exclude` — categories hidden from taxonomy listings.
- `params.hire_me` — `false`, so the for-hire banner never renders.
- `params.social` — platform keys to handles; see "Social Media Platform Registry".
- `params.date_format` — Go time layout; this site uses `"02 January 2006"`.
- `params.legal.license` — `name`, `url`, `title` for the footer copyright line.
- `disableKinds` — suppresses `taxonomy` and `term` output because there is no blog yet; `categories` front matter still drives projects.
- `privacy` — DNT for X and Vimeo, `privacyEnhanced` for YouTube, `anonymizeIP` and `respectDoNotTrack` for Analytics.
- `menu.nav` — site navigation entries, duplicated per language for the Arabic labels.
- `languages` — two languages, `en` (default) and `ar` (RTL), with explicit `weight` values.

`data/style.yaml` is not configuration and is covered under "Brand palette and dark mode".
`data/llms.yaml` is covered under "Machine-readable index".

## Brand palette and dark mode

Light-mode brand colors are compiled into the stylesheet; dark-mode values ship as an inline `<style>` block, so the dark set can change without a rebuild.

The palette is dusty rose with a deep teal accent.
Light: primary `#A25666`, secondary `#FAF0E6`, accent `#1F4E4A`, background `#FDF8F2`.
Dark: primary `#7C4250`, accent `#2E4744`, background `#121212`; secondary carries over unchanged.

The navbar is `--primary` with `--secondary` text, which constrains dark mode: brightening the primary drops cream nav text below AA, so this site deepens it and headings recover contrast through the theme's `color-mix(in srgb, var(--primary) 60%, white)`.

`--accent-color` is both a surface behind `--secondary` text (brand chip, active nav item, theme toggle, dropdown hover, tag badges) and the nav hover underline, which is why the accent stays deep.
That dual role means the underline is still low-contrast, 1.81:1 in light mode and 1.31:1 in dark; the hover state also changes the background, so it stays perceivable.
The real fix is the accent-token split, tracked upstream in the theme repo, not a local workaround.

**Neutral override.** Toph's `#767676` muted gray is calibrated against white and reaches only 4.30:1 on this cream page, under the 4.5:1 floor.
`data/style.yaml` darkens `text.secondary` and `text.anchor` to `#6E6E6E`, which restores 4.83:1 on cream, 5.10:1 on white cards, and 4.63:1 on the hover surface.
That file restates the whole `text` group on purpose: Hugo replaces a top-level data group wholesale rather than merging into it, so overriding only two keys would drop `body`, `link`, and `muted` to Bootstrap's defaults.
If the theme gains a key in that group, add it here too, and keep the `light:` / `dark:` nesting because a flat `text.secondary` key is silently ignored.

Measured against palette C: headings 4.90:1 light and 6.38:1 dark, nav text 4.60:1 and 6.78:1, links 11.94:1 and 8.02:1, chips 8.31:1 and 8.89:1.
Re-check contrast against the lightest surface a text role can land on, not just the page background, and verify both modes with a `params.color_mode: dark` overlay build.

## Internationalization

Two languages are active: `en` (default, weight 1) and `ar` (RTL, weight 2).
Both must keep explicit `weight` values; without them, `hreflang` tags and the navbar switcher are ordered non-deterministically across builds.
Arabic content lives beside its English sibling as `.<lang>` files, not in a `content/ar/` subdirectory.
Per-language overrides belong under `languages.ar`: `title`, `params.description`, `params.biography.name`, `params.biography.tagline`, and both translated `menu.nav` entries live there today.
The `biography.name` override is required, not decorative: the hero reads `.Param "biography.name"`, so without it the Arabic homepage would show the English name.
This site ships no `i18n/` directory; interface strings come from the theme's `i18n/ar.yaml`.
RTL layout comes from the theme's `_rtl.css` through `direction: rtl`; changing it is upstream work.

## Machine-readable index

`data/llms.yaml` enables `/llms.txt` and native source publishing.
The index itself is written in exactly one language (`index.language: en`), while `sources.languages: all` publishes both languages' sources under `/source/en/` and `/source/ar/`.
`sources.prefix: source` must agree with `--source-prefix source` in the CI verification step; changing one without the other fails the build.
The `sections` list names the homepage, the about page, and the whole `projects` section, so new projects are indexed without editing the data file.
Delete the file to turn the feature off; the theme ships no default, so absence is the off switch.
The schema and its invariants are documented in `themes/toph/README.md`; this repo owns only the data file.

## Social Media Platform Registry

Social media configuration uses a single source of truth:

1. **`data/social.yaml`** (theme) — platform registry mapping keys to display names, URL templates, and Bootstrap Icons classes. Entries are alphabetized.
2. **`params.social`** (site config) — simple key-value map of platform key to username or handle.

```yaml
# Example in config.yaml
params:
  social:
    instagram: drfarahaldali
    linkedin: in/farah-aldali-2388461a9
    threads: drfarahaldali
```

All consumers read from `params.social` plus `data/social.yaml`:

- `partials/hero.html` — icon buttons on the homepage.
- `partials/nav.html` — the social dropdown in the navbar.
- `partials/meta-seo.html` — the JSON-LD `sameAs` array.

**`menu.social` is NOT used.** Social links derive entirely from `params.social`.
Keys must match `data/social.yaml` exactly, or the platform silently produces nothing.
LinkedIn's template is `https://www.linkedin.com/%s`, so its value must include the `in/` path segment, as above.

## Homepage Architecture

The homepage renders through the theme's biography layout, in this order:

1. **Hero partial** — profile photo, tagline (`params.biography.tagline`), social icon buttons, about page link.
2. **For-hire banner** — only when `params.hire_me` is true, which it is not here.
3. **Page content** — the body of `_index.adoc`, rendered through `partials/render-page-content.html`.
4. **Recent posts** — the partial self-guards on "at least one visible blog post", so it renders nothing today.
5. **Projects** — carousel plus list, rendered because a page carries `categories: ["projects"]`.

The `h1` page title comes from `partials/header.html`, not from the hero partial.

**The layout declaration is load-bearing.** `_index.adoc` and `_index.ar.adoc` must both declare `layout: biography` in front matter, because the theme ships no `layouts/index.html`.
Without it, Hugo silently falls back to `_default/list.html` and the homepage becomes a paginated list of every regular page, including ones marked `hide_sitemap: true`, with no warning.
The Arabic homepage needs its own declaration; front matter is not inherited across languages.

## Taxonomy Templates (dormant)

`disableKinds: ["taxonomy", "term"]` suppresses all generated category and tag pages because there is no blog yet.
Nothing in this list renders today, and you should not build against it until a blog exists:

- `layouts/categories/terms.html` — three-column magazine cards with images, descriptions, and recent posts.
- `layouts/tags/terms.html` — word cloud with font-size and opacity scaled by post count, pill buttons, default sort by most-used.
- `layouts/_default/terms.html` — fallback list with a sort toggle and exclusion filtering.
- `layouts/_default/term.html` — single term page with a post list and plaintext excerpts.

When a blog lands, remove the `disableKinds` line, add `content/categories/` and `content/tags/` `_index.md` files, and keep the conventions the theme already uses.
Tags are lowercase; categories use Title Case from their `_index.md` title.
`hide_sitemap: true` in a term's `_index.md` hides it from listing pages and the word cloud.
Dates display as `02 January 2006` (e.g., "09 May 2023") and `datetime` attributes use ISO `2006-01-02`.
Excerpts use `.Plain | htmlUnescape | truncate 250`, which avoids HTML entity artifacts.

## Heading Anchors

Section headings display a clickable 🔗 anchor on hover for sharing direct links.

- **Markdown:** a render hook at `layouts/_default/_markup/render-heading.html`.
- **AsciiDoc:** `replaceRE` on page content in `layouts/partials/render-page-content.html`, because Asciidoctor bypasses Hugo render hooks. Before v1.3.0 this pass lived in `single.html`.

Both produce identical `hanchor` markup and share the same CSS.
Do not move the AsciiDoc pass back into a single layout: section intros, embedded project pages, and RSS descriptions all go through the shared partial for the same reason.

## GitHub API Access (CRITICAL)

**ALWAYS** get explicit user consent before any mutating GitHub action under their account.
Mutating means any POST, PATCH, PUT, or DELETE: comments, replies, issue and PR creation, edits to an existing title, body, or comment, labels, assignees, milestones, review requests, state changes such as closing or merging, and branch, tag, or release operations.
Read-only calls need no approval.

The user and the agent work as a team.
Communication on GitHub must be effective, genuine, and honest, which requires a human-in-the-loop check before every public-facing action.

**Use the `github` MCP server first.** It is the configured path for this repository and covers issues (`issue_write`, `add_issue_comment`, `update_issue_comment`), pull requests (`create_pull_request`, `pull_request_read`, `pull_request_review_write`), and reads (`search_issues`, `list_pull_requests`, `get_file_contents`).
Call `get_me` first when you need to confirm which account would act.
Fall back to the `gh` CLI only when the MCP server lacks the operation, and say so when you do.
`run_secret_scanning` can check a payload before it leaves the machine; use it on anything containing configuration or credentials.

Workflow:

1. Draft the exact payload and write it to a uniquely named file under `/tmp/opencode/`, for example `/tmp/opencode/comment-pr-12-reply.txt`. That directory is always writable, so no approval is needed to save a draft.
2. Present the payload for review. For edits to existing content, show a precise diff and confirm nothing else changed.
3. Wait for a free-text approval of that exact content.
4. Then send it, either through the MCP tool or by presenting a copy-pasteable `gh api` command for the user to run.

Being asked to make a change is a task assignment, not approval of the change itself: "edit the PR description" means draft the edit and show it.
Approval never carries forward, not even between two comments in the same task, and a menu selection or plan approval is not consent because it picks a direction rather than authorizing a payload.
Anything composed after the approval — a header, a trailer, a title — is new unreviewed content and needs another round.
If you are unsure whether consent exists, it does not; stop and ask.

Never reply to a PR review comment until after the fix is committed and pushed to the remote.

### Formatting GitHub comments

Comments posted through the API follow different conventions than files in the repo.

- Wrap paragraphs normally; the one-sentence-per-line rule governs `.md` and `.adoc` files, not comments.
- Never wrap commit hashes in backticks: a bare hash autolinks, a backticked one does not. Use `owner/repo@hash` to link a commit in another repository.
- Do wrap color hex codes in backticks, since GitHub renders a color swatch for them.
- Closing keywords do not work across repositories: `Closes owner/repo#12` from another repo creates a backlink but will not close the issue, so say that it needs closing manually.
- Disclose AI authorship as "LLM-gen-AI" and do not name the model or vendor in the comment. The point is to tell readers the content is machine-generated and needs verification; naming a vendor reads as branding. This does not change the `Assisted-by:` commit trailer, which still cites the exact model.

## Git Conventions (CRITICAL)

Commit history is read by humans and by agents, so message shape is part of the contract.
**Never run `git add`, `git commit`, `git push`, or `git merge`, and never open a pull request.** The user stages, commits, and pushes; you prepare the message and hand over the exact commands.

### The 50/72 rule and formatting

- **Subject**: `<gitmoji> <component>: <summary>`, at most 50 characters, filled close to that limit rather than truncated early. One or two gitmojis, always first. Canonical list: <https://gitmoji.dev>.
- **Component scope**: a concise lowercase subsystem such as `theme`, `ci`, `config`, `content`, `a11y`, or `llms`.
- **Body**: at most 72 characters per line, wrapped greedily — each line packed with as many words as fit without reaching 73, never broken early for visual balance.
- **Body content**: three to six sentences on motivation, constraints, and the alternatives rejected. Do not narrate the diff or restate mechanics.
- **Blank lines**: one between the subject, each paragraph, and the trailer block.
- **Identifiers**: wrap paths, file names, layout names, and code entities in backticks. A backtick pair must open and close on the same line. A commit hash stays bare, never backticked, because GitHub only autolinks plain text.

### AI disclosure trailer

Every commit assisted by a model ends with an `Assisted-by:` trailer on its own line, after a blank line.
The model changes between sessions, so verify the active model and its context window from the runtime environment instead of carrying one forward from earlier in the conversation.
Format it exactly as `Assisted-by: <model> <version> (<context window>)`; for this harness that reads `Assisted-by: DeepSeek V4.1 Flash (200K context)`.
`Signed-off-by` comes only from `--signoff`; never write it into the message by hand.

### Commit preparation workflow

1. Print the drafted message in a fenced code block.
2. Write it to `/tmp/opencode/commit-<descriptive-name>.txt`, using a name that is unique and tab-completable. Never use a generic `commit-msg.txt`, because several sessions can run at once, and write each artifact once rather than mirroring it to a second path.
3. Present the copy-pasteable commands, staging first and commit second.
4. Keep commit flags in strict alphabetical order: `--edit --file=/tmp/opencode/commit-<name>.txt --gpg-sign --signoff`. `--edit` lets the user review the message before it lands.
5. For a multi-commit change set, present a table pairing each subject with its command.
6. An amendment of an already-pushed commit needs `git push --force-with-lease`, with `--amend` first in the alphabetical order: `--amend --edit --file=/tmp/opencode/commit-<name>.txt --gpg-sign --signoff`. The user decides whether to do it.

A line beginning with `#` is a comment to Git and is stripped from the message, so confirm `grep -c '^#'` returns 0 before handing over.
`/tmp` is periodically cleaned; if a message file disappears before it is used, rewrite it.

```bash
git add <paths>
git commit --edit --file=/tmp/opencode/commit-<name>.txt --gpg-sign --signoff
```

### Never

- `--no-gpg-sign`, `--no-verify`, or any other hook bypass.
- `git add`, `git commit`, `git push`, `git merge`, or `git reset` on the user's behalf.
- Force-pushing without explicit permission.
- Suggesting a command with an abbreviated flag; always write `--signoff`, `--file`, `--init`, and so on in full.

All commits must be GPG-signed (`commit.gpgsign = true`).
The user creates branches and approves every change.

## JavaScript Conventions

- Vanilla JS only, with no dependency beyond Bootstrap.
- This repo ships no JavaScript of its own; anything scripted belongs to the theme.
- Use `const` for variables that are never reassigned, and `var` only when reassignment is needed.
- Always pass an explicit radix to `parseInt()`, for example `parseInt(value, 10)`.

## Style Guide for Agents

When demonstrating or suggesting shell commands, use fully expanded flags and parameters, for example `--signoff` instead of `-s`, `--file` instead of `-F`, and `--init` instead of `-i`. This promotes learning for the user.
Write prose one sentence per line in every repository file.
Reproduce CI conditions with `hugo --gc --minify` and the same `TZ` and `HUGO_ENVIRONMENT` before blaming a build difference on the content.
Cite files by their real paths in this checkout; do not extrapolate a path from the theme's layout or from an older version of this document.
