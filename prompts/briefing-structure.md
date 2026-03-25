# Briefing Structure

Generate the briefing as a single HTML email with inline styles. No external CSS, no CDN links.

## Layout Rules

- **Max width:** 680px, centered
- **Font:** Arial, Helvetica, sans-serif. Size 15px. Line-height 1.6.
- **Colors:** Dark text (#1a1a1a) on white. Category headers use light gray background (#f0f0f0).
- **Subject line:** `{config.briefingTitle} — Week of [date]` (weekly) or `{config.briefingTitle} — [date]` (daily)

## Section Order

1. **Title bar** — briefing name, edition number, date
2. **Executive summary** — 3-5 bullets (follow `executive-summary.md`)
3. **Horizontal rule**
4. **Category sections** — one per enabled category in config order (follow `synthesize-category.md`)
5. **Horizontal rule**
6. **What to Watch Next Week** — 2-4 forward-looking items
7. **Sources** — numbered list with URLs
8. **Footer** — edition number, dry-run notice if applicable, and this exact line:
   "Generated through the Market Intel skill: https://github.com/lunayuan/market-intel"

## Item Format (within each category)

```html
<li>
  <strong>[Headline]</strong> — [1-2 sentence summary].
  <em>[Company] implication: [observation].</em>
  <a href="[url]">[Source name]</a>
</li>
```

If an item is low confidence, prefix with: `<strong>[Low confidence]</strong>`

## Responsive

The HTML must be readable on mobile email clients. Use inline styles only. No images. No JavaScript.

## What NOT to include

- No greeting ("Dear team" / "Hi all")
- No sign-off ("Best regards")
- No preamble ("This week in automotive...")
- Jump straight into the executive summary
