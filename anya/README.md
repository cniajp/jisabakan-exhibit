# anya の展示アプリ

このディレクトリの Kubernetes マニフェストを、namespace `exhibit-anya` に同期します。

- [Omni ログイン・kubectl 接続手順](../docs/omni-login.md)
- 素の YAML はこのディレクトリ直下に置き、`metadata.namespace` は省略するか `exhibit-anya` を指定してください。
- `main` への反映後、ArgoCD が同期します。削除したマニフェストのリソースも削除されます。
- パスワード、トークン、kubeconfig はコミットしないでください。

現在は案内のみです。アプリの Deployment / Service などを追加してください。
