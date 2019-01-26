## Cant change service typt from Nodeport to ClusterIP

@threetreeslight

on shinjuku mokumoku programming #32

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

VP of Engineering at [Repro](https://repro.io)

実態はEventおじさん😇

---

## 今日やること

- servicegraphの導入
- jaegerの導入

---

## できたこと

- 🙅♀️servicegraphの導入
- 🙅jaegerの導入

---

## `Service` の理解が甘い

Istioの各種custom resourceとkubeneretes resourceの理解が常にいまいち

（毎日触っていないと知識がホントダメですね）

---

## なので

serviceとistio trafficについてひたすら触ったりdocument呼んだりして

## 時間が終わりました😇

---

# そんな中でのtips

---

## service typeのnodeport -> clusteripに変える

```diff
spec:
-  type: NodePort
+  type: ClusterIP
  selector:
    app: blog
  ports:
  - name: blog
    protocol: TCP
    port: 8080
    targetPort: 8080
```

---

## んん？🤔

applyすると

> may not be used when `type` is 'ClusterIP'

---

## もちろんググる

ありますよね

> Unable to change service from type=NodePort to type=ClusterIP with kubectl #221
>
> https://github.com/kubernetes/kubectl/issues/221

---

## 既知の問題とのこと😇

> This is caused by a known issue: mergeKey in ports is portNumber (both are 53) cannot uniquely identify an entry in this case.
> `kubectl replace` can be the workaround.

再作成するworkaroundがオススメということなので

---

## んん？🤔

```sh
% kubectl replace -f ./kubernetes/app/service.yaml
The Service "blog" is invalid: spec.clusterIP: Invalid value: "": field is immutable
```

> spec.clusterIP: Invalid value: "": field is immutable

immutable field?

---

## 似たようなことやらかしている人がいるんでしょうね😇

> Note: it requires specifying the clusterIP field to be the same IP as the allocated ClusterIP of this service

---

## Fix `clusterIP` & Apply

Kubenetesにて割り当てられたk8s IPを追記

```diff
spec:
  type: NodePort
+ clusterIP: 10.59.240.13
  selector:
    app: blog
  ports:
  - name: blog
    protocol: TCP
    port: 8080
    targetPort: 8080
```

---

## Change `ClusterIP` -> Apply

```diff
spec:
-  type: NodePort
  clusterIP: 10.59.240.13
  selector:
    app: blog
  ports:
  - name: blog
    protocol: TCP
    port: 8080
    targetPort: 8080
```

---

## 今後の決意

### Programming a hour every day

どんなに忙しくても毎日1hはプログラミング
