# CLAUDE.md

## What this is

Udesh Kumarasinghe's personal academic website — a Jekyll site built on
[al-folio](https://github.com/alshedivat/al-folio) **v1.2**, served at
`https://www.udesh.xyz` (see `CNAME`). Almost all work here is **content
editing**, not code: adding a publication, a news item, or a project.

al-folio v1 is a thin starter: layouts, includes, Sass and feature JS live in
versioned `al_*` gems (theme `al_folio_core`), not in this repo. `AGENTS.md`,
`docs/` and `.agents/skills/` are upstream's and are written for the al-folio
starter repo itself — in particular its "stop sign" against `_layouts/` /
`_sass/` paths does **not** apply to this site, which keeps two deliberate
local overrides (see below). Prefer this file.

## Build & run

CI uses **Ruby 3.3.5** (`.github/workflows/deploy.yml`) and `Gemfile.lock` is
committed. The system Ruby on this machine is 2.6, so a bare `bundle install`
fails; use Docker or a Ruby 3.x install.

```bash
bundle exec jekyll serve                   # local preview at http://localhost:4000/
docker compose up -d                       # or: serve on http://127.0.0.1:8080/
bundle exec al-folio upgrade audit         # v1 upgrade contract checks (also run in CI)
bundle exec al-folio upgrade overrides audit
```

- **baseurl is blank** (site served at the domain root). Upstream docs assume
  `/al-folio`; ignore that.
- **Adding a gem means editing `Gemfile`, `_config.yml` `plugins:`, and
  regenerating `Gemfile.lock`.** A plugin listed in only one of the first two is
  silently inert.
- **imagemagick is required** for `jekyll-imagemagick` (responsive `.webp`
  variants of everything under `assets/img/`).

## Deployment

Push to `main` → the `Deploy site` workflow builds with `JEKYLL_ENV=production`,
runs purgecss, and publishes `_site` to the **`gh-pages`** branch. Pull requests
run the build without deploying. Never commit to `gh-pages` by hand.

## Content model

| What | Where | Notes |
|---|---|---|
| Bio / landing page | `_pages/about.md` | `layout: about`; toggles `announcements`, `selected_papers`, `social` via front matter |
| Publications | `_bibliography/papers.bib` | rendered by `jekyll-scholar`, grouped by year automatically |
| News | `_news/announcement_N.md` | numbered sequentially; `announcements.limit: 5` in `about.md` caps the homepage list |
| Projects | `_projects/{r,e}N_slug.md` | `r` = Research, `e` = Engineering |
| Nav pages | `_pages/*.md` | ordered by `nav_order`: publications 1, projects 2, cv 4 |
| Social icons | `_data/socials.yml` | order in the file = order on the page; email intentionally omitted |

### Adding a publication

1. Add the entry to `_bibliography/papers.bib`. Useful custom keys:
   `selected={true}` (shows on the homepage), `bibtex_show={true}`,
   `preview={file.png}`, `pdf`, `code`, `slides`, `poster`, `website`, `abbr`.
2. Drop the preview image in `assets/img/publication_preview/`.
3. Any *new* custom BibTeX key must also be added to `filtered_bibtex_keywords`
   in `_config.yml`, or it leaks into the displayed BibTeX block.

The publications page no longer needs a hardcoded `years:` list.

### Adding a news item

Create the next `_news/announcement_N.md`. `inline: true` renders a one-line
item with no detail page (the common case); `inline: false` needs a `title` and
gets its own page.

### Adding a project

Copy an existing file in `_projects/`. The `category` must be one of the values
in `display_categories` on `_pages/projects.md` (`[Research, Engineering]`) or
the card is silently dropped. `importance` sorts within a category — each
category has its own 1..N sequence. The `<div class="project-tech"><span>…</span></div>`
chip block is a local addition, styled in `assets/css/main.scss`.

## Local deviations from upstream al-folio

Worth knowing before "fixing" something that looks broken:

- **Two local overrides of `al_folio_core` files**, recorded with checksums in
  `.al-folio-overrides.yml`:
  - `assets/css/main.scss` — verbatim gem copy plus a "Site-local
    customizations" block at the bottom (blue theme color, card/news/social
    spacing, `project-tech` chips). Put new styling there, using
    `var(--global-theme-color)` rather than hardcoded hex.
  - `_layouts/about.liquid` — capitalized "News"/"Publications" headings,
    contact note above the social icons, `bio` class on the article.
  After bumping `al_folio_core`, run `bundle exec al-folio upgrade overrides audit`,
  reconcile with `... overrides diff <path>`, then `... overrides accept <path>`.
- **`_pages/cv.md` and `_pages/ipdb_demo.md` are redirects** via al-folio's
  built-in `redirect:` front matter (meta refresh after ~3s) to a Google Drive
  PDF and a SharePoint video respectively. `_data/cv.yml` is still the upstream
  placeholder and is not rendered.
- **No blog, books, or teaching.** Those collections, pages and demo posts were
  removed; pagination, blog archives, Disqus and `external_sources` are off in
  `_config.yml`. `latest_posts` is disabled on the about page.
- Dark mode and medium-zoom are off in `_config.yml`.
- Upstream maintainer CI workflows (unit tests, visual regression, star history,
  screenshots, citation updates, …) were removed; only deploy, docker, and
  `upgrade-check` remain.
- Upstream placeholder images under `assets/img/` are still present and unused.
