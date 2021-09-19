# homelab

## Setup cluster
1) `k3sup install --k3s-extra-args '--no-deploy traefik' --cluster --local --k3s-version v1.21.4+k3s1`
2) `kubectl create namespace argocd`
3) `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

## Argocd

1) Run `kubectl apply -f setup/`