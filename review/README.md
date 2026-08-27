# Review protocol

Status: native Japanese review complete; Chinese, Korean, and independent implementation reviews pending.

Completed responses and conclusions are published in [results.md](results.md).

The review checks two claims that parser tests cannot establish:

1. Native Japanese, Chinese, and Korean readers do not find the ordinary-language results clearly wrong.
2. A Markdown parser implementer can derive the tested behavior from the normative prose without consulting our implementations.

The two reviews are intentionally separate. Language reviewers judge author expectations, not Unicode formulas. The implementer reviews the rule, not whether the sample sentences sound natural.

## Required reviews

Obtain one completed [language review](language-review.md) from a native reader, or a reader with near-native professional writing fluency, for each language:

- Japanese, recorded as `JA-R1`;
- Simplified or Traditional Chinese, recorded as `ZH-R1` with the variety noted;
- Korean, recorded as `KO-R1`.

Obtain one completed [implementer review](implementer-review.md) from someone who has implemented or substantially maintained a Markdown inline parser. Record it as `IMPL-R1`.

Additional reviews may be recorded as `JA-R2`, `ZH-R2`, `KO-R2`, and so on. One person may review more than one language only if their relevant fluency is recorded for each review. The parser review should come from someone who has not read the candidate implementation patches before completing the exercise.

## Procedure

1. Send only the relevant form. Do not coach the reviewer toward the proposed result.
2. Ask the reviewer to return the case IDs and answers. Free-form comments are optional.
3. Remove names, handles, email addresses, and other contact details from the project record.
4. Record conclusions under an anonymous reviewer ID, preserving disagreements and uncertainty.
5. For `IMPL-R1`, compare the predictions with `proposed_html` in [`fixtures/emphasis.json`](../fixtures/emphasis.json) only after the reviewer has submitted them.
6. If feedback suggests a semantic change, revise the rule and fixture oracle together, then rerun the complete verification suite.

## Acceptance rule

The review is complete when:

- the Japanese, Chinese, and Korean ordinary-prose results have no `clearly wrong` judgment;
- any `context-dependent` or `prefer another result` answer is documented as a policy consideration;
- no reviewer identifies a protected non-change that makes the narrow proposal misleading or harmful;
- `IMPL-R1` predicts every scored case consistently with the fixture corpus, or any mismatch is traced to wording that we correct and retest.

Silence is not evidence. Missing reviews remain open. Review feedback informs the proposal but does not by itself determine a rule change.

## How answers affect the proposal

The answer labels are evidence, not votes that automatically change code:

- `A` supports the proposed result and requires no change.
- `B` identifies context that we must record. It may improve an example or explanation, but it does not by itself change semantics.
- `C` identifies a preferred alternative. It requires investigation of the scope and compatibility cost.
- `D` identifies a clearly wrong result. It blocks review completion until the result is revised, justified, or removed from scope.
- `E` is an abstention. It supplies neither support nor an objection and triggers no code change.

If every reviewer answers `E` to a shared policy check such as `L-01` or `L-02`, retain the narrow behavior and record the uncertainty. Seek another opinion if practical, but do not expand the proposal without affirmative evidence and compatibility analysis.

## Privacy boundary

The published review record contains only:

- an anonymous reviewer ID;
- the relevant language variety or parser experience in broad terms;
- answers and technical comments;
- the date received;
- any resulting project decision.

Do not commit a reviewer's name, account handle, employer, location, contact information, or a link that identifies them.

## Fixture traceability

The review forms contain no new expected behavior. Their cases map to the fixture oracle as follows:

| Review cases | Fixture IDs |
| --- | --- |
| `JA-01`, `JA-02` | `ja-simple-quoted`, `ja-sentence-punctuation` |
| `ZH-01`, `ZH-02` | `zh-sentence-punctuation`, `zh-punctuation-only` |
| `KO-01`, `KO-02` | `ko-parenthetical`, `ko-link-particle` |
| `L-01` | `ja-corner-quotes-strong`, `cjk-quotes-underscore` |
| `L-02` | `space-after-opener`, `space-before-closer` |
| `L-03` | `latin-cjk-quotes` |
| `I-01` through `I-12` | In table order: `cjk-ascii-parentheses`, `latin-cjk-quotes`, `latin-curly-quotes`, `cjk-quotes-underscore`, `space-after-opener`, `space-before-closer`, `default-emoji`, `cjk-outside-emoji-content`, `ideographic-variation-deferred`, `non-bmp-han`, `triple-nested`, `line-start` |
