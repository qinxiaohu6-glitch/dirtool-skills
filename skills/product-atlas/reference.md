# Product Atlas — field reference

Companion to `SKILL.md`. Read this when you need to map a field; the recipes in the skill cover most tasks.

All shapes below were captured from the live site on 2026-11-09 and are stable, but the values (counts, listings) change.

## Listing object

Returned inside `/api/tools` (`tools[]`) and by `/api/tools/{slug}`.

| Field | Type | Notes |
|---|---|---|
| `slug` | string | URL key. The page is `/tools/{slug}`, Chinese `/zh/tools/{slug}`. |
| `name_en` / `name_zh` | string | Display name. Often identical when the product has no localised name. |
| `short_en` / `short_zh` | string | One-line tagline (card text). |
| `long_en` / `long_zh` | string | Longer description. May equal the tagline when the submitter supplied only one. |
| `url` | string | The product's own site. |
| `logo` | string | Absolute image URL, may be empty. |
| `category` | string | Category **slug**, not a display name. Resolve via `/api/directory/meta`. |
| `tags` | string[] | Tag slugs. May be empty. |
| `status` | string | `approved` for anything this endpoint returns. |
| `is_featured` | boolean | Always `false` on this endpoint — submitted listings are not sold featured placement. |
| `is_paid` | boolean | Listing was bought, rather than the free reviewed tier. |
| `featured_until` | string | Always `""` here, for the same reason as `is_featured`. |
| `votes_total` | number | **Same value as `votes_week`** on this endpoint: the current ISO week's count. Do not present it as an all-time total. |
| `votes_week` | number | Votes in the current ISO week. |
| `created_at` | string | `YYYY-MM-DD`. |
| `maker` | string | Submitter's display name. May be empty. |
| `shots` | string[] | Screenshot URLs. May be empty. |
| `detail_en` / `detail_zh` | object | **Optional** — present only when the operator wrote a localised detail block. Absent is normal, not an error. |

### `/api/tools/{slug}`

```json
{ "ok": true, "tool": { ...listing object... } }
```

There is **no top-level `votes` field** — the week's count arrives inside the tool as `votes_week` (and `votes_total`, which carries the same number).

| Status | Body | Meaning |
|---|---|---|
| 200 | `{ ok:true, tool, votes }` | Live. |
| 404 | `{ ok:false, reason:"not_found" }` | Never existed, still pending, or rejected — deliberately indistinguishable. |
| 410 | `{ ok:false, reason:"removed" }` | Was live, pulled by the operator. |

## `/api/directory/meta`

```json
{
  "ok": true,
  "categories": [
    {
      "slug": "chatbots",
      "icon": "message-square",
      "count": 5,
      "name_en": "AI Assistants",
      "name_zh": "AI 助手",
      "desc_en": "General-purpose assistants you can talk to.",
      "desc_zh": "通用对话助手，什么问题都能接。"
    }
  ],
  "tags": [{ "slug": "api", "count": 6, "name_en": "API", "name_zh": "API" }],
  "collections": []
}
```

- `categories` — 8 entries. `icon` is a sprite name, useful only for rendering.
- `tags` — 15 entries, same shape minus descriptions.
- `collections` — curated groupings; **empty array** on the live site, so do not assume a collection exists.
- `count` is live listings in that category/tag, curated plus approved submissions.
- Sends `cache-control: public, max-age=0, s-maxage=30`.

## `/api/board`

```json
{ "ok": true, "week": null }
```

`week` is the newest **closed** weekly board. `null` means no week has closed yet — that is the normal answer on a young install, not an error. When present, the object carries the archived week id and its ranked entries.

## `/api/ads`

```json
{ "ok": true, "board": null, "sidebar": null }
```

One ad per slot, the oldest live booking winning; `null` when the slot is unsold. A populated ad exposes its public fields only — never the buyer's email or the order id.

## `/api/ads/availability`

`GET /api/ads/availability?slot=board|sidebar&weeks=1|4`

```json
{
  "ok": true,
  "slot": "board",
  "weeks": 1,
  "starts": ["2026-11-16", "2026-11-23", "2026-11-30", "2026-12-07", "2026-12-14", "2026-12-21"],
  "available": { "2026-11-16": true, "2026-11-23": true, "2026-11-30": true, "2026-12-07": true, "2026-12-14": true, "2026-12-21": true }
}
```

- `starts` — the next six Mondays, ascending. A window is `weeks` weeks long starting on that Monday.
- `available` — keyed by the same dates; `false` means booked.
- `weeks` must be `1` or `4`. `slot` must be `board` or `sidebar`. Anything else is `400`.
- On a database failure: `{ ok:false, starts:[], available:{} }` — render nothing rather than inventing dates.

## `/api/dr`

`POST /api/dr`, `content-type: application/json`.

| Request | Result |
|---|---|
| `{"domain":"example.com"}` | DR4 for that domain. |
| `{"domain":"example.com","compare":true}` | Same, plus `compare: { dr3, dr4, delta }`. Costs two rate-limit slots. |

Success:

| Field | Type | Notes |
|---|---|---|
| `ok` | `true` | |
| `domain` | string | Normalised: lowercased, scheme and `www.` stripped. |
| `provider` | `"ahrefs"` | |
| `metric` | `"DR"` | |
| `version` | `"dr4"` | Always DR4, even when `compare` also returns DR3. |
| `value` | number | 0–100. |
| `checked_at` | number | Unix seconds. |
| `cached` | boolean | `true` when served from the 24-hour in-memory memo. |
| `license` | string | Present when upstream supplies one. Keep the `Domain Rating by Ahrefs` credit regardless. |
| `compare` | object | Only with `compare: true`: `{ dr3: {value, cached}, dr4: {value, cached}, delta }`, `delta = dr4 - dr3`. |

Failure is always HTTP 200 with `{ ok:false, reason, version:"dr4" }`, plus `failed_version` and `upstream_status` when relevant.

| `reason` | What it means | What to tell the user |
|---|---|---|
| `invalid_domain` | Empty or unparseable domain. | Ask for a bare domain such as `example.com`. |
| `not_configured` | Server has no upstream key. | The lookup tool is unavailable; do not fall back to an external checker. |
| `bad_key` | Upstream rejected the key (401/403). | Unavailable right now. |
| `rate_limited` | Per-IP window exhausted, or upstream 429. | Wait — the window is 10 minutes. |
| `no_data` | Upstream had no rating for the domain. | Common for new or tiny domains; report "no rating yet", not `0`. |
| `provider_error` / `bad_response` / `timeout` / `network_error` | Upstream failed. | Unavailable right now; retry later. |

A genuine DR of zero is returned as `ok:true, value:0` — it is **not** the same as `no_data`.

## Rate limits, caching and licence

- DR: 12 requests per 10 minutes per IP; `compare` consumes two.
- DR results memoised server-side for 24 hours, keyed by `domain|version`.
- `Domain Rating by Ahrefs` with a link to https://ahrefs.com/ is required wherever a DR number is displayed.
- Bulk harvesting of domain ratings is forbidden by the Domain Rating licence. Do not loop over domain lists.
