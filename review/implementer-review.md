# Independent implementer review

This exercise tests whether a Markdown implementer can derive the candidate behavior from its prose. It should take about 15 to 20 minutes.

Please complete it before reading this repository's implementation patches, fixture outputs, or compatibility report. Do not run the candidate parsers. Familiarity with the existing CommonMark emphasis algorithm and consultation of the published CommonMark specification are allowed.

## Candidate rule

A CJK character is an assigned Unicode character that meets either condition:

1. its East Asian Width property is `W`, `F`, or `H`, and it does not have the `Emoji_Presentation` property;
2. its Script property is `Hangul`.

Let `before` and `after` be the Unicode code points immediately before and after an emphasis delimiter run. A line boundary counts as Unicode whitespace.

Apply the existing CommonMark flanking definitions, then add:

```text
left_flanking  = commonmark_left_flanking
              or not after_whitespace and adjacent_cjk

right_flanking = commonmark_right_flanking
               or not before_whitespace and adjacent_cjk

adjacent_cjk = before_is_cjk or after_is_cjk
```

The existing rules that turn flanking runs into openers and closers remain unchanged. This includes the additional restrictions for `_` and `__`. The new conditions apply only while scanning `*` and `_` runs. Classification uses immediate Unicode code points only, with Unicode 17.0 property data.

## Prediction exercise

For each input, write either the complete paragraph HTML or `same as current CommonMark` if the proposal does not affect the parse. The exercise is scored on the parse, not insignificant serialization differences such as a final newline.

| ID | Markdown | Your predicted HTML or unchanged judgment |
| --- | --- | --- |
| I-01 | `語**(ABC)**語` | |
| I-02 | `a**「x」**b` | |
| I-03 | `a**“x”**b` | |
| I-04 | `語__「強調」__語` | |
| I-05 | `語** 強調**語` | |
| I-06 | `語**強調 **語` | |
| I-07 | `a**😀。**b` | |
| I-08 | `語**😀**文` | |
| I-09 | `禰󠄀**(ね)**豆子` | |
| I-10 | `𠮷**(U+20BB7)**語` | |
| I-11 | `語***「引用」***語` | |
| I-12 | `**「先頭」**語` | |

In `I-09`, the character immediately before the opening `**` is the ideographic variation selector U+E0100. In `I-10`, `𠮷` is the single supplementary-plane code point U+20BB7.

## Implementation questions

1. Does the rule determine every prediction without consulting an implementation? If not, identify the ambiguous term or case.
2. Would a conforming implementation need more than the immediately preceding and following code points for this version?
3. Can the CJK classifier be generated from Unicode property files without a hand-maintained block list?
4. Do the unchanged underscore outcomes follow from the existing CommonMark underscore restrictions, or does this prose need an extra exception?
5. Could applying the override in code shared with smart-quote handling accidentally change quotation marks? If so, is the prose clear that this would be an implementation defect?
6. Identify any likely correctness, complexity, or portability problem not covered above.

## Return format

Return the table answers, the six implementation answers, and only this broad experience description:

```text
Markdown inline-parser experience: [implemented / maintained / substantial review]
```

Do not include your name, handle, employer, project affiliation, or contact details. The project will assign the anonymous ID `IMPL-R1` when recording the response.
