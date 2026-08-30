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

ブロックを置いて、micro:bitを接続して、LEDを光らせたり、ボタンや傾きなどを入力として使ったりできる、あれです。

普通に使っていると、ScratchがそのままBluetoothでmicro:bitと通信しているように見えます。

しかし、Scratchとmicro:bitの間には **Scratch Link** という別のアプリケーションが入っています。

今回はこの境界を調べて、Scratchから見た「Scratch Link」をPythonで実装してみました。

作ったものが **22552/link-copy** です。

結論から言うと、これは単なる通信ログ取得用のモックではありません。

**micro:bit拡張については実際にScratchから接続して動かせます。**

一方で、現状の実装はmicro:bit向けの通信を前提にかなり直接書いているので、Scratch Link全体を置き換える汎用実装にはなっていません。

この記事では、どうやってScratch Linkとの境界を見つけ、どこまで実装すればScratchを動かせるのか、そして今のlink-copyに何が足りないのかを書きます。

## Scratchはmicro:bitと直接話していない

まず構造をかなり単純化すると、こうなります。

```text
Scratch
  ↓ WebSocket / JSON-RPC 2.0
Scratch Link
  ↓ Bluetooth Low Energy
micro:bit
```

Scratchのmicro:bit拡張から見ると、Bluetooth APIを直接叩いているわけではありません。

ローカルで動いているScratch Linkへ命令を送り、Scratch LinkがBLE側の処理を担当します。

つまり、観察する対象を二つのレイヤーに分けられます。

```text
[ Scratch側 ]
Scratch
  ↓
Scratch Linkへ送るRPC

[ ハードウェア側 ]
Scratch Link
  ↓
BLE / GATT
micro:bit
```

最初は「micro:bitとのBluetooth通信をそのまま再現しないといけないのでは」と考えそうになります。

でも、Scratchから見えているのはBluetoothそのものではありません。

Scratchにとって重要なのは、Scratch Linkらしい相手がローカルに存在し、期待したRPCへ期待した形式で返事をすることです。

そこで発想を逆にしました。

> Scratch Linkを観察するのではなく、自分がScratch Linkになればいいのでは？

これがlink-copyの出発点です。

## Scratch Linkとの境界はかなり扱いやすい

link-copyはPythonの`websockets`を使い、デフォルトで次のアドレスを待ち受けます。

```text
ws://127.0.0.1:20111
```

そこでScratchから来るJSON-RPC 2.0のメッセージを受け取ります。

JSON-RPCなので、基本形はかなり読みやすいです。

例えば接続要求なら、概念的には次のようなJSONが来ます。

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

こちらは同じ`id`を付けて返します。

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": true
}
```

HTTPのように一回のrequest / responseで全部が終わるわけではなく、WebSocketなのでScratch Link側から通知を投げることもできます。

この「双方向」という点が、後で仮想センサーを実装するときに重要になります。

## まずはScratchが何を要求するか全部見る

link-copyでは受信したメッセージをそのまま表示します。

```python
print("<< RECV:", msg)
```

そしてレスポンスも表示します。

```python
print(">> SEND:", resp)
```

これだけでもかなり便利です。

Scratchでmicro:bit拡張を開き、接続操作をすると、ScratchがScratch Linkへ何を要求しているのかが順番に見えます。

分からないメソッドが来たら、そのメソッドを実装する。

次の要求が来たら、また実装する。

という形で、Scratchが期待する最小限の境界を少しずつ埋めていけます。

パケットキャプチャだけだと「このデータは何のために送られたのか」を外側から推測する必要があります。

しかし自分がRPCサーバーになると、Scratchが呼んだメソッド名、パラメータ、要求の順序がそのまま見えます。

この差はかなり大きいです。

## 存在しないmicro:bitをScratchに見せる

本物のScratch Linkなら、デバイス探索を要求されたときにBluetoothデバイスを探します。

link-copyでは、実際のBLEスキャンをする代わりにPython上へ仮想Peripheralを作ります。

構造はだいたい次のようになっています。

```text
VirtualPeripheral
├── peripheral_id
├── name
├── rssi
├── connected
└── gatt
```

`peripheral_id`はUUIDから生成します。

```python
peripheral_id: str = field(default_factory=lambda: str(uuid.uuid4()))
```

名前は現在、

```text
microbit-virtual
```

です。

Scratchから探索要求が来たら、この仮想デバイスを「発見したデバイス」として返します。

さらに`didDiscoverPeripheral`の通知も送ります。

```text
Scratch
   ↓ discover
link-copy
   ↓ didDiscoverPeripheral
