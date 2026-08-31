---
title: "Versioning"
free: true
---

# Versioning

txiki.jsはCalendar Versioning（CalVer）を使い、versionは `YY.MM.MICRO` 形式です。

- `YY`: release年の下2桁
- `MM`: release月。zero paddingなし
- `MICRO`: その月内のpatch番号。0から開始

例えば `24.6.1` は2024年6月の2回目のreleaseを表します。

## Version確認

CLI:

```bash
tjs --version
```

runtime内:

```js
console.log(tjs.version);
console.log(tjs.engine.versions.quickjs);
console.log(tjs.engine.versions.uv);
```

`tjs.engine.versions` ではtxiki.js本体だけでなく、bundleされているQuickJS-ng、libuv、libwebsockets、WAMR、SQLite、mimallocなど主要componentのversionも確認できます。

## Stability

txiki.jsは最新ECMAScriptとWinterTC compatibilityを目標にしています。core APIはreleaseをまたいでstableであることを意図していますが、明示的にExperimentalとされているsurfaceは変更される可能性があります。

代表例がTPK App Packagesと `tjs app` commandsです。Experimental機能を使うapplicationではversion pinningとrelease note確認が重要です。

## Release / Changelog

releaseはGitHub Releasesで公開され、実質的なchangelogとして利用できます。CalVerなのでversion番号を見るだけで「いつ頃のreleaseか」が分かるのも特徴です。

参照: txiki.js `website/docs/versioning.md`
