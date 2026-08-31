---
title: "HTTP / WebSocketサーバー"
free: true
---

# HTTP / WebSocketサーバー

txiki.js にはHTTPサーバー機能があり、`Request` / `Response` を使うWeb標準寄りの形でハンドラを書けます。

## 最小のHTTPアプリ

`server.js`:

```js
export default {
  fetch(request) {
    return new Response('Hello from txiki.js!');
  }
};
```

起動:

```bash
tjs serve server.js
```

デフォルトではポート8000で待ち受けます。

```text
http://localhost:8000/
```

ポートを変える場合:

```bash
tjs serve -p 3000 server.js
```

## Requestを読む

ブラウザのService WorkerやWinterTC系ランタイムに近く、URLやmethodを `Request` から取得できます。

```js
export default {
  fetch(request) {
    const url = new URL(request.url);

    if (url.pathname === '/api/time') {
      return Response.json({ now: new Date().toISOString() });
    }

    return new Response('Not Found', { status: 404 });
  }
};
```

巨大なframeworkを入れなくても、小さなAPIならこのまま十分書けます。

## WebSocket

HTTPとWebSocketを同じアプリで扱えます。

```js
export default {
  fetch(request, { server }) {
    if (request.headers.get('upgrade') === 'websocket') {
      server.upgrade(request);
      return;
    }

    return new Response('WebSocket server');
  },

  websocket: {
    open(ws) {
      console.log('connected');
    },
    message(ws, message) {
      ws.send(message);
    },
    close(ws) {
      console.log('closed');
    }
  }
};
```

これだけで簡単なecho serverになります。

## TLS

`tjs serve` には証明書と秘密鍵を渡してHTTPSで起動するオプションもあります。

```bash
tjs serve \
  --tls-cert cert.pem \
  --tls-key key.pem \
  server.js
```

両方をセットで指定します。

## 何を作ると楽しい？

- 小さなJSON API
- SQLiteを使った掲示板
- WebSocketチャット
- ローカル開発ツール
- 1バイナリで配布する管理画面

特に `tjs:sqlite` とTPKを組み合わせると、「Node.jsもnode_modulesも不要な小型Webアプリ」というtxiki.jsらしい構成を作れます。