Scratch
```

Scratchから見ると、Scratch Linkがmicro:bitらしいPeripheralを発見した状態になります。

ここでは本物のBluetoothアダプタも、本物のmicro:bitも必要ありません。

Bluetooth自体をエミュレートしているのではなく、**ScratchがBluetoothを見るために使っている窓口をエミュレートしています。**

## なぜこれでハードウェアまで偽物にできるのか

Scratch側が見ている世界を考えると分かりやすいです。

Scratch自身は、目の前に本物のBLEデバイスが存在することを確認しているわけではありません。

Scratch Linkから、

```text
このPeripheralを見つけた
このServiceがある
このCharacteristicを読める
このCharacteristicへ書ける
値が変化した
```

という情報を受け取っています。

なら、その情報をScratch Link互換の相手が生成すればよいことになります。

```text
本来

Scratch
  ↓
Scratch Link
  ↓
BLEデバイス

link-copy

Scratch
  ↓
link-copy
  ↓
Python上の状態
```

Scratchから見たインターフェースが一致していれば、その下が本物のBLEでもPythonの辞書でも構わないわけです。

これはWeb APIのモックにかなり近い発想ですが、結果としてハードウェアまで仮想化できるので見た目以上に面白いです。

## GATTも最低限だけPython上に作る

BLEではGATTのServiceとCharacteristicを使ってデータをやり取りします。

link-copyでは、micro:bit拡張を動かすために必要なGATT情報をPython上に持たせています。

現在の実装では主に次のUUIDを扱っています。

```text
Service
0000f005-0000-1000-8000-00805f9b34fb

Characteristic
5261da01-fa7e-42ab-850b-7c80220097cc
5261da02-fa7e-42ab-850b-7c80220097cc
```

内部では、かなり単純に、

```text
service UUID
  └── characteristic UUID
        └── bytes
```

という形で値を保持しています。

つまりBLEスタックそのものをPythonで再実装しているわけではありません。

Scratch LinkのRPCから見える範囲だけを表現しています。

この割り切りのおかげで実装をかなり小さくできます。

## `discover`から`connect`まで

仮想Peripheralを用意したら、次は接続です。

`connect`ではScratchから渡された`peripheralId`が、自分が作った仮想PeripheralのIDと一致するかを確認します。

一致すれば、

```python
self.peripheral.connected = True
```

にするだけです。

物理的な接続処理はありません。

それでもScratchとの境界では「接続済み」という状態を作れます。

この時点で重要なのは、実世界のBLE接続処理を再現することではなく、**Scratchが次の処理へ進むために必要な状態遷移を再現すること**です。

## `getServices`でmicro:bitらしさを返す

Scratchが接続後にServiceを確認したら、仮想GATTに登録してあるServiceを返します。

```python
def list_services(self) -> List[str]:
    return list(self.gatt.keys())
```

これも非常に単純です。

Scratch Linkの下に本当にGATTサーバーがあるわけではありません。

ただしScratchから見ると、必要なServiceが存在していることになります。

ここまで来ると、

```text
探索
↓
発見
↓
接続
↓
Service確認
```

という、実機を接続したときの流れを仮想デバイスだけで進められます。

## `write`を取るとScratchからmicro:bitへ何を送っているか分かる

個人的に一番楽しいのが`write`です。

Scratchがmicro:bitへ出力しようとすると、そのデータはScratch Linkへの`write`としてlink-copyに届きます。

link-copyでは`message`がBase64ならデコードします。

```python
if enc == "base64":
    data = _b64d(str(msg))
else:
    data = str(msg).encode("utf-8")
```

これでScratchがハードウェア側へ送ろうとしていた生のバイト列をPythonから直接見られます。

本物のScratch Linkを使う場合、ここはScratch LinkからBLEへ流れていく部分です。

link-copyでは自分がScratch Linkなので、その場で好きに観察できます。

## `0x81`と`0x82`を少し覗く

現在の実装では、受け取ったデータの先頭バイトによって簡単な表示を行っています。

先頭が`0x81`なら、その後ろをUTF-8として表示します。

```python
if data[0] == 129:
    print("UTF8:", data[1:].decode("utf-8", errors="replace"))
```

先頭が`0x82`なら、後続バイトを5bit表現にして逆順に表示します。

```python
elif data[0] == 130:
    for i in data[1:]:
        print(str(format(i, "05b"))[::-1])
```

ここはScratch Link公式のJSON-RPCプロトコルではありません。

**Scratch Linkの上を流れているmicro:bit向けデータを、自分が調べやすくするために追加した観察処理**です。

レイヤーとしてはこうです。

```text
JSON-RPC
└── write
    └── Base64
        └── bytes
            └── micro:bit向けの意味
