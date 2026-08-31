---
title: "miseで使う"
free: true
---

# miseで使う

[mise](https://mise.jdx.dev/) は複数language / toolを扱えるversion managerです。txiki.jsのversionをproject単位で固定し、system-wide installなしで `tjs` をPATHへ入れられます。

現時点でtxiki.jsはmise registryの通常packageとしてではなく、GitHub Releasesから取得する `github` backendを使います。release binaryは主にmacOS arm64 / x86_64、Windows x86_64向けです。Linuxやその他Unixでrelease binaryがない場合はsource buildを使います。

## Projectへ追加

```bash
mise use "github:saghul/txiki.js[exe=tjs]"
```

これでlatest releaseをinstallし、projectに `mise.toml` が作られます。

```toml
[tools]
"github:saghul/txiki.js" = { version = "latest", exe = "tjs" }
```

`exe = "tjs"` はrelease archive内で公開する実行file名をmiseへ伝えます。`mise.toml` をcommitすればteamやCIでも同じ設定を共有できます。

## Version固定

```bash
mise use "github:saghul/txiki.js[exe=tjs]@26.6.0"
```

利用可能version:

```bash
mise ls-remote "github:saghul/txiki.js"
```

upgradeはversion specに従って `mise upgrade` するか `mise.toml` を編集します。

## 実行

```bash
mise exec -- tjs run app.js
mise exec -- tjs --version
```

shell integrationでmiseをactivateしていればproject directory内で直接 `tjs` を使えます。

## Project task

miseはtask runnerとしても使えます。

```toml
[tools]
"github:saghul/txiki.js" = { version = "26.6.0", exe = "tjs" }

[tasks.start]
run = "tjs run src/main.js"

[tasks.test]
run = "tjs test tests/"
description = "Run tests"
```

```bash
mise run start
mise run test
```

## CI

```bash
mise install
mise exec -- tjs run app.js
```

GitHub Actionsではmise Actionを使い、`mise.toml` からtoolをinstallして `tjs` をPATHへ追加できます。

## 古いtxiki.js

公式Docsでは26.5.0以前について、miseの旧 `ubi` backendを使う方法も案内されています。ただしubi backend自体がdeprecated扱いなので、新しいversionでは `github` backendを使う方がよいです。

参照: txiki.js `website/docs/guides/mise.md`
