## Node Label OR Taints

@threetreeslight

on shinjuku mokumoku programming #31

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

実態はEventおじさん 😇

---

## 今日やること

- [ ] node poolごとの配置戦略を考える
- [ ] 潤沢なリソースを持つnode poolにpilotを展開するようにする
- [ ] istio resourceを叩き込んで調整

---

## できたこと

- [x] node poolごとの配置戦略を考える
- [x] 潤沢なリソースを持つnode poolにpilotを展開するようにする
- [x] istio resourceを叩き込んで調整

---

### 全体的にすんなり？

---

#### でもなくめっちゃ悩んだ結果
# 雑にやった

---

## 今回やりたかったことを再掲

---

## Pilot podの要求リソースがでかーい

```yaml
  # Resources for a small pilot install
  resources:
    requests:
      cpu: 500m
      memory: 2048Mi
```

---

## 現行のnode pool構成

- resource pool: small
  - g1-small (1 vCPU, 1.7 GB memory) * 4 instance

---

## メモリ絶対足りない

- resourceの大きいnode poolを作る
- そこにpilotが配置されうようにする

---

## それ用のnode poolを作る

- node pool: small
  - g1-small (1 vCPU, 1.7 GB memory) * 2
- node pool: medium
  - n1-standard-1 (1 vCPU, 3.75 GB memory) * 1

---

## とはいえ

node pool mediumにいっぱい配置されたら死ぬよね

---

# Label OR Taints
# Preferred OR Requried

---

## Label OR Taints

- label
  - 各種サービスと同じようにnodeにlabelをつけ、合致する場所に展開
  - e.g. disk=ssd
- Taints
  - nodeに印をつけ、node側では許容するtaintsを許可する
  - e.g. env=production

---

## Preferred OR Requried

- 必ず守らせるのか、そうでないのか
- もちろん条件を組み合わせることもできる

---

## 悩むのはLabel OR Taints

resourceの大きさで配置をかけるのどうしったらよいの？

labelっぽいし、Tainstsっぽいし、、、GPUほど特殊でもないし、、、うーーーん

---

## labelを眺める

```sh
% kubectl describe nodes gke-blog-cluster-small-c43c7079-0tgd
Name:               gke-blog-cluster-small-c43c7079-0tgd
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/fluentd-ds-ready=true
                    beta.kubernetes.io/instance-type=g1-small
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-nodepool=small
                    cloud.google.com/gke-os-distribution=cos
                    failure-domain.beta.kubernetes.io/region=us-west1
                    failure-domain.beta.kubernetes.io/zone=us-west1-a
                    kubernetes.io/hostname=gke-blog-cluster-small-c43c7079-0tgd
Annotations:        container.googleapis.com/instance_id=5472602061798695923
                    node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
```

---

## とりま

node poolでマッチさせればいっか

```yaml
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: cloud.google.com/gke-nodepool
                operator: In
                values:
                - small

```

---

## 今後

- Istioの設定がうまくできているのか確かめるための可視化どうするか？ServiceGraphを使うか？
- jaegerを使った分散トレーシング
- prometheusでistioのalert設定

