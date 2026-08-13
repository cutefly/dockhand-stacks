# Vault transit

```sh
vault secrets enable -path=transit transit
vault secrets tune -description="transit secret engine" transit
```

```sh
vault policy write transit-admin -<<EOF
path "sys/mounts/transit" {
  capabilities = [ "create", "read", "update", "delete", "list" ]
}

path "sys/mounts" {
  capabilities = [ "read" ]
}

path "transit/*" {
  capabilities = [ "create", "read", "update", "delete", "list" ]
}
EOF

vault policy write transit-manager -<<EOF
path "sys/mounts/transit" {
  capabilities = [ "read", "update", "list" ]
}

path "sys/mounts" {
  capabilities = [ "read" ]
}

path "transit/*" {
  capabilities = [ "read", "update", "list" ]
}
EOF
```

vault write -f transit/keys/rustfs-default-key


