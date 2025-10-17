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

# Next.js History

| 年      | 概要           | 詳細                              |
| ------- | -------------- | --------------------------------- |
| 2022/05 | Layout RFC発表 | 後のApp Router                    |
| 2022/10 | Next.js@v13.0  | App Router(Beta)                  |
| 2023/05 | Next.js@v13.4  | App Router(Stable)                |
| 2024/10 | Next.js@v15.0  | `params`などの破壊的変更、PPRなど |
| 2025/10 | Next.js@v16.0  | TBW                               |

---

# Next.jsの価値観

For performance, efficiency and developer experience.

- Next.jsは「デフォルトで高いパフォーマンス」を非常に重視している
- App Routerのパフォーマンスを支える要素
  - SSR
  - Streaming
  - <span v-mark="{ color: 'red', type: 'circle' }" class="font-semibold">Caching</span>

---
layout: section
---

<h2><span class="legacy">Legacy</span> Cache</h2>

---
transition: fade
---

# Next.js Cache

Next.jsには4種類のCacheがある

- Request Memoization
- Data Cache
- Full Route Cache
- Router Cache

---

# Next.js Cache

Next.jsには4種類のCacheがある

<div class="flex justify-center">
  <img src="/cache-architecture.png" alt="Next.js Cache Architecture" class="h-100">
</div>

---

<h1><span class="legacy">Legacy</span> Cache</h1>

App Router初期のCacheは不評だった

- デフォルトでCacheを活用し、Opt-out方式だった
- 当初のバグや複雑な仕様が絡み合い、[議論が加熱](https://github.com/vercel/next.js/discussions/54075)した
  - 参考: [2年前のJSConf JP](https://jsconf.jp/2023/talk/akfm-sato-1/)

---
transition: fade
---

# 複雑な仕様

`fetch`が動くかどうかわかりづらい

TBW

---
transition: fade
---

# 複雑な仕様

必要な設定ができない、複雑な設定が多すぎる

TBW: 30s破棄できない実装について、ドキュメントもなかったこと明記

---
transition: fade
---

# 複雑な仕様

キャッシュ周りの実装が煩雑で、バグが多かった

TBW: Intercepting RoutesがセルフCache poisoningみたいなことになってた
