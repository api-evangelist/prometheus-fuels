---
name: Browse Prometheus Fuels news and press articles
description: Pull Prometheus Fuels' published news and press coverage from the site's `news-articles` custom post type, with pagination, filtering and media hydration.
api: openapi/prometheus-fuels-news-articles-api-openapi.yml
operations: [listWpV2NewsArticles, getWpV2NewsArticlesById, listWpV2Types, listWpV2Media]
---

# Browse Prometheus Fuels news and press articles

Prometheus Fuels publishes no product API. The one machine-readable surface it operates is the
WordPress REST API behind its corporate site, which exposes a site-specific `news-articles` custom
post type carrying the company's press and news coverage. All steps below are anonymous — no
credentials required.

## Base
`https://prometheusfuels.ai/wp-json`

## Steps

1. **Confirm the content type is present** — `listWpV2Types` (`GET /wp/v2/types`). The response object
   is keyed by type slug; `news-articles` is the Prometheus Fuels custom type. This step makes the
   skill safe against the type being renamed or removed.
2. **List the articles** — `listWpV2NewsArticles` (`GET /wp/v2/news-articles`). Paginate with
   `page` / `per_page` (`per_page` is bounded 1..100). Read `X-WP-Total` and `X-WP-TotalPages` from
   the response headers rather than looping until an empty page. Narrow with `search`, `slug`,
   `after` / `before` (ISO 8601), `include` / `exclude`, and sort with `order` / `orderby`.
3. **Trim the payload** — add `_fields=id,slug,date,title,excerpt,link` to the list call. The default
   response carries the full rendered `content` plus a large `yoast_head` SEO block for every item.
4. **Hydrate one article** — `getWpV2NewsArticlesById` (`GET /wp/v2/news-articles/{id}`) with the
   `id` from the list. `title.rendered`, `excerpt.rendered` and `content.rendered` are HTML strings;
   `link` is the canonical public URL on prometheusfuels.ai.
5. **Attach imagery** — `listWpV2Media` (`GET /wp/v2/media`) and read `source_url` / `media_details`.
   Note that on this site `news-articles` items do not expose a `featured_media` field on the public
   `view` context, so media must be matched by hand rather than followed from the article.

## Conventions
- **Pagination**: `page` / `per_page` with `X-WP-Total` and `X-WP-TotalPages` response headers, plus
  an RFC 8288 `Link` header carrying `rel="next"` / `rel="prev"`.
- **Errors**: flat WordPress envelope `{ code, message, data: { status } }` — a bad id returns
  `rest_post_invalid_id` (404), an out-of-range parameter returns `rest_invalid_param` (400). See
  `errors/prometheus-fuels-problem-types.yml`.
- **No idempotency keys** are supported; every operation here is a safe, retryable read.
- **Volume is small**: `X-WP-Total` was 3 on 2026-08-05. Do not build a crawler loop around this.
