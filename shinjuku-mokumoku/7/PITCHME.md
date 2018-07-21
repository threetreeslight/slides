# Custermize node logging agent on GKE

shinjuku mokumoku programming #7

@threetreeslight on 2018-07-15

---

### 今日やりたいこと

blogを運用するGKE上にて

1. data lostせずnode logging agentを更新する方法を探る
1. node logging agentの配置
1. [papertrail](https://papertrailapp.com/) にもlogを飛ばすようにしておく


---

### 図にすると

```txt
nginx prometheus grafana
  |       |         |
  -------------------
          |
  node logging agent(fluentd)  <- ここらへん
      |              |
  stack driver   papertrail
      |
     ...
```

---

## data lost

node logging agentとして利用するfluentdを終了するときにはいくつか気をつけなければいけないことがある

---

### buffer lost

fluentd containerをSTOPすると

1. SIGTERMがcontainerに送られる
1. handleされ、container内で稼働するfluentdにSIGTERMが伝搬する
1. fluentdがmemory bufferのflushを試みる
1. **失敗してもretryしない**

---

### この問題を解決するために

受付を停止した上で、いずれかの方法をとる

1. bufferのflushが完了していることを確認して終了する
1. 適切に処理をresumeできる

see also: [Fluentd's Signal Handling](https://docs.fluentd.org/v0.12/articles/signals)

---

## forwarder data lost

直接データを送信している場合は、接続先のfluentdがdownしても問題ない状態を作らなければいけない

1. service discoveryを行い、生存しているlogging agentに自動で再接続されなければいけない
1. retryが適切になされなければいけない
1. その間、可能であれば処理が継続できなければい

---

### 解決するにはHA構成おすすめ

fluentdのHA構成にておくことで、送信元の問題の解決も比較的容易である。

![](https://docs.fluentd.org/images/fluentd_ha.png)

see also: [Fluentd High Availability Configuration](https://docs.fluentd.org/v1.0/articles/high-availability)

---

## On Kubernates

kubernatesでは以下の戦略が一般的。flushさえ問題なくできれば大丈夫そう

1. stdout, stderrに出力したデータをnodeに蓄積する
1. nodeに蓄積されたデータをnode logging agentが監視し、tailでデータを見る
1. tailではposがあるのでそこらへんは気にせず投げられる。

---

# Resumeは問題ないのか？

1. ここについても fluentd tail pluginを使っているので概ね問題ない

---

## remote_syslogを使ってpapertrailに流すの試す

---

### fluentd conf

```html
<source>
  @type tail
  @id input_tail
  format json
  time_key time
  path /var/log/sample/*
  pos_file /var/log/sample/sample.log.pos
  time_format %Y-%m-%dT%H:%M:%S.%N%Z
  tag sample
  read_from_head true
</source>

<filter **>
  @type stdout
</filter>

<match **>
  @type remote_syslog
  @id papertrail
  host logs7.papertrailapp.com
  port 45121
  severity info
  program sample

  protocol tcp
  timeout 20
  timeout_exception true
  tls true
  ca_file /etc/papertrail-bundle.pem
  keep_alive true
  keep_alive_cnt 9
</match>
```

---

### dockerfile

```dockerfile
FROM fluent/fluentd:v1.2.2

RUN apk --no-cache --update add \
    curl \
  && curl -sL -o /etc/papertrail-bundle.pem https://papertrailapp.com/tools/papertrail-bundle.pem \
  && fluent-gem install fluent-plugin-remote_syslog -v 1.0.0
```

---

### うまくいきました

![]()

---

## customize

---

### ここらへんの情報を使う

以下のrepoを使うこともできるが、すでにfluentd daemonsetが動いているので、設定を吐き出すことにする

- [GoogleCloudPlatform/k8s-stackdriver](https://github.com/GoogleCloudPlatform/k8s-stackdriver)
- [GoogleCloudPlatform/container-engine-customize-fluentd](https://github.com/GoogleCloudPlatform/container-engine-customize-fluentd)
- [kubernetes/kubernetes - fluentd-gcp](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/fluentd-gcp/README.md)

---

### 既存設定を書き出す

```sh
# daemonset
% kubectl get ds --namespace kube-system | grep fluentd | awk '{ print $1 }'
fluentd-gcp-v2.0.9

% kubectl get ds fluentd-gcp-v2.0.9 --namespace kube-system -o yaml > fluentd/fluentd-gcp-ds.yaml
fluentd-gcp-v2.0.9

# fluentd conf
% kubectl get cm  --namespace kube-system | grep fluentd | awk '{ print $1 }'
fluentd-gcp-config-v1.2.2

kubectl get cm fluentd-gcp-config-v1.2.2 --namespace kube-system -o yaml > fluentd/fluentd-gcp-configmap.yaml
```

---

### enable logging customize

```sh
gcloud beta container clusters update --logging-service=none blog-cluster
```

---

### Dockerfile

```dockerfile
FROM gcr.io/google-containers/fluentd-gcp:2.0.9

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    curl \
  && curl -sL -o /etc/papertrail-bundle.pem https://papertrailapp.com/tools/papertrail-bundle.pem \
  && fluent-gem install fluent-plugin-remote_syslog -v 1.0.0
```

---

### build and publish image script

```sh
timestamp=`date +%Y%m%d%H%M%S`

docker build -t threetreeslight/fluentd-gcp:$timestamp -f ./fluentd/Dockerfile .
docker tag threetreeslight/fluentd-gcp:$timestamp threetreeslight/fluentd-gcp:latest

echo "\nimage name is threetreeslight/fluentd-gcp:$timestamp\n"

docker push threetreeslight/fluentd-gcp:$timestamp
docker push threetreeslight/fluentd-gcp:latest
```

---

### daemonset

```diff
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd-gcp-v2.0.9
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - env:
        - name: FLUENTD_ARGS
          value: --no-supervisor -q
-        image: gcr.io/google-containers/fluentd-gcp:2.0.9
+        image: threetreeslight/fluentd-gcp:20180721153509
        imagePullPolicy: IfNotPresent
```

---

### configmap

```diff
    <match **>
+     @type copy
+
+     <store>
        @type google_cloud

        detect_json true
        enable_monitoring true
        monitoring_type prometheus
        detect_subservice false
        buffer_type file
        buffer_path /var/log/fluentd-buffers/kubernetes.system.buffer
        buffer_queue_full_action block
        buffer_chunk_limit 1M
        buffer_queue_limit 2
        flush_interval 5s
        max_retry_wait 30
        disable_retry_limit
        num_threads 2
+     </store>

+     <store>
+       @type remote_syslog
+       host <host>
+       port <port>
+       severity info
+       program ${tag[0]}
+
+       protocol tcp
+       timeout 20
+       timeout_exception true
+       tls true
+       ca_file /etc/papertrail-bundle.pem
+       keep_alive true
+       keep_alive_cnt 9
+     </store>
    </match>
```

---

## apply

CrashLoopBackOff :sob:

```
fluentd-gcp	Jul 21, 2018, 4:29:00 PM	2018-07-21 07:29:00 +0000 [error]: /usr/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
fluentd-gcp	Jul 21, 2018, 4:29:00 PM	2018-07-21 07:29:00 +0000 [error]: /usr/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
fluentd-gcp	Jul 21, 2018, 4:29:00 PM	2018-07-21 07:29:00 +0000 [error]: /var/lib/gems/2.1.0/gems/fluent-plugin-remote_syslog-1.0.0/lib/fluent/plugin/out_remote_syslog.rb:3:in `<top (required)>'
fluentd-gcp	Jul 21, 2018, 4:29:00 PM	2018-07-21 07:29:00 +0000 [error]: /var/lib/gems/2.1.0/gems/fluent-plugin-remote_syslog-1.0.0/lib/fluent/plugin/out_remote_syslog.rb:4:in `<module:Fluent>'
fluentd-gcp	Jul 21, 2018, 4:29:00 PM	2018-07-21 07:29:00 +0000 [error]: unexpected error error="Plugin is not a module"
```

---

## 標準がfluentd-0.12.40なのか、、、

古いよGKE。あげなくては

---
## ここまでしかできんかった 😇

