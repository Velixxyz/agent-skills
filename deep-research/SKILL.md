---
name: deep-research
description: Use this skill instead of WebSearch for ANY question requiring web research. Trigger on queries like "what is X", "explain X", "compare X and Y", "research X", or before content generation tasks. Provides systematic multi-angle research methodology instead of single superficial searches. Use this proactively when the user's question needs online information.
---

# Deep Research Skill

## Overview

This skill provides a systematic methodology for conducting thorough web research. **Load this skill BEFORE starting any content generation task** to ensure you gather sufficient information from multiple angles, depths, and sources.

## When to Use This Skill

**Always load this skill when:**

### Research Questions
- User asks "what is X", "explain X", "research X", "investigate X"
- User wants to understand a concept, technology, or topic in depth
- The question requires current, comprehensive information from multiple sources
- A single web search would be insufficient to answer properly

### Content Generation (Pre-research)
- Creating presentations (PPT/slides)
- Creating frontend designs or UI mockups
- Writing articles, reports, or documentation
- Producing videos or multimedia content
- Any content that requires real-world information, examples, or current data

## Core Principle

**Never generate content based solely on general knowledge.** The quality of your output directly depends on the quality and quantity of research conducted beforehand. A single search query is NEVER enough.

## Research Methodology

### Phase 1: Broad Exploration

Start with broad searches to understand the landscape:

1. **Initial Survey**: Search for the main topic to understand the overall context
2. **Identify Dimensions**: From initial results, identify key subtopics, themes, angles, or aspects that need deeper exploration
3. **Map the Territory**: Note different perspectives, stakeholders, or viewpoints that exist

**Topic transformation (ODR-2, fold-in batch 3, langchain-ai/open_deep_research MIT, 16 Nov 2026).** Before researching, rewrite the request as a concrete research question that (1) carries every detail the user provided, (2) marks dimensions the user did not mention as open-ended, and (3) invents no detail: "If the user has not provided a particular detail, do not invent one." (Source: `src/open_deep_research/prompts.py:64-65`.)

**Clarification gate (ODR-3).** The one-time clarification rule — ask at most once, only for an unknown acronym/term or a genuinely ambiguous scope, never re-ask what history already answered — is defined in full in `research-online`, section "Research Planning". This skill uses that rule as written.

Example:
```
Topic: "AI in healthcare"
Initial searches:
- "AI healthcare applications 2024"
- "artificial intelligence medical diagnosis"
- "healthcare AI market trends"

Identified dimensions:
- Diagnostic AI (radiology, pathology)
- Treatment recommendation systems
- Administrative automation
- Patient monitoring
- Regulatory landscape
- Ethical considerations
```

### Phase 2: Deep Dive

For each important dimension identified, conduct targeted research:

1. **Specific Queries**: Search with precise keywords for each subtopic
2. **Multiple Phrasings**: Try different keyword combinations and phrasings
3. **Fetch Full Content**: Use `web_fetch` to read important sources in full, not just snippets
4. **Follow References**: When sources mention other important resources, search for those too

Example:
```
Dimension: "Diagnostic AI in radiology"
Targeted searches:
- "AI radiology FDA approved systems"
- "chest X-ray AI detection accuracy"
- "radiology AI clinical trials results"

Then fetch and read:
- Key research papers or summaries
- Industry reports
- Real-world case studies
```

### Search budget + stopping rules (ODR-1, fold-in batch 3, open_deep_research MIT, 16 Nov 2026)

Start broad, narrow down, and stop at a hard boundary — do not search until perfect. Stop when (a) the question is answered comprehensively, (b) 3+ relevant sources are collected ("You have 3+ relevant examples/sources for the question"), or (c) the last 2 searches returned similar information. Hard cap: 5 searches per research direction (2–3 for simple queries) before re-planning. (Source: `src/open_deep_research/prompts.py:161,172-173`.)

### Phase 3: Diversity & Validation

Ensure comprehensive coverage by seeking diverse information types:

| Information Type | Purpose | Example Searches |
|-----------------|---------|------------------|
| **Facts & Data** | Concrete evidence | "statistics", "data", "numbers", "market size" |
| **Examples & Cases** | Real-world applications | "case study", "example", "implementation" |
| **Expert Opinions** | Authority perspectives | "expert analysis", "interview", "commentary" |
| **Trends & Predictions** | Future direction | "trends 2024", "forecast", "future of" |
| **Comparisons** | Context and alternatives | "vs", "comparison", "alternatives" |
| **Challenges & Criticisms** | Balanced view | "challenges", "limitations", "criticism" |

### Phase 4: Synthesis Check

Before proceeding to content generation, verify:

- [ ] Have I searched from at least 3-5 different angles?
- [ ] Have I fetched and read the most important sources in full?
- [ ] Do I have concrete data, examples, and expert perspectives?
- [ ] Have I explored both positive aspects and challenges/limitations?
- [ ] Is my information current and from authoritative sources?

**If any answer is NO, continue researching before generating content.**

