# CommonMark CJK emphasis

This repository is for improving CommonMark emphasis around CJK text and punctuation.

The immediate goal is a focused, reviewable change to the CommonMark specification. It is not a campaign to patch every Markdown engine or product independently.

## Current status

The narrow candidate has passed conformance, compatibility, robustness, and performance verification in both reference parsers. Native Japanese review is complete. Native Chinese, native Korean, and independent parser-implementer reviews remain open.

The candidate builds on the narrow adjacent-CJK rule prototyped in CommonMark issue 650. It adds CJK-aware flanking for emphasis delimiter runs, uses a Unicode 17 property definition, preserves whitespace and underscore restrictions, and defers sequence-aware variation handling.

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

- `reports/baseline.md` records the pinned upstream revisions and untouched test results.
- `proposal/behavior-matrix.md` lists intended changes and protected behavior.
- `proposal/rule.md` and `proposal/unicode-policy.md` define the narrow candidate.
- `proposal/compatibility-report.md` and `proposal/performance-report.md` record verification results.
- `review/README.md` describes the language and independent implementer review method.
- `review/language-review.md` and `review/implementer-review.md` are the two review forms.
- `review/results.md` contains anonymized completed responses and conclusions.
- `patches/commonmark-spec.patch`, `patches/cmark.patch`, and `patches/commonmark.js.patch` contain the three pinned candidate diffs.
- `scripts/verify` reproduces the baseline and the local parser candidates with read-only upstream access.
- `scripts/verify --baseline` runs only the untouched-parser checks.
- `scripts/generate-unicode-tables` reproduces the shared Unicode 17 classifier.
- `scripts/differential-test` and `scripts/robustness-test` exercise compatibility and failure boundaries.
- `scripts/benchmark` reproduces the performance measurement separately.
- `upstreams.lock` pins upstream commits and the commonmark.js dependency lock.
- `research/narrow-rule-evidence.md` documents the candidate rule provenance, proof-implementation behavior, and known hazards.
- `.worktrees/` is reserved for disposable upstream checkouts and is not committed.

## Verification

Run the complete reproducible check with:

```sh
./scripts/verify
```

No CommonMark specification pull request has been submitted from this project.
