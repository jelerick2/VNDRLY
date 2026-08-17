# AGENTS.md

## Cursor Cloud specific instructions

This repository is a **Jekyll / GitHub Pages documentation site** (custom domain `vndrly.ai`, see `CNAME`), not an application. Content is Markdown: `README.md` (rendered as the site index) and `docs/testing/vndrly-uat-script.md`. There is no application source, database, or test suite here. The actual VNDRLY platform lives in a separate repo (`vndrly/VNDRLY.ai`), referenced by the UAT doc.

### Local preview (the "app")

- Serve: `bundle exec jekyll serve --host 0.0.0.0 --port 4000` (add `--livereload` for auto-reload). Auto-regeneration on file save works out of the box.
- Build only: `bundle exec jekyll build` → output in `_site/` (git-ignored).
- Gems are installed project-locally under `vendor/bundle` (git-ignored). If `bundle exec` fails with a missing-gem error, run `bundle install` (see update script).

### Non-obvious gotchas

- The Markdown files have **no YAML front matter**. They only render to HTML because the pinned `github-pages` gem bundles `jekyll-optional-front-matter` (renders front-matter-less Markdown) and `jekyll-readme-index` (uses `README.md` as `index.html`). A bare `jekyll` install would copy the `.md` files verbatim instead — always use the bundled `github-pages` gem (Jekyll 3.10) via `bundle exec`, which mirrors GitHub Pages.
- CI/deploy uses `actions/jekyll-build-pages` (its own preinstalled `github-pages` gem), not the repo `Gemfile`. The committed `Gemfile`/`Gemfile.lock` exist only for local preview; the action ignores them (it only prints a warning if it can't satisfy them, which it can since we pin `github-pages`), so they do not affect the deployed site.
- `bundle exec github-pages health-check` prints a GitHub API auth warning locally (no token); this is expected and harmless for preview.

### There is no lint or test step

Validation = a clean `bundle exec jekyll build` and visually checking served pages.
