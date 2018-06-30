## blog availability monitoring via blackbox_exporter

@threetreeslight

---

## 今日取り組むこと

やる
- [done] [prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter) を利用したblogの死活監視

できれば
- [WIP] prometheus alert managerの設定
- GKEで稼働するコンテナのlogを自前のfluentdに集約させ、bigquery や papertrailにfunoutする

---

## blackbox expoter

https://github.com/prometheus/blackbox_exporter
> The blackbox exporter allows blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP and ICMP.

めちゃめちゃ便利やん

---

## prometheusと同じpodにblackbox_exporterを配置

特にblackbox expoter defaultの設定でいい感じにできているのでconfigはいじらない。

```diff
      containers:
+      - image: prom/blackbox-exporter:latest
+        imagePullPolicy: Always
+        name: blackbox-exporter
+        resources: {}
```

---

## prometheusからscriping

prometheus.yaml

```diff
+- job_name: 'blackbox'
+  metrics_path: /probe
+  params:
+    module: [http_2xx]  # Look for a HTTP 200 response.
+  static_configs:
+    - targets:
+      - https://threetreeslight.com
+      - https://grafana.threetreeslight.com/login
+  relabel_configs:
+    - source_labels: [__address__]
+      target_label: __param_target
+    - source_labels: [__param_target]
+      target_label: instance
+    - target_label: __address__
+      replacement: localhost:9115
```

---

## 見てみる

DEMO

---

## めちゃかんたんやな！！

---

## ここからalertが地獄

---

# Prometheus alerting

Prometheusのalartは２つのパートに分かれている

- alert ruleは alert managerに alertを送信する
- alert managerは
  - silencing, inhibition, aggregation をしたり
  - その通知を行う

---

## alert manager?

えっまた新しいcontainer立てなきゃだめですね。

---

## とりあえずalertmangerの挙動をlocalで把握する

---

## Alert manager

```yaml
  alertmanager:
    image: prom/alertmanager
    volumes:
      - ./prometheus/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    environment:
      WEBHOOK_URL: $SLACK_WEBHOOK_URL
    ports:
      - 9093:9093
      - 9094:9094
      - 9095:9095
```

---

## alartmanager file

```
global:
  slack_api_url: xxxx

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: slack

receivers:
- name: slack
  slack_configs:
  - channel: '#blog'
```

---

## ちなみに作者は環境変数は嫌いなので許さんスタイル

こんなポストを投げつける

http://movingfast.io/articles/environment-variables-considered-harmful/
> Environment Variables Considered Harmful for Your Secrets

---

## わかる。わかるんだがさ、、、、

---

## 立ち上がったのに投げる

```
curl -XPOST -d'[{"labels":{"alertname":"mistely alert","serverity":"critical","instance":"example"}}]' http://localhost:9093/api/v1/alerts
curl -XPOST -d'[{"labels":{"alertname":"mistely alert","serverity":"critical","instance":"example"}}]' http://localhost:9093/api/v1/alerts
curl -XPOST -d'[{"labels":{"alertname":"mistely alert","serverity":"critical","instance":"example"}}]' http://localhost:9093/api/v1/alerts
```

---

## みてみる

DEMO

---

## 今のつまりどころ

configmapを使ってalertmanagerをk8s上に展開したら、、、crashloop???why??


