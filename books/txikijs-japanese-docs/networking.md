---
title: "Networking"
free: true
---

# Networking

txiki.jsのlow-level networkingはWHATWG Direct Sockets系のmodelで提供されます。TCP、TLS、UDP、Unix domain socket / named pipeを扱え、入出力はWeb Streamsです。

## Socket model

socket constructorはすぐobjectを返し、実際の接続完了は `.opened` Promiseで待ちます。

```js
const sock = new TCPSocket('example.com', 80);
const { readable, writable, remoteAddress, remotePort } = await sock.opened;
```

主要memberは `opened`、`closed`、`close()` です。client socketの `opened` からは `ReadableStream<Uint8Array>` と `WritableStream<Uint8Array>` が得られます。server socket側のreadableにはacceptされたclient socketが流れてきます。

socket classは `AsyncDisposable` に対応し、`await using` でscope終了時に自動closeできます。

## TCP

serverは `TCPServerSocket`、clientは `TCPSocket` を使います。

```js
const server = new TCPServerSocket('127.0.0.1', { localPort: 1234 });
const { readable } = await server.opened;

for await (const conn of readable) {
  const { readable: r, writable: w } = await conn.opened;
  r.pipeTo(w);
}
```

client optionには `noDelay`、`keepAliveDelay`、`dnsQueryType`、server optionには `localPort`、`backlog`、`ipv6Only` などがあります。

## TLS

`TLSSocket` / `TLSServerSocket` はTCPと同じstream modelでTLSを処理します。clientは通常組み込みMozilla CAを信頼します。`sni`、`alpn`、`ca`、`cert`、`key`、`verifyPeer` などを設定できます。

serverでは `cert` と `key` が必要です。client証明書を要求するmTLSも設定できます。`opened` からnegotiated ALPNも取得できます。

HTTP/HTTPS用途ではraw TLS socketより `tjs.serve()` の方が通常は高levelです。

## UDP

`UDPSocket` はconnectionless socketで、受信chunkは `{ data, remoteAddress, remotePort }` 形式です。

```js
const sock = new UDPSocket({ localPort: 1234 });
const { readable, writable } = await sock.opened;
```

remote address / portをconstructorで指定するとconnected UDPとして扱えます。

### Multicast

`multicastController` から `joinGroup()` / `leaveGroup()` を呼べます。TTL、loopback、address sharingもoptionで指定できます。

## Unix domain socket / named pipe

`PipeSocket` と `PipeServerSocket` はhost/portの代わりにpathを使います。Unixではdomain socket、Windowsではnamed pipeとして動きます。

## `await using`

```js
async function readBanner(host, port) {
  await using client = new TCPSocket(host, port);
  const { readable } = await client.opened;
  const { value } = await readable.getReader().read();
  return new TextDecoder().decode(value);
}
```

scopeを抜ける際にcloseと `closed` の待機まで行われます。

## Promise-based API

class constructorのほか `tjs.connect()` / `tjs.listen()` もあります。

```js
const tcp = await tjs.connect('tcp', 'example.com', 80);
const server = await tjs.listen('tcp', '127.0.0.1', 1234);
```

transportには `tcp`、`tls`、`pipe`、`udp` を指定できます。

## DNS

`tjs.lookup()` は `getaddrinfo` を利用します。通常は最初の結果を `{ family, ip }` として返し、`{ all: true }` なら全結果を配列で返します。

```js
const addr = await tjs.lookup('example.com');
console.log(addr.ip, addr.family);
```

参照: txiki.js `website/docs/guides/networking.md`
