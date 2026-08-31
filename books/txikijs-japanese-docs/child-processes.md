---
title: "Child Processes"
free: true
---

# Child Processes

txiki.jsは `tjs.spawn()` で外部programを起動できます。返される `Process` objectから終了待ち、stdio pipe、signal送信などを行えます。現在processを別programへ置き換える `tjs.exec()` もあります。

## Processを起動する

```js
const p1 = tjs.spawn('uname');
const p2 = tjs.spawn(['ls', '-la', '/tmp']);
console.log(p2.pid);
```

既定ではstdin/stdout/stderrをparentからinheritします。

```js
const proc = tjs.spawn([tjs.exePath, 'eval', 'console.log(1 + 1)']);
const status = await proc.wait();
console.log(status);
```

`tjs.exePath` は現在実行中の `tjs` binary pathです。

## Options

`env`、`cwd`、POSIXの `uid` / `gid`、`stdin` / `stdout` / `stderr` を指定できます。stdioは `inherit`、`pipe`、`ignore` のいずれかです。

## stdout / stderrを読む

`pipe` にすると `proc.stdout` / `proc.stderr` がReadableStreamになります。さらに `text()`、`bytes()`、`arrayBuffer()` という全量読み込み用helperがあります。

```js
const proc = tjs.spawn(['echo', 'hello'], { stdout: 'pipe' });
const text = await proc.stdout.text();
const status = await proc.wait();
console.log(text.trim(), status.exit_status);
```

stdoutとstderrの両方をpipeする場合、片方を最後まで読んでからもう片方を読むとbufferが詰まる可能性があるため、通常は `Promise.all()` などで同時にdrainします。

## stdinへ書く

`stdin: 'pipe'` にするとWritableStreamが得られます。

```js
const proc = tjs.spawn('cat', { stdin: 'pipe', stdout: 'pipe' });
const writer = proc.stdin.getWriter();
await writer.write(new TextEncoder().encode('hello'));
await writer.close();
console.log(await proc.stdout.text());
await proc.wait();
```

## 終了を待つ

`proc.wait()` は `{ exit_status, term_signal }` を返します。複数回awaitしても同じ終了結果を取得できます。

## Signal

`proc.kill()` は既定でSIGTERMを送り、signal名も指定できます。PIDを直接指定する場合は `tjs.kill(pid, signal)` を使います。

## `await using`

`Process` はAsyncDisposableです。scope終了時にbest-effortでSIGTERMを送り、process終了を待つため、long-running childのcleanupに使えます。

## `tjs.exec()`

`exec` はchildを作るのではなく、現在process imageを別programへ置き換えます。成功すれば戻ってこないため、その後のJavaScriptは実行されません。wrapper programなどで「準備してから別programそのものになる」場合に向いています。

## Runtime自身へのsignal

`tjs.addSignalListener()` / `tjs.removeSignalListener()` でtxiki.js processが受け取ったsignalを処理できます。listenerが登録されている間はevent loopをaliveに保ちます。

process spawningはlibuv上で非同期に動作し、pipeも標準Web Streamsなので、TransformStreamや `pipeTo()` など他のWeb APIと組み合わせられます。

参照: txiki.js `website/docs/guides/child-processes.md`
