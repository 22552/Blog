---
title: "HTTP Proxy Support"
free: true
---

# HTTP Proxy Support

txiki.jsは標準的なproxy環境変数を自動で読み、HTTP系のoutbound connectionをproxy経由にできます。追加flagは不要です。

対象は `fetch()`、XMLHttpRequest、WebSocket / WebSocketStream、HTTP(S) module importなどです。

## 環境変数

| 変数 | 用途 |
|---|---|
| `http_proxy` / `HTTP_PROXY` | `http://` / `ws://` target |
| `https_proxy` / `HTTPS_PROXY` | `https://` / `wss://` target |
| `all_proxy` / `ALL_PROXY` | scheme-specific設定がない場合のfallback |
| `no_proxy` / `NO_PROXY` | proxyを通さないhost一覧 |

lowercaseとuppercaseが両方ある場合はlowercaseが優先されます。

## Schemeごとの選択

どのproxy variableを使うかはproxy URLではなく**target URLのscheme**で決まります。

```bash
http_proxy=http://proxy.local:8080 tjs run app.js
https_proxy=http://proxy.local:8080 tjs run app.js
all_proxy=http://proxy.local:8080 tjs run app.js
```

scheme-specific variableと `all_proxy` の両方がある場合は前者が優先されます。

## Proxy authentication

Basic認証を使うproxyではcredentialをproxy URLに含められます。runtimeはそこから認証情報を取り出してProxy-Authorizationを構成します。credentialをscriptやrepositoryへ直接hard-codeしないようにし、環境変数やsecret managementで渡す方が安全です。

## `no_proxy`

```bash
no_proxy=localhost,127.0.0.1,.internal.example tjs run app.js
```

対応pattern:

- exact hostname
- `.example.com` のようなdomain suffix
- `example.com:8080` のようなport指定
- `*` ですべてproxy bypass

複数指定はcomma区切りで、周囲のwhitespaceは除去されます。

## 影響するAPI

- `fetch()`
- XMLHttpRequest
- WebSocket / WebSocketStream
- HTTP / HTTPS module import

つまりmodule取得も通常のHTTP requestと同じproxy policyに従います。

参照: txiki.js `website/docs/guides/http-proxy.md`
