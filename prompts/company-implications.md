# Company Implications

This is the key differentiator. Every news item gets a one-sentence "so what for YOUR company" implication line. This is what turns a generic news roundup into strategic intelligence.

## How to Write Implications

Read `config.company` to understand:
- **What the company does** (`config.company.description`)
- **Who reads this** (`config.company.audience`)
- **What decisions they make** (`config.company.decisions`)

Then for each item, ask: "If I were this reader making this decision, why would I care about this news?"

## Tone: Observational, Not Prescriptive

**GOOD — flag the signal:**
- "This opens a design-in window for [company]'s EV connector line"
- "[Competitor] is moving into a space [company] currently owns"
- "New regulation will require additional sensors in every vehicle — potential volume increase"
- "This OEM's platform delay pushes out [company]'s projected design-in timeline"
- "Customer consolidation reduces [company]'s negotiating leverage in this segment"

**BAD — never prescribe action:**
- "[Company] should pursue this opportunity"
- "The team should reallocate budget toward..."
- "We recommend accelerating the roadmap"
- "Management needs to respond to this"
- "This requires immediate attention"

## When No Clear Implication Exists

Use: "No direct [company name] implication — included for market context."

This is fine. Not every item needs a strategic implication. Including it for context is honest and useful.

## Think About These Dimensions

- **Revenue impact:** Does this create or threaten revenue?
- **Competitive position:** Does this strengthen or weaken a competitor?
- **Design-in windows:** Does this create a new opportunity to win business?
- **Regulatory exposure:** Does this change compliance requirements?
- **Customer shifts:** Is a key customer changing strategy?
- **Supply chain:** Does this affect sourcing, manufacturing, or logistics?

## Format

Render as italicized text below the item summary:
```
Tesla implication: This opens a new market segment for Tesla's energy storage business.
```

Use the actual company name from `config.company.name`, not "the company" or "your company."
