# ドメインを付けて展示を公開する

希望するドメイン・公開するアプリを NOC に連絡してください。**DNS レコードの追加は NOC が Terraform で管理します。** DNS だけでは公開できないため、HTTPS の証明書・Gateway の入口・アプリへの経路も一緒に設定します。

通常の Web アプリは、運営の公開 Gateway → 自分の namespace の `ClusterIP` Service → Pod の順に接続します。アプリごとのグローバル IP は不要です。

## 参加者が用意するもの

自分のディレクトリに Deployment と Service を置いてください。次は `app: web` の Pod が HTTP の 8080 番で待ち受けている場合の `service.yaml` です。Deployment 側の label と待受ポートを合わせます。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

この例の Service のポートは **80**、Pod のポートは **8080** です。NOC には Service のポートを伝えてください。アプリは Pod 内で `0.0.0.0:8080` など外部から接続できるアドレスで待ち受けます。

## NOC への依頼テンプレート

次を Discord の相談スレッドなどで送ってください。以下は記入例で、ドメインの払い出しを確約するものではありません。

```text
出展者名: example
namespace: exhibit-example
希望ホスト名: demo.jisaba.sh
Service 名 / ポート: web / 80
バックエンドのプロトコル: HTTP
公開したいパス: / 以下すべて
公開範囲: 来場者を含めインターネット公開
アプリのログイン: なし（展示用データのみ）
特記事項: WebSocket / SSE / 大きいアップロードの有無
マニフェストの場所: example/service.yaml
希望する公開期間: 展示期間中
```

独自ドメインを使いたい場合は、そのドメインの DNS を誰が変更できるかも伝えてください。DNS を参加者側で管理している場合は、NOC が確認した接続先を参加者側で設定します。管理画面などログインが必要なものは、その旨も先に相談してください。パスワードやトークンは依頼文やリポジトリに書かないでください。

## NOC が設定する経路の例

以下は **運営側で管理する HTTPRoute の説明用サンプル**です。通常の出展者には HTTPRoute の作成権限がないので、自分の同期ディレクトリには追加しないでください。NOC が実際の namespace・ホスト名に合わせ、HTTPS listener と証明書を用意してから反映します。

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web
  namespace: exhibit-example
spec:
  parentRefs:
    - name: public
      namespace: envoy-gateway
      sectionName: https-demo
  hostnames:
    - demo.jisaba.sh
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: web
          port: 80
```

`https-demo` は NOC が追加する listener の名前の例です。HTTPRoute の `hostnames`、Gateway listener のホスト名、証明書、DNS を同じ公開先に合わせます。バックエンドの Service は HTTPRoute と同じ namespace に置きます。

## toka の rproxy 展示・TCP/UDP を公開する場合

専用 Gateway / LoadBalancer を使う展示は、割り当てられた IP と Gateway、編集してよい Route を NOC から案内します。一般の展示には `LoadBalancer` や Gateway の作成権限はありません。toka の具体的な構成・反映状況は [toka の案内](../toka/README.md)を確認してください。

専用 Gateway に新しいドメインを追加するときも、DNS と TLS 証明書は NOC に依頼してください。Route のホスト名を変えただけでは DNS・証明書は増えません。

TCP/UDP の公開は、上のテンプレートに **TCP または UDP、外部ポート、Service のポート**を記載します。DNS の A/AAAA レコードはポート番号を指定しないため、クライアント側で公開ポートを指定します。通常の Cloudflare HTTP プロキシを任意の TCP/UDP 通信に使うことはできません。DNS の向き先と公開方式は NOC が確認します。

## 公開できたか確認する

NOC から反映完了の案内を受けたら、実際のホスト名で確認します。

```bash
curl -I https://demo.jisaba.sh/
```

DNS が引けるだけでは完了ではありません。ブラウザでも証明書エラーなくアプリが表示されること、必要なログインや WebSocket などが動くことを確認してください。困った場合は URL・発生時刻・エラーを NOC に伝えてください。

参考: [Gateway API の HTTP routing](https://gateway-api.sigs.k8s.io/guides/user-guides/http-routing/)
