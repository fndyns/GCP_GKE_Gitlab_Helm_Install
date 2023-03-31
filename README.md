# Gitlab Chart Link : https://gitlab.com/gitlab-org/charts/gitlab/-/tree/v6.9.3 
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
4. After installing gitlab via helm, there was cert issue occurred, then I fixed it on pods. All pods up and running but 404 error still exist. I have checked all pods and cert-manager throwing error as  registry.zekoder.net ,  kas.zekoder.net and  kas.zekoder.net hosts not found. Not sure where we need to define these domains or how (on nginx or somewhere else). We may need to make domain route smt from these domains to gitlab.zekoder.net (edited) . (We did not fix this issue but gitlab via domain was accessed successfully after fixing cert issue). Ahmed said "we must add all domains (registry.zekoder.net ,  kas.zekoder.net ) to the DNS  via same ip as gitlab.zekoder.net if the same ingress will serve them all.
5. After Gitlab via domain is accessible, we enabled email notification on Gitlab via SMTP. For that we use AWS SES service and connect it to our GKE Cluster. I had reapplied the chart to make the required SMTP changes. We also allowed SMTP traffic for the GKE cluster within the default VPC from Firewalls Section on Google Cloud. AWS SES using port 587 for SMTP. There is also Google Cloud mailgun service for SMTP. But we did not use it. We just used the SMTP service from amazon, everything else stayed in GCP.

6. For SMTP part, I have gotten too many errors. To fix them first I made sure that AWS SES service is verified successfully with the correct domain gitlab@zekoder.net. There is one online tool https://www.gmass.co/smtp-test which we can test if our SMTP address working success. We also checked the pod logs for smtp issues from side car and web pods. We also tested our SMTP via  https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp-client-command-line.html . Here is the index.txt file I use ;


dell@dell-Latitude-3420:~/Downloads$ cat input.txt 
EHLO email-smtp.eu-central-1.amazonaws.com
AUTH LOGIN
QUtJQVdBM1JLSTY2T040UFpFRVA=
QkdwVzhSeTNPb3AvQzY1cEJCTHdodVV5YWxUUnBjc2IwTU9MazdKdnBXekw=
MAIL FROM: gitlab@zekoder.net
RCPT TO: melike.o@cyberneticlabs.io
DATA
X-SES-CONFIGURATION-SET:
From: Sender Name <gitlab@zekoder.net>
To: melike.o@cyberneticlabs.io
Subject: Amazon SES SMTP Test

This message was sent using the Amazon SES SMTP interface.
.
QUIT

**** In SMTP part, I have gotten many errors and here is the current SMTP config below, and also here is the web link I followed for whole gitlab installation ;

https://docs.gitlab.com/charts/

  smtp:
    enabled: true
    address: email-smtp.eu-central-1.amazonaws.com
    port: 587
    user_name: AKIAWA3RKI66ON4PZEEP
    ## https://docs.gitlab.com/charts/installation/secrets#smtp-password
    password:
      secret: gitlab8-smtp-secret
      key: password
    domain: zekoder.net
    authentication: "plain"
    starttls_auto: true
    openssl_verify_mode: "peer"
    pool: false

  ## https://docs.gitlab.com/charts/charts/globals#outgoing-email
  ## Email persona used in email sent by GitLab
  email:
    from: gitlab@zekoder.net
    display_name: Gitlab Zekoder
    reply_to: gitlab@zekoder.net
    subject_suffix: ""
    smime:
      enabled: false
      secretName: ""
      keyName: "tls.key"
      certName: "tls.crt"


# AWS_SES_SMTP_Config_Info

SMTP 
Username:AKIAWA3RKI66ON4PZEEP
Password: BGpW8Ry3Oop/C65pBBLwhuUyalTRpcsb0MOLk7JvpWzL
address: email-smtp.eu-central-1.amazonaws.com

STARTTLS Port
25, 587 or 2587

please use gitlab@zekoder.net


# GKE Cluster Version : 1.23.14-gke.1800 (without autopilot)

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

