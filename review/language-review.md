# CJK emphasis language review

Thank you for reviewing a small proposed Markdown change. This should take about 5 to 10 minutes. No software installation or Markdown expertise is required.

Markdown commonly uses two asterisks to request bold text. Current CommonMark leaves the asterisks visible in some ordinary CJK sentences. The proposal would render the intended bold span.

Please review only the section for a language you read natively or write with near-native professional fluency. Judge what an author would reasonably expect when typing the shown Markdown. The sample wording does not need to be stylistically perfect.

For every scored case, answer with one letter. The labels have the same meaning throughout the form:

- `A`: proposed result is natural or acceptable;
- `B`: proposed result is acceptable only in some contexts;
- `C`: a different result is preferable;
- `D`: proposed result is clearly wrong;
- `E`: unsure.

You can respond with lines such as `JA-01: A`. Add a short reason for `B`, `C`, or `D`. An answer records evidence; it does not automatically change the proposal.

## Japanese

### JA-01: simple quoted phrase

Source Markdown:

```markdown
これは**「重要なこと」**です。
```

Current result:

<div lang="ja">これは**「重要なこと」**です。</div>

Proposed result:

<div lang="ja">これは<strong>「重要なこと」</strong>です。</div>

Answer, `A` through `E`: __________

### JA-02: Japanese full stop inside the bold span

Source Markdown:

```markdown
これは**私のやりたかったこと。**だからするの。
```

Current result:

<div lang="ja">これは**私のやりたかったこと。**だからするの。</div>

Proposed result:

<div lang="ja">これは<strong>私のやりたかったこと。</strong>だからするの。</div>

Answer, `A` through `E`: __________

## Chinese

Please state whether you are reviewing Simplified Chinese, Traditional Chinese, or both: __________

### ZH-01: Chinese full stop inside the bold span

Source Markdown:

```markdown
这是**我想做的事。**所以继续。
```

Current result:

<div lang="zh-Hans">这是**我想做的事。**所以继续。</div>

Proposed result:

<div lang="zh-Hans">这是<strong>我想做的事。</strong>所以继续。</div>

Answer, `A` through `E`: __________

### ZH-02: punctuation-only boundary test

This example is deliberately minimal. Judge the Markdown expectation, not whether it is a likely standalone sentence.

Source Markdown:

```markdown
**。**话
```

Current result:

<div lang="zh-Hans">**。**话</div>

Proposed result:

<div lang="zh-Hans"><strong>。</strong>话</div>

Answer, `A` through `E`: __________

## Korean

### KO-01: parenthetical text inside the bold span

Source Markdown:

```markdown
**안녕(hello)**하세요.
```

Current result:

<div lang="ko">**안녕(hello)**하세요.</div>

Proposed result:

<div lang="ko"><strong>안녕(hello)</strong>하세요.</div>

Answer, `A` through `E`: __________

### KO-02: linked term followed by a Korean particle

Source Markdown:

```markdown
문서를 만들 때 Excel 대신 **[Markdown](https://commonmark.org/help/)**을 사용하세요.
```

Current result:

<div lang="ko">문서를 만들 때 Excel 대신 **<a href="https://commonmark.org/help/">Markdown</a>**을 사용하세요.</div>

Proposed result:

<div lang="ko">문서를 만들 때 Excel 대신 <strong><a href="https://commonmark.org/help/">Markdown</a></strong>을 사용하세요.</div>

Answer, `A` through `E`: __________

## Shared policy checks

Review these after the language-specific section.

### L-01: asterisks and underscores remain different

The proposal makes the asterisk form bold but leaves the underscore form literal:

```markdown
語**「強調」**語
語__「強調」__語
```

Proposed result:

<div lang="ja">語<strong>「強調」</strong>語<br>語__「強調」__語</div>

Is the proposed difference between the asterisk and underscore lines acceptable?

Answer, `A` through `E`: __________

### L-02: spaces still prevent formatting

Both lines remain literal because a delimiter touches a space on its content side:

```markdown
語** 強調**語
語**強調 **語
```

Is the proposed literal behavior acceptable?

Answer, `A` through `E`: __________

### L-03: mixed Latin and CJK punctuation

The same character-based rule changes this mixed-text input:

```markdown
a**「x」**b
```

Proposed result:

<div lang="en">a<strong>「x」</strong>b</div>

Is this proposed behavior acceptable?

Answer, `A` through `E`: __________

### L-04: anything missing

Is there an ordinary emphasis pattern in your language that this review should include? If so, provide the exact Markdown and intended result: __________

Do not include your name, handle, employer, or contact details in the returned form.
