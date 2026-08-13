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
vault auth tune -description="oidc based credentials" oidc

vault write auth/oidc/config \
         oidc_discovery_url="https://keycloak.club012.com/realms/devops" \
         oidc_client_id="vault" \
         oidc_client_secret="$AUTH0_CLIENT_SECRET" \
         default_role="default"
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
user_claim="preferred_username" \
policies=default-policy

vault read auth/oidc/role/default

vault write auth/oidc/role/manager \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="preferred_username" \
policies=secret-manager

vault read auth/oidc/role/manager

vault write auth/oidc/role/admin \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="preferred_username" \
policies=secret-admin

vault read auth/oidc/role/admin
```

```
vault login -method=oidc role=default
```

### group mapping

```bash
vault write auth/oidc/role/default \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="preferred_username" \
groups_claim="groups" \
policies=default-policy

vault read auth/oidc/role/default

vault write auth/oidc/role/manager \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="preferred_username" \
groups_claim="groups" \
policies=default-policy

vault read auth/oidc/role/manager

vault write auth/oidc/role/admin \
bound_audiences="vault" \
allowed_redirect_uris="https://vault.club012.com/ui/vault/auth/oidc/oidc/callback" \
allowed_redirect_uris="http://localhost:8250/oidc/callback" \
user_claim="preferred_username" \
groups_claim="groups" \
policies=default-policy

vault read auth/oidc/role/admin
```

```bash
# create vault groups
vault write identity/group \
    name="secret-admin" \
    policies="secret-admin,transit-admin" \
    type="external" \
    metadata=organization="admin"
# group id : eaac9985-83b6-ce8a-9c3d-50140d585f4b
export ADMIN_GROUP_ID=eaac9985-83b6-ce8a-9c3d-50140d585f4b

vault write identity/group \
    name="secret-manager" \
    policies="secret-manager,transit-manager" \
    type="external" \
    metadata=organization="manager"
# grup id : aa965737-4ec6-557c-0254-c49f72bd0c37
export MANAGER_GROUP_ID=aa965737-4ec6-557c-0254-c49f72bd0c37

# Get AccessorId of OIDC auth
accessorId=$(vault auth list | grep oidc | awk '{print $3}')
# accessorId : auth_oidc_1f25907e

# map keycloak groups to vault groups
vault write identity/group-alias \
    name="jenkins-admin" \
    mount_accessor="$accessorId" \
    canonical_id="${ADMIN_GROUP_ID}"

vault write identity/group-alias \
    name="jenkins-manager" \
    mount_accessor="$accessorId" \
    canonical_id="${MANAGER_GROUP_ID}"
```
