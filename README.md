# maguroskey-manifest

misskeyを立ち上げるための設定を記載したリポジトリです

## VaultにSecretを登録する
1. DBで使うユーザ用のパスワードとユーザ名を作成する
```bash
$ kubectl -n vault exec -it vault-0 -- sh
$ vault kv put kv/maguroskey/db-misskey-user username=misskey password=<password>
$ vault kv put kv/maguroskey/db-super-user username=postgres password=<password>
```

2. Misskeyのコンフィグファイルを作成する
```bash
$ vi config
$ vault kv put kv/maguroskey/config default.yml="$(cat config)"
```

3. バックアップ用のキーを登録する
```bash
$ vault kv put kv/maguroskey/backup-key ACCESS_KEY_ID=<access key> ACCESS_SECRET_KEY=<secret key>
```

ポリシーとロールを作成する
```bash
$ vault policy write read-maguroskey -  << EOF
path "kv/data/maguroskey/*" {
    capabilities = ["read"]
}
EOF

$ vault write auth/kubernetes/role/read-maguroskey \
bound_service_account_names=default \
bound_service_account_namespaces=maguroskey \
policies=read-maguroskey
```

## アプリケーションを作成する

1. `apps`以下を`apply`する
```bash
$ kubectl apply -f apps
```

2. データベースの準備が完了してからmisskeyを起動する
```bash
$ kubectl -n maguroskey get clusters.postgresql.cnpg.io 
$ argocd app sync argocd/maguroskey-misskey 
```