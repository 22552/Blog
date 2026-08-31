---
title: "txiki.jsとは"
free: true
---

# txiki.jsとは

txiki.js は、小さく保たれた JavaScript ランタイムです。JavaScript エンジンには QuickJS-ng、OSとの橋渡しには libuv を使い、ブラウザで見慣れた Web API と、サーバー側で必要になる低レベルAPIの両方を扱えます。

Node.js の小型版というより、**Web標準寄りのJavaScriptをそのまま小さなランタイムで動かす**という感覚のほうが近いです。

代表的には次のようなものが使えます。

- `fetch`、`WebSocket`、`Crypto`、`setTimeout` などの Web API
- TCP / UDP / Unix socket
- HTTP / WebSocket サーバー
- ファイル操作、子プロセス、シグナル
- `tjs:sqlite`、`tjs:ffi`、`tjs:path` などの標準ライブラリ
- WASI / WebAssembly
- 単一実行ファイル化
- TPK App Packages

## この本について

この本は txiki.js 公式ドキュメントの逐語訳ではなく、公式ドキュメントと実装を参照しつつ、日本語で使い始めやすい順番に再構成した**非公式ガイド**です。

特に次の点を重点的に扱います。

1. txiki.js を動かす
2. ES Modules と HTTP import を使う
3. `tjs:` 標準ライブラリを使う
4. HTTPサーバーを立てる
5. 複数ファイルのアプリをTPKで1バイナリにまとめる

Node.js 前提の説明はできるだけ避け、txiki.js 自体の設計が見えるようにします。

## 公式情報

- 公式サイト: https://txikijs.org/
- ソースコード: https://github.com/saghul/txiki.js

txiki.js は活発に更新されており、特に TPK など実験的な機能は仕様が変わる可能性があります。本書でも upstream の変更に合わせて内容を更新していく予定です。
