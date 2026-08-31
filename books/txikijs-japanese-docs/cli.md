---
title: "CLIリファレンス"
free: true
---

# CLIリファレンス

`txiki.js` の入口は `tjs` コマンドです。スクリプト実行、式評価、HTTPサーバー、テスト、bundle、単一実行ファイル化、TPKアプリ操作、REPLをここから扱います。

```bash
tjs [options] [subcommand] [args]
```

## グローバルオプション

- `-v`, `--version`: バージョン表示
- `-h`, `--help`: ヘルプ表示
- `--memory-limit LIMIT`: JavaScriptランタイムのメモリ上限
- `--stack-size SIZE`: JavaScriptスタック上限。既定は1024KB
- `--wasm-stack-size SIZE`: WebAssemblyスタック上限。既定は512KB
- `--tls-ca FILE`: 組み込みCAの代わりに使うPEM CA bundle

## 主なサブコマンド

### `tjs run`

JavaScriptを実行します。

```bash
tjs run app.js
```

Import Mapを使う場合は `--import-map FILE` を指定できます。対象が `.wasm` の場合はWASIランナーとして動作します。

### `tjs eval`

```bash
tjs eval "console.log(1 + 1)"
```

コマンドライン上の式をそのまま評価します。

### `tjs serve`

```bash
tjs serve app.js
```

`fetch` ハンドラをdefault exportするmoduleをHTTP/HTTPSアプリとして起動します。

### `tjs test`

```bash
tjs test tests/
```

ディレクトリ内の `test-*.js` をそれぞれ別の `tjs run` 子プロセスとして実行します。`VERBOSE_TESTS`、`TJS_TEST_TIMEOUT`、`TJS_TEST_CONCURRENCY` で挙動を調整できます。

### `tjs bundle`

```bash
tjs bundle app.js bundle.js
```

esbuildを使って依存moduleをまとめます。TypeScript入力も扱えます。

### `tjs compile`

```bash
tjs compile bundle.js
```

単一JavaScriptをtxiki.jsランタイム込みのstandalone executableにします。

### `tjs app`

TPK App Packages向けです。

```bash
tjs app init
tjs app pack
tjs app compile
```

## REPL

TTY上で単に `tjs` を起動するとREPLになります。`.help`、`.time`、`.strict`、`.depth N`、`.hidden`、`.color`、`.dark`、`.light`、`.clear`、`.clear-history`、`.load FILE`、`.quit` といったdirectiveがあります。

履歴は通常 `$TJS_HOME/history.db` に保存されます。SQLiteなしでbuildした場合は履歴保存が無効になります。

## stdinからの実行

stdinがTTYでない場合は入力全体をJavaScriptとして評価できます。

```bash
echo "console.log('hi')" | tjs
tjs < script.js
```

## WASI

```bash
tjs run module.wasm arg1 arg2
```

`.wasm` を指定すると `wasi_snapshot_preview1` として実行し、引数や終了コードもWASI側へ引き継ぎます。細かく制御したい場合は `tjs:wasi` を使います。

## 環境変数

- `TJS_HOME`: キャッシュ・Cookie jar・REPL履歴などの保存先
- `TJS_CA_BUNDLE`: TLS CA bundle
- `SSL_CERT_FILE`: CA bundleのfallback

CAの優先順位は `--tls-ca` → `TJS_CA_BUNDLE` → `SSL_CERT_FILE` → 組み込みMozilla CAです。

`fetch()` は `Alt-Svc` によりHTTP/3が案内されたoriginで、次の新規接続からQUICを試します。利用できないネットワークではHTTP/1.1やHTTP/2へ自動fallbackします。HTTP系APIは標準proxy環境変数にも対応します。

参照: txiki.js `website/docs/cli.md`
