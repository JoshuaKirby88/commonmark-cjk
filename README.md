# CommonMark CJK emphasis

This repository is for improving CommonMark emphasis around CJK text and punctuation.

The immediate goal is a focused, reviewable change to the CommonMark specification. It is not a campaign to patch every Markdown engine or product independently.

## Current status

We are still choosing the exact normative boundary. No implementation scope has been accepted yet.

The leading option is the narrow rule tested by the CommonMark maintainers in 2024: when a delimiter run is adjacent to a defined CJK character, treat it as both left-flanking and right-flanking. Before implementing it, we need agreement on the character definition and on which Unicode sequence edge cases belong in the first proposal.

## Proposed first scope

Include:

- CommonMark emphasis and strong emphasis with `*`, `**`, `_`, and `__`
- representative Japanese, Chinese, and Korean examples
- negative and nested-emphasis compatibility cases
- a precise Unicode definition and update policy
- proof implementations in cmark and commonmark.js
- conformance and performance results

Defer:

- GFM strikethrough
- CJK line-breaking and spacing changes
- product-specific integrations
- every script that omits spaces between words
- unrelated emoji and keycap parsing behavior

## Repository layout

- `.worktrees/` is reserved for disposable upstream checkouts and is not committed.

