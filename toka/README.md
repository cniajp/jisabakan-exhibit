# toka の rproxy 展示

namespace は `exhibit-toka`。このディレクトリ直下の YAML が `main` から ArgoCD で同期されます。

| ファイル | 編集するもの |
| --- | --- |
| `gateway.yaml` | Gateway `toka` の listener（公開ポート） |
| `http-route.yaml` | HTTP/HTTPS の転送先 |
| `tcp-route.yaml` | TCP 18080 の転送先 |
| `udp-route.yaml` | UDP 18081 の転送先 |
| `demo-web.yaml` | HTTP と TCP の転送先の nginx / ClusterIP Service |
| `demo-udp.yaml` | UDP の echo サーバー / ClusterIP Service |

## 構成と担当

```text
来場者 → 202.226.7.203 → LoadBalancer Service → rproxy → デモ用 Service → Pod
                            ↑                   ↑
                    Gateway の listener    HTTPRoute / TCPRoute / UDPRoute
```

rproxy controller が Gateway から **type: LoadBalancer の Service を自動生成**します。生成された Service や rproxy Deployment をこのディレクトリに複製しないでください。変更は Gateway / Route に書きます。

toka さんはこのディレクトリのアプリ・Gateway・Route を編集できます。運営が controller・CRD・権限・公開 IP・DNS・証明書を管理します。既存の Envoy Gateway とは別の GatewayClass `rproxy` を使用します。

- Gateway の名前 `toka`、GatewayClass `rproxy`、IP `202.226.7.203` は固定です。
- listener は 80・443・10000〜19999 番。Gateway の listener を追加し、対応する Route の `sectionName` を合わせると、そのポートを公開する構成になります。
- Route はこの Gateway に接続し、同じ namespace の Service だけを転送先にします。
- HTTPS は NOC が用意する `toka.jisaba.sh` 用の `toka-jisaba-sh-tls` を使います。追加ドメインは [NOC へ依頼](../docs/publishing.md)してください。
- 443 の listener は `toka.jisaba.sh` 専用です。80 と TCP/UDP のデモは IP でも試せます。
- Git から消したリソースはクラスタからも削除されます。秘密情報はコミットしないでください。

## 反映の確認

[Omni の接続手順](../docs/omni-login.md)で自分用 kubeconfig を設定した後:

```bash
kubectl -n exhibit-toka get gateways,httproutes,tcproutes,udproutes
kubectl -n exhibit-toka get pods,services
kubectl -n exhibit-toka describe gateway toka
```

Gateway の `Accepted` / `Programmed`、Route の `Accepted` / `ResolvedRefs` が True になっていることを確認します。マニフェストが Git にあるだけでは、controller・DNS などが反映済みとは限りません。

```bash
curl --fail http://202.226.7.203/
curl --fail http://202.226.7.203:18080/
# DNS と証明書の設定完了後
curl --fail https://toka.jisaba.sh/
```

UDP の確認例（Python 3）:

```bash
python3 - <<'PY'
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.settimeout(5)
s.sendto(b'toka-demo', ('202.226.7.203', 18081))
assert s.recv(1024) == b'toka-demo'
print('UDP echo OK')
PY
```

## UI

UI は運営側で別に用意します。公開 URL・ログイン方法は準備が完了してから案内します。Gateway 管理の設定は UI で変更せず、このディレクトリの YAML を編集してください。