```

最初はJSONしか見えていなかったのに、Scratch Linkを再現すると、その中のBase64、その中のバイト列、その中の命令形式、と一段ずつ降りていけます。

## 逆方向の通信も必要になる

Scratchからmicro:bitへ送るだけなら`write`だけでもかなり遊べます。

しかしmicro:bit拡張には、

- 傾き
- ボタン
- タッチ
- ジェスチャー

など、ハードウェアからScratchへ戻ってくる入力もあります。

そこでlink-copyは逆方向の状態も持っています。

```python
sensors = {
    "tilt_x": 0,
    "tilt_y": 0,
    "button_a": 0,
    "button_b": 0,
    "touch": [0, 0, 0],
    "gesture": 0,
}
```

これらは実際のセンサーではなく、Python上にある変数です。

## Notificationで仮想センサーをScratchへ送る

BLEではCharacteristicの値が変化したことをNotificationとして受け取れます。

link-copyでも同じ役割の通知をScratchへ返します。

現在はセンサー状態から10byteのデータを組み立てています。

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

そしてWebSocket上でScratchへ送ります。

構造としては、

```text
Python上の変数
  ↓
10byteへエンコード
  ↓
Base64
  ↓
JSON-RPC notification
  ↓
Scratch
```

となります。

実機なら、

```text
micro:bitのセンサー
  ↓ BLE
Scratch Link
  ↓ WebSocket
Scratch
```

だった部分です。

この一番下だけをPythonへ差し替えています。

## `setSensor`という勝手なRPCも生やした

仮想センサーを外部から変更できないと不便なので、link-copyには`setSensor`という独自RPCもあります。

これはScratch Linkの標準APIではありません。

link-copy専用です。

例えば、概念的には、

```json
{
  "method": "setSensor",
  "params": {
    "name": "button_a",
    "value": 1
  }
}
```

のように状態を変えられるようにしています。

すると、

```text
外部プログラム
   ↓ setSensor
link-copy
   ↓ notification
Scratch
```

という経路を作れます。

つまり本物のmicro:bitだけでなく、別のプログラムの入力をmicro:bitの入力としてScratchへ見せることもできます。

例えば理屈の上では、キーボード入力、ゲームパッド、Web APIの値、別のセンサーなどを一度link-copyへ入れて、Scratch側ではmicro:bitとして扱うこともできます。

## 実際に動くところまで来た

ここは重要なので明確に書いておきます。

link-copyは「こうすれば動きそう」という途中のモックではありません。

**現在の実装でScratchのmicro:bit拡張を実際に動かせます。**

仮想Peripheralを発見させ、接続させ、Scratchから出力を受け取り、入力側もNotificationで返せます。

なので、問題点は「まだmicro:bitでも動かない」ことではありません。

問題は別のところにあります。

## 最大の問題はmicro:bit専用実装であること

今のlink-copyはScratch Linkという名前の仕組みを再現していますが、中身はかなりmicro:bitに寄っています。

例えば、

- 仮想デバイス名が`microbit-virtual`
- micro:bit向けService UUIDを前提にしている
- micro:bit向けCharacteristicを最初から登録している
- センサー状態が`tilt_x`、`button_a`などmicro:bit前提
- Notificationの10byte構造もmicro:bit前提
- `write`の観察処理もmicro:bit側のデータ形式を意識している

という状態です。

つまり、

```text
Scratch Link互換レイヤー
        ＋
micro:bit実装
```

がまだ一つのファイルの中に強く結びついています。

micro:bit用途では動きますが、このままでは別のScratchハードウェア拡張を持ってきて即座に動くわけではありません。

ここが現在の一番大きな設計上の制限です。

## 本当はScratch Link部分とデバイス部分を分離したい

理想的には、構造を次のようにしたいです。

```text
Scratch
  ↓
Generic Scratch Link server
  ↓
Device backend
  ├── micro:bit
  ├── LEGO系デバイス
  ├── 独自BLEデバイス
  └── Virtual device
```

Scratch Link側は、

- WebSocket
- JSON-RPC
- discover
- connect
- disconnect
- getServices
- read
- write
- startNotifications
- stopNotifications

など、デバイスに依存しない処理だけを担当します。

そしてmicro:bit固有の、

- Service UUID
- Characteristic UUID
- センサーのエンコード
- Scratchから届く出力のデコード

は別のbackendへ分離するのが自然です。

そうすればlink-copyは、

```text
micro:bitの偽物
```

から、

```text
Scratch Linkへ好きな仮想デバイスを接続するための基盤
```

へ近づきます。

## 今のコードは意図的に小さい

現状は一つのPythonファイルに、

```text
WebSocket server
JSON-RPC dispatcher
VirtualPeripheral
仮想GATT
micro:bit backend
sensor state
notification loop
CLI
```

が全部入っています。

設計として綺麗に分割されたライブラリではありません。

ただ、今回の目的ではこれがかなり役に立ちました。

プロトコルを追いかけている途中では、抽象化を先に作りすぎるより、来た要求をその場で処理して次の通信を見る方が速いからです。

まず動く境界を見つける。

その後、何が共通部分で何がmicro:bit固有なのかが分かってから分離する。

今回のようなプロトコル調査では、その順番の方が自然でした。

## パケットを盗み見るより、相手そのものになる

今回一番面白かったのはこの考え方です。

通信を調べるとなると、最初に思いつくのはWiresharkなどでパケットを取る方法です。

もちろんそれも有効です。

でも、アプリケーション同士の境界がWebSocket + JSON-RPCのように高水準なら、もっと直接的な方法があります。

```text
普通の観察

