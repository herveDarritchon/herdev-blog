---
agent: agent
description: This prompt is used to add footnotes styling to a Hugo blog using the PaperCSS
---

# Footnotes editorial rules (Hugo / Goldmark)

Goal: add sources without hurting readability.

## 1) When to add a footnote
- Use footnotes to source a claim (spec/doc/code) or add a non-essential technical aside.
- Don’t hide core reasoning in footnotes: if it’s central, keep it in the main text.

## 2) One note = one claim
- 1 footnote supports 1 specific claim.
- Avoid “note dump” with many URLs; only allow multi-links in a dedicated “Bibliography / Further reading” block.

## 3) IDs and dedup
- Use stable named IDs: `[^kotlin-null-safety]`, not `[^1]`.
- Same source (same URL) => same ID everywhere in the article.
- Never create multiple IDs for the same URL.
- Define each `[^id]: ...` exactly once in the Sources section at the end.

## 4) Callout placement (critical)
Footnote callout behaves like punctuation:
- No space before: `word[^id]` (never `word [^id]`).
- Put it after the phrase it supports.
- Put it before strong punctuation: `. , ; : ! ?`
  - ✅ `...bytecode[^id].`
  - ✅ `...bytecode[^id], which...`
  - ❌ `...bytecode.[^id]`
  - ❌ `...bytecode [^id].`

Special cases:
- Parentheses: if the source supports only the parenthetical, place inside `(...)`; otherwise before.
- Quotes: if the source is for the quote, place before the closing quote, then punctuation.

## 5) Reusing the same source
- Reuse the same `[^id]` if needed, but keep it sparse (2–4 times max per article: TL;DR + section entry + “proof” section).
- If a source would be repeated everywhere, restructure (sources per section or a References block).

## 6) Footnote content format
Keep it short, consistent:
`[^id]: Org — Title/Page: <https://...>.`

## 7) Hugo-specific constraint
- Avoid placing footnote callouts before `<!--more-->` if `.Summary` is used on list pages.

## Output requirements for the agent
- Insert `[^id]` in-text at the right spot per rule #4.
- Append/update a final `## Sources` section containing the `[^id]: ...` definitions.
- Deduplicate: unify IDs pointing to the same URL and update all callouts accordingly.
- Do not change meaning; minimal edits outside footnote insertion.
