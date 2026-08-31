---
title: "インストールと最初の実行"
free: true
---

# インストールと最初の実行

まずは `tjs` コマンドが動く状態を作ります。

公式READMEでは、ソースからビルドする最小手順として次の流れが案内されています。

```bash
git clone --recursive https://github.com/saghul/txiki.js --shallow-submodules
cd txiki.js
make
./build/tjs
```

環境ごとの詳しいビルド方法は公式の Building ドキュメントを参照してください。

https://txikijs.org/docs/building

## REPLを触る

`./build/tjs` のようにファイル名を指定せず起動すると、対話環境を使えます。

```js
> 1 + 2
3
```

Web API も試せます。

```js
> crypto.randomUUID()
```

## ファイルを実行する

`hello.js` を作ります。

```js
console.log('Hello from txiki.js!');
console.log(navigator.userAgent);
```

実行します。

```bash
tjs run hello.js
```

txiki.js の JavaScript ファイルは基本的に ES Module として扱われます。このため、今後の章でも `import` / `export` を前提に進めます。

## fetchしてみる

ブラウザに近い `fetch` API をそのまま使えます。

```js
const response = await fetch('https://example.com/');
console.log(response.status);
console.log(await response.text());
```

トップレベル `await` が使えるため、短いスクリプトなら非同期処理のためだけに関数で包む必要はありません。

## 覚えておくコマンド

```text
tjs run FILE       JavaScriptを実行
tjs eval EXPR      式を評価
tjs serve FILE     HTTPアプリを起動
tjs test DIR       テストを実行
tjs bundle ...     esbuildでbundle
tjs compile ...    単一JSを実行ファイル化
tjs app ...        TPKアプリを操作
```

このうち `serve` と `app` は後の章で詳しく触れます。
