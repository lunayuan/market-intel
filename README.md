# Market Intel

**Know what matters before the meeting starts.**

Most teams stay informed through a patchwork of LinkedIn scrolling, forwarded articles, and "did you see the GM announcement?" in Slack. Market Intel replaces that with a structured weekly briefing filtered through your company's strategic lens — every item gets a "so what for YOUR company" implication line.

Built as a Claude Code skill. Works for any industry.

## What You Get

- **Executive summary** — 3-5 biggest signals, scannable in 60 seconds
- **News by category** — customizable sections (Market, Technology, Customers, Competitors, Regulatory, Supply Chain)
- **Company implications** — every item filtered through your strategy ("This opens a design-in window for Tesla's EV connector line")
- **Source links** — every claim is verifiable, no hallucinated intelligence
- **"What to Watch"** — forward-looking items for the next week
- **Curated sources** — specify trade publications to always check, including paywalled sites you're logged into

Delivered as a Gmail draft you review before sending, or a local HTML file you can paste into Outlook, Teams, or whatever your company uses.

## Quick Start

1. Install the skill:
   ```bash
   git clone https://github.com/lunayuan/market-intel.git ~/.claude/skills/market-intel
   ```

2. In Claude Code, say **"set up market intel"** or type `/market-intel`.

3. Answer 8 setup questions about your company, industry, and preferences. That's it — your first briefing generates immediately.

No config files to edit. The agent walks you through everything conversationally.

## Example Industries

| Industry | Focus Areas | Categories |
|---|---|---|
| EV / Automotive | Battery tech, autonomous driving, charging | Competitor Vehicles, Battery & Supply Chain, Regulatory |
| Semiconductor Equipment | Lithography, deposition, metrology | Technology, Customers, Trade Policy |
| Renewable Energy | Solar, wind, battery storage | Market Trends, Policy, Supply Chain |
| Medical Devices | Surgical robotics, diagnostics | Regulatory (FDA), Technology, M&A |
| Enterprise SaaS | Cloud infrastructure, AI/ML | Customers, Competitors, Funding |

See `config/examples/tesla-ev-auto.json` for a complete sample.

## Changing Settings

Tell your agent conversationally — no files to edit:

- "Add BYD to my competitor watch list"
- "Switch to daily briefings"
- "The regulatory section was the most useful — emphasize it"
- "Skip supply chain noise unless there's a major acquisition"
- "Add reuters.com as a curated source"
- "Show me my current settings"

## Curated Sources

You can specify news sources to always check:

- "Add Electrek as a source — electrek.co, no login needed"
- "Add Bloomberg Green — requires my subscription login"

For paywalled sources, import your browser cookies so the skill can access content you're already logged into. Without cookies, paywalled sources are skipped gracefully.

## Customizing the Briefing

The briefing's behavior is controlled by plain-English prompt files in `prompts/`:

| File | What it controls |
|---|---|
| `briefing-structure.md` | HTML layout and section ordering |
| `synthesize-category.md` | How items are summarized |
| `company-implications.md` | How "so what" lines are written |
| `executive-summary.md` | How the exec summary is composed |
| `search-queries.md` | How search queries are built from your config |

These are plain English instructions, not code. To customize, copy any file to `~/.market-intel/prompts/` and edit it there. Your overrides always take priority.

## How It Works

1. Reads your config — company context, industry focus, categories, competitors
2. Checks your curated sources first (trade publications you specified)
3. Searches general news across each category via WebSearch
4. Deduplicates against the last edition so you never see the same item twice
5. Synthesizes items with company-specific implications
6. Creates a Gmail draft or saves an HTML file for your review
7. You edit and send when ready

The briefing gets smarter over time. Tell it what's useful and what's noise — it remembers.

## Delivery

| Method | How it works |
|---|---|
| Gmail draft | Creates a draft in Gmail. You review, edit, send. |
| Local file | Saves HTML to a folder. Copy into Outlook, Teams, or any system. |
| Both | Gmail draft + local file backup. |

The skill **never sends email directly**. You always review first. The first 2 editions go to you only (dry-run mode) so you can calibrate before sharing with your team.

## Requirements

- Claude Code with WebSearch
- Gmail MCP connector (for Gmail draft delivery) — optional if using local file
- Google Drive MCP (optional — enriches briefings with internal document context)

## Privacy

- All news is fetched via WebSearch at runtime — no central feed, no third-party service
- Your company context stays on your machine in `~/.market-intel/config.json`
- Search queries use industry terms, not proprietary company information
- Curated source cookies stay in your local browser profile
- Nothing is shared with anyone unless you hit send

## License

MIT
