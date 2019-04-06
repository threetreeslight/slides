### Have a nice firebase function day

@threetreeslight

on shinjuku mokumoku programming #39

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

VP of Engineering at [Repro](https://repro.io)

実態は人事オジサン😇

---

## 今日やること

- HackingHR!コミュニティ用blogをgatsbyで構築

---

# とみせかけて

---

## やったこと

shinjuku-mokumokuの準備作業は全部slash commandで実行するようにしていた😇

---

## command

- [x] `/prepare <num>`
  - slack channelの自動生成やプレゼンターの準備
- [x] `/presenters <num>`
  - 発表者の自動生成
- [ ] `/generateNextEvnet`
  - 次回イベント用dirを勝手に切り出す

あと環境のdockerizeとci deploy周り整備した

---

## 初めて知った👀

npm進化しているなぁ・・・

- npmに `prefix` optionがあって、project rootより深いネスト先 `package.json` を叩ける
  - e.g. `npm --prefix functions i`
- npm run にArgumentが受け取れるようになっていた
  - e.g. `npm run foo -- --super=hacker`
- `npx` なにこれ便利

---

## 悩ましかった 🤔

npm runのargumentに環境変数を与えると標準出力に評価結果が・・・

```sh
#!/bin/bash -eo pipefail
npm --prefix "functions" run deploy -- --token=$FIREBASE_DEPLOY_TOKEN

> shinjuku-mokumoku@1.0.0 deploy /home/circleci/project/functions
> firebase deploy --only functions -f "--token=評価済み"
```

---

## みんなどうしているの？

- npmのslientもquiet optionぜんぜん関係ないところなので、隠れない
- やっぱりshellなどwrapして制御するのが定石かしら？

---

# 久しぶりのjs楽しい😇


