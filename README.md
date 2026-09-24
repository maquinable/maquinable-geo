# Maquinable GEO

[Español](README.es.md)

A Claude Code plugin for GEO (generative engine optimization) and technical SEO. Four skills that do real work on your articles and your site, in English and Spanish.

## Skills

| Skill | What it does |
|---|---|
| `content-brief` | Researches a keyword in the SERP (your language and market) and writes a brief: top 10 structure, required terms, target length, real questions for the FAQ. Saves it to `briefs/<slug>.md`. |
| `geo-optimize` | Adds an AI-friendly layer to a finished article: TL;DR at the top, a 4-6 question FAQ at the end and FAQPage JSON-LD. HTML or Markdown. |
| `seo-gate` | Scores an article /100 before publishing, fixes what it can and re-scores until it reaches 85 or 3 passes. Internal links come from your real sitemap. Includes a human-voice check that counts and quotes AI writing tics. |
| `seo-audit` | Read-only audit of a site: sitemap, titles, meta descriptions, canonical, JSON-LD, H1, alt text, robots.txt verdicts for search and AI crawlers, `/llms.txt`. Returns fixes by severity. |

Typical flow: `content-brief` → write the article → `geo-optimize` → `seo-gate` → publish.

## Example prompts

- "Content brief for `crm for small law firms`, US market"
- "Optimize this article for AI" (paste the article or give a file)
- "Run the SEO gate on `drafts/my-post.md`, keyword `invoice automation`, domain example.com"
- "Audit example.com. Are AI bots blocked?"

## Optional config

Create `maquinable-geo.config.md` at the root of your project so the skills don't have to ask every time:

```markdown
# maquinable-geo config
- Domain: example.com
- Language / market: English, US
- Audience: owners of small accounting firms
- Topic clusters: 1) invoicing, 2) payroll, 3) AI for accountants
- Voice: plain, specific, first person plural, no hype
- CMS: WordPress (sanitizes <script> in post body: yes)
```

## What it doesn't do

- It doesn't publish anything or edit your site. `seo-audit` only reads.
- It doesn't promise rankings or AI citations. FAQPage has not produced Google rich results since 2023, except for government and health sites. Its value for GEO is plausible, not proven.
- It doesn't compute a global SEO or GEO score for your site.
- It doesn't invent URLs, search volumes or facts. What it can't verify, it marks as not verifiable.

## Permissions and external services

- Uses Claude's built-in tools: web search, web fetch and bash (Python standard library for sitemaps and robots.txt).
- No required external services and no API keys.
- Optional, with your own keys: PageSpeed Insights (performance and mobile in `seo-audit`), SerpAPI or DataForSEO (more exact SERP data in `content-brief`).
- Optional: `pip install protego` to check robots.txt with a second parser (RFC 9309).

## Install

```
/plugin marketplace add maquinable/maquinable-geo
/plugin install maquinable-geo@maquinable
```

## Credits

The anti-AI-tics rules in `seo-gate` are adapted from [blader/humanizer](https://github.com/blader/humanizer).

## About

Made by [Maquinable](https://maquinable.com). If you want an AI citability score and a prioritized fix plan for your site, that's what we do: [request a diagnosis](https://maquinable.com/en/request?ref=claude-plugin).

Support: hola@maquinable.com

Privacy: [PRIVACY.md](PRIVACY.md). The plugin collects no data.

License: MIT
