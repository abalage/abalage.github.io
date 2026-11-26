# AGENTS.md — Balagetech blog (abalage.github.io)

## Project overview

This repository is the home of my personal blog, **Balagetech** — "Selfhosting
journey of a hardcore Linux fan" (author: Balázs Németh). It previously ran as a
self-hosted WordPress site at `https://balagetech.com`.

Right now the repo contains **only a backup of that WordPress production site**.
The goal is to convert its content to **Jekyll** and publish the site on
**GitHub Pages** as `https://abalage.github.io`. Later I will point the custom
domain **`balagetech.com`** at the GitHub Pages site.

## Current state of the repo

Tracked files: only `.gitignore` and `LICENSE`. There is no Jekyll site yet.

The `wp/` directory is **gitignored** (see `.gitignore`) and must stay that way:

- `wp/balagetech.WordPress.2026-11-18.xml` — WordPress WXR export (129 items:
  19 posts, 2 pages, 97 attachments, 1 draft; language en-US). Source of truth
  for all content to migrate.
- `wp/backup/` — full WordPress root backup, including
  `wp/backup/wp-content/uploads/2018..2026/` (post images, organized by year/month),
  `wp/backup/softsql.sql` (database dump), `wp/backup/wp-config.php` (contains
  secrets), and a `tar.gz` of the whole site.

**Never commit `wp/`.** It contains credentials, database dumps and personal
data. Do not delete it either — it is the migration source until migration is
verified complete.

## Target architecture (Jekyll on GitHub Pages)

- Jekyll site lives at the **repo root**: `_config.yml`, `_posts/`, `_layouts/`,
  `_includes/`, `assets/`, etc. GitHub Pages builds from the `main` branch.
- GitHub Pages only supports a whitelist of plugins (and safe themes such as
  `minima`). Do not rely on arbitrary plugins; if something is needed beyond
  the whitelist, use a GitHub Actions workflow that builds Jekyll and deploys
  to Pages instead of branch-source deployment.
- Local development: `Gemfile` with the `github-pages` gem (or plain `jekyll`),
  then:
  - `bundle install`
  - `bundle exec jekyll serve` — local preview
  - `bundle exec jekyll build` — verify the build passes
- `_config.yml` initially: `url: https://abalage.github.io`, `baseurl: ""`.
  When `balagetech.com` is pointed here, add a `CNAME` file containing
  `balagetech.com` and change `url:` to `https://balagetech.com`.

## Migration conventions (WordPress WXR → Jekyll)

Source: `wp/balagetech.WordPress.2026-11-18.xml`. Images:
`wp/backup/wp-content/uploads/YYYY/MM/...`.

- Parse the WXR export; migrate **published posts and pages**. The single draft
  is excluded unless explicitly marked `published: false` in front matter.
- Posts go to `_posts/YYYY-MM-DD-slug.md` — keep the original WordPress
  `wp:post_date` and `wp:post_name` (slug) so URLs and chronology are preserved.
- Front matter per post:
  ```yaml
  ---
  title: "..."
  date: YYYY-MM-DD HH:MM:SS +0000
  categories: [Tutorials]   # from wp:category, lowercase nicenames preferred
  tags: [linux, nfs]        # from wp:tag
  ---
  ```
- Convert post content from HTML to Markdown; keep code samples as fenced code
  blocks with a language hint. Do not lose shortcodes' intent — expand them into
  plain Markdown/HTML.
- Never change content or spelling. If your are in doubt ask the user for clarification.
- Rewrite image/media references from
  `/wp-content/uploads/YYYY/MM/file.ext` to site-local paths, and copy the
  referenced files from `wp/backup/wp-content/uploads/` into
  `assets/uploads/YYYY/MM/`. Only migrate files actually referenced by migrated
  content (the 97 attachments include many unused images).
- Preserve old permalink shape (`/:year/:month/:day/:title/`) via
  `permalink: /:year/:month/:day/:title/` in `_config.yml` so existing links
  keep working; add redirects only where a URL cannot be preserved.
- Existing categories in the export: Automation, Docker, Entertainment,
  File storage, Kubernetes, Script, Security, Syslog, Tutorials, Uncategorized.
  Tags include: Android, Ansible, Linux, Network, NFS, OpenWRT, Python,
  fail2ban, kubernetes, openSUSE, syslog-ng, and more.
- Content language: English. Tone: practical technical selfhosting guides.

## Working rules for agents

1. Never commit anything under `wp/`; keep `.gitignore` as is.
2. Keep all Jekyll site files at the repo root so GitHub Pages can serve them.
3. Verify every change with `bundle exec jekyll build` before considering it
   done; fix warnings that affect output.
4. Use theme [mmistakes/minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/docs) as the base
   and customize via `_layouts`/`_includes`/assets rather than plugins.
5. When the custom domain is switched on, the only changes should be the
   `CNAME` file and `url:` in `_config.yml` — do not hardcode
   `abalage.github.io` into content or templates.