Scratch
  ↓
Scratch Link
  ↓
自分は横から見る
```

ではなく、

```text
今回

Scratch
  ↓
自分
```

にしてしまう方法です。

すると、

- 何のメソッドが呼ばれたか
- パラメータは何か
- どの順番で呼ばれるか
- どの返答をすると次へ進むか
- どこでNotificationが必要か

を一つずつ確認できます。

分からない要求が来ればエラーになります。

そのエラー自体が「次に実装すべきもの」を教えてくれます。

かなり原始的ですが、ブラックボックスの境界を理解する方法として強いです。

## Scratch Linkを丸ごとコピーする必要はなかった

最初に「Scratch Link互換実装」と聞くと、Scratch Linkの全機能を再実装しないといけないように感じます。

しかし、micro:bit拡張を動かすという目的だけなら必要ありませんでした。

Scratchが実際に通る経路だけを実装すればよいからです。

```text
必要なRPCだけ実装
↓
必要なServiceだけ返す
↓
必要なCharacteristicだけ扱う
↓
必要なNotificationだけ送る
```

という形で進められます。

これは互換実装を作るときの面白い点だと思います。

仕様全体を最初から理解しなくても、対象クライアントが使う最小部分から動かせます。

ただし、その最小実装をそのまま「Scratch Link全体と互換」と呼ぶのは違います。

link-copyはまさにその状態で、**micro:bitについては使えるが、Scratch Link一般としてはまだ狭い**です。

## これを汎用化するともっと面白そう

今後やるなら、最も大きい変更はmicro:bit部分の分離です。

例えばデバイスごとに、

```text
backend
├── scan_results()
├── connect()
├── disconnect()
├── services()
├── read()
├── write()
└── notify()
```

のような共通インターフェースを持たせれば、Scratch Link側のJSON-RPC処理を使い回せます。

その上で、

```text
MicrobitBackend
VirtualBleBackend
OtherDeviceBackend
```

のように差し替えられるようにすれば、今の実験をかなり一般化できます。

さらに、本物のBLEへ流すbackendと仮想backendを同じScratch Link互換サーバーへ接続できれば、

```text
Scratch
  ↓
link-copy
  ├── 実BLE
  ├── 仮想BLE
  └── ログ・変換
```

のようなプロキシ兼実験環境にもできます。

単なる偽物のmicro:bitより、こちらの方が最終的には面白そうです。

## まとめ

今回やったことをまとめると、

1. Scratchとmicro:bitの間にScratch Linkがいることを見る
2. Scratch Linkとの通信境界をWebSocket + JSON-RPCとして扱う
3. `127.0.0.1:20111`で互換サーバーを待ち受ける
4. 仮想PeripheralをScratchへ発見させる
5. 仮想GATT Service / Characteristicを返す
6. `connect`、`read`、`write`、Notificationなど必要な処理を実装する
7. Scratchからmicro:bitへ送られるデータを直接観察する
8. Python上の仮想センサーをScratchへ入力として返す
9. 実際にmicro:bit拡張を動かす

という流れでした。

Scratchの画面だけを見ると、

```text
micro:bit拡張
```

という一つの機能に見えます。

しかし中身は、

```text
Scratch VM
↓
WebSocket
↓
JSON-RPC
↓
Scratch Link
↓
BLE
↓
GATT
↓
micro:bit
```

という複数のレイヤーに分かれています。

そして、一つ境界が見つかれば、その境界を自分の実装へ差し替えられます。

今回の場合は、ハードウェアを直接エミュレートするのではなく、**ハードウェアとの仲介役であるScratch Linkをエミュレートすることで、その下にあるmicro:bitまで仮想化しました。**

しかもmicro:bitについては実際に動いています。

次の課題は動作させることではなく、**micro:bitにベタ付けされた実装を剥がして、Scratch Linkとして汎用化すること**です。

コードは`22552/link-copy`に置いてあります。
