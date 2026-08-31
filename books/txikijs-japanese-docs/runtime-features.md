---
title: "Runtime Features"
free: true
---

# Runtime Features

txiki.jsはES2025をほぼ網羅することを目標にしつつ、server-side / native runtimeとして必要な機能を追加しています。

## Modules

JavaScript moduleは標準ES Modulesです。local file、HTTP(S) URL、`tjs:` 標準module、Import Attributes、Import Mapsを利用できます。

## Core features

- TCP / TLS / UDP / Unix domain socket
- child process起動とsignal処理
- asynchronous filesystem API
- DNS lookup（`getaddrinfo`）
- HTTP / HTTPS serverとWebSocket
- WASI
- standalone executable生成
- built-in test runner

これらはNode.js互換APIとしてではなく、可能な部分はWeb StreamsやRequest/ResponseなどWeb Platformのprimitiveと組み合わせて設計されています。

例えばnetwork socketの入出力は `ReadableStream<Uint8Array>` / `WritableStream<Uint8Array>` として扱え、file handleやprocess pipeとも同じstream modelで接続できます。

## Build featureの確認

custom buildではWebAssemblyやSQLiteを無効化できるため、実行時には `tjs.engine.features` で利用可能か判定できます。

```js
console.log(tjs.engine.features);
```

詳細なglobal APIはAPI Reference、各機能の実例は後続のGuidesを参照してください。

参照: txiki.js `website/docs/features/runtime-features.md`
