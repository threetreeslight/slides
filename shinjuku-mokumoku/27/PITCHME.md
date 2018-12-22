## Calm down for troubleshooting

@threetreeslight

on shinjuku mokumoku programming #27

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=300&v=4)

VP of Engineering at [Repro](https://repro.io)

実態はEventおじさん 😇

---

## 今日やること

- GKE上で可動しているblackbox exporterが急に期待通り動かなくなってしまったの治す
- prometheus, grafanaのminor version up
- k8s上で稼働するblogのSSL証明書更新をcert-managerに移行

---

## できたこと

- 🙆‍GKE上で可動しているblackbox exporterが急に期待通り動かなくなってしまったの治す
- 🙆‍prometheus, grafanaのminor version up
- 🙅‍k8s上で稼働するblogのSSL証明書更新をcert-managerに移行

---

## About
## blackbox exporter task

---

## あれー？11月末に行った証明書更新以降、死活監視落ちてる？

---

## chromeで眺めてみると？

![](/shinjuku-mokumoku/27/browser-cert-1.png)

---

## よさそうやん

![](/shinjuku-mokumoku/27/browser-cert-2.png)

---

## 12月に相次ぐリリース

imageのversionをlatestとか雑なことしてたから死んだんじゃね？🤔

- prometheus
- alertmanager
- blackbox exporter

---

## 当時のimage versionに戻してもダメ😇

---

## 手元で `blackbox exporter` のprobe結果を見ると、、、

```sh
Logs for the probe:
msg="Beginning probe" probe=http timeout_seconds=9.5
msg="Resolving target address" preferred_ip_protocol=ip6
msg="Resolution with preferred IP protocol failed, attempting fallback protocol" fallback_protocol=ip4 err="address threetreeslight.com: no suitable address found"
msg="Resolved target address" ip=35.201.67.24
msg="Making HTTP request" url=http://[35.201.67.24]/healthy host=threetreeslight.com
msg="Received redirect" url=https://threetreeslight.com/healthy
msg="Error for HTTP request" err="Get https://threetreeslight.com/healthy: x509: certificate signed by unknown authority"
msg="Response timings for roundtrip" roundtrip=0 start=2018-12-22T03:40:57.8203383Z dnsDone=2018-12-22T03:40:57.8203383Z connectDone=2018-12-22T03:40:57.8209252Z gotConn=2018-12-22T03:40:57.8209672Z responseStart=2018-12-22T03:40:57.9258131Z end=2018-12-22T03:40:57.9259149Z
msg="Response timings for roundtrip" roundtrip=1 start=2018-12-22T03:40:57.9262533Z dnsDone=2018-12-22T03:40:57.9300509Z connectDone=2018-12-22T03:40:57.9371498Z gotConn=0001-01-01T00:00:00Z responseStart=0001-01-01T00:00:00Z end=2018-12-22T03:40:57.942842Z
msg="Probe failed" duration_seconds=0.1346038
```

---

### `x509: certificate signed by unknown authority`

# アーーーー

---

certificate、中間証明書いれてなかったやん😇

---

## まとめ

- chromeなどのbrowserは中間証明書がなくても結構いい感じに解釈する
- middlewareはそんなことはせん
- don't guess measure

---

## 次回以降

- cert-managerについての理解を誤っていたので、それをいい感じにまとめて発表したい
- それを踏まえてnginx-ingressは本当に必要なのか？ちゃんと練り込んでいく
- istioがスマートに稼働するにはまだ道程は長い

