# CLAUDE.md

## What this is

Udesh Kumarasinghe's personal academic website — a Jekyll site built on the
[al-folio](https://github.com/alshedivat/al-folio) theme, served at
`https://www.udesh.xyz` (see `CNAME`). Almost all work here is **content
editing**, not code: adding a publication, a news item, or a project.

The theme's own docs live in `README.md` (upstream, 30KB — mostly irrelevant to
day-to-day edits). Prefer this file.

## Build & run

`Gemfile.lock` is gitignored and no `vendor/` is checked in, so there is no
pinned dependency set locally. CI uses **Ruby 3.0.2** (`.github/workflows/deploy.yml`);
the system Ruby on this machine is 2.6, so a bare `bundle install` may fail.

```bash
bundle exec jekyll serve          # local preview (needs Ruby 3.x + imagemagick)
bash bin/cibuild                  # = bundle exec jekyll build
bash bin/docker_build_image.sh    # build the al-folio:latest image
bash bin/docker_run.sh            # serve on :8080 inside Docker (deletes Gemfile.lock first)
```

Two build-environment notes that bite:

- **imagemagick is required.** `jekyll-imagemagick` is enabled in `_config.yml`
  and generates `-480/-800/-1400.webp` variants for everything under
  `assets/img/`; `_includes/figure.html` points `<source srcset>` at them.
- **Minification only happens in production.** `jekyll-minifier` is active when
  `JEKYLL_ENV=production` (which `bin/deploy` exports), so local output differs
  from deployed output.

## Deployment

Push to `main` → the `deploy` workflow runs `bin/deploy`, which builds with
`JEKYLL_ENV=production`, wipes the worktree, moves `_site/*` to the root, adds
`.nojekyll`, and force-pushes to the **`gh-pages`** branch. Pull requests run
the same thing with `--no-push` as a build check.

Never commit to `gh-pages` by hand — it is fully regenerated each deploy.

## Content model

| What | Where | Notes |
|---|---|---|
| Bio / landing page | `_pages/about.md` | `layout: about`; toggles news, selected papers, social via front matter |
| Publications | `_bibliography/papers.bib` | rendered by `jekyll-scholar` through `_layouts/bib.html` |
| News | `_news/announcement_N.md` | numbered sequentially; `news_limit: 5` in `_config.yml` caps the homepage list |
| Projects | `_projects/{r,e}N_slug.md` | `r` = Research, `e` = Engineering |
| Nav pages | `_pages/*.md` | ordered by `nav_order`: publications 1, projects 2, cv 4 |

### Adding a publication

1. Add the entry to `_bibliography/papers.bib`. Useful custom keys al-folio
   understands: `selected={true}` (shows on the homepage via
   `_includes/selected_papers.html`), `bibtex_show={true}`, `preview={file.png}`,
   `pdf`, `code`, `slides`, `poster`, `website`, `abbr`.
2. Drop the preview image in `assets/img/publication_preview/` — `preview` is
   resolved against that directory unless it contains `://`.
3. **Update `years:` in `_pages/publications.md`.** That list is hardcoded
   (`[2024, 2023, 2022]`) and the page loops over it, so a paper in a year
   missing from the list renders nowhere. This is the single easiest thing to
   forget.
4. Any *new* custom BibTeX key must also be added to `filtered_bibtex_keywords`
   in `_config.yml`, or it leaks into the displayed BibTeX block
   (`_plugins/hideCustomBibtex.rb` strips keys by that list).

Author display is capped at `max_author_limit: 3`, with the rest revealed on click.

### Adding a news item

Create the next `_news/announcement_N.md`. `inline: true` renders a one-line
item with no detail page (the common case); `inline: false` needs a `title` and
gets its own page at `/news/<slug>/`.

### Adding a project

Copy an existing file in `_projects/`. The `category` must be one of the values
in `display_categories` on `_pages/projects.md` (`[Research, Engineering]`) or
the card is silently dropped. `importance` sorts within a category — each
category has its own 1..N sequence, so `r*` and `e*` numbering is independent.
The `<div class="project-tech"><span>…</span></div>` chip block is a local
addition to the theme, styled at `_sass/_base.scss:680`.

## Local deviations from upstream al-folio

Worth knowing before "fixing" something that looks broken:

- **Disabled pages are commented out wholesale**, not deleted:
  `_pages/teaching.md`, `_pages/repositories.md`, `_pages/dropdown.md`. The
  whole file is inside an HTML comment, front matter included.
- **`_pages/cv.md` is a redirect**, via `jekyll-redirect-from`'s `redirect_to`,
  to a Google Drive PDF. Consequently `_data/cv.yml` is still the upstream
  Albert Einstein placeholder and is **not** rendered anywhere — don't treat it
  as real content.
- **`_pages/ipdb_demo.md`** likewise redirects to a SharePoint video. It
  declares `layout: ipdb_demo`, which does not exist in `_layouts/`; the
  redirect takes precedence, so it works anyway.
- **No blog.** There is no `_posts/` directory and `blog_nav_title` is
  commented out in `_config.yml`, so the blog nav entry never renders.
  `_plugins/external-posts.rb` (RSS import into `_posts/`) is inert because
  `external_sources` is unset.
- Several theme features are off in `_config.yml`: `enable_darkmode`,
  `enable_medium_zoom`, `enable_tooltips`, analytics, and site verification.
  `enable_masonry` and `enable_math` (MathJax) are on.
- Upstream placeholder images (`assets/img/1.jpg`…`12.jpg`, `*-screenshot.png`)
  are still present and unused.

## Conventions

- Keep theme files (`_layouts/`, `_includes/`, `_sass/`) close to upstream;
  customizations are small and localized (e.g. `project-tech`). Note any new
  divergence here so the next al-folio rebase is tractable.
- Styling goes in `_sass/_base.scss`; color tokens come from
  `_sass/_themes.scss` / `_variables.scss` — use `var(--global-theme-color)`
  rather than hardcoded hex.
