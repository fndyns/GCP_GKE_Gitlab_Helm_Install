# Steps to Install Gitlab via Helm on Google Kubernetes GKE Cluster

1. Install the GKE Cluster and make connection to the cluster from your terminal
2. Add static IP in GCP for Gitlab URL and add this ip to your local values yaml (as seen below) and register this ip to the gitlab domain (for us it was gitlab.zekoder.net)

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

4. Created Postgres on Google Cloud (Google Cloud SQL) and connect it to our Gitlab on GKE. https://cloud.google.com/sql/docs/postgres To do that, setted postgres Install as False below.

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
    
    
And enabled external psql config as seen below.  Setted postgres private ip 10.40.160.3 as host ip below. Cluster access external Postgres DB via its Private IP since both are in the same default VPC. ello Ahmed. I have enabled "Private IP" for Postgres. Then this Private ip is needed to connect Google Cloud SQL to GKE. (There was another way to connect Google Cloud SQL to GKE . This is Cloud SQL Auth proxy, but to do that, we need to prepare Service account with permission to access Cloud SQL  but I dont have permission  to grant access in existing service account or create a service account with the Cloud SQL role assigned. Please help on this. Ahmed said "according to documentation, using cloud managed redis and postgresql is recommended for production environment so let’s stick with that, it is always better to create non default VPC and use it for K8s and other resources, so you can open only what you need in the new VPC, I have granted you a security admin and SQL admin, this should cover all your needs. By the way you can add permissions to yourself, I have added K8s Admin)


  psql:
    connectTimeout:
    keepalives:
    keepalivesIdle:
    keepalivesInterval:
    keepalivesCount:
    tcpUserTimeout:
    password: {}
      # useSecret:
      # secret:
      # key:
      # file:
    host: 10.40.160.3
    port: 5432
    username: postgres
    database: gitlabhq_production


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

