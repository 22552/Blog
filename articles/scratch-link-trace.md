---
title: "Scratch Linkをトレースして、Scratchに偽物のmicro:bitを接続する"
emoji: "🛰️"
type: "tech"
topics:
  - "scratch"
  - "microbit"
  - "bluetooth"
  - "websocket"
  - "reverseengineering"
published: false
---

Scratchにはmicro:bit拡張があります。

ブロックを置いて、micro:bitを接続して、LEDを光らせたりボタンの状態を読んだりできる、あれです。

普通に使っているとScratchが直接Bluetoothでmicro:bitと通信しているように見えます。

しかし実際には、その間に **Scratch Link** というアプリケーションが入っています。

今回はこのScratch Linkとの通信を観察して、Scratchから見た「Scratch Linkっぽいもの」をPythonで作ってみました。

作ったものが **22552/link-copy** です。

## Scratchはmicro:bitと直接話していない

ざっくり構造を書くとこうなります。

```text
Scratch
  ↓ WebSocket / JSON-RPC 2.0
Scratch Link
  ↓ Bluetooth Low Energy
micro:bit
```

Scratch Linkは、Scratch 3.0からハードウェアへ接続するための仲介役です。

公式のScratch Linkリポジトリを見ると、Scratchとの通信にはWebSocketとJSON-RPC 2.0が使われています。

つまりScratch側から見ると、重要なのは最初からBluetoothではありません。

**ローカルで動いているScratch Linkに、決められたJSONを送ること**です。

ここで一つ思いつきました。

> Scratch Linkと同じように返事をするサーバーを立てたら、Scratchは本物のScratch Linkだと思うのでは？

これがlink-copyの始まりです。

## とりあえず20111番ポートで待つ

現在のlink-copyはPythonの`websockets`を使い、デフォルトでは次の場所で待ち受けます。

```text
ws://127.0.0.1:20111
```

Scratch Link側のプロトコルはJSON-RPC 2.0なので、Scratchからは例えば`discover`、`connect`、`read`、`write`などのメソッドが飛んできます。

受け取ったJSONを見て、同じ`id`を付けて結果を返せば、基本的なRPCとして会話できます。

イメージとしてはこうです。

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "connect",
  "params": {
    "peripheralId": "..."
  }
}
```

これに対して、成功したことにして返します。

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": true
}
```

もちろん実際には、接続前にデバイス探索などの流れがあります。

## 存在しないmicro:bitを生やす

本物のScratch Linkなら、`discover`されたときにBluetoothデバイスを探します。

link-copyではBluetoothを探す代わりに、Python上に仮想デバイスを一つ作ります。

```text
VirtualPeripheral
├── peripheralId
├── name
├── rssi
├── connected
└── GATT services
```

そしてScratchに対して、

```text
「microbit-virtualというデバイスを見つけました」
```

という通知を送ります。

Scratch側からすれば、Scratch Linkがデバイスを発見したように見えます。

ここがかなり面白いところです。

Bluetoothそのものをエミュレートしているわけではありません。

**ScratchとBluetoothの間にあるScratch Linkをエミュレートすることで、Bluetoothデバイスまで仮想化しています。**

## GATTっぽいものもPython上に作る

micro:bitとのBLE通信ではGATTのServiceとCharacteristicが使われます。

link-copyにも最低限の仮想GATTを持たせています。

現在のコードでは主に次のUUIDを扱います。

```text
Service
0000f005-0000-1000-8000-00805f9b34fb

Characteristic
5261da01-fa7e-42ab-850b-7c80220097cc
5261da02-fa7e-42ab-850b-7c80220097cc
```

Scratchから`write`されたデータは、この仮想Characteristicへ保存します。

逆に`read`やNotificationでは、Python側で作ったデータをScratchへ返せます。

つまり、実物のmicro:bitがいなくても、

```text
Scratch
  ↓
link-copy
  ↓
Python上の仮想GATT
```

という経路を作れます。

## Scratchから何が送られているのかを見る

Scratch Linkの代わりになれると、Scratchから送られるデータをそのまま観察できます。

link-copyでは`write`されたバイト列を少しだけデコードしています。

