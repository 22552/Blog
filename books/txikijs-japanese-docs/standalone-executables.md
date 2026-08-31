---
title: "Standalone Executables"
free: true
---

# Standalone Executables

`tjs compile` を使うと、JavaScriptとtxiki.js runtimeをまとめた単一実行ファイルを作れます。別途C compilerは不要です。

## 基本

`bundle.js` がある場合:

```bash
tjs compile bundle.js
```

Unix系では通常 `bundle`、Windowsでは `bundle.exe` が生成されます。

## Output名を指定する

```bash
tjs compile bundle.js myapp
```

## 先にbundleする

`tjs compile` 自体は複数fileのmodule graphをbundleしません。inputは単一JavaScriptです。複数moduleやTypeScript projectでは先に `tjs bundle` を使います。

```bash
tjs bundle --minify src/main.ts bundle.js
tjs compile bundle.js myapp
```

## TPKとの使い分け

複数file構成をそのまま保持し、assetも含めてpackageしたい場合は `tjs app compile` のTPKを使います。

```text
tjs compile
  単一JSを埋め込む
  module graphは事前bundleが必要

tjs app compile
  app/ directory全体をpackage
  source treeやassetを保持できる
```

1 fileへbundle済みなら `tjs compile`、複数file applicationをそのまままとめたいならTPKという使い分けです。

参照: txiki.js `website/docs/guides/standalone-executables.md`
