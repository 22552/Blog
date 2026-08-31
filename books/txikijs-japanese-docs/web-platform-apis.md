---
title: "Web Platform APIs"
free: true
---

# Web Platform APIs

txiki.jsは、ブラウザ以外のJavaScript runtimeでもWeb標準に近い書き味を使えるよう、多数のWeb Platform APIを実装しています。

## 主な対応API

`AbortController` / `AbortSignal`、`atob` / `btoa`、`Blob`、`BroadcastChannel`、`MessageChannel` / `MessagePort`、`CompressionStream` / `DecompressionStream`、Console、Crypto / SubtleCrypto、Direct Sockets、DOMException、Encoding API、EventSource、EventTarget、`fetch`、File / FileReader、FormData、Performance、`queueMicrotask`、timer、Storage API、Streams API、`structuredClone`、URL / URLPattern / URLSearchParams、WebAssembly、WebSocket / WebSocketStream、Web Workers、XMLHttpRequestなどを利用できます。

Import AttributesではJSON / text / bytesも扱えます。`localStorage` は通常 `$TJS_HOME/localStorage.db` のSQLiteへ永続化され、`sessionStorage` はmemory上です。

## Web Crypto

Global `crypto` は `getRandomValues()` と `randomUUID()` に加え、`crypto.subtle` を実装しています。

利用できる主要な処理はdigest、encrypt/decrypt、sign/verify、key生成、derive、import/export、wrap/unwrapです。

対応algorithmにはSHA-1/256/384/512、AES-CBC/CTR/GCM、AES-KW、RSA-OAEP、RSASSA-PKCS1-v1_5、RSA-PSS、ECDSA、Ed25519、HMAC、ECDH、X25519、PBKDF2、HKDFなどがあります。

```js
const data = new TextEncoder().encode('hello');
const hash = await crypto.subtle.digest('SHA-256', data);
console.log(new Uint8Array(hash));
```

同期hashやincremental hashing、SHA-3などが必要なら `tjs:hashing` を使います。

## WebSocket拡張

txiki.jsの `WebSocket` と `WebSocketStream` には、handshake requestへ独自HTTP headerを追加できる非標準拡張があります。認証headerなどに使えます。ただし `Connection`、`Upgrade`、`Sec-WebSocket-*` などhandshakeを構成するheaderは設定できません。

```js
const ws = new WebSocket('wss://example.com/ws', {
  protocols: ['chat'],
  headers: { 'X-App-Client': 'demo' }
});
```

## WebAssembly

WebAssemblyはWAMR interpreterを利用します。`validate`、`compile`、`instantiate`、streaming variants、Module / Instance / Memory / Table / Globalなど主要APIを実装しています。reference types、SIMD、bulk memoryも一定範囲で利用できます。

一方、table import、JS-backed imported functionでの一部reference type、imported functionからのmulti-value return、同一Moduleを異なるimport objectで再instantiateするケースなどには制約があります。独立したinstanceが必要ならbytesから改めてinstantiateする方法を使います。

## WinterTC

txiki.jsはWinterTC準拠を目標としており、browser専用ではないWeb API互換runtimeとしての共通surfaceを重視しています。

参照: txiki.js `website/docs/features/web-platform-apis.md`
