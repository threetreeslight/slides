### How to probe prometheus & grafana
### What is helm

@threetreeslight

on shinjuku mokumoku programming #20

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

Event Organizer おじさん 😇

---

## 今日やること

- blog監視のgrafanaにおいて外形監視が落ちるを解決する
- Istioのchartを基にprometheusとgrafanaの設定見直す

---

## できたこと

- 🙆‍♀️blog監視のgrafanaにおいて外形監視が落ちるを解決する
- 🙆‍♀️Istioのchartを基にprometheusとgrafanaの設定見直す
- あわせて
  - helm chartの構成理解
  - istioの正常（多分）稼働

---

## 迷ったこと

1. すでにPrometheus, Grafanaでcluster監視している場合、Istioに同梱されているchartは使わないよね？
1. istioは個別のnamespaceに分けたほうがつかやすかったりするのか？
1. helmを複数のservice account, 複数のclusterで使う場合どうするのか？

---

### せっかくなので

# いくつかtips

### ご紹介

---

# healthcheck
# Prometheus
# Grafana

---
## Prometheus helth check

CNCF graduagted projectのprometheus、health check endpoint実装されてた

https://github.com/prometheus/prometheus/blob/47a673c3a0d80397f715105ab6755db2aa6217e1/web/web.go#L303

```go
	router.Get("/-/healthy", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		fmt.Fprintf(w, "Prometheus is Healthy.\n")
	})
	router.Get("/-/ready", readyf(func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		fmt.Fprintf(w, "Prometheus is Ready.\n")
	}))
```

---

### 思ったより?

揉めずにサクッとはいっていた模様。`/-/ready` が先にあったからかな？

[Add `/-/healthy` and `/-/ready` endpoints #2831](https://github.com/prometheus/prometheus/issues/2831)

---

### Prometheus Probeこんな感じ

シンプルになった 😇

```yaml
livenessProbe:
  httpGet:
    path: /-/healthy
    port: 9090
readinessProbe:
  httpGet:
    path: /-/ready
    port: 9090
```

---

## Grafana health check

[Grafana 4.3](http://docs.grafana.org/guides/whats-new-in-v4-3/#health-check-endpoint)で `/api/health` endpointが提供されていた。

https://github.com/grafana/grafana/blob/e78c1b4abc7eda7a065e390dc04b7a0c0435268c/pkg/api/http_server.go#L250

```go
func (hs *HTTPServer) healthHandler(ctx *macaron.Context) {
	notHeadOrGet := ctx.Req.Method != http.MethodGet && ctx.Req.Method != http.MethodHead
	if notHeadOrGet || ctx.Req.URL.Path != "/api/health" {
		return
	}

	data := simplejson.New()
	data.Set("database", "ok")
	data.Set("version", setting.BuildVersion)
	data.Set("commit", setting.BuildCommit)
```

---

### ぼちぼちコメントが

それなりにこまっていたということだろうか？ぼちぼちコメントが有る。

grafanaのiconが帰ってくるかどうかでwork aroundしているひともいるぐらい。

アクセスすると認証前だったらlogin画面に飛ばされたり、そもそもそのログイン画面がちょいと重かったりするから欲しい気持ちめっちゃわかる。

[Monitoring Grafana #3302](https://github.com/grafana/grafana/issues/3302)

---

### Grafana Probeこんな感じ

シンプルになった 😇

```yaml
readinessProbe:
  httpGet:
    path: /api/health
    port: 3000
```

---

## health checkの仕組みが提供されているとよいよね

---

# helm chart

---

## What is helm

helm ( https://helm.sh/ ) とは、CNCF ( https://www.cncf.io/ ) でhostingされているkubernetes上のpackage manager。

---

## stop the copy-and-paste madness.

この表現がなされるほどのyaml wall 🙀

1. Helmは単純にkubernetesのresourceをGo templatingしているだけ
1. localにchartをおいて複数clusterに展開することもできるので便利だったりする

細かい話はblogにあげていく

---

# Tips終わり

---

# 頑張っていく 💪
