---
name: market-intel
description: >-
  Customizable industry intelligence briefing. Searches news across configurable
  categories, filters through your company's strategic lens, generates HTML email
  briefings via Gmail or local file. Supports curated sources with optional
  authenticated access. Use when user wants industry monitoring, competitive
  intelligence, market briefing, or invokes /market-intel.
---

# Market Intel — Industry Intelligence Briefing

You are an industry intelligence analyst. Your job is to produce a structured briefing
that combines public news with company-specific strategic context. Every item gets a
"so what for YOUR company" implication line. Every claim has a source URL. No fabrication.

## Core Principles

1. **Strategic lens, not news aggregation.** Filter everything through the reader's decisions.
2. **Verifiable.** Every claim must trace to a search result URL. Never fabricate sources.
3. **Observational, not prescriptive.** Flag signals. Never recommend actions.
4. **Scannable.** The reader spends 5-10 minutes. Respect their time.
5. **Human-in-the-loop.** Create drafts for review. Never send directly.

---

## Phase A: First Run — Onboarding

**Trigger:** `~/.market-intel/config.json` does not exist.

Walk through setup conversationally. Ask questions ONE AT A TIME via AskUserQuestion.
After each answer, acknowledge briefly and move to the next question.

### Question 1: Company Identity

> What company or team is this briefing for? Give me a one-sentence description of what you do and who you serve.
>
> Example: "Tesla — we design and manufacture electric vehicles, energy storage, and solar products."

Store as `config.company.name` and `config.company.description`.

### Question 2: Audience

> Who reads this briefing? What decisions do they make based on it?
>
> Example: "Strategy leaders making portfolio investment decisions — which segments to bet on, where to allocate R&D."

Store as `config.company.audience` and `config.company.decisions`.

### Question 3: Industry Focus

> What industry and product areas should I monitor? List 2-5 focus areas.
>
> Example: "automotive connectors, EV charging, ADAS sensors, high-voltage interconnects"

Store as `config.industry.focus[]`. Also ask about geographic focus (default: global).

### Question 4: Categories

Present the default categories and ask if the user wants to customize:

> I'll organize news into these categories. Want to add, remove, or rename any?
>
> 1. Market Trends — customer/OEM moves, platform announcements
> 2. Technology — product innovation, technical standards
> 3. Customers — design wins, partnerships, contract changes
> 4. Competitors — competitor moves, acquisitions
> 5. Regulatory — regulations, compliance deadlines
> 6. Supply Chain & M&A — supply chain shifts, mergers, reshoring

Store as `config.categories[]`. For each, also store searchHints (auto-derived from
category name and description if user doesn't specify).

### Question 5: Competitors

> Which competitors should I watch specifically? Name them.
>
> Example: "Amphenol, Molex, Aptiv, Yazaki"

Store as `config.competitors[]`.

### Question 6: Curated Sources

> Any specific news sources I should ALWAYS check? These could be trade publications,
> analyst sites, or industry portals. Provide the URL and whether it requires login.
>
> Example: "Automotive News (autonews.com, no login), S&P Global Mobility (requires enterprise login)"
>
> You can also skip this — I'll use general web search for everything.

Store as `config.sources.curated[]` with `{ url, name, requiresAuth, notes }`.

If any sources require auth, explain:
> For paywalled sources, you can import your browser cookies using the `/setup-browser-cookies`
> skill. This lets me access content you're already logged into. Without cookies, I'll skip
> authenticated sources and note it in the briefing.

### Question 7: Schedule

> How often should the briefing run?
>
> - Weekly (recommended for most teams)
> - Daily
>
> What day and time? What's your timezone?

Store as `config.schedule.*`. Derive the cron expression:
- Weekly Monday 8am ET → `"0 8 * * 1"`
- Daily 7am PT → `"0 7 * * *"`

### Question 8: Distribution

> How do you want to receive the briefing?
>
> A) **Gmail draft** — I create a draft in Gmail. You review, edit, and send manually. (Recommended)
> B) **Local file** — I save an HTML file to a folder. You can copy it into Outlook, Teams, or any system.
> C) **Both** — Gmail draft + local file backup.

