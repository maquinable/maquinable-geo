---
name: geo-optimize
description: Takes a finished article and adds an AI-friendly layer so it is easier for ChatGPT, Claude, Gemini and Perplexity to quote - a TL;DR callout at the top, a 4-6 question FAQ at the end and FAQPage JSON-LD. Use when the user says "optimize this article for AI", "GEO this post", "add an FAQ to this article", "make this AI-friendly", "geo-optimize". También en español - "optimiza este artículo para IA", "hazle GEO al post", "añade un FAQ a este artículo", "formato AI-friendly".
---

# geo-optimize

Turn a finished article into an AI-friendly version: TL;DR at the top, FAQ at the end, FAQPage schema. The article's own content is not rewritten.

**Language:** answer and write in the language of the article. If there is no article yet, use the user's language.

## Inputs

- **Article** (required): HTML or Markdown body.
- **Main keyword** (optional): if missing, propose the strongest candidate from the title and intro and confirm it.
- **Brief** (optional): if the user has one (for example from `content-brief`), take the FAQ questions from it.
- **Output format**: HTML (default) or Markdown for CMSs that don't accept raw HTML. Ask if it's not obvious from the input.
- **Project config** (optional): if `maquinable-geo.config.md` exists at the root of the user's project, read it for domain, audience, voice and CMS.

## 1. TL;DR callout

Two short paragraphs at the start of the article:

- **Paragraph 1:** what the article concretely says. If it's a case study, only use facts the article states.
- **Paragraph 2:** who it's relevant for (audience, sector, situation), so the reader can decide whether to keep reading.

Neutral semantic HTML, no inline colors. The user's theme styles `.tldr`:

```html
<aside class="tldr">
  <p><strong>TL;DR</strong></p>
  <p>[Paragraph 1]</p>
  <p>[Paragraph 2]</p>
</aside>
```

Use the label in the article's language ("TL;DR", "Resumen", "En resumen").

Markdown version:

```markdown
> **TL;DR**
>
> [Paragraph 1]
>
> [Paragraph 2]
```

## 2. FAQ section

4-6 Q&A at the end of the article. Question sources, in order:

1. The brief, if there is one.
2. Real questions found with web search for the main keyword, in the article's language and market ("People also ask"-style questions, forums, Q&A sites). Use Claude's web search - do not scrape Google directly.
3. If that's not enough: questions a reader of this article would naturally ask an AI assistant about the topic.

**Suggested mix:**
- 2-3 questions specific to the topic of the article.
- 2-3 general questions about the industry or the problem (how long it takes, how much it costs, who it applies to).

**Editorial rules:**
- Don't state certifications, awards, exact metrics or regulations the article doesn't support.
- Use the article's own content for facts.
- For general questions, talk about the industry in the abstract.
- Answers: 1-3 sentences, 50 words max.
- Keep the author's voice. No marketing tone.

HTML:

```html
<h2>FAQ</h2>

<h3>[Question 1]</h3>
<p>[Answer 1]</p>

<h3>[Question 2]</h3>
<p>[Answer 2]</p>
```

Localize the heading ("Frequently asked questions", "Preguntas frecuentes"). In Markdown use `## ` and `### ` headings.

## 3. FAQPage JSON-LD

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "[Question 1]",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "[Answer 1, plain text]"
      }
    }
  ]
}
</script>
```

**Rules:**
- `text` must not contain HTML. Strip every tag before including it.
- The number of `mainEntity` items must equal the number of H3 questions in the FAQ section. Check it before delivering.
- The JSON must parse. Validate it (for example with `python3 -c "import json,sys; json.load(sys.stdin)"`).

**Where it goes:** ask whether the user's CMS sanitizes `<script>` tags in the post body (many do). If it doesn't, append the script at the end of the body. If it does, or the user doesn't know, deliver the JSON-LD separately so they can inject it in the page `<head>` (theme, SEO plugin or tag manager). In Markdown output, always deliver it separately.

**What to expect, stated plainly:** since August 2023 Google only shows FAQ rich results for government and health sites. For everyone else the FAQPage won't produce a rich result, and that is not an error. Its value for GEO (being quoted by AI assistants) is plausible but not proven. Don't promise rankings or citations.

## 4. Lists and tables where they help

If the article has content that reads better as structure, suggest it:

- "X lets you do A, B and C" → bulleted list.
- A comparison between options → table.
- Steps of a process → numbered list.

Only when every item answers the same implicit question and makes sense on its own. Don't break up prose that has causal flow.

## 5. Keyword check

Check that the main keyword appears:
- In the first paragraph of the original content (the TL;DR doesn't count).
- In at least one H2.
- 3-5 times in total (more is stuffing).

If it falls short, suggest the rewording to the user. Don't edit the body yourself: the voice belongs to the author.

## Final order

```
[TL;DR callout]
[Original article content]
[FAQ section]
[FAQPage JSON-LD, or delivered separately]
```

## Output

- The full article with TL;DR + FAQ + schema.
- A short summary: "GEO layer added: TL;DR ([N] words), [N] Q&A, FAQPage schema ([N] mainEntity = [N] H3). [Where the JSON-LD goes]."
- Suggest running `seo-gate` next, before publishing.

## Checking after publishing

Download the published HTML, extract every `<script type="application/ld+json">` and parse each one. The FAQPage has to parse and have as many `mainEntity` items as H3 in the FAQ section. Google's Rich Results Test won't list FAQPage for most sites (see above), so don't use it to validate this schema.
