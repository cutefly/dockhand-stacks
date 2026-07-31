# Hashicorp Vault

```
$ docker compose up -d

$ docker exec -it vault-server bash
/ # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
/ # vault operator init -key-shares=1 -key-threshold=1

```

## Integrate with keycloak

> https://medium.com/@sauravkumarsct/integrate-keycloak-as-oidc-jwt-provider-with-hashicorp-vault-ae9ebcf8e335

```bash
vault secrets enable -path=secret kv
vault secrets tune -description="key/value store" secret

vault auth enable oidc
vault read auth/oidc/config

# default-policy
vault policy write default-policy - <<EOF
path "/secret/*" {
    capabilities = ["read", "list"]
}
EOF

# secret-manager
vault policy write secret-manager - <<EOF
path "/secret/*" {
    capabilities = ["read", "update", "list"]
}
EOF

# secret-admin
vault policy write secret-admin - <<EOF
path "/secret/*" {
    capabilities = ["create", "read", "update", "delete", "list"]
}
EOF

vault write auth/oidc/role/default \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="sub" \
policies=default-policy

vault read auth/oidc/role/default

vault write auth/oidc/role/manager \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="sub" \
policies=secret-manager

vault read auth/oidc/role/manager

vault write auth/oidc/role/admin \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="sub" \
policies=secret-admin

vault read auth/oidc/role/admin
```

```
vault login -method=oidc role=default
```

### group mapping

```
vault write auth/oidc/role/default \
bound_audiences="vault" \
allowed_redirect_uris="<https://vault.club012.com/ui/vault/auth/oidc/oidc/callback>" \
allowed_redirect_uris="<http://localhost:8250/oidc/callback>" \
user_claim="sub" \
groups_claim="groups" \
policies=default-policy

vault read auth/oidc/role/default

vault write auth/oidc/role/manager \
bound_audiences="vault" \
allowed_redirect_uris="<https://vault.club012.com/ui/vault/auth/oidc/oidc/callback>" \
allowed_redirect_uris="<http://localhost:8250/oidc/callback>" \
user_claim="sub" \
policies=default-policy

vault read auth/oidc/role/manager

vault write auth/oidc/role/admin \
bound_audiences="vault" \
allowed_redirect_uris="<https://vault.club012.com/ui/vault/auth/oidc/oidc/callback>" \
allowed_redirect_uris="<http://localhost:8250/oidc/callback>" \
user_claim="sub" \
policies=default-policy

vault read auth/oidc/role/admin
```
