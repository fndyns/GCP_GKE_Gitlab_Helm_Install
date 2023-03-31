# GCP_GKE_Gitlab_Helm_Install
GCP_GKE_Gitlab_Helm_Install

helm repo upgrade
helm repo update
helm upgrade --install -f values.yaml gitlab8 gitlab/gitlab --set certmanager-issuer.email=ahmed@cyberneticlabs.io --version=6.9.3