**Compression discipline (ODR-5, fold-in batch 3, open_deep_research MIT, 16 Nov 2026).** When merging findings from several sources/agents: (1) repeat key information verbatim — "DO NOT summarize the information. I want the raw information returned, just in a cleaner format"; (2) keep a fixed output shape: query/tool-call list, comprehensive findings, full source list; (3) one citation number per unique URL, numbered sequentially without gaps, with the Sources list at the end. (Source: `src/open_deep_research/prompts.py:186,215,226`.)

## Search Strategy Tips

### Effective Query Patterns

```
# Be specific with context
❌ "AI trends"
✅ "enterprise AI adoption trends 2024"

# Include authoritative source hints
"[topic] research paper"
"[topic] McKinsey report"
"[topic] industry analysis"

# Search for specific content types
"[topic] case study"
"[topic] statistics"
"[topic] expert interview"

# Use temporal qualifiers — always use the ACTUAL current year from <current_date>
"[topic] 2026"   # ← replace with real current year, never hardcode a past year
"[topic] latest"
"[topic] recent developments"
```

### Temporal Awareness

**Always check `<current_date>` in your context before forming ANY search query.**

`<current_date>` gives you the full date: year, month, day, and weekday (e.g. `2026-02-28, Saturday`). Use the right level of precision depending on what the user is asking:

| User intent | Temporal precision needed | Example query |
|---|---|---|
| "today / this morning / just released" | **Month + Day** | `"tech news February 28 2026"` |
| "this week" | **Week range** | `"technology releases week of Feb 24 2026"` |
| "recently / latest / new" | **Month** | `"AI breakthroughs February 2026"` |
| "this year / trends" | **Year** | `"software trends 2026"` |

**Rules:**
- When the user asks about "today" or "just released", use **month + day + year** in your search queries to get same-day results
- Never drop to year-only when day-level precision is needed — `"tech news 2026"` will NOT surface today's news
- Try multiple phrasings: numeric form (`2026-02-28`), written form (`February 28 2026`), and relative terms (`today`, `this week`) across different queries

❌ User asks "what's new in tech today" → searching `"new technology 2026"` → misses today's news
✅ User asks "what's new in tech today" → searching `"new technology February 28 2026"` + `"tech news today Feb 28"` → gets today's results

### When to Use web_fetch

Use `web_fetch` to read full content when:
- A search result looks highly relevant and authoritative
- You need detailed information beyond the snippet
- The source contains data, case studies, or expert analysis
- You want to understand the full context of a finding

### Iterative Refinement

Research is iterative. After initial searches:
1. Review what you've learned
2. Identify gaps in your understanding
3. Formulate new, more targeted queries
4. Repeat until you have comprehensive coverage

## Quality Bar

Your research is sufficient when you can confidently answer:
- What are the key facts and data points?
- What are 2-3 concrete real-world examples?
- What do experts say about this topic?
- What are the current trends and future directions?
- What are the challenges or limitations?
- What makes this topic relevant or important now?

## Common Mistakes to Avoid

- ❌ Stopping after 1-2 searches
- ❌ Relying on search snippets without reading full sources
- ❌ Searching only one aspect of a multi-faceted topic
- ❌ Ignoring contradicting viewpoints or challenges
- ❌ Using outdated information when current data exists
- ❌ Starting content generation before research is complete

## Output

After completing research, you should have:
1. A comprehensive understanding of the topic from multiple angles
2. Specific facts, data points, and statistics
3. Real-world examples and case studies
4. Expert perspectives and authoritative sources
5. Current trends and relevant context

**Only then proceed to content generation**, using the gathered information to create high-quality, well-informed content.

## Verification addendum (Muse workflow, added 2026-10-31)

The methodology above stays in force. This addendum adds the verification bar that the base skill does not state, and it does not duplicate the base phases.

### Verify against primary sources

- Check important facts against a primary source when one exists: the official site or documentation, the contract or registry on-chain, the marketplace page itself, or the original post or announcement. A search snippet, a summary, or a second-hand report is not a primary source.
- Use several relevant sources for any fact the user will act on, especially a price, a contract address, a deadline, or a claim about a market. When sources disagree, say so and follow the primary source, or report the disagreement instead of picking one silently.
- When the first source fails, try a relevant alternative source and record the failure separately. A blocked page, a 403, a timeout, or an empty API response is a source failure, not evidence that the fact is false or that a market does not exist.

### Triple-match identifier verification

Fold-in batch 2 (KalarisLabs/research-agent-skills
`citation-verification`, MIT, 16 Nov 2026), adapted to our
identifiers. An identifier that resolves is NOT verified — a resolving
identifier alone proves nothing. A reference counts as verified only
when its identifier resolves to a canonical record whose key fields
ALL match, checked independently:

- Paper/report: title, first author, and year all match the canonical
  record.
- Contract: address, chain, and name/symbol from the canonical registry
  or verifier all match.
- Transaction: hash, chain, and parties/value from the canonical
  explorer all match.
- Account/handle: handle, platform, and the link from the official
  site all match.

