---
title: "Filesystem"
free: true
---

# Filesystem

txiki.jsはfilesystemをglobal `tjs` から扱います。多くのoperationは非同期でPromiseを返し、event loopをblockしないようlibuvのthread pool上で実行されます。

## File全体を読む / 書く

`tjs.readFile()` は `Uint8Array` を返します。

```js
const bytes = await tjs.readFile('hello.txt');
console.log(new TextDecoder().decode(bytes));
```

`tjs.writeFile()` はstringまたは `Uint8Array` を受け取り、fileを作成または上書きします。

```js
await tjs.writeFile('hello.txt', 'hello\n');
await tjs.writeFile('data.bin', new Uint8Array([1, 2, 3]));
```

作成時のmodeも指定できます。

## `tjs.open()` とFileHandle

細かいread/write、random access、streamingには `tjs.open()` を使います。

```js
const f = await tjs.open('notes.txt', 'r');
const buf = new Uint8Array(4096);
const n = await f.read(buf);
await f.close();
```

flagは `r`、`w`、`a`、`x`、`+` を組み合わせます。`read()` はEOFなら `null`、それ以外は読み込んだbyte数を返します。`write()` にはoffsetも指定できます。

FileHandleには `stat()`、`truncate()`、`sync()`、`datasync()`、`chmod()`、`utime()` などもあります。

## `await using`

FileHandleはAsyncDisposableです。

```js
async function readFirst(path) {
  await using f = await tjs.open(path, 'r');
  const buf = new Uint8Array(256);
  const n = await f.read(buf);
  return new TextDecoder().decode(buf.subarray(0, n ?? 0));
}
```

例外やearly returnでもcloseされます。

## Streaming

FileHandleには `readable` / `writable` Web Streamがあります。

```js
await using src = await tjs.open('large.bin', 'r');
await using dst = await tjs.open('copy.bin', 'w');
await src.readable.pipeTo(dst.writable);
```

大きなfileを全量memoryへ載せず処理できます。

## Directory

`tjs.readDir()` はasync iterableな `DirHandle` を返します。

```js
for await (const entry of await tjs.readDir('.')) {
  console.log(entry.name, entry.isDirectory);
}
```

entryにはfile、directory、symlink、device、FIFO、socketなどのtype判定があります。

## Metadata

`tjs.stat()` はtargetを、`tjs.lstat()` はsymlinkそのものを調べます。size、mode、uid/gid、inode、block数、各timestamp、file typeなどを取得できます。

filesystem全体の容量情報には `tjs.statFs()` を使います。

## Temporary file / directory

`tjs.makeTempDir()` と `tjs.makeTempFile()` があり、template末尾の `XXXXXX` がrandom文字へ置換されます。

```js
const dir = await tjs.makeTempDir('work-XXXXXX');
await using tmp = await tjs.makeTempFile('upload-XXXXXX');
```

## その他のpath operation

`tjs.makeDir()`、`remove()`、`rename()`、`copyFile()`、`realPath()`、`symlink()`、`link()`、`readLink()`、`chmod()`、`chown()`、`utime()` などがあります。

`remove()` はdirectory treeもrecursiveに削除できます。

## Watch

`tjs.watch()` はpath変更を監視し、`change` / `rename` eventをcallbackへ渡します。

```js
using watcher = tjs.watch('./src', (filename, event) => {
  console.log(event, filename);
});
```

watcherはDisposableなので `using` で自動closeできます。

pathのjoin / normalize / parseなど高level operationには `tjs:path` を使います。

参照: txiki.js `website/docs/guides/filesystem.md`
