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
3) create new role `vault write auth/approle/role/argocd secret_id_ttl=10m token_num_uses=10 token_ttl=20m token_max_ttl=30m secret_id_num_uses=40`
4) Read settings
    1) `vault read auth/approle/role/argocd/role-id` = your_role_id
    1) `vault write -force auth/approle/role/my-role/secret-id` = your_secret_id
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
  AVP_ROLE_ID: <your_role_id>
  AVP_SECRET_ID: <your_secret_id>
  AVP_TYPE: vault
  AVP_VAULT_ADDR: http://vault.vault
EOF
```
6) Run `kubectl apply -f setup/`

# Secrets
```
traefik/http-auth/htpasswd-admin: $(htpasswd -nb username password)
```