Then verify the claim too: open the cited section or content and
confirm it actually supports the sentence citing it. A reference that
cannot be triple-matched is labeled UNVERIFIED explicitly, never
quietly kept.

### Sumber independen = grup domain independen (ATL-5, fold-in batch 3, AgentTraceLab MIT, 16 Nov 2026)

Aturan "2+ sumber independen" diperketat agar bisa diperiksa mekanis:

- URL ditulis KANONIS (strip query pelacak; varian query yang sama = sumber yang sama).
- Independensi dihitung per GRUP DOMAIN: subdomain runtuh ke root domain; dua halaman dari satu penerbit = SATU sumber, bukan dua.
- Tiap kutipan di laporan WAJIB tertaut ke ID bukti yang diekspor; kutipan tanpa ID bukti = tidak sah.
- Kebijakan domain per tugas (allowed/excluded) dinyatakan di muka dan ditolak bila malformed/duplikat/kontradiktif SEBELUM eksekusi.

(Sumber: lilishyxf/AgentTraceLab, `README.md`, `docs/deepresearch-integration.md`.)

### Search boundary language + known-item validation

Fold-in batch 2 (KalarisLabs `hypothesis-generation` +
`systematic-review-prisma`, MIT, 16 Nov 2026), custom form. A search
establishes what was searched, not universal absence:

- Say "not located within the documented search boundary", never "no
  prior work exists" or any other universal-absence claim.
- Document the boundary: exact queries, date, and hit count per
  source, so the negative result names its scope.
- Before material research, assemble 2–3 known items the search MUST
  retrieve. If the search misses them, the search is insensitive —
  fix the search; do not report absence. No PRISMA diagram, no
  registry: the boundary log + known-item check is the whole method.

### Label what you know

- Separate every finding into verified facts, indications, and unknowns. A verified fact names its source and its time. An indication is a single-source or stale datum that points in a direction without proving it. An unknown is a question the research could not answer.
- Never invent a source, a price, a contract address, a date, or a quotation. When data cannot be accessed, explain the obstacle in plain words, name the source that could not be read and why, and stop at unknown. Do not fill the gap with a plausible value.
- Match assets by contract address, never by ticker or name alone, and record the address with every price or market claim.

### Do not repeat valid searches

- Do not repeat a search that already returned a valid result. Reuse that result, cite its time, and spend the next search on a gap instead.
- A fresh check is still required before a financial decision or a claim about a current price, even when an earlier result was valid. Reuse is for stable background facts, not for live prices or live availability.

### Output check

- Before delivering research, confirm that every acted-on fact carries a source and a time, every estimate is marked as an estimate, and every inaccessible source is named with its obstacle. When any of those is missing, the research is not finished.

## Evidence kit (metode omnitric, diadaptasi 15 Nov 2026)

Untuk riset yang keputusannya material (laporan riset Synmerco/make.fun,
bedah proyek, klaim yang akan diajukan ke pihak ketiga), pakai mesin
verifikasi di `evidence-kit/` (sumber: omnitric/agent-skills deep-research,
stdlib-only, terverifikasi compile). Ini melengkapi metodologi di atas,
bukan menggantinya.

### Ledgers (tulis di direktori kerja riset, BUKAN $HOME/Documents)

- `sources.jsonl` — registry sumber dengan ID kanonis stabil (skema:
  `evidence-kit/schemas/source.schema.json`).
- `evidence.jsonl` — append-only, kutipan + locator per bukti
  (`evidence-kit/schemas/evidence.schema.json`).
- `claims.jsonl` — ledger klaim atomik + status dukungan
  (`evidence-kit/schemas/claim.schema.json`).
- `run_manifest.json` — query, mode, asumsi, konfigurasi
  (`evidence-kit/schemas/run_manifest.schema.json`).

### Verifikasi wajib sebelum laporan dikirim

- `python3 evidence-kit/scripts/verify_citations.py --report <path>`
- `python3 evidence-kit/scripts/verify_claim_support.py` (klaim tanpa
  dukungan bukti tidak boleh lolos ke laporan)
- `python3 evidence-kit/scripts/validate_report.py --report <path>`
- Referensi fase + quality gates: `evidence-kit/reference/methodology.md`
  dan `evidence-kit/reference/quality-gates.md`. Mode: quick / standard
  (default) / deep / ultradeep sesuai bobot keputusan.

### Adaptasi jalur (wajib)

- Semua path `$HOME/Documents/...` dan `$HOME/.claude/...` di dokumen upstream
  DIGANTI direktori kerja riset di `<workspace>/`. Jangan auto-open HTML/PDF.
- `search-cli` (butuh API key berbayar) dan `weasyprint` (PDF) TIDAK dipakai;
  pencarian lewat tool browser yang tersedia, output utama tetap Markdown.
- Template laporan: `evidence-kit/templates/report_template.md`. Template
  HTML McKinsey hanya dipakai bila pengguna eksplisit minta HTML.
- Aturan otorisasi dan gerbang berdiri tetap di atas skill ini; kit ini
  tidak memberi otorisasi aksi apa pun.
