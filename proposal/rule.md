# Candidate narrow rule

Status: local proposal for behavior-matrix review. This is not ready for upstream submission.

## Objective

Make `*` and `**` emphasis delimiters work next to ordinary CJK text and punctuation while preserving CommonMark's whitespace rules and underscore restrictions.

## Character definition

A CJK character is an [assigned Unicode character](https://www.unicode.org/glossary/#assigned_character) that meets either condition:

1. its East Asian Width property is `W`, `F`, or `H`, and it does not have the `Emoji_Presentation` property;
2. its Script property is `Hangul`.

The definition uses properties instead of hand-written Unicode blocks. The accompanying reference data is pinned to Unicode 17.0. See [unicode-policy.md](unicode-policy.md).

## Flanking override

Let `before` and `after` be the Unicode code points immediately before and after a delimiter run. A line boundary counts as Unicode whitespace, as it does in CommonMark.

Apply the existing CommonMark flanking definitions, then add these two conditions:

- A delimiter run is also left-flanking when `after` is not Unicode whitespace and either `before` or `after` is a CJK character.
- A delimiter run is also right-flanking when `before` is not Unicode whitespace and either `before` or `after` is a CJK character.

Equivalent expressions are:

```text
left_flanking  = commonmark_left_flanking
              or not after_whitespace and adjacent_cjk

right_flanking = commonmark_right_flanking
               or not before_whitespace and adjacent_cjk

adjacent_cjk = before_is_cjk or after_is_cjk
```

This wording is more exact than saying every CJK-adjacent run is automatically both left-flanking and right-flanking. Whitespace still prevents opening on its right and closing on its left.

## Delimiter consequences

The existing rules that turn flanking runs into openers and closers remain unchanged.

For `*` and `**`, left-flanking controls opening and right-flanking controls closing. The new condition therefore fixes the simple Japanese quotation:

```markdown
これは**「重要なこと」**です。
```

Proposed HTML:

```html
<p>これは<strong>「重要なこと」</strong>です。</p>
```

For `_` and `__`, CommonMark's additional intraword restrictions still apply. The shared flanking calculation changes, but inputs such as `語__強調__語` remain literal. This proposal does not make underscores behave like asterisks.

The override applies only while scanning `*` and `_` delimiter runs. An implementation that shares delimiter-scanning code with smart quotes must not change smart-quote behavior.

## Intentional compatibility change

CJK punctuation itself triggers the override, including in mixed text. For example:

```markdown
a**「x」**b
```

changes from literal markers to:

```html
<p>a<strong>「x」</strong>b</p>
```

This is intentional. The syntax cannot know the language of the surrounding letters. It only classifies the code points adjacent to the delimiter.

Ambiguous-width curly quotes do not trigger the override:

```markdown
a**“x”**b
```

Their East Asian Width is `A`, so this input remains literal unless another CommonMark rule makes the delimiters valid.

## Explicit deferrals

This candidate examines immediate code points only. It does not reinterpret a base character by looking at a following variation selector. The following remain outside version one:

- CJK base characters followed by standard or ideographic variation selectors next to a delimiter;
- emoji characters forced to text presentation with `U+FE0E`;
- ambiguous punctuation forced to a fullwidth form by a standardized variation selector;
- keycap and other multi-code-point emoji sequences;
- scripts outside the stated CJK definition;
- GFM strikethrough and other Markdown extensions.

These cases can be proposed separately. They do not block the ordinary Japanese, Chinese, and Korean cases in the behavior matrix.

## Evidence

John MacFarlane summarized the candidate in May 2024 as treating a delimiter run as both left-flanking and right-flanking when either surrounding character is CJK, subject to defining CJK ([issue comment](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2098683604)). After the cmark experiment passed its collected tests, he called the small change promising and restated the rule in August 2024 ([test result](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2295336967), [rule sketch](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2295337497)).

The cmark experiment preserves the whitespace checks while adding the adjacent-CJK condition. Its pinned research commit is `1c2e49a9e108ef2ee8ef29dd773ede4a1e8dbdf5`. The later proposal to derive character ranges from East Asian Width data received maintainer support ([property proposal](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2587525413), [maintainer response](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2587599860)).

The code-point-only boundary follows MacFarlane's request to avoid increasing the rule's complexity for rare presentation sequences ([complexity comment](https://github.com/commonmark/commonmark-spec/issues/650#issuecomment-2367245456)). The complete sequence-aware behavior remains documented by the independent [CJK-friendly amendment](https://github.com/tats-u/markdown-cjk-friendly/blob/main/specification.md).
