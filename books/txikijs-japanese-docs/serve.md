---
title: "HTTPサーバー"
free: true
---

# HTTPサーバー

txiki.jsにはHTTP/HTTPS serverとWebSocket supportが組み込まれています。最短は `tjs serve` です。

```bash
tjs serve app.js
```

moduleは `fetch` methodを持つobjectをdefault exportします。

```js
export default {
  fetch(request) {
    const url = new URL(request.url);
    return new Response(`path: ${url.pathname}\n`);
  }
};
```

handlerは標準 `Request` を受け取り、`Response` またはそれにresolveするPromiseを返します。既定portは8000です。

## CLI option

- `-p`, `--port PORT`: listen port
- `--tls-cert FILE`: TLS certificate PEM
- `--tls-key FILE`: TLS private key PEM

certificateとkeyはセットで指定します。

## Request context

`fetch` の第2引数にはserverとremote addressが入ります。

```js
export default {
  fetch(request, { server, remoteAddress }) {
    console.log(remoteAddress);
    return new Response('ok');
  }
};
```

## WebSocket

upgrade requestを受けたら `server.upgrade(request)` を同期的に呼び、そのrequestではResponseを返さず終了します。

```js
export default {
  fetch(request, { server }) {
    if (request.headers.get('upgrade') === 'websocket') {
      server.upgrade(request, { data: { connectedAt: Date.now() } });
      return;
    }
    return new Response('HTTP endpoint');
  },
  websocket: {
    open(ws) {
      console.log('open');
    },
    message(ws, data) {
      ws.sendText(`echo: ${data}`);
    },
    close(ws, code, reason) {
      console.log(code, reason);
    }
  }
};
```

server-side WebSocketでは `sendText()`、`sendBinary()`、`close()`、upgrade時に付けた `data` が使えます。

## Programmatic API

`tjs serve` は `tjs.serve()` のCLI wrapperです。

```js
const server = tjs.serve({
  port: 8080,
  fetch(request) {
    return new Response('hello');
  }
});

console.log(server.port);
await server.close();
```

fetch handlerだけ渡すshorthandもあります。Serverはasync-disposableなので `await using` も利用できます。

TLSをprogrammaticに使う場合はPEM stringを `tls: { cert, key }` に渡します。CA、passphrase、client certificate要求なども設定できます。

## HTTP/2

HTTPSではALPNによってHTTP/2を自動negotiationします。clientが `h2` を提示しなければHTTP/1.1へfallbackします。`tls.alpn` でserver側のprotocol listを制限できます。

## HTTP/3

`http3: true` とTLS設定を有効にすると、同じportのUDP/QUICでHTTP/3もserveできます。TCP側ではHTTP/1.1 / 2も維持され、`Alt-Svc` によりHTTP/3 availabilityをclientへ知らせます。

```js
tjs.serve({
  tls: { cert, key },
  http3: true,
  fetch() {
    return new Response('h1 / h2 / h3');
  }
});
```

handlerの書き方はHTTP versionによって変わりません。

参照: txiki.js `website/docs/guides/serve.md`
