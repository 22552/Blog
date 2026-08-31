---
title: "Code Bundling"
free: true
---

# Code Bundling

txiki.jsはES Modulesをnativeで扱えますが、配布時にsourceを1 fileへまとめたい場合はbundleを使います。standalone executableを作る場合やTypeScript projectでは特に便利です。

公式推奨bundlerはesbuildです。txiki.js自身に `tjs bundle` commandがあり、必要なesbuild binaryを自動取得してtxiki.js向けoptionで実行します。

## 基本

```bash
tjs bundle my-app/index.js bundle.js
```

outputを省略するとinput名を元に `.bundle.js` が作られます。

```bash
tjs bundle my-app/index.js
```

## TypeScript

esbuildはTypeScript syntaxからtype annotationを除去できるため、`.ts` / `.tsx` もinputにできます。

```bash
tjs bundle my-app/index.ts bundle.js
```

ただしesbuildはtype checkingをしないので、必要なら `tsc --noEmit` を別途実行します。

## Minify

```bash
tjs bundle --minify my-app/index.ts bundle.js
```

`-m` も使えます。standalone binaryへ埋め込むcodeを小さくしたい場合に有効です。

## esbuildのoption

`--define` や `--drop` など追加のesbuild CLI optionも渡せます。

```bash
tjs bundle --minify --define:DEBUG=false --drop:debugger src/main.ts bundle.js
```

## esbuild binaryの取得

初回実行時、対応するesbuildをnpm registryから取得し、通常 `$TJS_HOME/esbuild/<version>/` にcacheします。その後はcacheを再利用します。取得処理自体もtxiki.jsの `fetch` や `DecompressionStream` を使います。

内部では概ね次の方針でesbuildを実行します。

- importsをbundleする
- `tjs:*` はruntime提供なのでexternal扱い
- targetは最新JavaScript
- Node/browser固有前提を置かないneutral platform
- ESM output
- packageのentry pointを適切に選択

## Source Map

txiki.jsはsource mapを読み、bundle後のstack traceを元sourceへmapできます。

inline source map:

```bash
tjs bundle --sourcemap=inline src/main.ts bundle.js
```

external source map:

```bash
tjs bundle --sourcemap src/main.ts bundle.js
```

error発生時に `sourceMappingURL` を読み、bundle内のline/columnをoriginal sourceへ戻します。inline `data:` URLと外部 `.map` fileの両方に対応します。

## esbuildを直接使う

`tjs bundle` を使わず `npx esbuild` を呼ぶ場合も、`tjs:*` をexternalにし、ESM / neutral platform / modern targetとしてbundleすれば同じ方向のoutputを作れます。

参照: txiki.js `website/docs/guides/code-bundling.md`
