## Faillicies of kubernetes setting

@threetreeslight

on shinjuku mokumoku programming #18

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

Event Organizer おじさん 😇

---

# 今日やること

- 自blogをIstio Readyに
- その上でIstioをちゃんと設定し直す
- さらにJaegerでtracingする

---

# これしかできんかった 😇

> 自blogをIstio Readyに

---

# isitoを利用しやすくするために

- namespaceの整理
- 更新しやすいような構成に整理

---

ほぼ作り直しなので

# 改めてハマったところを出す

---

# kubernetes yaml

- yamlのvalue値に `-` を含む場合、 single OR double-quotationでくくらないと適用されないケースとそうでないケースが有った
- しかもerrorにはならない。その項はなかったこととしてapplyされる
- 例えばGKE ingressのannotaiton値は single OR double-quotationでくくらないとだめ

---

## GEK ingress

- 設定が適用されたとしても、不要なresourceは削除されない
- そのため以下のサービスは手動で調整する必要がある
  - forwarding rule
  - backend service

---

## service account

- namespace間で同名のservice account resourceがあるとどちらが使われるかわからない？

---

## namespace

- `istio` はns単位で `istio-injection: enabled` を制御できる、`istio-injection` したい管理単位でnamespaceをうまく分けるの大事
- namespaceを変更して適用したときの `--prune` optionは期待通りに動かなかった（やりかた悪いだけ？）
- なので粛々と `kubectl` を使ってリソースを削除

---

### 最終的には

![](/shinjuku-mokumoku/18/blog-system.png)

---

# 頑張っていくしか無い 😇
