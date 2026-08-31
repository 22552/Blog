---
title: "ビルド方法"
free: true
---

# ビルド方法

txiki.jsをソースからbuildするにはCMakeが必要です。また、repositoryは複数のgit submoduleに依存しているため、recursive cloneするか `git submodule update --init` を実行しておきます。

## GNU/Linux

Debian / Ubuntu系では、少なくともbuild toolchain、CMake、libffiのdevelopment packageが必要です。公式例ではUbuntu 24.04以降を前提にしています。

```bash
sudo apt install build-essential cmake libffi-dev
```

Amazon Linux 2023ではGCC 14系、CMake、libffi、libatomicなどを導入し、必要に応じて `cc` / `c++` をGCC 14へ向けます。

## macOS

```bash
brew install cmake
```

その後、通常はrepositoryを取得して `make` します。

```bash
git clone --recursive https://github.com/saghul/txiki.js --shallow-submodules
cd txiki.js
make
./build/tjs
```

## Windows

Visual Studio 2022またはBuild Toolsと、vcpkgを使用します。vcpkgをbootstrapしたあと `libffi` をinstallし、Developer PowerShellなどからCMakeを実行します。

```powershell
cmake -B build -DCMAKE_TOOLCHAIN_FILE=path/to/vcpkg/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release
```

生成物は通常 `build\Release\tjs.exe` です。テストは次のように実行できます。

```powershell
.\build\Release\tjs.exe test tests/
```

## 機能を削って小さくする

一部のsubsystemはbuild時に無効化できます。

| CMake設定 | 既定 | 無効化したとき |
|---|---:|---|
| `BUILD_WITH_WASM` | ON | WebAssembly / WASIを外す |
| `BUILD_WITH_SQLITE` | ON | `tjs:sqlite` を外す |

WASMを外すと `WebAssembly` globalと `tjs:wasi` が使えません。SQLiteを外すと `tjs:sqlite` がなくなり、`localStorage` は永続DBではなくmemory上へfallbackします。利用可能なfeatureは `tjs.engine.features` で確認できます。

```js
console.log(tjs.engine.features.wasm);
console.log(tjs.engine.features.sqlite);
```

## サイズ最適化

機能を削らずにbinaryを小さくするため、symbol strip、LTO、dead-code stripping、`MinSizeRel` なども利用できます。

- `BUILD_WITH_STRIP=ON`
- `BUILD_WITH_LTO=ON`
- `BUILD_WITH_GC_SECTIONS=ON`
- `CMAKE_BUILD_TYPE=MinSizeRel`

`MinSizeRel` はサイズ優先の最適化になるため、compute-heavyなJavaScriptでは通常のReleaseより遅くなる可能性があります。

## runtime内部のJavaScriptを変更した場合

runtimeへ埋め込まれるJSを変更した場合は、bundleし直してC sourceを再生成します。

```bash
npm install
make js
```

参照: txiki.js `website/docs/building.md`
