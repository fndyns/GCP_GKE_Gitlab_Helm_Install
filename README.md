# Steps to Install Gitlab via Helm on Google Kubernetes GKE Cluster

1. Install the GKE Cluster and make connection to the cluster from your terminal
2. Add static IP in GCP for Gitlab URL and add this ip to your local values yaml (as seen below) and add this ip to the gitlab domain (for us it was gitlab.zekoder.net)

  hosts:
    domain: zekoder.net
    hostSuffix:
    https: true
    externalIP: "34.28.205.131"
    ssh: ~
    gitlab: {}
    minio: {}
    registry: {}
    tls: {}
    smartcard: {}
    kas: {}
    pages: {}

4. Created Postgres on Google Cloud and connect it to our Gitlab on GKE. https://cloud.google.com/sql/docs/postgres To do that setted postgres Install as False below  

postgresql:
  postgresqlUsername: gitlab
  # This just needs to be set. It will use a second entry in existingSecret for postgresql-postgres-password
  postgresqlPostgresPassword: bogus
  install: false
  postgresqlDatabase: gitlabhq_production
  image:
    tag: 12.7.0
  usePasswordFile: true
  existingSecret: bogus
  initdbScriptsConfigMap: bogus
  master:
    extraVolumeMounts:
      - name: custom-init-scripts
        mountPath: /docker-entrypoint-preinitdb.d/init_revision.sh
        subPath: init_revision.sh
    podAnnotations:
      postgresql.gitlab/init-revision: "1"
  metrics:
    enabled: true
    ## Optionally define additional custom metrics
    ## ref: https://github.com/wrouesnel/postgres_exporter#adding-new-metrics-via-a-config-file


3. Domain zekoder.net needs to be registered somewhere like on AWS(Route 53) or Google Cloud.
# AWS_SES_SMTP_Config_Info

SMTP 
Username:AKIAWA3RKI66ON4PZEEP
Password: BGpW8Ry3Oop/C65pBBLwhuUyalTRpcsb0MOLk7JvpWzL
address: email-smtp.eu-central-1.amazonaws.com

STARTTLS Port
25, 587 or 2587

please use gitlab@zekoder.net


# GKE Cluster Version : 1.23.14-gke.1800 

# GCP_GKE_Gitlab_Helm_Install

gcloud auth login
gcloud container clusters get-credentials gitlab--cluster --zone us-central1-f --project devops-376610
sudo gcloud container clusters get-credentials gitlab--cluster --zone us-central1-f --project devops-376610
gcloud components install kubectl
gcloud components install gke-gcloud-auth-plugin
gcloud config set account ACCOUNT
gcloud auth login
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
./google-cloud-sdk/bin/gcloud init
gcloud init
gcloud config list
gcloud components install kubectl
sudo ./google-cloud-sdk/bin/gcloud init
sudo gcloud init
sudo gcloud components install kubectl
gke-gcloud-auth-plugin --version


# GCP_GKE_Gitlab_Helm_Install


helm repo upgrade
helm repo update
helm upgrade --install -f values.yaml gitlab8 gitlab/gitlab --set certmanager-issuer.email=ahmed@cyberneticlabs.io --version=6.9.3
kubectl create secret generic gitlab8-smtp-secret --from-literal=password=BGpW8Ry3Oop/C65pBBLwhuUyalTRpcsb0MOLk7JvpWzL
kubectl get secret gitlab8-smtp-secret -o yaml  #You should be seeing encode64 format of password in output
kubectl logs gitlab8-sidekiq-all-in-1-v2-7b97965c64-hznn6   #Its output giving smtp, auth error
kubectl logs gitlab8-webservice-default-58fd46dd44-hxnnv --all-containers

