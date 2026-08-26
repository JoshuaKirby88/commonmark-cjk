# Baseline verification

Verified: 2026-08-26

## Pinned upstreams

| Repository | Commit |
| --- | --- |
| `commonmark/commonmark-spec` | `3da939428d80f146f270cd1765e4ba462e96bb1b` |
| `commonmark/cmark` | `7042d9978b20fea86ca9cc98bda55f10be392e69` |
| `commonmark/commonmark.js` | `f149165c833d3c4877cab3bb0d7f50f898046d68` |

`upstreams.lock` is the machine-readable source for these pins.

## Toolchain used

- Linux 7.0.0, x86-64
- GCC 13.3.0
- CMake 3.28.3
- GNU Make 4.3
- Python 3.14.7
- Node.js 24.17.0
- npm 11.13.0

These versions record the first successful run. They are not yet minimum-version requirements.

## Results

| Check | Result |
| --- | --- |
| cmark native CTest suite | 9 of 9 tests passed |
| commonmark.js suite | 742 tests passed |
| Current `commonmark-spec` examples | 655 examples extracted |
| Current spec against cmark | 655 passed, 0 failed |
| Current spec against commonmark.js | 655 passed, 0 failed |
| Upstream checkout status after testing | All three clean |

The cmark conformance invocation uses `--unsafe`. Current cmark suppresses raw HTML by default, while the CommonMark examples expect raw HTML to pass through. Without this option, 74 raw-HTML examples fail for an invocation reason unrelated to emphasis parsing.

The commonmark.js repository does not include a package lock. This project records the resolved dependency graph in `locks/commonmark.js-package-lock.json`, verifies its SHA-256 digest through `upstreams.lock`, and temporarily supplies it to `npm ci`. The upstream checkout stays clean.

The initial npm installation reported one moderate-severity advisory in the development dependency graph. It did not affect the baseline tests. Dependency remediation is an upstream concern and outside this proposal.

## Reproduction

Run:

```bash
./scripts/verify
```

The command prepares the pinned checkouts, builds cmark, installs the locked commonmark.js dependencies, runs both native suites, and tests both parsers against the current CommonMark specification.
