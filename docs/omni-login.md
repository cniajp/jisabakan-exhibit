# Omni にログインして kubectl を使う

自鯖缶の Kubernetes は、[Omni](https://omni.jisaba.sh/)を通して利用します。ログインは **Google または GitHub** を選べます。メールのワンタイムコード認証はありません。Cloudflare Access への登録は、このログインには不要です。

## 先に運営へ伝えるもの

- ログインに使うメールアドレス
- 出展者名（運営から案内された namespace は `exhibit-<名前>`）

**GitHub でログインして構いませんが、Omni の登録は GitHub ユーザー名ではなく、Auth0 が返すメールアドレスで照合します。** GitHub の `@username`、Git の `user.email`、コミット用の `noreply` アドレスを伝えるだけでは登録できません。

GitHub の Settings → Emails で、使うアカウントの確認済みメールを確認して運営へ伝えてください。プロフィールのメールを公開する必要はありません。複数のメールを登録していてログインできない場合は、Auth0 に返っているメールを運営と確認します。GitHub にそのメールを追加しただけで、Omni の登録と必ず一致するとは限りません。

運営が Omni のユーザー・権限・namespace を作った後で、以下へ進んでください。リポジトリへの push 権限は別の設定です。

## 1. ツールを入れる

必要なのは `omnictl`、`kubectl`、`kubectl oidc-login` です。macOS で Homebrew を使う例:

```bash
brew tap siderolabs/tap
brew install siderolabs/tap/sidero-tools
brew install kubectl

omnictl --version
kubectl version --client
kubectl oidc-login --help
```

`sidero-tools` には `omnictl` と `oidc-login` が含まれます。他の OS や手動インストールは [Omni 公式のインストール手順](https://docs.siderolabs.com/omni/getting-started/install-and-configure-omnictl)を参照してください。`oidc-login` は [kubelogin の手順](https://github.com/int128/kubelogin#setup)でもインストールできます。

## 2. 自鯖缶用の接続設定を作る

以下の `you@example.com` は、**運営に登録してもらったメール**に置き換えてください。自鯖缶専用のファイルを作り、他のクラスタの設定と分けます。

```bash
mkdir -p "$HOME/.config/jisabakan"
export OMNICONFIG="$HOME/.config/jisabakan/omniconfig.yaml"

# 初回だけ実行。すでに設定済みなら作り直さない
if [ ! -e "$OMNICONFIG" ]; then
  omnictl config new "$OMNICONFIG" \
    --url https://omni.jisaba.sh/ \
    --identity you@example.com
fi

omnictl config contexts
```

接続先が `https://omni.jisaba.sh/` になっていることを確認します。メールを変更するときは、先に運営へ連絡して登録を変更してから `omnictl config identity <新しいメール>` を実行してください。

## 3. ブラウザでログインして kubeconfig を取得する

```bash
omnictl kubeconfig "$HOME/.config/jisabakan/kubeconfig" \
  --cluster jisabakan --merge=false
```

ブラウザの認証画面で Google または GitHub を選び、登録メールに対応するアカウントでログインします。確認画面が出たら内容を確認して進み、ターミナルに戻ってコマンドの完了を待ってください。kubectl の初回実行でもログインを求められることがあります。

ブラウザが自動で開かない場合は、同じコマンドの先頭に `BROWSER=echo` を付け、表示された URL を自分のブラウザで開きます。既存の kubeconfig がある場合は、エラー内容を確認し、無条件に `--force` で上書きしないでください。

出展者は CLI 用の権限（global role `None` とクラスタの ACL）なので、**Omni の Web 管理画面には入れません**。Web 画面に入れないことだけでは、kubectl の権限が無いとは限りません。この手順は管理画面から設定をダウンロードせず進められます。

## 4. 自分の namespace を確認する

`exhibit-<名前>` を運営から案内されたものに置き換えてください（例: `exhibit-anya`、`exhibit-toka`）。

```bash
export KUBECONFIG="$HOME/.config/jisabakan/kubeconfig"
kubectl config set-context --current --namespace='exhibit-<名前>'
kubectl auth whoami
kubectl get pods
kubectl get resourcequota
```

`kubectl auth whoami` のグループに自分の `exhibit:<名前>` があり、`system:masters` が無いことを確認します。Pod がまだ無ければ `No resources found` は正常です。

自分の namespace 内ではアプリを操作できます。クラスタ全体の `kubectl get nodes` / `kubectl get namespaces` や、別の出展者・`kube-system` の操作はできません。

新しいターミナルを開いたら、上記の `OMNICONFIG` と `KUBECONFIG` の `export` を再実行してください。ログインの期限が切れたら、表示に従ってブラウザで再認証します。

## 困ったとき

- `oidc-login` が見つからない: 手順1のインストールと `PATH` を確認する。
- ログイン後に未登録・権限エラー: Google/GitHub のアカウント、登録メール、`OMNICONFIG` の identity を確認する。メールが合わなければ運営へ連絡する。
- 自分の namespace で `Forbidden`: `KUBECONFIG` と `kubectl auth whoami` の結果を確認し、運営に伝える。管理者の kubeconfig を借りて回避しない。
- 他のクラスタにつながる: `KUBECONFIG` が上記の専用ファイルを指しているか確認する。

問い合わせでは、実行したコマンド・エラー文・namespace・使用したログイン方法を伝えてください。パスワード、認証コード、ログイン URL、kubeconfig やトークンは public な Issue / PR に貼らないでください。

## 確認状況

2026-11-14 時点で、運営によるユーザー・ACL・namespace の反映と権限検査は確認済みです。接続設定を作るコマンドもローカルで確認しています。出展者本人のブラウザ認証から kubectl 操作までの一連の確認はこれからなので、初回に詰まった場合は運営へ連絡してください。

参考: [Omni のユーザー管理](https://docs.siderolabs.com/omni/security-and-authentication/manage-user-in-omni)、[ACL と None ユーザーの扱い](https://docs.siderolabs.com/omni/security-and-authentication/how-to-manage-acls)。
