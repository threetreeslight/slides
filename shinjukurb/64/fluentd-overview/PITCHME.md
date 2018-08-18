## fluentd Overview

@threetreeslight

on shinjuku.rb #64

---

## [fluentd](https://github.com/fluent/fluentd) とは？

一元的なロギングレイヤー( Unified Logging Layer )を提供する middleware

log aggregator, log shipper

![](shinjukurb/64/fluentd-architecture.png)

---

## ユースケース

1. アプリのログ集約
1. サービスログの監視
1. データ分析
1. データストアへの接続
1. ストリーム処理

---

### なぜUnifiedする必要があるのか？

> so much effort is wasted trying to make various backend systems understand log data.

ログデータを利用できるようにするためにバックエンド側のシステムでは多大な努力を支払ってきた

💪 😭

---

write complex regular expressions skill

is

# **a heroic skill**

---

### 欲しいもの

1. log producers と consumers で 共通なインターフェース
1. 水平方向にスケールし、リトライ・レジュームも可能
1. 新しいデータ入出力系統への対応がpluginでお手軽にできる


---

# fluentd<br>全てを満たす

---

## イベントの加工も簡単 😇

---

## Filters

> A Filter aims to behave like a rule to pass or reject an event.
> The Filter basically will accept or reject the Event based on its type and rule defined

いらないデータすてたり、データのマスキング(アプリケーションでやったほうが良い)とか容易

---

```sh
<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^logout$
  </exclude>
</filter>
```

---

## Labels

> to define new Routing sections that do not follow the top to bottom order

label使って特定環境のログデータの処理とか容易

---

```sh
<label @STAGING>
  <filter test.cycle>
    @type grep
    <exclude>
      key action
      pattern ^logout$
    </exclude>
  </filter>

  <match test.cycle>
    @type stdout
  </match>
</label>
```

---

## Buffers

> Output plugin in buffered mode stores received events into buffers first and write out buffers to a destination by meeting flush conditions.

bufferingしたりretryしてくれるのでアウトプット先への負荷コントロールなども容易

---

# Plugins

新しいデータ入出力への対応を容易にする

---

1. Input Plugin
1. Output Plugin
1. Filter Plugin
1. Parser Plugin
1. Formatter Plugin
1. Buffer Plugin
1. Storage Plugin

---

## Input Plugin

データソース用、ファイルを呼んだりloggerからforwardingされてきたりできる

- An input plugin typically creates a thread socket and a listen socket. 
- It can also be written to periodically pull data from data sources.

---

## Parser Plugin

インプットソースのデータ処理するやつ。filterとの使い分けが重要。

cannot parse the user’s custom data format (for example, a context-dependent grammar that can’t be parsed with a regular expression).

---

## Filter Plugin

labelやtagに基づいてデータ整形するやついろいろできる

1. 削ったり
1. イベントの追加したり
1. マスキングしたり

---

## Output Plugin

いろんな出力先に接続するやつ

---

## Formatter Plugin

Sometimes, the output format for an output plugin does not meet one’s needs. 

出力フォーマットを決定するやつ

---

## Buffer Plugin

memoryやfileによるbuffering storage

---

## Storage Plugin

pluginによる状態（接続先への転送量など）の統計情報を蓄積できたり

---

# いろいろできる

---

# もう少し内部を見てみる

---?image=shinjukurb/64/fluentd-v0.14-plugin-api-overview.png


---

## ここからは!

abickyさんのblogで内部構造やdata lost について見ていく!

https://abicky.net/2017/10/23/110103/

---

## ref

- [Fluentd Blog - Unified Logging Layer: Turning Data into Action](https://www.fluentd.org/blog/unified-logging-layer)
- [fluentd - Buffer Plugin Overview](https://docs.fluentd.org/v1.0/articles/buffer-plugin-overview)
- [あらびき日記 - fluentd の基礎知識](https://abicky.net/2017/10/23/110103/)
- [sonots:blog - fluentdでログが欠損する可能性を考える](http://blog.livedoor.jp/sonots/archives/44690980.html)

