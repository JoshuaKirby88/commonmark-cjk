# Behavior matrix

Status: implemented and verified in the pinned cmark and commonmark.js candidates.

This matrix applies the [candidate narrow rule](rule.md) and [Unicode policy](unicode-policy.md). Exact Markdown and HTML strings live in [`fixtures/emphasis.json`](../fixtures/emphasis.json). The generated delimiter-boundary truth table lives in [`fixtures/generated-boundaries.json`](../fixtures/generated-boundaries.json).

## Design choices represented here

This matrix applies four choices:

1. A CJK character is assigned and either has East Asian Width `W`, `F`, or `H` without `Emoji_Presentation`, or has Script `Hangul`.
2. Only the immediate code points around the delimiter are classified. Variation-selector-dependent forms are deferred.
3. The new flanking condition applies to all emphasis delimiter runs, but CommonMark's underscore-specific opener and closer restrictions remain unchanged.
4. CJK punctuation triggers the rule in mixed text such as `a**「x」**b`.

## Ordinary prose that changes

| Case | Markdown | Proposed interpretation |
| --- | --- | --- |
| Simple Japanese quotation | `これは**「重要なこと」**です。` | The quoted phrase becomes `<strong>` |
| Japanese sentence ending | `これは**私のやりたかったこと。**だから` | The sentence fragment becomes `<strong>` |
| Japanese corner quotes | `語**「引用」**語` | `「引用」` becomes `<strong>` |
| Japanese single emphasis | `語*「引用」*語` | `「引用」` becomes `<em>` |
| Chinese sentence ending | `这是**我想做的事。**所以` | The sentence fragment becomes `<strong>` |
| Chinese punctuation | `**。**话` | `。` becomes `<strong>` |
| Korean parenthetical | `**안녕(hello)**하세요.` | `안녕(hello)` becomes `<strong>` |
| Korean link and particle | `**[Markdown](https://commonmark.org/help/)**을 사용하세요.` | The linked word becomes `<strong>` |
| ASCII punctuation in CJK | `語**(ABC)**語` | `(ABC)` becomes `<strong>` |
| Supplementary Han | `𠮷**(U+20BB7)**語` | `(U+20BB7)` becomes `<strong>` |

The Korean case is a deliberate improvement over the 2024 experimental block ranges, which omitted ordinary Hangul syllables.

## Mixed-text compatibility changes

These are not accidental. They follow directly from classifying the adjacent code points rather than detecting a document language.

| Markdown | Proposed interpretation | Why |
| --- | --- | --- |
| `a**「x」**b` | `「x」` becomes `<strong>` | `「` and `」` are CJK punctuation |
| `a**（x）**b` | `（x）` becomes `<strong>` | Fullwidth parentheses have East Asian Width `F` |
| `a**㊙x**b` | `㊙x` becomes `<strong>` | `㊙` is wide and does not have default emoji presentation |
| `語**😀**文` | `😀` becomes `<strong>` | The outside characters `語` and `文` trigger the runs |
| `語***「引用」***語` | Nested `<em><strong>` | Existing three-asterisk pairing runs after the flanking change |
| `語**「*引用*」**語` | Outer `<strong>`, inner `<em>` | Existing inner emphasis is preserved |
| `語**[項目](https://example.com)。**文` | Link and punctuation become `<strong>` | Link precedence is unchanged |
| ``語**`code`。**文`` | Code span and punctuation become `<strong>` | Code-span precedence is unchanged |

The full curated set has 21 changed outputs. The JSON fixture records the complete expected HTML for each one.

## Protected behavior

| Markdown | Result | Protection |
| --- | --- | --- |
| `a**“x”**b` | Remains literal | Curly quotes have ambiguous width `A`, not CJK |
| `foo**bar**baz` | Remains `<strong>` | Existing intraword asterisk behavior |
| `語__強調__語` | Remains literal | Underscore intraword restriction |
| `語__「強調」__語` | Remains literal | Underscore restriction is not relaxed |
| `語** 強調**語` | Remains literal | Whitespace after an opener still blocks opening |
| `語**強調 **語` | Remains literal | Whitespace before a closer still blocks closing |
| `a**😀。**b` | Remains literal | Default emoji alone does not trigger the opener |
| `禰󠄀**(ね)**豆子` | Remains literal in version one | Immediate preceding code point is a variation selector |
| U+2FA20 around `**(x)**` | Remains literal | The code point has a default wide value but is unassigned |
| `ไทย**(ข้อความ)**ไทย` | Remains literal | Thai is outside this proposal |

There are 11 protected curated outputs in total. The remaining one is the simple Japanese quotation rewritten with underscores, which remains literal for the same underscore-specific reason.

## Generated boundary coverage

The generator crosses nine immediate-character classes on both sides of a delimiter:

- whitespace;
- ASCII punctuation;
- ambiguous-width Unicode punctuation;
- ASCII text;
- CJK punctuation;
- CJK text;
- default emoji presentation;
- variation selectors;
- an unassigned code point whose default East Asian Width is `W`.

It evaluates all 81 before-and-after combinations for both `*` and `_`, producing 162 boundary records.

Results:

- 14 asterisk boundary records gain opener or closer eligibility.
- 14 underscore records change raw left-flanking or right-flanking classification.
- Zero underscore records change final opener or closer eligibility because the existing underscore rules filter those changes.
- Whitespace never gains invalid opener or closer eligibility.
- Default emoji, variation selectors, and unassigned code points never count as CJK by themselves.

## Deterministic checks

`scripts/check-fixtures` verifies:

- both untouched reference parsers match `current_html` for all 32 curated cases;
- cmark and commonmark.js agree on every current result;
- every `change` flag matches the old-versus-proposed HTML pair;
- the committed 162-record boundary matrix exactly matches the generator.

Both candidate parsers also pass all 677 proposed CommonMark examples. The fixture oracle was derived from the normative formula before implementation, then checked independently against both candidates.

## Compatibility boundary

The most consequential tradeoffs are the mixed-text changes and the deliberate deferral of variation-selector sequences. Both are recorded explicitly rather than hidden as parser-specific exceptions.
