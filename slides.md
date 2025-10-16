---
theme: default
class: text-center
highlighter: shiki
lineNumbers: false
colorSchema: "dark"
drawings:
  persist: false
transition: slide-left
title: Next.js Caching - Legacy, Improvement, Re-Architecture
mdc: true
---

# Next.js Caching

<div>
  <span class="legacy">Legacy</span> -> 
  <span class="improvement">Improvement</span> -> 
  <span class="re-architecture">Re-Architecture</span>
</div>

---

# Profile

<div class="pb-5">
  <img src="https://avatars.githubusercontent.com/u/25711332?v=4" width="100" height="100">
</div>

```jsonc
// profile.jsonc
{
  "name": "佐藤 昭文",
  "alias": ["akfm_sato", "あっきー"],
  "job": "フロントエンドエキスパート",
  "tags": ["Next.js", "React", "Test", "リーン開発"],
  "sns": {
    "x": "akfm_sato",
    "zenn.dev": "akfm",
  },
}
```

---
layout: fact
---

## テーマ: Next.jsとCache

---
layout: fact
---

## 結論: `"use cache"`は素晴らしい

---
layout: section
---

# History

Next.jsのCacheの物語

---
transition: fade
---

# Next.js History

| 年      | 概要            | 詳細                                |
| ------- | --------------- | ----------------------------------- |
| 2013/05 | Reactが公開     |                                     |
| 2016/10 | Next.jsが公開   | React＋SSRが広く認知される          |
| 2018~   | Gatsby.jsの台頭 | SSR->SSGが注目される                |
| 2019/07 | Next.js@v9.0    | dynamicルーティング・Typescript対応 |
| 2020/03 | Next.js@v9.3    | SSG対応                             |
| 2020/07 | Next.js@v9.5    | ISR対応                             |

---
transition: fade
---

# Next.js History

| 年      | 概要           | 詳細                              |
| ------- | -------------- | --------------------------------- |
| 2022/05 | Layout RFC発表 | 後のApp Router                    |
| 2022/10 | Next.js@v13.0  | App Router(Beta)                  |
| 2023/05 | Next.js@v13.4  | App Router(Stable)                |
| 2024/10 | Next.js@v15.0  | `params`などの破壊的変更、PPRなど |
| 2025/10 | Next.js@v16.0  | TBW                               |

---

# Next.js History

Next.jsの歴史とキャッシュ

- Next.jsが重視する2つの価値
  - 高いパフォーマンス
  - 優れた開発体験
- App Routerのパフォーマンスを支える要素
  - Streaming
  - **Caching**

---
layout: section
---

<h2><span class="legacy">Legacy</span> Cache</h2>

---

# App Router登場当初のCache

Cacheに関してはOpinionatedな設計と仕様だった

- 積極的にキャッシュを活用
- バグと複雑な仕様が絡み合い、[議論が加熱](https://github.com/vercel/next.js/discussions/54075)した
- [2年前のJSConf JP](https://jsconf.jp/2023/talk/akfm-sato-1/)では、Router Cacheがいかに複雑な仕様か解説した
  TBW

---

# Router Cache

TBW: 30s破棄できない実装について、ドキュメントもなかったこと明記
TBW: Intercepting RoutesがセルフCache poisoningみたいなことになってた

---

# Data Cache

TBW: Otp out方法が複数あり自明でなかった
