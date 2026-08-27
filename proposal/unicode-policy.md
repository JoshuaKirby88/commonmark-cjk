# Candidate Unicode policy

Status: implemented and verified in the pinned cmark and commonmark.js candidates.

## Normative classification

The proposed rule uses Unicode assignment status and three Unicode properties:

- assigned-character status from the [Unicode glossary](https://www.unicode.org/glossary/#assigned_character) and General Category data in [Unicode Standard Annex 44](https://www.unicode.org/reports/tr44/);
- East Asian Width from [Unicode Standard Annex 11](https://www.unicode.org/reports/tr11/);
- Script from [Unicode Standard Annex 24](https://www.unicode.org/reports/tr24/);
- `Emoji_Presentation` from [Unicode Technical Standard 51](https://www.unicode.org/reports/tr51/).

An assigned character is CJK when:

```text
assigned
and (
  East_Asian_Width in {W, F, H} and not Emoji_Presentation
  or Script = Hangul
)
```

`Emoji_Presentation` is the specific property in `emoji-data.txt`. The broader `Emoji` property is not suitable because it includes ASCII digits, `#`, and `*`.

In generated tables, `assigned` means General Category is neither `Cn` nor `Cs` in `DerivedGeneralCategory.txt`. This excludes reserved code points, noncharacters, and surrogate code points. Some excluded code points would otherwise inherit default East Asian Width values.

## Reference data version

The first reference implementations will generate their tables from Unicode 17.0.0:

| Data | SHA-256 |
| --- | --- |
| `DerivedGeneralCategory.txt` | `d62e5bab70ca74f099343f71224fa051cb1fdd61a1ab45c0488c44cfc0b6102e` |
| `EastAsianWidth.txt` | `ea7ce50f3444a050333448dffef1cadd9325af55cbb764b4a2280faf52170a33` |
| `Scripts.txt` | `9f5e50d3abaee7d6ce09480f325c706f485ae3240912527e651954d2d6b035bf` |
| `emoji-data.txt` | `2cb2bb9455cda83e8481541ecf5b6dfda66a3bb89efa3fa7c5297eccf607b72b` |

Official data URLs:

- <https://www.unicode.org/Public/17.0.0/ucd/extracted/DerivedGeneralCategory.txt>
- <https://www.unicode.org/Public/17.0.0/ucd/EastAsianWidth.txt>
- <https://www.unicode.org/Public/17.0.0/ucd/Scripts.txt>
- <https://www.unicode.org/Public/17.0.0/ucd/emoji/emoji-data.txt>

The generated tables, source script, Unicode version, and input hashes will be committed. Regeneration must produce byte-for-byte identical output.

The normative CommonMark prose should name the properties without making Unicode 17 permanent. CommonMark already defines whitespace and punctuation through Unicode properties without pinning one permanent Unicode version. Conformance examples will use assigned characters whose classifications are stable. A later Unicode data update should be an explicit implementation change with its own generated diff and tests.

## Why these properties

Hand-written block ranges caused three problems in the 2024 experiments:

- they require manual updates when Unicode adds characters or blocks;
- broad block intervals include gaps and unrelated symbols;
- the tested ranges omitted ordinary Hangul syllables.

East Asian Width directly captures wide, fullwidth, and halfwidth code points used in East Asian text. The separate Hangul condition includes neutral-width Hangul Jamo such as `U+1160`. Excluding `Emoji_Presentation` prevents ordinary default emoji from changing Markdown flanking merely because Unicode assigns them width `W`.

## Representative classifications

| Character | Code point | EAW | Script | Emoji presentation | CJK |
| --- | --- | --- | --- | --- | --- |
| `語` | `U+8A9E` | `W` | Han | no | yes |
| `「` | `U+300C` | `W` | Common | no | yes |
| `。` | `U+3002` | `W` | Common | no | yes |
| `（` | `U+FF08` | `F` | Common | no | yes |
| `안` | `U+C548` | `W` | Hangul | no | yes |
| `ᅠ` | `U+1160` | `N` | Hangul | no | yes |
| `𠮷` | `U+20BB7` | `W` | Han | no | yes |
| `㊙` | `U+3299` | `W` | Common | no | yes |
| `😀` | `U+1F600` | `W` | Common | yes | no |
| `🈯` | `U+1F22F` | `W` | Common | yes | no |
| `“` | `U+201C` | `A` | Common | no | no |
| `󠄀` | `U+E0100` | `A` | Inherited | no | no |
| unassigned | `U+2FA20` | `W` | Unknown | no | no |

These values come from the pinned Unicode 17 data files above.

## Code points, not display sequences

The candidate classifies the immediate code points around a delimiter. Variation selectors are assigned characters, but they are not CJK under the property definition. It does not inspect a second code point to decide whether an adjacent symbol has emoji, text, narrow, or fullwidth presentation.

This creates a known limitation:

```markdown
禰󠄀**(ね)**豆子
```

The code point immediately before the opening delimiter is the ideographic variation selector `U+E0100`, not `禰`. The narrow rule leaves the input literal. The full CJK-friendly amendment handles it, but doing so requires sequence definitions and extra lookaround that are intentionally outside this proposal.

## Malformed input

CommonMark defines a character as a Unicode code point and does not mandate one encoding. The normative proposal therefore does not assign CJK behavior to isolated UTF-16 surrogates or malformed UTF-8 byte sequences.

Reference implementations must not crash, hang, or read outside input bounds on malformed runtime input. They may apply their existing replacement-character behavior. Robustness tests belong in the implementation package, but malformed code units do not expand the syntax definition.

## Scope escalation rule

If presentation-sequence behavior becomes a requirement, the proposal must be reconsidered before implementation changes. Adding that behavior changes the parser from immediate-code-point classification to sequence-aware lookaround and activates the larger amendment scope.
