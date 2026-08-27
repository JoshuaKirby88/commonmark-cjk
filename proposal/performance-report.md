# Performance report

Status: verified local measurement.

## Result

No measured median slowdown reached the 5 percent investigation threshold.
The largest slowdown was 3.66 percent for commonmark.js on normal prose. No
complexity regression appeared in the upstream pathological suites or the
additional long-input checks.

## Method

`scripts/benchmark` compared each untouched parser with its candidate on three
corpora:

- normal Markdown prose with ordinary emphasis and links;
- CJK-heavy prose containing the proposed emphasis patterns;
- delimiter-heavy mixed CJK input with valid and deliberately literal runs.

Each corpus was at least 2 MiB. The script performed two warmup runs and 13
measured runs for every engine, version, and corpus. It alternated baseline and
candidate order. Timings include process startup and discard rendered output.
The table reports wall-clock medians and population variance.

Command:

```sh
./scripts/benchmark --output tmp/performance.json
```

## Environment

| Item | Value |
| --- | --- |
| Date | 2026-08-27 |
| OS | Linux 7.0.0-30-generic, x86-64, glibc 2.39 |
| CPU | Intel Core i5-9400F at 2.90 GHz |
| Python | 3.14.7 |
| Node.js | 24.17.0 |
| C compiler | GCC 13.3.0 |

## Measurements

Negative deltas mean the candidate was faster in this run. They are treated as
no regression, not as an optimization claim.

| Engine | Corpus | Baseline median ms | Candidate median ms | Delta | Baseline variance ms² | Candidate variance ms² |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| cmark | Normal prose | 68.78 | 70.45 | +2.42% | 0.14 | 0.52 |
| cmark | CJK-heavy | 49.68 | 38.85 | -21.80% | 1.56 | 0.01 |
| cmark | Delimiter-heavy | 140.62 | 134.51 | -4.35% | 9.55 | 10.85 |
| commonmark.js | Normal prose | 331.74 | 343.90 | +3.66% | 153.18 | 248.79 |
| commonmark.js | CJK-heavy | 213.41 | 219.18 | +2.70% | 5.03 | 107.87 |
| commonmark.js | Delimiter-heavy | 440.97 | 422.78 | -4.12% | 519.16 | 253.65 |

Actual corpus sizes ranged from 2,097,167 to 2,097,220 bytes because the script
repeats complete UTF-8 units rather than cutting a code point at the size
boundary.

## Complexity and robustness

The classifier uses binary search over 68 non-overlapping ranges. A delimiter
scan therefore adds a fixed maximum of seven range comparisons per adjacent
code point. It does not scan backward through grapheme clusters or inspect a
second code point.

Both candidates passed their existing 10,000-level pathological emphasis
tests. Additional cases covered 20,000 repeated CJK spans, 20,000
supplementary-plane spans, a 100,000-character unmatched delimiter run, 1,000
levels of CJK-adjacent nesting, and 20,000 deferred variation sequences. All
terminated within the test timeout, and both parsers produced identical output
for valid inputs.

## Conclusion

The candidate passes the performance gate. No median slowdown exceeded 5
percent, no run exceeded the 10 percent submission blocker, and no complexity
regression was observed.
