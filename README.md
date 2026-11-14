# jisabakan-exhibit

自鯖缶の Kubernetes クラスタ `jisabakan` に、出展者のアプリを載せるためのリポジトリです。

ドメインを付けて公開する場合は、[公開方法と NOC への依頼例](docs/publishing.md)を確認してください。DNS・TLS・公開経路は NOC が設定します。

まず [Omni のログイン・kubectl の接続手順](docs/omni-login.md)を確認してください。Google／GitHub でログインできます。

運営が Git 同期を有効にしたディレクトリ `<名前>/` の中身が、その出展者の namespace `exhibit-<名前>` に入ります。`main` に push すると、数分で ArgoCD が反映します。namespace の払い出しだけでは Git 同期は有効になりません。

```
<名前>/          → namespace exhibit-<名前>
  deployment.yaml
  service.yaml
  ...
```

- ディレクトリは運営が出展者ごとに用意します。自分の名前のディレクトリだけを触ってください
- 素の YAML なら、ディレクトリの直下のファイルだけを入れます（サブディレクトリは見ない）。`kustomization.yaml` があれば Kustomize、`Chart.yaml` があれば Helm として描画します
- **ディレクトリから消したリソースは、クラスタからも消えます。** kubectl で直したリソースも、ここの内容に戻ります
- **このリポジトリは public です。** パスワードやトークンなどの値は置かず、kubectl で Secret を作ってください
- 入れられるのは namespace の中のリソースのうち、Pod・Deployment・StatefulSet・Job・Service（ClusterIP）・ConfigMap・Secret・PVC・NetworkPolicy などです。Ingress・HTTPRoute・Gateway・Certificate や、`LoadBalancer`・`NodePort` の Service は入りません。Web で見せるときは運営に相談してください

`trial/` は、運営が動作を確かめるためのサンプルです。

使い方の詳しい説明（kubectl のつなぎ方、マニフェストの書き方、できること・できないこと）は、[出展者向けガイド](https://github.com/cniajp/jisabakan/blob/main/k8s/platform/exhibits/exhibitor-guide.md)を見てください。ログインについては、このリポジトリの [Omni 接続手順](docs/omni-login.md)を参照してください。
