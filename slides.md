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

<h1><span class="legacy">Legacy</span> Cache</h1>

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

# Pain point: `fetch`とキャッシュ

`fetch`が動くかどうかわかりづらい

```tsx
export default async function Page() {
  // Q. この`getRandomTodo()`はキャッシュされる/されない？🤔
  const { todo } = await getRandomTodo();
  // ...
}
```

<div v-click class="mt-5">

### A. 設定や実装次第

- Full Route Cache: 設定や実装次第
  - デフォルトではCacheされる
  - PageやLayoutの設定でCacheをOpt outできる
  - `cookies()`や`headers()`をどこかで使ってたらCacheされない
- Data Cache: `fetch()`のオプション次第ではCacheされない
- Router Cache: Router Cacheは5m or 30s必ずCacheされる

</div>

---
transition: fade
---

# Pain point: 設定の設計

必要な設定ができない、複雑な設定が多すぎる

- Router Cache
  - 5m or 30s必ずCacheされる（ドキュメントに記載なし）
  - 時間を設定できない
- Full Route Cache, Data Cache
  - 実装と設定が遠くなりがち
  - 設定が複雑すぎ

```tsx
export const fetchCache = "auto";
// 'auto' | 'default-cache' | 'only-cache'
// 'force-cache' | 'force-no-store' | 'default-no-store' | 'only-no-store'
```

---

# Pain point: バグや脆弱性

煩雑な実装起因のバグや脆弱性

- e.g. Intercepting RoutesでRouter Cacheが不安定な挙動になる
  - 参考: [vercel/next.js#52748](https://github.com/vercel/next.js/pull/52748)
  - キャッシュのキーの明らかな考慮漏れ
  - 修正するには抜本的な再設計が必要だった

---
layout: fact
---

## App Router登場当初のCacheは<br>ともかく悩みの種だった

---
layout: fact
---

## Next.js開発チームには<br>多くのフィードバックが寄せられた

---
layout: section
---

<h1>Cache <span class="improvement">Improvement</span></h1>

---

<h1>Cache <span class="improvement">Improvement</span></h1>

Next.js開発チームはコミュニティとの議論を開始

- Discussion: [_Deep Dive: Caching and Revalidating_](https://github.com/vercel/next.js/discussions/54075)
  - 開発チームとコミュニティで論点やユースケースを擦り合わせつつ、慎重に方針を検討
  - 「コミュニティが求めてるユースケースは何か？」
  - 「Router Cacheの寿命は設定できれば解決するのか？他の解決方法はないのか？」
- 主にv14~15で少しづつ改善が進む
  - ドキュメントの充実
  - `staleTimes`オプション
  - `fetch()`のデフォルトCache廃止

---

# Improvement: Document

あまりに不足していたドキュメントの拡充

- ドキュメントがなかった頃は挙動観察やコードリーディングしかなかった
  - [Next.js App Router 知られざるClient-side Cacheの仕様](https://zenn.dev/akfm/articles/next-app-router-client-cache)
- [Caching in Next.js](https://nextjs.org/docs/app/guides/caching)
  - Cacheの種類、デフォルトの挙動、図など詳細な説明が追加
  - Static, Dynamic Renderingやこれらの条件になるDynamic APIsの定義など

---

# Improvement: Router Cache

初見殺しな仕様が改善

- v14.2: `staleTimes`オプションが追加、Router Cacheの寿命を設定可能に
- v15: `staleTimes`のデフォルトが30sから0sに

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
      static: 180,
    },
  },
};

export default nextConfig;
```

---

# Improvement: `fetch()` cached by default

`fetch()`のデフォルトCacheが廃止

- v15: `fetch()`のデフォルトが変更
  - Opt-in型になったことで直感的に

```tsx
const res = await fetch(`https://...`, {
  // `undefined`(default): Data Cacheは無効、ただしDynamic Renderingには**ならない**
  // `"no-store"`: Data Cacheは無効、Dynamic Renderingを強制
  // `"force-cache"`: Data Cacheは有効
  cache: "force-cache",
});
```

---

# Improvement: Partial Pre-Rendering(PPR)

PPRの発見が議論を大きく変えた

- 従来はページ単位でDynamic, Static Renderingを決定する必要があった
- PPR: ページをStatic Renderingに、部分的にDynamic Renderingが可能に

<div class="flex justify-center">
  <img src="/ppr.png" alt="PPR" class="h-80">
</div>

---
layout: fact
---

## Next.js v13~v15でCacheは<br>丁寧に着実に改善された

---
layout: fact
---

## しかし、4種類のCacheは<br>根本的な複雑さを伴った

---
layout: fact
---

## そこに唐突に現れたのが<br>`"use cache"`である

---
layout: section
---

<h1>Cache <span class="re-architecture">Re-Architecture</span></h1>

---
transition: fade
---

<h1>Cache <span class="re-architecture">Re-Architecture</span></h1>

根本的なCacheの再設計

- `"use cache"`でCache対象のファイルや関数をマークする
  - 既存のCacheとは全く異なる仕組み
  - CacheのキーはNext.jsが自動で検出する
- Cacheの属性はNext.jsのAPIで設定
  - `cacheLife(profile)`, `cacheTag(tagName)`

---
transition: fade
---

# Cache <span class="re-architecture">Re-Architecture</span>

可読性が明らかに向上

```tsx {all|3}
// Before
export default async function Page() {
  // 🤔この`getRandomTodo()`はキャッシュされる/されない？
  const { todo } = await getRandomTodo();
  // ...
}
```

```tsx {all|2,5}
// After
"use cache";

export default async function Page() {
  // 💡`"use cache"`があるのでこのComponentは明らかにCache対象
  const { todo } = await getRandomTodo();
  // ...
}
```

---

# Architecture: RSCの拡張

TBW
