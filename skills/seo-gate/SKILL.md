---
name: seo-gate
description: Quantitative pre-publish quality gate for articles. Scores the draft /100 (keyword placement, headings, meta, slug, real internal links from the site's sitemap, GEO layer, human voice / anti-AI-tics, readability, length), fixes what it can and re-scores in a loop until it reaches 85 or 3 passes. Use when the user says "run the gate", "score this article", "SEO score", "SEO loop", "check the draft before publishing", "seo-gate". También en español - "pásale el gate", "puntúa el artículo", "revisa el borrador antes de publicar", "loop SEO".
---

# seo-gate

**Philosophy:** don't report problems, fix them. Score, optimize, re-score, repeat until the number says it's ready.

**Target: 85/100. Ceiling: 3 passes.** If pass 3 doesn't reach 85, stop and deliver an honest "COULD NOT FIX" list. Never claim 85 if it wasn't reached.

**Language:** answer in the language of the article. The anti-tics rules are chosen by the article's language (Spanish or English set below). For any other language, apply the rules that transfer and say which ones you skipped.

## Inputs

- **Article** (HTML or Markdown), ideally already with TL;DR + FAQ + schema.
- **Main keyword** (required). If missing, propose the strongest candidate from the title and intro and confirm it.
- **Site domain** (required for internal links and SERP preview). Take it from `maquinable-geo.config.md` at the project root if it exists, otherwise ask.
- **Brief terms** (optional): enable the full semantic coverage check.
- **Topic clusters** (optional, from the user or the config file): used to prioritize internal links.
- **meta_title / meta_description / slug** if they already exist. Otherwise they are generated.

If the article has no TL;DR / FAQ / schema, run `geo-optimize` first instead of duplicating that logic here.

## Step 0: internal link candidates from the sitemap

Before scoring, collect the site's real URLs. **Never use placeholders or invented URLs.**

1. Fetch `https://<domain>/sitemap.xml`. If it doesn't exist, check the `Sitemap:` lines in `robots.txt`.
2. If it's a sitemap index (`<sitemapindex>`), recurse into each child sitemap.
3. Keep the article-like URLs (blog, resources, guides). Exclude the article itself, tag/category pages and pagination.
4. Rank candidates by topical affinity with the article (URL slug, and the page title if you fetch it). Same cluster first when clusters are defined.

```python
import urllib.request
from xml.etree import ElementTree as ET

NS = {"sm": "http://www.sitemaps.org/schemas/sitemap/0.9"}
UA = "Mozilla/5.0 (compatible; maquinable-geo/0.1; +https://github.com/maquinable/maquinable-geo)"

def fetch(url):
    # Always send a descriptive User-Agent: many sites (e.g. Cloudflare's default
    # Browser Integrity Check) answer 403 to the generic "Python-urllib" one.
    req = urllib.request.Request(url, headers={"User-Agent": UA})
    with urllib.request.urlopen(req, timeout=20) as r:
        return r.read()

def sitemap_urls(url, depth=0):
    root = ET.fromstring(fetch(url))
    if root.tag.endswith("sitemapindex") and depth < 3:
        out = []
        for loc in root.findall("sm:sitemap/sm:loc", NS):
            out += sitemap_urls(loc.text.strip(), depth + 1)
        return out
    return [loc.text.strip() for loc in root.findall("sm:url/sm:loc", NS)]
```

If a fetch returns 403 (or a challenge page), don't report the site as blocking crawlers: it's usually the site rejecting your client. Retry with the descriptive User-Agent above, or with Claude's web fetch tool, and only report a block if it persists.

If the sitemap can't be read, score anyway and mark internal links as "NOT VERIFIABLE". Don't invent URLs.

## Step 1: SCORE (before)

| Category | Pts | What is measured |
|---|---|---|
| Keyword in title | 10 | Keyword in the first 30 chars; title ≤ 60 chars |
| Keyword in first paragraph | 10 | In the first 100 words of the original content (the TL;DR doesn't count) |
| Keywords in headings | 5 | Keyword or a variation in 2-3 H2/H3; no stuffing in all of them |
| Heading hierarchy | 5 | Exactly one H1 on the rendered page and no skipped levels (H2 → H4). Ask whether the CMS renders the post title as H1; if it does, the body must not contain another H1 |
| Semantic coverage | 10 | Brief terms present in the body. Without a brief: keyword 3-5 times in total, not more |
| Meta title + description | 10 | meta_title 50-60 chars with keyword; meta_description 100-160 chars with keyword + hook |
| Slug | 5 | 3-5 words, kebab-case, keyword included, no stop words (EN: the, a, of, for, with / ES: el, la, de, para, con) |
| Internal links | 10 | 3-5 links to real pages of the site (Step 0) with descriptive anchor text (never "click here" / "haz clic aquí") |
| GEO | 15 | TL;DR present + FAQ with 4-6 Q&A + FAQPage JSON-LD that parses, has no HTML in `text` and has as many `mainEntity` as FAQ H3 |
| Human voice (anti-AI-tics) | 10 | Count of violations of the 10 rules for the article's language. See below |
| Readability | 5 | Paragraphs < 150 words; list-worthy prose turned into lists/tables |
| Length | 5 | ≥ 600 words; ideal 800-1200 |

### Human voice: how it is scored

**What counts as 1 violation.** One discrete textual occurrence, not one broken rule:

- A sentence with 4 bold phrases = **1** violation of rule 1 (not 4).
- Two "not X, but Y" antitheses in different paragraphs = **2** violations of rule 2.
- One paragraph can collect violations of different rules.
- Every violation is quoted with its exact text in the report. If you can't quote the fragment, it doesn't count. No violations "by general feel".

**Cap of 2 per rule.** No rule adds more than 2 to the total. If a rule has 3+ real occurrences, count 2 and report it separately as `SYSTEMIC TIC: rule N (X occurrences)`. The cap keeps one repeated tic from sinking an otherwise well-written article. Max countable: 20 (10 rules × 2).

| Violations (after cap) | Pts | Reading |
|---|---|---|
| 0-2 | 10 | Clean |
| 3-4 | 8 | Normal editing noise |
| 5-6 | 6 | The pattern shows |
| 7-8 | 4 | Obvious to a frequent reader |
| 9-10 | 2 | Sounds machine-written |
| 11+ | 0 | See escalation |

**Escalation.** With 11+ violations the problem is no longer editing. It needs a rewrite, and rewriting conflicts with voice protection (Step 2). In that case the gate does NOT try to reach 85 through this category: it leaves it at 0, keeps optimizing the rest and adds a `VOICE REWRITE` entry to "COULD NOT FIX", with the violations quoted, for the user to decide. Below 11 the loop fixes them without asking: they are local edits.

**Scope.** Count over the article body, the TL;DR and the FAQ answers. NOT over meta_title, meta_description, anchor text or headings: SEO criteria rule there.

### Anti-tics rules: Spanish (articles in Spanish)

1. Negritas solo en términos escaneables; nunca 3 o más en una misma oración.
2. Prohibido "no es X, es Y" salvo que el contraste sea el punto real.
3. No anunciar el punto ("Y lo más importante:", "La clave está en"); arrancar con el contenido.
4. Sin punchlines fragmentadas ("Sin excusas. Sin rodeos.") ni frases-slogan ("X es el lenguaje de Y").
5. Raya solo para incisos con apertura y cierre; nunca raya suelta antes de una coletilla.
6. Tríadas solo si hay 3 elementos reales; no rellenar a tres.
7. Verbos simples ("es", "tiene"); no "se erige como" ni "cuenta con" decorativo.
8. Sin aperturas falso-cándidas ("Seamos honestos") ni finales genéricos positivos; cerrar con un dato o un próximo paso.
9. Sin filler: "con el fin de" → "para", "debido al hecho de que" → "porque".
10. Afirmaciones vagas ("expertos señalan") → fuente concreta, o se sacan.

### Anti-tics rules: English (articles in English)

1. Bold only for scannable terms; never 3 or more in one sentence.
2. No "It's not X, it's Y" / "not just X, but Y" unless the contrast is the actual point.
3. Don't announce the point ("Here's the thing:", "The key takeaway?", "Let's dive in"); start with the content.
4. No fragmented punchlines ("No fluff. No excuses.") or slogan lines ("Data is the new oil").
5. Em dashes only in pairs around an aside; never a single em dash hanging a tagline off the end of a sentence.
6. Triads only when there are 3 real items; don't pad to three.
7. Plain words over inflated ones: no "delve", "leverage", "navigate the landscape", "unlock", "crucial", "pivotal", "robust", "seamless", "game-changer" when "use", "important" or a concrete detail works.
8. No mock-candid openers ("Honestly,", "Let's be real,", "In today's fast-paced world,") and no generic upbeat endings ("The future is bright."); end with a fact or a next step.
9. No filler: "in order to" → "to", "due to the fact that" → "because", "it's important to note that" → cut.
10. Vague attributions ("experts say", "studies show") → a concrete source, or cut.

**Exception for both sets:** the rules don't apply to deliberate SEO/GEO structure (lists, FAQ, TL;DR, scannable headings) as defined by the structure test below.

### When a list is structure and when it's padding

Readability rewards turning prose into lists. Rules 1 (bold) and 6 (triads) punish list-ified formatting. A list or table is **deliberate structure** (and exempt from rules 1 and 6) only if it passes **all three** questions:

1. **Parallelism.** Do all items answer the same implicit question, and can they be reordered without breaking the meaning? If there are causal or temporal connectors between items ("then", "that's why", "once"), it's prose split into bullets.
2. **Extraction.** Is an item copied out of the article still a complete unit of information? That's literally the criterion an LLM uses to quote it.
3. **Own contribution.** Does each item bring information no other item has? An item that restates the previous one is padding.

**Passes all three:** it's structure. The bold mini-title at the start of each item does NOT count for rule 1, and the number of items does NOT count for rule 6.
**Fails any:** it's disguised prose. Apply rules 1 and 6 like any paragraph, and it does NOT add to Readability.

**Fixed rulings for cases the test doesn't cover:**
- Bold **inside the body** of an item (not the mini-title): counts normally for rule 1.
- Bold table headers: always exempt.
- A three-element enumeration **inside a prose sentence** is never a "list"; rule 6 always applies.

**Precedence.** If a change raises Readability but creates an anti-tics violation, anti-tics wins: it's worth 10, Readability 5.

### Score output format

```
═══════════════════════════════════════
 SEO SCORE (BEFORE): XX/100
═══════════════════════════════════════
Target keyword: "[keyword]"

 Keyword in title:     XX/10  [detail]
 First paragraph:      XX/10  [keyword at word X / NOT FOUND]
 Headings:             XX/5   [X/3 headings with keyword]
 Hierarchy:            XX/5   [valid / H2→H4 skip / 2 H1]
 Semantic coverage:    XX/10  [X/Y brief terms]
 Meta title + desc:    XX/10  [status of each]
 Slug:                 XX/5   [ok / too long / no keyword]
 Internal links:       XX/10  [X real links / NOT VERIFIABLE]
 GEO:                  XX/15  [TL;DR ✓/✗ · FAQ X Q&A · schema ✓/✗]
 Human voice:          XX/10  [X violations (Y raw)]
 Readability:          XX/5   [X long paragraphs]
 Length:               XX/5   [X words]

ANTI-TICS VIOLATIONS:
 R2 (antithesis)  x2 · "[exact quote]"
 R3 (announcing)  x1 · "[exact quote]"
 [SYSTEMIC TIC: rule 2 (7 occurrences), 2 counted]

CRITICAL ISSUES:
 1. [worst]
 2. [second]
 3. [third]
═══════════════════════════════════════
```

Write the labels in the article's language.

## Step 2: OPTIMIZE

Apply changes directly to the text, category by category, starting with the ones losing the most points.

**DO (changes SEO needs):**
- Rewrite the title with the keyword in the first 30 chars, ≤ 60 chars.
- Put the keyword in the first paragraph by adjusting existing sentences naturally.
- Add the keyword or variations to 2-3 headings without breaking the structure.
- Insert 3-5 real internal links (Step 0) into existing sentences or natural transitions, spread through the article, early when possible.
- Split paragraphs > 150 words. Turn enumerations in prose into lists/tables only if the result passes the structure test.
- Fix anti-tics violations with the minimum edit: split the sentence with 3+ bold phrases or keep only the scannable term; drop the antithesis and leave the claim alone; delete the announcement and start with the content; close the dash or swap it for a comma; replace the inflated word with a plain one; cut the filler. All local edits: don't rewrite a whole paragraph to fix one tic.
- Generate `meta_title`, `meta_description` and `slug`.

**DON'T (no SEO gain, breaks the voice):**
- Add padding for length.
- Rewrite paragraphs that are already fine.
- Add new content the author didn't write.
- Change the order of sections or the article's structure.
- Rewrite a whole paragraph to lower the anti-tics count. If the tic doesn't go away with a local edit, it stays and is reported.
- List-ify prose that fails the structure test.
- Change the author's voice.
- State facts about clients, products or results that the article doesn't support.

**Voice protection:** if reaching 85 requires changing the voice or adding content the author didn't write, STOP and say so. Don't force the number.

## Step 3: RE-SCORE + loop

Re-score the optimized version with the same rubric.

- **Score ≥ 85:** target met. Show before/after and deliver.
- **Score < 85:** take the 2-3 lowest categories, rewrite ONLY the sections those categories touch (don't touch what already passes), re-score. Repeat until ≥ 85 or 3 passes in total (the first optimization counts as pass 1).

**Run the whole loop without pausing.** Don't ask between passes. The user sent a draft to optimize, not to be interviewed.

**Hard stops:**
- 3 passes max. Never more.
- If pass 3 is < 85: STOP. Deliver the final score + a "COULD NOT FIX" list: each failing category, why rewriting doesn't fix it (for example: length needs more material from the author; semantic coverage needs a brief), and what the user has to do manually.
- A stopped loop with an honest list is worth more than a fake 85.

### Final output

```
═══════════════════════════════════════
 BEFORE / AFTER
═══════════════════════════════════════
 [the 12 categories]   XX → XX
 ─────────────────────────────────
 TOTAL:                XX → XX

LOOP LOG:
 Pass 1: XX → XX  [categories rewritten]
 Pass 2: XX → XX  [...]
 RESULT: TARGET MET (XX) | STOPPED AT 3 PASSES (XX, see COULD NOT FIX)

SERP PREVIEW:
 Title:       [meta_title]
 URL:         [domain]/[path]/[slug]
 Description: [meta_description]

METADATA:
 meta_title / meta_description / slug
═══════════════════════════════════════
```

For the preview URL, use the path pattern of the site's existing articles (from the sitemap). If you can't tell it, show `[domain]/.../[slug]` and say the path is an assumption.

## Rules

1. Score first, optimize after. Always show the "before".
2. Internal links are always real URLs from the sitemap. If it fails, mark "NOT VERIFIABLE", never invent URLs.
3. The author's voice survives every pass. If 85 costs the voice, stop.
4. Don't invent ranking data or promise positions or AI citations. This gate audits structure, not the live SERP.
5. The gate doesn't publish anything.
6. If the input score is already ≥ 85: report it and stop. Don't optimize for the sake of it.
7. Anti-tics violations are quoted verbatim or they don't exist. Never report a count without the fragment that justifies it.
8. The anti-tics rubric grades the text, not the author's judgment. If a violation is a deliberate choice marked by the author in the draft, remove it from the count and note it.

## Output

- Optimized article + metadata (`meta_title`, `meta_description`, `slug`).
- Before/after report with loop log.
- If it ended < 85: "COULD NOT FIX" list with manual actions for the user.
