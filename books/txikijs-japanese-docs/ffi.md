---
title: "FFI（Native Libraries）"
free: true
---

# FFI（Native Libraries）

`tjs:ffi` はJavaScriptからnative shared libraryを直接呼ぶためのFFIです。libffi上に実装され、scalar、string、buffer、pointer、struct、callbackなどを扱えます。

FFIは型宣言を誤るだけでprocess crashやmemory corruptionにつながり得ます。ここで指定するsignatureは、実際のC ABIと一致している必要があります。

## Libraryを開く

基本は `dlopen()` です。

```js
import { dlopen } from 'tjs:ffi';

const lib = dlopen('c', {
  getpid: { returns: 'i32' },
  abs: { args: ['i32'], returns: 'i32' }
});

console.log(lib.symbols.getpid());
console.log(lib.symbols.abs(-5));
lib.close();
```

`'c'` と `'m'` はC standard libraryとmath library用のportable aliasです。自作libraryなどでは `suffix` を使うとplatformごとの `so` / `dylib` / `dll` suffixを扱えます。

## Type

argumentとreturn valueにはC ABI上の型を指定します。

主なalias:

- `i8` / `u8` ～ `i64` / `u64`
- `int`, `long`, `char`, `size_t` など
- `f32`, `f64`
- `ptr`
- `string`
- `buffer`
- `bool_u8`, `bool_u32`
- `void`

`returns` の既定は `void`、`args` の既定は空配列です。

## String / Buffer

`string` はNUL終端 `char *` とJS stringの間を自動変換します。raw memoryを渡す場合は `Uint8Array` を `buffer` argumentとして渡せます。

`bufferToString()` と `stringToBuffer()` もあります。

variadic C functionでは `fixed` に固定argument数を指定します。同じsymbolを異なるarityで使うなら、JS側で別名を付けて複数bindingを作れます。

## Symbol名とoptional symbol

entryの `name` を使うと、C symbol名とは異なるJS property名でbindできます。

`optional: true` を付けたsymbolがlibraryに存在しない場合、`dlopen()` 全体を失敗させず、そのpropertyだけ省略できます。platform差やlibrary version差を吸収する際に使えます。

## Global variable

functionではなくdata symbolを扱う場合、`args` / `returns` の代わりに `type` を指定します。結果はaddressを指すPointerで、`deref()` して値を取得できます。

## Struct

`defineStruct()` はfield定義からC struct layoutを作ります。

```js
import { defineStruct } from 'tjs:ffi';

const Point = defineStruct([
  ['x', 'i32'],
  ['y', 'i32']
]);

const bytes = Point.pack({ x: 3, y: 4 });
console.log(Point.unpack(bytes));
console.log(Point.size, Point.align);
```

layoutとpaddingはlibffiがplatform ABIに従って決定します。`describe()` でfield offsetを確認できます。

Structはfunction argument / return valueとしてby-valueで使うことも、packed bufferやtyped pointerを使ってby-referenceで渡すこともできます。

### Nested struct / array / enum

field typeとして別の `defineStruct()` を使えばnested structをinline配置できます。pointer-to-arrayとlength fieldの組み合わせ、counted string、`defineEnum()`、固定長array、固定長stringなども扱えます。

field optionではdefault value、optional field、validationや変換、特定platformだけ存在するfield、`lengthOf`、nested structをpointer化する指定などを表現できます。

C側にstructを埋めてもらう用途には `allocStruct()` があり、array field用bufferまで準備できます。大量のstructを扱う場合はlist向けのpack / unpack APIでallocation回数を減らせます。

## Callback

`JSCallback` を使うとJavaScript functionをC function pointerとして渡せます。

Callback objectはC側から呼ばれる可能性がある間ずっと生存させる必要があります。GCされるとnative側にdangling function pointerが残るため注意が必要です。

native libraryがexportするfunction addressを別のC APIへ渡す用途では、symbolをPointerとしてbindしてaddressを利用できます。

## Pointer

`tjs:ffi` にはraw native addressを表すpointerと、型情報を持つtyped `Pointer` があります。`Pointer.createRef(type, value)` で値をmemoryへ置き、そのaddressをCへ渡すこともできます。

`deref()` は1段、`derefAll()` は複数levelのindirectionを辿ります。

## Native memoryのzero-copy view

native memoryを `Uint8Array` / `ArrayBuffer` としてcopyせず参照するAPIもあります。このviewのlifetimeはruntimeが追跡しません。native memoryがfree・reallocされた後にviewを触ると未定義動作になるため、所有権とlifetimeを自分で管理する必要があります。

native memoryを解放した後はexternal ArrayBufferの `detach()` でviewを無効化できます。native pointer自体はWorkerへstructured cloneできないため、必要ならaddressの `bigint` を渡して受信側でpointerを再構成します。ただしlibrary自体がthread-safeとは限りません。

## C prototypeからbindingを作る

`dlopenCProto()` はC declarationをparseし、functionやtypeをまとめてbindingできます。手でsymbol mapを書く代わりにheader相当のprototypeから生成したい場合に利用します。

## Error handling

C libraryが `errno` を使う場合、`errno()` でcodeを取得し、`strerror()` でmessageへ変換できます。

## Close

`dlopen()` / `dlopenCProto()` の結果には `close()` があり、library handleを解放します。`Symbol.dispose` にも対応しているため `using` でscope-based cleanupが可能です。

参照: txiki.js `website/docs/guides/ffi.md`
