# Product Atlas — agent skill

An [Agent Skill](https://www.skills.sh/) that teaches AI agents to read the **Product Atlas** AI directory at [dirtool.com](https://dirtool.com) — its products, categories, listings, weekly board, ad-slot availability and Ahrefs Domain Rating (DR4).

No API key, no account, no write access. Every endpoint the skill uses is public and read-only.

## Install

```bash
npx skills add qinxiaohu6-glitch/dirtool-skills
```

Or run it without installing:

```bash
npx skills use qinxiaohu6-glitch/dirtool-skills@product-atlas
```

## What the skill covers

| Capability | Source |
|---|---|
| Curated AI products with URLs and one-line descriptions | `GET https://dirtool.com/llms.txt` |
| Categories, tags and live listing counts | `GET https://dirtool.com/api/directory/meta` |
| Community-submitted live listings | `GET https://dirtool.com/api/tools` |
| A single submitted listing | `GET https://dirtool.com/api/tools/{slug}` |
| Newest closed weekly board | `GET https://dirtool.com/api/board` |
| Which weekly ad slots are still open | `GET https://dirtool.com/api/ads/availability` |
| Ahrefs Domain Rating, DR4 (optionally DR3 → DR4 delta) | `POST https://dirtool.com/api/dr` |

## Layout

```
skills/
  product-atlas/
    SKILL.md      # frontmatter + the instructions an agent follows
    reference.md  # field-by-field shapes, error codes, rate limits
```

## Limits worth knowing

- **DR lookups: 12 per 10 minutes per IP.** `compare: true` costs two slots.
- **Attribution is required.** Any Domain Rating you display must carry `Domain Rating by Ahrefs`, with `Ahrefs` linked to https://ahrefs.com/. That is a condition of the Domain Rating licence, not a courtesy.
- **No ranking guarantees.** A paid listing or an ad slot does not change search rankings, traffic or domain authority.
- **Counts move.** The curated catalogue holds 30 products; submitted listings change daily.

## Licence

Code and skill text: MIT (see `LICENSE`).

The Domain Rating values returned by `/api/dr` are provided by Ahrefs and remain subject to the [Domain Rating License](https://ahrefs.com/legal). This repository grants no rights to that data beyond what that licence allows.
