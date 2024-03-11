# maguroskey-manifest

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