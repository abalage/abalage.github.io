# AGENTS.md — Balagetech blog (abalage.github.io)

## Project overview

This repository is the home of my personal blog, **Balagetech**.

## Target architecture (Jekyll on GitHub Pages)

- Jekyll site lives at the **repo root**: `_config.yml`, `_posts/`, `_layouts/`,
  `_includes/`, `assets/`, etc. GitHub Pages builds from the `main` branch.
- GitHub Pages only supports a whitelist of plugins. Do not rely on arbitrary plugins; if something is needed beyond the whitelist, prompt the user for decision.
- Local development: `Gemfile` with the `github-pages` gem (or plain `jekyll`),
  then:
  - `bundle install`
  - `bundle exec jekyll serve` — local preview
  - `bundle exec jekyll build` — verify the build passes

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

1. Keep all Jekyll site files at the repo root so GitHub Pages can serve them.
2. Verify every change with `bundle exec jekyll build` before considering it
   done; fix warnings that affect output.
3. Use theme [mmistakes/minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/docs) as the base and customize via `_layouts`/`_includes`/assets rather than plugins.