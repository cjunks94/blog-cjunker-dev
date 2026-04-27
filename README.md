# blog.cjunker.dev

## Overview
Personal engineering blog. Astro static site with MDX content, served by nginx in a container on Railway. Sibling to the portfolio at https://cjunker.dev — design tokens (forest theme, brutalist borders + offset shadows) are duplicated across the two repos so they can drift independently.

## Architecture

```
src/content/blog/*.mdx
        │  (Zod-validated frontmatter)
        ▼
   astro build  ──►  dist/  ──►  nginx:alpine (Docker)  ──►  Railway
        │
        ├─ /index.html, /<slug>/index.html       (post pages)
        ├─ /tags/index.html, /tags/<slug>/...    (tag pages — slugified URLs)
        ├─ /rss.xml                              (full feed, CORS for portfolio)
        ├─ /sitemap-index.xml                    (@astrojs/sitemap)
        └─ /404.html
```

- **Astro 6** (static output) with `@astrojs/mdx`, `@astrojs/sitemap`, `@astrojs/rss`
- **Shiki** (`github-dark`) for code highlighting
- **Strict TypeScript** via `astro/tsconfigs/strict`
- **Node** `>= 22.12.0`

Source layout:

```
src/
├── components/        # BaseHead, Header, Footer, PostCard, TableOfContents, TagList
├── content/blog/      # *.md / *.mdx posts
├── content.config.ts  # blog collection schema
├── layouts/           # BaseLayout, PostLayout
├── pages/             # index, [...slug], 404, rss.xml, tags/
├── styles/            # tokens.css, global.css, article.css
└── utils/slug.ts      # tagSlug() — used everywhere tags appear in URLs
```

## Setup

```bash
# requires Node >= 22.12.0
npm install
npm run dev    # http://localhost:4321
```

## Configuration

No runtime env vars — output is fully static. Build-time config:

- `astro.config.mjs` — `site` is the canonical URL (drives sitemap + RSS absolute links)
- `src/content.config.ts` — frontmatter schema (Zod)
- `nginx.conf` — caching, CORS, 404 fallback
- `railway.json` — Railway deploy config (Dockerfile builder, healthcheck on `/`)

### Authoring posts
Drop a file into `src/content/blog/`. `.md` and `.mdx` both work.

```yaml
---
title: "Post title"
description: "Shown in list, RSS, and meta tags."
date: 2026-04-26
updatedDate: 2026-05-01      # optional
tags: ["observability", "go"]
draft: false                  # drafts excluded from list, RSS, sitemap, tag pages
radarQuadrant: "techniques"   # optional — adds "View on Tech Radar" link
---
```

`radarQuadrant` accepts `languages-frameworks | platforms | tools | techniques`.

### Tags
Display strings are preserved as authored. URLs are slugified through `tagSlug()` (`lowercase`, non-alphanumeric → `-`), so `"Go (1.22)"` becomes `/tags/go-1-22/`. Don't hardcode `/tags/<raw>` — always route through `tagSlug()`.

## Usage

| Command           | Action                              |
| :---------------- | :---------------------------------- |
| `npm run dev`     | Dev server at `localhost:4321`      |
| `npm run build`   | Static build to `./dist/`           |
| `npm run preview` | Preview the production build        |

## Testing

No test suite yet — content site with no business logic. Pre-merge sanity:

```bash
npm run build      # surfaces schema/type errors via astro build
```

A future `astro check` script + lint config + CI would close the gap against the workspace standards (typecheck, lint, ≥80% coverage). Tracked as a follow-up.

## Deployment

Railway builds from `Dockerfile` (multi-stage: `node:20-alpine` → `nginx:alpine`, listens on `8080`). `railway.json` sets the healthcheck and restart policy.

`nginx.conf` handles:

- Clean URLs (`try_files $uri $uri/ $uri.html`)
- Long-cache `immutable` for hashed static assets
- CORS on `/rss.xml` for `https://cjunker.dev` (the portfolio embeds the feed)
- `404.html` fallback (generated from `src/pages/404.astro`)

**Rollback:** redeploy the prior Railway build, or `git revert` and let CI redeploy. No DB, no migrations — rollback is purely the static bundle.

**Analytics:** Umami is wired as a paused placeholder in `src/components/BaseHead.astro`. Re-enable when ready.
