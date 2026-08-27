# Compatibility report

Status: verified against the pinned cmark and commonmark.js candidates.

This report covers the frozen narrow rule implemented in the pinned cmark and
commonmark.js candidates. Exact Markdown and HTML pairs are in
[`fixtures/emphasis.json`](../fixtures/emphasis.json).

## Result

The candidate has no unexplained output differences and no parser
disagreements in the tested corpus.

- Both parsers preserve all 655 existing CommonMark examples.
- Both parsers pass the 22 new CJK examples, for 677 proposed examples total.
- Both parsers match all 32 curated fixtures. Twenty-one change under the proposed rule and 11
  remain protected.
- Both parsers agree across 13,122 systematic delimiter documents. Exactly 833
  outputs change. Every change uses asterisks and has an opener or closer
  eligibility change predicted by the frozen rule.
- Zero systematic underscore outputs change.
- Both parsers agree across 5,000 seeded mixed-script documents. The candidate
  changes 691 of those outputs.
- Five long-input cases agree across the candidates. Four malformed UTF-8 cases
  terminate without a crash, hang, or excessive output in both runtimes.

## Intended core changes

These 11 curated changes directly exercise the reported failure or the same
failure in Japanese, Chinese, and Korean.

| Fixture | Purpose |
| --- | --- |
| `ja-simple-quoted` | Covers a short neutral Japanese quotation pattern |
| `ja-sentence-punctuation` | Closes strong emphasis after `。` |
| `ja-corner-quotes-strong` | Handles Japanese corner quotes with `**` |
| `ja-corner-quotes-emphasis` | Applies the same rule to `*` |
| `zh-sentence-punctuation` | Handles the equivalent Chinese sentence pattern |
| `zh-punctuation-only` | Fixes the existing commonmark.js issue 108 example |
| `ko-parenthetical` | Includes ordinary Hangul text |
| `ko-link-particle` | Handles linked strong text followed by a Korean particle |
| `cjk-ascii-parentheses` | Allows ASCII punctuation inside CJK emphasis |
| `non-bmp-han` | Handles supplementary-plane Han as a code point |
| `line-start` | Handles CJK punctuation at the start of a line |

Representative change:

```markdown
これは**「重要なこと」**です。
```

```html
<p>これは<strong>「重要なこと」</strong>です。</p>
```

The later `ko-link-particle` fixture adds coverage for behavior already determined by the proposed rule. It introduces no new semantic choice.

## Documented direct consequences

These ten outputs also change. They are consequences of the proposed
flanking rule, not separate syntax features.

| Fixture | Compatibility effect |
| --- | --- |
| `latin-cjk-quotes` | CJK corner quotes trigger emphasis inside Latin text |
| `latin-fullwidth-parentheses` | Fullwidth parentheses trigger emphasis inside Latin text |
| `cjk-outside-emoji-content` | CJK outside the delimiters allows an emoji to be emphasized |
| `cjk-symbol-text` | A wide text-default symbol triggers the rule |
| `triple-nested` | Existing three-asterisk pairing runs after reclassification |
| `nested-inner-emphasis` | Existing inner emphasis stays nested inside the new strong span |
| `link-at-end` | Existing link precedence stays intact inside the new strong span |
| `code-at-end` | Existing code-span precedence stays intact inside the new strong span |
| `non-ascii-backslashes` | Literal backslashes remain inside the new strong span |
| `empty-cjk-quotes` | Two CJK punctuation characters can become emphasized content |

The most visible mixed-text change is:

```markdown
a**「x」**b
```

```html
<p>a<strong>「x」</strong>b</p>
```

## Protected behavior

The following 11 curated fixtures remain unchanged:

- ambiguous-width curly quotes;
- existing intraword asterisk emphasis;
- three underscore cases, including the simple Japanese quotation rewritten with `__`;
- whitespace after an opener or before a closer;
- default emoji presentation without outside CJK adjacency;
- an ideographic variation-selector sequence;
- an unassigned code point with a default wide value;
- Thai text, which is outside this proposal.

The systematic matrix strengthens the underscore result. Fourteen underscore
boundaries change raw flanking classification, but the existing
underscore-specific rules filter all of them. None of the 6,561 constructed
underscore documents changes output.

## Parser-specific risk checks

The JavaScript implementation reads both adjacent characters as Unicode code
points, including supplementary-plane characters. The C and JavaScript
fixtures agree on `U+20BB7` cases.

The CJK alternative runs only for `*` and `_`. Smart quote tests confirm that
`「"quote"」` still renders as opening and closing curly quotes in both parsers.

The Unicode table excludes General Category `Cn` and `Cs`, default emoji
presentation, and variation selectors that do not independently meet the CJK
definition. The generator reproduces the same 68 ranges for both parsers.

## Classification

| Class | Count | Submission status |
| --- | ---: | --- |
| Intended core changes | 11 | Accept |
| Documented direct consequences | 10 | Requires the stated mixed-text policy |
| Protected curated outputs | 11 | Required non-changes |
| Unexplained differences | 0 | None |
| Known defects | 0 | None |

## Compatibility conclusion

The candidate intentionally changes all 21 curated cases listed above, including ten direct consequences of the character-based rule. It also intentionally defers variation-selector-dependent behavior. No tested output difference remains unexplained.
