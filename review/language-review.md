# CJK emphasis language review

Thank you for reviewing a small proposed Markdown change. This should take about 5 to 10 minutes. No software installation or Markdown expertise is required.

Markdown commonly uses two asterisks to request bold text. Current CommonMark leaves the asterisks visible in some ordinary CJK sentences. The proposal would render the intended bold span.

Please review only the section for a language you read natively or write with near-native professional fluency. Judge what an author would reasonably expect when typing the shown Markdown. The sample wording does not need to be stylistically perfect.

For each case, answer one of:

- `proposal is natural`;
- `proposal is acceptable but context-dependent`;
- `current literal result is preferable`;
- `proposal is clearly wrong`;
- `unsure`.

Add a short reason for any answer other than `proposal is natural`.

## Japanese

### JA-01: quoted phrase

Source Markdown:

```markdown
これは**「重要なこと」**です。
```

Current result:

<div lang="ja">これは**「重要なこと」**です。</div>

Proposed result:

<div lang="ja">みたいに、<strong>「何かの状態を判断する指標・目安」</strong>という比喩的な意味でも使います。</div>

Answer: __________

### JA-02: Japanese full stop inside the bold span

Source Markdown:

```markdown
これは**私のやりたかったこと。**だからするの。
```

Current result:

<div lang="ja">これは**私のやりたかったこと。**だからするの。</div>

Proposed result:

<div lang="ja">これは<strong>私のやりたかったこと。</strong>だからするの。</div>

Answer: __________

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

Answer: __________

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

Answer: __________

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

Answer: __________

### KO-02: linked term followed by a Korean particle

Source Markdown:

```markdown
문서를 만들 때 Excel 대신 **[Markdown](https://commonmark.org/help/)**을 사용하세요.
```

Current result:

<div lang="ko">문서를 만들 때 Excel 대신 **<a href="https://commonmark.org/help/">Markdown</a>**을 사용하세요.</div>

Proposed result:

<div lang="ko">문서를 만들 때 Excel 대신 <strong><a href="https://commonmark.org/help/">Markdown</a></strong>을 사용하세요.</div>

Answer: __________

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

Should the underscore line also become bold? `yes`, `no`, or `unsure`: __________

### L-02: spaces still prevent formatting

Both lines remain literal because a delimiter touches a space on its content side:

```markdown
語** 強調**語
語**強調 **語
```

Is keeping these literal reasonable? `yes`, `no`, or `unsure`: __________

### L-03: mixed Latin and CJK punctuation

The same character-based rule changes this mixed-text input:

```markdown
a**「x」**b
```

Proposed result:

<div lang="en">a<strong>「x」</strong>b</div>

Is this behavior acceptable? `yes`, `no`, `context-dependent`, or `unsure`: __________

### L-04: anything missing

Is there an ordinary emphasis pattern in your language that this review should include? If so, provide the exact Markdown and intended result: __________

Do not include your name, handle, employer, or contact details in the returned form.
