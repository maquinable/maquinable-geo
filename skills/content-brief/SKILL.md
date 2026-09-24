---
name: content-brief
description: Builds a SERP-based, AI-friendly content brief for a target keyword - top 10 results, common headings, recurring terms, average length, freshness and real questions for the FAQ, in the market and language you choose. Saves it to briefs/<slug>.md. Use when the user says "brief for [keyword]", "content brief", "research the keyword X", "plan an article about X". También en español - "brief para [keyword]", "haz un brief", "investiga la keyword X", "prepárame el artículo sobre X".
---

# content-brief

Build a structured brief for one target keyword. The brief plans the article; it doesn't write it.

**Language:** write the brief in the article's language (the keyword's language unless the user says otherwise).

## Inputs

- **Target keyword** (required): the exact phrase to rank for.
- **Language** (default: the user's language).
- **Country / market** (optional, no default). Ask if the keyword is ambiguous across markets (for example Spanish in Spain vs Mexico vs Argentina). Without a country, search in the language only and say so in the notes.
- **Intent** (optional): informational / commercial / transactional / navigational. If missing, infer it from the SERP and state it.
- **Project config** (optional): if `maquinable-geo.config.md` exists at the project root, read domain, audience, topic clusters and voice from it.

## 1. SERP research

Use Claude's web search with the keyword, in the target language and market. Don't scrape Google, "People also ask" or Perplexity directly with scripts.

If the user configured their own SerpAPI or DataForSEO key (in the environment or the config file), you can use that API for more exact positions and PAA. Say which source you used.

Capture:
- Top 10 results (title, URL, snippet).
- Questions people ask about the topic (PAA-style questions, forums, Q&A sites).
- Related searches or angles that show up repeatedly.
- Featured snippet or answer box, if visible: what format (list, paragraph, table)?

## 2. Top 10 analysis

For each result, fetch the page and extract:

- **Length** (words).
- **H2 and H3** headings (structure).
- **Key terms** that appear in at least 5 of the 10 results: the "required topics".
- **Publication or last-update date**. If they're all old, a fresh article is an angle.

If a page can't be fetched (paywall, blocking), skip it and say how many were analyzed.

## 3. Real questions for the FAQ

GEO needs the questions people actually ask.

**Phrasing rule.** The substance of an answer can come from any source: English-language discussions, developer communities (Reddit, Hacker News, GitHub issues, Stack Overflow), documentation. But the **phrasing of the FAQ questions** comes from real searches in the article's language and market. Don't translate questions literally from another language or from a technical audience: they won't match any real search and they sound imported.

> Example. A developer forum discusses inference latency and token cost of AI agents on top of an ERP.
> ✗ FAQ: "What is the inference latency of AI agents on an ERP?"
> ✓ FAQ: "Will my system get slower if I add an AI agent?" Same substance, phrased like the person who actually searches.

Extract:
1. **Real objections and friction**: what people complain about, what doesn't work for them, what they ask twice. This feeds the FAQ and usually isn't in the SERP.
2. **The community's own terms**: what people really call it, which often differs from the marketing term. Cross-check with the required terms from step 2.
3. **Recent changes**: releases, prices, deprecations. A fresh-content angle against an outdated top 10.

Record where each thing came from in the brief's notes.

## 4. Brief structure

Save it to `briefs/<slug>.md` in the user's working directory (create the folder if needed).

```markdown
# Brief: [keyword] ([date])

## Meta
- **Target keyword:** [exact phrase]
- **Language / market:** [language, country or "no country"]
- **Intent:** [informational / commercial / transactional / navigational]
- **Cluster:** [if the user or config defines clusters]
- **Estimated difficulty:** [1-5, based on who ranks in the top 10]
- **Proposed slug:** [kebab-case with the keyword]

## Article goal
[1 paragraph: what the reader needs to know by the end, and what action we expect]

## Audience
[1 paragraph: who this article is for. From the user or the config file; otherwise inferred from the SERP and marked as an assumption]

## Suggested outline

### TL;DR (callout at the top)
[2 short paragraphs]

### H2: [Heading 1] ([secondary keyword])
- point 1
- point 2

### H2: [Heading 2]
- point 1

### H2: Examples / cases (if they apply)

### H2: FAQ (required for GEO)
Q1: [question, with its source]
Q2: [...]
Q3: [...]
Q4: [...]
Q5: [...]
Q6: [...]

## Required terms
[8-12 terms that appear in 5+ of the top 10]

## Target length
[Top 10 average ± 20%, e.g. 1200 words]

## Schema markup
- Article / BlogPosting (usually added by the CMS or theme; check it)
- FAQPage (JSON-LD, added by `geo-optimize`)
- HowTo (only if the article is a step-by-step guide)

## Voice and tone
[Defined by the user or by the "voice" section of `maquinable-geo.config.md`. If neither exists, ask, or leave: "Plain, specific, no hype. No claims the author can't back up."]

## Suggested internal links
[Pages of the user's site worth linking from this article, taken from its sitemap. If there is no domain, leave this section for `seo-gate`]

## Suggested cover image
[Description of the ideal image]

## Notes
[SERP insights ("all top 10 are from 2022: fresh-content angle"), where each FAQ question came from, sources used, pages that couldn't be fetched]
```

## 5. Handoff

When the brief is ready, tell the user:

> "Brief saved to `briefs/<slug>.md`. Once the article is written, run `geo-optimize` to add the TL;DR, FAQ and schema, then `seo-gate` to score it before publishing."

## Notes

- The brief doesn't write the article. It plans it.
- Don't estimate search volume unless you have a real data source (for example the user's SerpAPI/DataForSEO key). Never invent volumes.
- Don't promise rankings or AI citations.