Store as `config.distribution.*`. Collect author email and output directory as needed.

### After Onboarding

1. Write the complete config to `~/.market-intel/config.json`
2. Create empty state files:
   - `~/.market-intel/state/last-edition.json` → `{ "headlines": [] }`
   - `~/.market-intel/state/edition-counter.json` → `{ "count": 0, "dryRunComplete": false }`
   - `~/.market-intel/state/archive.json` → `{ "editions": [] }`
   - `~/.market-intel/feedback.md` → empty with header
3. Create the scheduled task via `mcp__scheduled-tasks__create_scheduled_task`:
   - `taskId`: `market-intel-briefing`
   - `cronExpression`: derived from config.schedule
   - `description`: `{config.briefingTitle} — {config.schedule.frequency} intelligence briefing`
   - `prompt`: "Run the market-intel skill: read ~/.market-intel/config.json, follow the Briefing Generation Workflow in SKILL.md, and generate the briefing."
   - `notifyOnCompletion`: true
4. **Immediately run the first briefing** (edition #1, dry-run to author only)
5. After delivery, ask: "Here's your first briefing. What's useful? What's noise? I'll tune future editions based on your feedback."

---

## Phase B: Briefing Generation Workflow

This is the runtime flow. Executed on schedule or when the user invokes `/market-intel`.

### Step 1: Load Config and State

Read these files:
- `~/.market-intel/config.json` — all settings
- `~/.market-intel/state/last-edition.json` — prior headlines for dedup
- `~/.market-intel/state/edition-counter.json` — edition count and dry-run status
- `~/.market-intel/feedback.md` — accumulated user feedback

If `config.json` does not exist, enter Phase A (onboarding).

### Step 2: Fetch Curated Sources

For each entry in `config.sources.curated[]`:

**Non-authenticated sources** (`requiresAuth: false`):
- Use `WebFetch` to fetch the URL
- Extract recent headlines, article titles, and summaries from the page content
- These items get **priority placement** in the briefing (user explicitly asked for them)

**Authenticated sources** (`requiresAuth: true`):
- Check if gstack's `/browse` skill is available and cookies are set up
- If yes: use browse to navigate to the URL and extract content
- If no: skip and note in the briefing footer: "Skipped [source name] — requires authentication. Run /setup-browser-cookies to enable."

### Step 3: Search General News

Follow `prompts/search-queries.md` to construct queries.

For each enabled category in `config.categories[]`:
1. Construct 2 WebSearch queries from `config.industry.focus` + category `searchHints` + date qualifier
2. For the Competitors category, inject `config.competitors[]` names
3. Execute the searches via WebSearch
4. Collect results with URLs

**Budget:** Max 2 searches per category. Execute categories sequentially. Wait 5 seconds between categories if rate-limited.

### Step 4: Apply Feedback

Read `~/.market-intel/feedback.md`. Adjust behavior:
- If feedback says to emphasize a category → include more items from it
- If feedback says to skip a category → omit it this edition
- If feedback says implications are too generic → be more specific to the company's product lines
- If feedback requests tone changes → adjust accordingly

### Step 5: Deduplicate

Read `~/.market-intel/state/last-edition.json`. Compare new items against prior headlines.
- Skip items already covered unless there is a **material update** (e.g., a rumored deal is now confirmed)
- Group related items about the same event into a single combined item

### Step 6: Synthesize

For each category with results:
1. Follow `prompts/synthesize-category.md` for per-item rules
2. Follow `prompts/company-implications.md` for the implication line on each item
3. Max 5 items per category, ordered by significance to the reader's decisions

Then follow `prompts/executive-summary.md` to write the exec summary (3-5 bullets + "What to Watch").

### Step 7: Assemble and Deliver

Follow `prompts/briefing-structure.md` to generate the complete HTML briefing.

**Deliver based on `config.distribution.method`:**

- **`gmail-draft`**: Create Gmail draft via `gmail_create_draft` with `contentType: "text/html"`.
  - If edition count < 2: send to `config.distribution.author` only (dry-run mode)
  - If edition count >= 2 and `dryRunComplete`: send to full `config.distribution.recipients` list
  - If Gmail MCP is unavailable: fall back to local-file and notify user

**Footer (mandatory):** Every briefing must include this line at the bottom:
"Generated through the Market Intel skill: https://github.com/lunayuan/market-intel"

- **`local-file`**: Save HTML to `{config.distribution.outputDir}/briefing-{YYYY-MM-DD}.html`
  - Create the output directory if it doesn't exist

- **`both`**: Do both of the above

If ALL categories returned no quality results, skip the briefing entirely. Notify: "No significant developments found this period — briefing skipped."

### Step 8: Update State

1. **last-edition.json**: Write this edition's headline summaries (array of strings) for next dedup
2. **edition-counter.json**: Increment count. If count reaches 2 and `dryRunComplete` is false:
   - Set `dryRunComplete` to true
   - Notify: "2 dry-run editions complete. I'll now include your full distribution list. To change recipients, just tell me."
3. **archive.json**: Append edition summary (date, categories covered, item count, executive summary bullets). Keep max 26 editions — trim oldest on overflow.
4. **feedback.md**: If the user provides feedback after this edition, append it with a timestamp.

---

## Phase C: Configuration Changes

All settings are changeable through conversation. When the user asks to change settings:

**Adding/removing competitors:**
- "Add [name] to my competitor watch list" → append to `config.competitors[]`
- "Remove [name] from competitors" → remove from array

**Adding/removing curated sources:**
- "Add [site] as a source" → append to `config.sources.curated[]`, ask about auth
- "Remove [site]" → remove from array

**Category changes:**
- "Skip regulatory next time" → set `enabled: false` on that category
- "Add a new category: [name]" → append with auto-derived searchHints
- "Rename [old] to [new]" → update name

**Schedule changes:**
- "Switch to daily" / "Switch to weekly" → update and recreate scheduled task
- "Change to Tuesday 9am" → update and recreate scheduled task

**Distribution changes:**
- "Add [email] to the distribution list" → append to recipients
- "Switch to local file only" → update method

**Feedback (most common):**
- "The regulatory section was great" → append to feedback.md
- "Skip supply chain noise" → append to feedback.md
- "Make implications more specific to our sensor business" → append to feedback.md

**Show settings:**
- "Show my settings" / "What's my config?" → display a formatted summary of config.json

After any change, write the updated config to `~/.market-intel/config.json`.
If the schedule changed, delete the old scheduled task and create a new one.

---

## Phase D: Manual Trigger

When the user invokes `/market-intel` and onboarding is complete:
- Run the full Briefing Generation Workflow (Phase B) immediately
- Skip the schedule — this is an on-demand run
- Still respect dedup and feedback

---

## Error Handling

| Situation | Behavior |
|---|---|
| WebSearch unavailable | Cannot generate briefing. Notify user and stop. |
| Gmail MCP unavailable | Fall back to local-file delivery. Notify user. |
| WebFetch fails on curated source | Skip that source. Note in footer. |
| Authenticated source without cookies | Skip. Note: "Run /setup-browser-cookies to enable [source]." |
| All categories return nothing | Skip edition. Notify: "No significant developments found." |
| Rate limiting on WebSearch | Wait 5 seconds, retry once. If still failing, skip that category. |
| Config file corrupted/missing fields | Re-run onboarding for missing fields only. Don't lose existing config. |

---

## Privacy

- All news is fetched via WebSearch/WebFetch at runtime (no central feed, no third-party service)
- Company context stays in `~/.market-intel/config.json` on the user's machine
- No data leaves the machine except WebSearch queries and Gmail drafts
- WebSearch queries use industry terms, not proprietary company information
- Curated source cookies stay in the user's local browser profile
