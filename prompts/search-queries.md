# Search Query Construction

For each enabled category in `config.categories`, construct exactly 2 WebSearch queries.

## Query Template

```
[industry focus terms] [category searchHints] [date qualifier]
```

## Rules

- **Max 15 words per query.** WebSearch works better with focused queries.
- **Date qualifier:** Use current month and year (e.g., "March 2026"). For daily briefings, use "this week" or specific date range.
- **No quotes inside queries.** WebSearch handles them differently from Google.
- **For Competitors category:** Inject competitor names from `config.competitors[]` into the query.
- **For Customers category:** Emphasize `config.industry.focus` terms to find customer-relevant news.
- **Geography:** If `config.industry.geography` is not "global", add geographic qualifier (e.g., "North America", "Europe").

## Curated Sources (priority)

Before general searches, check `config.sources.curated[]`:
- For each curated source with `requiresAuth: false`: use WebFetch to fetch the page URL and extract recent headlines.
- For each curated source with `requiresAuth: true`: use gstack `/browse` with imported cookies. If cookies not set up, skip and note it.
- Curated source items get priority placement in the briefing.

## General Search Strategy

For each category, combine the first 2-3 items from `config.industry.focus` with the category's `searchHints`:

| Config | Example |
|---|---|
| `industry.focus`: ["automotive connectors", "EV charging"] | |
| Category: "Regulatory" with hints: ["regulation", "standard"] | |
| **Query 1:** | `automotive connector regulation standard change March 2026` |
| **Query 2:** | `EV charging regulation compliance update March 2026` |

## Rate Limiting

- Execute categories sequentially, not in parallel
- If you hit rate limits, wait 5 seconds between categories
- If a search fails, skip that query and note in the briefing footer

## Source Domain Scoping

If `config.sources.preferredDomains` is populated, consider adding `site:` qualifiers to at least one query per category for higher-quality results. Example:
```
automotive OEM platform announcement site:reuters.com OR site:autonews.com March 2026
```

Only use domain scoping on one of the two queries per category — keep the other broad to catch items from unexpected sources.
