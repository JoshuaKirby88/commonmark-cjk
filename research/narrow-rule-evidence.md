# Evidence for a narrow CJK-aware emphasis rule

Research date: 2026-08-26 (UTC)

## Decision summary

The August 2024 candidate is small enough to freeze without adopting the full
sequence-aware amendment. Its essential change is:

> Keep CommonMark's whitespace requirements. When neither adjacent character
> is whitespace, a delimiter run also satisfies the punctuation part of both
> flanking tests if either immediately adjacent Unicode code point is CJK.

This is more exact than the shorthand used in the issue, which says the run is
"counted as both left- and right-flanking." Both proof implementations retain
the outer whitespace gates. CJK adjacency does not override whitespace
([cmark implementation](https://github.com/commonmark/cmark/blob/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5/src/inlines.c#L479-L490),
[commonmark.js implementation](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/lib/inlines.js#L312-L333)).

The parser rule and the character classifier should be frozen separately. The
current proof branches combine a good local parser rule with a coarse range
table, incomplete Korean coverage, limited variation-selector behavior, and a
few implementation bugs. Copying either branch verbatim would silently make
policy decisions that the narrow proposal does not need.

For the narrow proposal, the smallest defensible character policy is a single-code-point
predicate based on Unicode properties:

1. The code point is assigned, and either:
   - its `East_Asian_Width` value is `W`, `F`, or `H`, and its
     `Emoji_Presentation` property is false; or
   - its `Script` value is `Hangul`.
2. Only the code point immediately before and immediately after the delimiter
   run is examined.
3. Variation selectors and multi-code-point sequences are deferred. There is
   no normalization, grapheme-cluster inspection, or second-code-point
   lookahead in this proposal.

This property direction was proposed in January 2025, and John MacFarlane said
he liked it and suggested generating C code from `EastAsianWidth.txt`
([proposal](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2587525413),
[maintainer response](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2587599860)).
The property inputs have first-party Unicode definitions and machine-readable
data: [UAX #11](https://www.unicode.org/reports/tr11/),
[UAX #24](https://www.unicode.org/reports/tr24/),
[UTS #51](https://www.unicode.org/reports/tr51/), and the pinned Unicode 17
files for [General Category](https://www.unicode.org/Public/17.0.0/ucd/extracted/DerivedGeneralCategory.txt),
[East Asian Width](https://www.unicode.org/Public/17.0.0/ucd/EastAsianWidth.txt),
[Script](https://www.unicode.org/Public/17.0.0/ucd/Scripts.txt), and
[emoji properties](https://www.unicode.org/Public/17.0.0/ucd/emoji/emoji-data.txt).

The current independent amendment uses this same single-code-point foundation,
then adds CJK sequences, punctuation sequences, standardized variation
sequences, and two-code-point rules. Those additions are exactly what the
narrow proposal defers
([current amendment](https://github.com/tats-u/markdown-cjk-friendly/blob/2f4a9a739520681fa9a38fcaa643788dcdf99f19/specification.md#L11-L37)).

## Provenance of the August rule

The parser rule did not first appear in August. MacFarlane implemented it in
cmark commit
[`5ffd6b0`](https://github.com/commonmark/cmark/commit/5ffd6b0d5f5bf70a19138bb3e2929f566c2f4993)
on May 7, 2024. Its commit message gives the concise formulation: if either
the preceding or following character is CJK, count the run as both left- and
right-flanking. The issue discussion records the same proposal and the need to
define the affected code points
([May 7 summary](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2098683604)).

The state validated in August contained more than that one sentence:

- cmark PR [#556](https://github.com/commonmark/cmark/pull/556) added a large
  CJK test file and variation-selector fixes.
- At commit
  [`6a6bd55`](https://github.com/commonmark/cmark/commit/6a6bd55b924876ccc9093ec45dcc844b0b382a03),
  the implementation recognized U+FE00 through U+FE02 before a delimiter by
  looking back to the preceding base code point. It also classified
  U+E0100 through U+E01EF directly as CJK.
- MacFarlane reported that all collected tests passed, then called the small
  rule promising while repeating that "CJK" still needed a definition
  ([test result](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2295336967),
  [assessment](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2295337497)).
- Support for U+FE0E, VS15, arrived immediately afterward in
  [`39b2bdf`](https://github.com/commonmark/cmark/commit/39b2bdf03cb3b3e1ad3fb340f10de001aebc0cba).

The useful conclusion is narrow. August validated the adjacent-character
flanking idea against the collected corpus. It did not establish a final
Unicode taxonomy, complete variation-sequence semantics, or coverage of every
emphasis marker.

## What the current proof implementations actually do

### Shared flanking calculation

Both current implementations reduce to these expressions:

```text
eitherCJK = CJK(before) OR CJK(after)

leftFlanking =
  NOT afterWhitespace AND
  (normalLeftPunctuationConditions OR eitherCJK)

rightFlanking =
  NOT beforeWhitespace AND
  (normalRightPunctuationConditions OR eitherCJK)
```

The normal punctuation conditions are the existing CommonMark definitions
([left-flanking](https://spec.commonmark.org/0.31.2/#left-flanking-delimiter-run),
[right-flanking](https://spec.commonmark.org/0.31.2/#right-flanking-delimiter-run)).
The CJK term is an extra alternative inside those definitions. It is not an
override of the definitions as a whole.

For `*`, CommonMark maps left-flanking directly to `can_open` and
right-flanking directly to `can_close`. For `_`, the extra intraword
restrictions still apply
([CommonMark emphasis rules](https://spec.commonmark.org/0.31.2/#emphasis-and-strong-emphasis),
[cmark code](https://github.com/commonmark/cmark/blob/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5/src/inlines.c#L491-L505)).
As a result, the narrow rule changes ordinary `*` and `**` cases but does not
make embedded `_` or `__` work. For example, `日_本_語` and
`太郎は__「こんにちは」__と` remain literal.

### Current cmark `cjk` branch

The immutable head inspected here is
[`1c2e49a`](https://github.com/commonmark/cmark/commit/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5),
dated September 23, 2024. The relevant code is in
[`src/inlines.c`](https://github.com/commonmark/cmark/blob/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5/src/inlines.c#L423-L507)
and
[`src/utf8.c`](https://github.com/commonmark/cmark/blob/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5/src/utf8.c#L430-L483).

Before a delimiter, cmark skips U+FE00 through U+FE02 and U+FE0E, then tests
the preceding base. It does not skip U+FE03 through U+FE0D or U+FE0F. The
classifier itself includes U+E0100 through U+E01EF, so an ideographic
variation selector immediately before a delimiter counts as CJK even if it is
not preceded by a valid base. U+FE0F intentionally does not inherit CJK status
from its base.

The branch's dedicated
[`cjkemphasis.txt`](https://github.com/commonmark/cmark/blob/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5/test/cjkemphasis.txt)
contains 86 examples and 180 `**` tokens. It contains no single `*`, `_`, or
`__` marker. Passing this file therefore supports strong asterisk emphasis,
but it does not validate the whole CommonMark emphasis-marker space.

### commonmark.js PR #291

Draft [PR #291](https://github.com/commonmark/commonmark.js/pull/291) remains
open and conflicted. Its immutable head is
[`726e9fb`](https://github.com/commonmark/commonmark.js/commit/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f).
The classifier and variation-selector expressions appear at
[`lib/inlines.js` lines 37 through 41](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/lib/inlines.js#L37-L41),
and the delimiter calculation appears at
[`lib/inlines.js` lines 248 through 345](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/lib/inlines.js#L248-L345).

Its added coverage consists of the existing issue #108 case plus one example
containing four CJK paragraphs
([tests](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/test/regression.txt#L58-L64),
[additional tests](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/test/regression.txt#L522-L535)).
All use `**`.

## Exact classifier in both current branches

cmark's commented C expression and commonmark.js's regular expression compile
to the same nine inclusive intervals:

```text
U+2E80..U+4DBF
U+4E00..U+A4CF
U+F900..U+FAFF
U+FE10..U+FE1F
U+FE30..U+FE6F
U+FF00..U+FFEE
U+1B000..U+1B16F
U+20000..U+3FFFF
U+E0100..U+E01EF
```

This is a hand-written operational table, not a defensible normative
definition of CJK. Its most important consequences are:

- It includes internal unassigned gaps and noncharacters. Examples include
  U+2FE0 through U+2FEF, U+2FA20 through U+2FFFF, and U+323B0 through U+3FFFF.
- It includes all Yi characters between U+A000 and U+A4CF and fullwidth or
  halfwidth forms through U+FFEE.
- It excludes Yijing Hexagrams U+4DC0 through U+4DFF. The branch first added
  them, then deliberately reverted that decision in cmark
  [PR #564](https://github.com/commonmark/cmark/pull/564) because corresponding
  trigram symbols occur in another block.
- It excludes most Korean text. Hangul Jamo U+1100 through U+11FF and Hangul
  Syllables U+AC00 through U+D7AF are absent. Only Compatibility Jamo inside
  U+3130 through U+318F are covered.
- It treats any isolated U+E0100 through U+E01EF selector as CJK.

Those boundaries explain why the code was useful for experimentation but
should not become specification prose.

## Recommended narrow Unicode definition

Use a property definition rather than those nine ranges:

> A CJK code point is an assigned Unicode code point whose
> `East_Asian_Width` property is `Wide`, `Fullwidth`, or `Halfwidth` and whose
> `Emoji_Presentation` property is false, or whose `Script` property is
> `Hangul`.

For reproducible generated tables, the proposal should pin Unicode 17.0.0. A later
Unicode update should be a visible repository change that regenerates the
table and behavior report. The normative prose can name the properties while
the C implementation uses generated intervals.

The "assigned" condition is deliberate. It prevents a CommonMark behavior
change for reserved code points and noncharacters merely because UAX #11 gives
some unassigned East Asian ranges a default width. New assignments can enter
the generated table when the pinned Unicode version is updated.

The emoji exclusion is also behavioral, not cosmetic. Many default emoji have
East Asian Width `W`; counting them as CJK would enable punctuation-adjacent
emphasis in emoji contexts. `Emoji_Presentation`, not the broader `Emoji`
property, is the relevant exclusion. The broader property includes characters
such as ASCII digits that render as text unless combined into an emoji
sequence
([UTS #51 emoji properties](https://www.unicode.org/reports/tr51/#Emoji_Properties_and_Data_Files)).

This definition intentionally includes East Asian punctuation and fullwidth
forms. They are needed for mixed examples such as `a**「x」**b`, where neither
outside character is itself Han, Hiragana, or Katakana. It also includes
Hangul explicitly, fixing the major omission in the proof branches.

### Explicit sequence deferral

Version 1 should say all of the following plainly:

- Adjacent means one Unicode code point, not one grapheme cluster.
- U+FE00 through U+FE0F and U+E0100 through U+E01EF are not independently CJK.
- A CJK base followed by a variation selector immediately before a delimiter
  receives no special treatment.
- Standardized variation sequences and ambiguous CJK punctuation sequences
  are outside this proposal.
- Ill-formed UTF-8 and isolated UTF-16 surrogate handling stay at each
  implementation's existing boundary. They cannot acquire CJK status.

This knowingly leaves rare cases such as `塚︀**(variant)**漢` unchanged. That
is a clean, testable boundary. The broader amendment can address those cases
later without being smuggled into a supposedly adjacent-code-point rule.

## Deterministic edge cases

The following table follows directly from the recommended rule. "Current"
means CommonMark 0.31.2 behavior. "Narrow" means the proposal in this note.

| Markdown | Current | Narrow | Reason |
|---|---|---|---|
| `これは**「重要なこと」**です。` | literal `**` | strong | Both runs have CJK code points or CJK-width punctuation adjacent. |
| `a**「x」**b` | literal `**` | strong | `「` and `」` have East Asian width and unlock the punctuation cases. |
| `漢**字**漢` | strong | strong | Existing CommonMark already permits intraword asterisk emphasis when both sides are non-punctuation. |
| `漢*字*漢` | emphasis | emphasis | The common intraword single-asterisk case already works. |
| `漢_字_漢` | literal `_` | literal `_` | The underscore-specific intraword restriction still applies. |
| `漢__字__漢` | literal `__` | literal `__` | The same restriction applies to strong underscores. |
| `漢 **字** 漢` | strong | strong | Whitespace behavior is unchanged. |
| `漢**"x"**a` | literal `**` | literal `**` | The closing run has no adjacent CJK code point. Each run is classified locally. |
| `a***「x」***b` | literal `***` | nested emphasis and strong | The normal delimiter matching algorithm runs after the new flanking classification. |
| `**x!**𠮷` | literal `**` | strong | The supplementary-plane Han code point after the closing run is CJK. |
| `**안녕(hello)**하세요.` | literal `**` | strong | Hangul is included by the Script property. |
| `a**😀**b` | literal `**` | literal `**` | A default emoji is excluded even though its East Asian width is `W`. |
| `塚︀**(variant)**漢` | literal `**` | literal `**` | U+FE00 sequence handling is explicitly deferred. |
| `㊙︎**(Top)**㊙︎` | literal `**` | literal `**` | U+FE0E sequence handling is explicitly deferred. |

The nested case is an intended consequence, not a separate CJK algorithm.
The CommonMark delimiter-stack rules and multiple-of-three rule remain
unchanged.

## Implementation hazards discovered in the proof branches

These are reasons to reimplement the candidate on the pinned baselines instead
of rebasing either experiment blindly.

### Supplementary-plane lookahead in commonmark.js

PR #291 reads the character before a delimiter with a surrogate-aware helper,
but reads the character after it with `peek()`. `peek()` uses `charCodeAt`, so
it returns only the high surrogate for a supplementary-plane character
([`peek()`](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/lib/inlines.js#L131-L139),
[after-character path](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/lib/inlines.js#L294-L319)).
Consequently, a case such as `**x!**𠮷` works in cmark's branch but cannot
classify U+20BB7 correctly in the JavaScript branch. The implementation needs a
code-point-aware reader in both directions.

### Variation-selector disagreement

cmark replaces `before_char` with the base code point before calculating both
CJK and punctuation status. commonmark.js retains the selector for punctuation
status and consults the base only for `either_is_cjk`. This makes underscore
behavior disagree. For example, cmark can open the underscore in
`㊙︎_foo_`, while PR #291 cannot. The narrow policy avoids this disagreement by
deferring variation sequences completely.

### Smart-quote spillover

Both branches reuse the emphasis delimiter scanner for smart single and double
quotes. The CJK override therefore changes smart-quote direction even though
smart punctuation is not part of the CommonMark proposal
([cmark quote branch](https://github.com/commonmark/cmark/blob/1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5/src/inlines.c#L464-L505),
[commonmark.js quote branch](https://github.com/commonmark/commonmark.js/blob/726e9fb38f325bbcb9bcc6f18d7e8e4a4b51cb1f/lib/inlines.js#L270-L343)).
A local differential run shows cmark `--smart` changes `「"quote"」` from
opening-plus-closing curly quotes to two closing quotes on the CJK branch. Our
implementation must apply the new CJK term only when scanning `*` or `_`, or
calculate quote flanking separately.

### Test coverage gap

The cmark experiment has substantial realistic Japanese and Chinese coverage,
but every dedicated case uses `**`. The JavaScript PR adds only five CJK
paragraphs, also using `**`. The fixture oracle therefore needs its own matrix for single
asterisks, underscores, whitespace, nesting, supplementary-plane characters,
emoji exclusion, and protected literal cases. Existing proof-branch tests are
inputs to that matrix, not a complete oracle.

## Recommendation

Freeze these four decisions before generating the behavior matrix:

1. Add the CJK alternative inside the punctuation portion of the two flanking
   definitions. Preserve their whitespace prerequisites.
2. Use the assigned single-code-point Unicode property definition above,
   pinned to Unicode 17.0.0 for generated artifacts.
3. Keep all existing underscore-specific and delimiter-matching rules.
4. Defer every variation-selector and multi-code-point special case.

This is enough to fix the reported Japanese pattern, mixed CJK punctuation,
supplementary Han, and Korean without taking on the full amendment. It also
gives the behavior matrix an exact answer for every input. The remaining uncertainty is
standards policy, not parser determinism.
