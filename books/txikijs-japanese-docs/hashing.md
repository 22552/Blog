---
title: "Hashing"
free: true
---

# Hashing

`tjs:hashing` は同期式のhash APIです。one-shotだけでなく複数回 `update()` するincremental / streaming hashingにも対応します。

```js
import { createHash } from 'tjs:hashing';

const data = new TextEncoder().encode('hello world');
const digest = createHash('sha256').update(data).digest();
console.log(digest);
```

## API

`createHash(type)` はHash objectを返します。algorithm名はcase-insensitiveです。未知のalgorithmは `TypeError` になります。

- `update(Uint8Array)`: dataを追加し、自分自身を返す
- `digest()`: lowercase hex stringを返す
- `bytes()`: raw digestを `Uint8Array` で返す

`digest()` / `bytes()` はhashをfinalizeします。結果はcacheされるため同じHashから複数回読めますが、finalize後に `update()` しないようにします。

## 大きなdataをstreamingする

```js
import { createHash } from 'tjs:hashing';

const hash = createHash('sha256');
await using file = await tjs.open('large.bin', 'r');
const buf = new Uint8Array(65536);

while (true) {
  const n = await file.read(buf);
  if (n === null) break;
  hash.update(buf.subarray(0, n));
}

console.log(hash.digest());
```

全fileをmemoryへ読み込む必要がありません。

## 対応algorithm

`SUPPORTED_TYPES` から一覧を取得できます。MD5、SHA-1、SHA-2 family（SHA-224/256/384/512、SHA-512/224、SHA-512/256）、SHA-3 family（224/256/384/512）を扱えます。

MD5やSHA-1は互換用途や既存format確認には使えますが、新しいsecurity-sensitiveな設計ではcollision resistanceの強いalgorithmを選ぶべきです。

## Web Cryptoとの違い

`crypto.subtle.digest()` はWeb標準で非同期、入力はone-shotで、SHA-1/256/384/512を扱います。一方 `tjs:hashing` は同期でincremental inputに対応し、hex出力が直接得られ、MD5やSHA-3も利用できます。

browserとも共有するcodeならWeb Crypto、large streamやtxiki.js固有のalgorithmが必要なら `tjs:hashing` が向いています。

参照: txiki.js `website/docs/guides/hashing.md`
