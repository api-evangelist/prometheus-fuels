---
name: Search Prometheus Fuels site content
description: Run a unified search across Prometheus Fuels pages, posts and news articles using the WordPress REST search endpoint, then hydrate each match with its full record.
api: openapi/prometheus-fuels-search-api-openapi.yml
operations: [listWpV2Search, getWpV2PagesById, getWpV2PostsById, getWpV2NewsArticlesById]
---

# Search Prometheus Fuels site content

The WordPress REST search endpoint returns lightweight, cross-type matches across the Prometheus
Fuels corporate site — the Home, About, Technology and ULDES pages, the blog posts, and the
`news-articles` press items — which you then hydrate with the full resource. Anonymous — no
credentials required.

## Base
`https://prometheusfuels.ai/wp-json`

## Steps

1. **Search** — `listWpV2Search` (`GET /wp/v2/search`) with `search=<terms>`. Optionally constrain
   `type` and `subtype`, and paginate with `page` / `per_page`. Each result carries `id`, `title`,
   `url`, `type` and `subtype`.
2. **Hydrate a page match** — where `subtype` is `page`, call `getWpV2PagesById`
   (`GET /wp/v2/pages/{id}`) with the result `id` for the full rendered content and `parent`.
3. **Hydrate a post match** — where `subtype` is `post`, call `getWpV2PostsById`
   (`GET /wp/v2/posts/{id}`) for content, author, categories and tags.
4. **Hydrate a news match** — where `subtype` is `news-articles`, call `getWpV2NewsArticlesById`
   (`GET /wp/v2/news-articles/{id}`).
5. **Enrich or trim** — add `_embed` to a hydrate call to inline author and featured media where the
   type exposes them, or `_fields` to cut the response down (the default carries a large `yoast_head`
   SEO block).

## Conventions
- **Pagination**: `page` / `per_page` with `X-WP-Total` / `X-WP-TotalPages` headers.
- **Field selection**: `context` (`view` | `embed` | `edit`) — `edit` requires authentication;
  `_fields` for sparse fieldsets.
- **Errors**: flat WordPress envelope `{ code, message, data: { status } }` — see
  `errors/prometheus-fuels-problem-types.yml`.
- Search is read-only and safe to retry.

## Scope note
This searches the Prometheus Fuels *website*. Prometheus Fuels is an electrofuels manufacturer and
exposes nothing about its fuel production, ULDES systems, plant telemetry or offtake agreements
through any API.