現在の実装では、先頭バイトが`0x81`なら後続をUTF-8として表示します。

```python
if data[0] == 129:
    print("UTF8:", data[1:].decode("utf-8", errors="replace"))
```

また、先頭が`0x82`なら、後続バイトを5bitの列として表示する処理を入れています。

```python
elif data[0] == 130:
    for i in data[1:]:
        print(str(format(i, "05b"))[::-1])
```

これは「Scratch LinkのJSON-RPC仕様」そのものではなく、**その上を流れてくるmicro:bit向けデータを観察するためにlink-copy側へ入れた処理**です。

Scratch Linkとの通信を再現したことで、そのさらに上のレイヤーにあるデータ形式も調べやすくなりました。

## 入力側も偽物にできる

出力を受け取るだけではなく、逆方向も試せます。

link-copyには仮想的なセンサー状態として、

```text
tilt_x
tilt_y
button_a
button_b
touch0
touch1
touch2
gesture
```

などを持たせています。

そしてNotificationを使って、その状態をScratch側へ送ります。

Scratch LinkのBLE通信ではCharacteristicの値が変わったことを通知できます。

link-copyでは一定間隔で10byteのデータを組み立てて、Scratchへ通知する処理を作りました。

```text
2byte tilt_x
2byte tilt_y
1byte button_a
1byte button_b
1byte touch0
1byte touch1
1byte touch2
1byte gesture
```

さらにデバッグ用として`setSensor`という独自RPCも追加しています。

これはScratch Link公式のAPIではなく、仮想センサーを書き換えるためにlink-copy側へ勝手に生やしたものです。

このように、

```text
現実のmicro:bit → Scratch
```

だけではなく、

```text
任意のプログラム → link-copy → Scratch
```

という経路も作れます。

## 「通信を盗み見る」より「相手そのものになる」

今回一番面白かったのはここでした。

通信を調べるとき、最初はパケットキャプチャのような方法を考えがちです。

もちろんそれも有効です。

しかしScratch Linkの場合、ScratchとScratch Linkの間には比較的読みやすいWebSocket + JSON-RPCという境界があります。

だったら、その境界に自分の実装を置けばいい。

```text
Scratch
   ↓
本物のScratch Linkを観察する
```

だけでなく、

```text
Scratch
   ↓
自分がScratch Linkになる
```

という調べ方ができます。

Scratchから届いた要求をログへ出し、分からないメソッドが来たら実装し、次へ進む。

これはプロトコルを調べる方法としてかなり分かりやすいです。

## まだ完全互換ではない

link-copyはScratch Linkの完全な再実装ではありません。

現在のコードでも、扱っているRPCやGATTの挙動は必要な部分だけです。

Scratchやmicro:bit側のバージョンによって通信内容が変わる可能性もあります。

特に、

- 接続時の細かな状態遷移
- Service / Characteristicの扱い
- Notificationの正確な形式
- micro:bit向けメッセージの全種類
- エラー時の挙動

などは、まだ詰める余地があります。

そのため「Scratch Linkクローン」というより、今のところは **Scratch Linkを観察するための小さな互換サーバー** と考えるのが近いです。

## まとめ

今回やったことをまとめると、

1. Scratchがmicro:bitと直接通信しているわけではないことを確認する
2. Scratch Linkとの境界がWebSocket + JSON-RPC 2.0だと分かる
3. 同じ場所で待ち受けるPythonサーバーを書く
4. 仮想micro:bitを`discover`させる
5. `read` / `write` / Notificationを実装する
6. Scratchから送られるデータを観察する

という流れでした。

普段は「micro:bit拡張」という一つの機能に見えますが、中身を分解すると、

```text
Scratch VM
WebSocket
JSON-RPC
Scratch Link
BLE
GATT
micro:bit
```

という複数のレイヤーが重なっています。

そしてレイヤーの境界が見つかると、そこだけを差し替えて遊べます。

**ハードウェアをエミュレートするのではなく、ハードウェアとの仲介役をエミュレートする。**

思ったより小さなPythonコードでScratchの裏側に入り込めたので、かなり面白い実験でした。

コードは`22552/link-copy`に置いてあります。
