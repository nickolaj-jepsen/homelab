# homelab

## Setup cluster
1) `k3sup install --k3s-extra-args '--no-deploy traefik' --cluster --local --k3s-version v1.21.4+k3s1`
2) `kubectl create namespace argocd`
3) `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
4) `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

## Argocd

1) Setup this repo in argocd
2) Run `kubectl apply -f setup/`


## Longhorn

1) Setup disks in longhorn admin
2) disable local-storage default `kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'`


## Vault

1) Setup vault
2) Enable approle
3) create new role `vault write -force auth/approle/role/argocd`
4) Read settings
    1) `vault read auth/approle/role/argocd/role-id` = your_role_id
    1) `vault write -force auth/approle/role/argocd/secret-id` = your_secret_id
5) Deploy yaml 
```bash
cat <<EOF | kubectl apply -f -
kind: Secret
apiVersion: v1
metadata:
  name: argocd-vault-plugin-credentials
  namespace: argocd
type: Opaque
stringData:
  AVP_AUTH_TYPE: approle
  AVP_ROLE_ID: ff3f67af-e4dc-002f-f311-aa710cc310fa
  AVP_SECRET_ID: 09a0864c-e98f-3708-86a2-512e9d1a2939
  AVP_TYPE: vault
  VAULT_ADDR: http://vault.vault:8200
EOF
```
6) Run `kubectl apply -f setup/`
7) Create policy "read-all" `path "kv/data/*" {  capabilities = ["read"]}`
8) Add policy to generated entity

# Secrets
```
traefik/http-auth/htpasswd-admin: $(htpasswd -nb username password)
cert-manager/cloudflare/api-key: $(cloudflare > User Profile > API Tokens > API Keys > Global API Key > View)
keycloak/traefik-forward-auth/issuer-url: $("https://keycloak.k8s.nickolaj.com:8443/auth/realms/master")
keycloak/traefik-forward-auth/client-id: $(keycloak > Client > create > Client ID)
keycloak/traefik-forward-auth/client-secret: $(keycloak > Client > "Client ID" > Creditentials > Secret))
```