# jisabakan-exhibit

自鯖缶の Kubernetes クラスタ `jisabakan` に、出展者のアプリを載せるためのリポジトリです。

ディレクトリ `<名前>/` の中身が、その出展者の namespace `exhibit-<名前>` に入ります。`main` に push すると、数分で ArgoCD が反映します。

```
<名前>/          → namespace exhibit-<名前>
  deployment.yaml
  service.yaml
  ...
```

- ディレクトリは運営が出展者ごとに用意します。自分の名前のディレクトリだけを触ってください
- `main` は保護されています。変更は PR にしてください。そのディレクトリのコードオーナー（`.github/CODEOWNERS`。出展者本人と運営）のうち、作者以外の承認でマージできます。1 人で出展している人の PR は運営が承認します
- 素の YAML なら、ディレクトリの直下のファイルだけを入れます（サブディレクトリは見ない）。`kustomization.yaml` があれば Kustomize、`Chart.yaml` があれば Helm として描画します
- **ディレクトリから消したリソースは、クラスタからも消えます。** kubectl で直したリソースも、ここの内容に戻ります
- **このリポジトリは public です。** パスワードやトークンなどの値は置かず、kubectl で Secret を作ってください
- 入れられるのは namespace の中のリソースのうち、Pod・Deployment・StatefulSet・Job・Service（ClusterIP）・ConfigMap・Secret・PVC・NetworkPolicy などです。Ingress・HTTPRoute・Gateway・Certificate や、`LoadBalancer`・`NodePort` の Service は入りません。Web で見せるときは運営に相談してください

`trial/` は、運営が動作を確かめるためのサンプルです。

使い方の詳しい説明（kubectl のつなぎ方、マニフェストの書き方、できること・できないこと）は、運営から渡す出展者向けのガイドを見てください。
