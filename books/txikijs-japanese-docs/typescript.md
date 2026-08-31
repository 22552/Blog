---
title: "TypeScript"
free: true
---

# TypeScript

txiki.js向けの型定義はnpm package `@txikijs/types` として提供されています。

```bash
npm install @txikijs/types --save-dev
```

これを入れるとeditorでtxiki.js固有APIの補完やtype checkを利用できます。

## TypeScriptファイルの実行

txiki.jsは `.ts` を直接実行するruntimeではありません。TypeScriptは先にJavaScriptへ変換する必要があります。推奨されている方法はesbuildで、type strippingとbundleを同時に行えます。

```bash
tjs bundle src/main.ts bundle.js
tjs run bundle.js
```

`tjs bundle` はesbuildを利用するため高速ですが、esbuild自身はTypeScriptの型検査を行いません。厳密なtype checkが必要なら別途次のように実行します。

```bash
npx tsc --noEmit
```

つまり、txiki.jsでのTypeScript利用は「型検査はTypeScript compiler、実行用JS生成はesbuild / `tjs bundle`」と分けて考えると分かりやすいです。

参照: txiki.js `website/docs/typescript.md`
