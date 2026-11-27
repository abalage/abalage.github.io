# Balagetech

Tech focused blog of an Engineer working in IT.

The site is a static [Jekyll](https://jekyllrb.com/) website built on the
[minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/) theme and
published on [GitHub Pages](https://pages.github.com/) at
<https://abalage.github.io> (custom domain `balagetech.com`, see the `CNAME`
file).

## Repository layout

```
├── _config.yml          # Jekyll configuration (url, permalink, theme settings)
├── _posts/              # Blog posts: YYYY-MM-DD-slug.md with YAML front matter
├── assets/
│   ├── images/          # Logo, header image, author photo
│   ├── uploads/         # Post images, organized by year/month
│   └── css/             # Custom stylesheets
├── index.html           # Home page (splash layout with post grid)
├── privacy-policy.md    # Privacy policy page
├── Gemfile              # Ruby dependencies (github-pages gem)
├── CNAME                # Custom domain: balagetech.com
└── wp/                  # WordPress backup (gitignored, never commit this)
```

Post URLs keep the original WordPress shape `/:year/:month/:day/:title/`
(see `permalink:` in `_config.yml`), so old links keep working.

## Local development

Requires Ruby and Bundler. Install dependencies once:

```bash
bundle install
```

Then preview the site locally (usually at <http://localhost:4000>):

```bash
bundle exec jekyll serve
```

Verify the production build passes:

```bash
bundle exec jekyll build
```

## Writing a post

Create `_posts/YYYY-MM-DD-slug.md` with front matter:

```yaml
---
title: "Post title"
date: YYYY-MM-DD HH:MM:SS +0000
categories: [tutorials]
tags: [linux, nfs]
---
```

Content is Markdown (kramdown); code samples go in fenced code blocks with a
language hint. Images referenced from a post live under
`assets/uploads/YYYY/MM/` and are referenced as `/assets/uploads/YYYY/MM/file.ext`.

## License

Blog articles and photographs are licensed under
[CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.en).
The site name, logo, favicon, design, and layout are not covered by that
license — see [LICENSE](LICENSE) for the full terms.
