---
Title: Gitlab Installation
tags: [normal]
---
___

## Introduction

In this article, I will describe all the steps required to setup GitLab CI/CD in kuberntes using kustomize.
We will go through how to run GitLab on Kubernetes when you have related resources `postgres`, `redis`, `minio`, `tls certificates` etc already available in your setup.

This is a very common scenario in companies and also for self-hosting that you are already using these services in your environment and prefer to use the same for gitlab.


*The all in one production installation may be easily performed with Helm. You can refer to official documentation from gitlab if that is your requirement.*


### Requirements

You will need the following in order to run gitlab.

1. Database : Postgres database is required for gitlab.
2. Cache :   Redis is used for caching.
3. Storage : Minio is used as object storage for `container registry`, `gitlab backups`, `terraform storage backend`, `gitlab artifacts` etc.
4. Ingress Controller : Nginx ingress is part of installation.
5. Persistent Volume : Gitaly will store `repository data` data on disk, for that your kubernetes cluster must have a way of provisioning storage. You can install [local path provisioner](https://github.com/rancher/local-path-provisioner) in your cluster for dynamically provisioning volumes.

* Repositories
  1. [Gitlab Manifests](https://github.com/kha7iq/gitlab-k8s)
  2. [SubVars App](https://github.com/kha7iq/subvars)

*Info: You can swap minio with any other object storage i.e S3 by changing connection info secret*

## Lets get Started

When installing gitlab with helm it generates the configmaps after rendering the templates with parameters, we can manually change these values in configmaps but its a hassle and not convenient.

To make this process easy we will use a tool called [subvars](https://github.com/kha7iq/subvars) which will let us render these values from command line. Install it by following the instructions on [github](https://github.com/kha7iq/subvars) page, we will use it later.


1. Download the release with manifests from [github](https://github.com/kha7iq/gitlab-k8s) alternatively you can clone the repo, if you are cloning the repo remove the `.git` folder afterwards as it creates issues some times when rendering multiple version of the same file with subvars.

```bash
export RELEASE_VER=1.0
wget -q https://github.com/kha7iq/gitlab-k8s/archive/refs/tags/v${RELEASE_VER}.tar.gz
tar -xf v${RELEASE_VER}.tar.gz
cd gitlab-k8s-${RELEASE_VER}
```


2. Lets start by setting the url for gitlab in our [kustomization file](https://github.com/kha7iq/gitlab-k8s/blob/master/ingress-nginx/kustomization.yaml) within ingress-nginx folder. You will find two blocks, one for web-ui and second for registry along with tls-secret-name for https.

```yaml
patch: |-
    - op: replace
      path: /spec/rules/0/host
      value: your-gitlab-url.example.com
    - op: replace
      path: /spec/tls/0/hosts/0
      value: your-gitlab-url.example.com
    - op: replace
      path: /spec/tls/0/secretName
      value: example-com-wildcard-secret
```

3. We can create `minio-conn-secret` containing configuration for minio. It will be used for all the enabled S3 buckets except gitlab backups, we will create that separately. Input the information as per your setup and create the kubernetes secret.

* minio.config
```bash
cat << EOF > minio.config
provider: AWS
region: us-east-1
aws_access_key_id: 4wsd6c468c0974006d
aws_secret_access_key: 5d5e6c468c0974006cdb41bc4ac2ba0d
aws_signature_version: 4
host: minio.example.com
endpoint: "https://minio.example.com"
path_style: true
EOF
```
* Kubernetes secret
```bash
kubectl create secret generic minio-conn-secret \
--from-file=connection=minio.config --dry-run=client -o yaml >minio-connection-secret.yml
```

4. Next step is to create a secret with mino configuration for gitlab backup storage. Just replace minio endpoint, bucket name, access key & secret key.

```bash
cat << EOF > storage.config
[default]
access_key = be59435b326e8b0eaa
secret_key = 6e0a10bd2253910e1657a21fd1690088
bucket_location = us-east-1
host_base = https://minio.example.com
host_bucket = https://minio.example.com/gitlab-backups
use_https = True
default_mime_type = binary/octet-stream
enable_multipart = True
multipart_max_chunks = 10000
multipart_chunk_size_mb = 128
recursive = True
recv_chunk = 65536
send_chunk = 65536
server_side_encryption = False
signature_v2 = True
socket_timeout = 300
use_mime_magic = False
verbosity = WARNING
website_endpoint = https://minio.example.com
EOF
```

```bash
kubectl create secret generic storage-config --from-file=config=storage.config \
--dry-run=client -o yaml > secrets/storage-config.yml
```

> All other secrets can be used as is from repository or you can change all of them following [gitlab documentation](https://docs.gitlab.com/charts/installation/secrets.html)

5. One of the most important secret is `gitlab-rails-secret`, in case of a disaster where you have to restore gitlab from a backup you must apply the same secret in your cluster as these keys will be used to decrypt the database etc from backup. Make sure you keep this consistent after first install.


6. We reached the last part, Its alot of work to change database and other parameters one by one in configmaps.
I have implemented some templating for this which can provide all the values vi environment variables and render the manifests with subvars, it will output these to destination folder and replace all the parameters defined as go templates.
The values are self explanatory, `GITLAB_GITALY_STORAGE_SIZE` variable is used to specify how much storage is needed for gitaly and `GITLAB_STORAGE_CLASS` is the name of storage class.

```bash
GITLAB_URL=gitlab.example.com \
GITLAB_REGISTRY_URL=registry.example.com \
GITLAB_PAGES_URL=pages.example.com \
GITLAB_POSTGRES_HOST=192.168.1.90 \
GITLAB_POSTGRES_PORT=5432 \
GITLAB_POSTGRES_USER=gitlab \
GITLAB_POSTGRES_DB_NAME=gitlabhq_production \
GITLAB_REDIS_HOST=192.168.1.91:6379 \
GITLAB_GITALY_STORAGE_SIZE=15Gi \
GITLAB_STORAGE_CLASS=local-path \
subvars dir --input gitlab-k8s-1.0 --out dirName
```
Change into `dirName/gitlab-k8s-1.0` you can have a look to confirm if everything is in order before applying this in cluster.

7. The final step is to create the namespace `gitlab` and build with kustomize or kubectl. I prefer kustomize but you can also use kubectl with `-k` flag.

* Create namespace
```bash
kubectl create namespace gitlab
```

* Apply the final manifest
```bash
kustomize build gitlab-k8s-1.0/ | kubectl apply -f -
# or following if you have already changed into directory
kustomize build . | kubectl apply -f -

# With kubectl
kubectl apply -k  gitlab-k8s-1.0/
# or following if you have already changed into directory
kubectl apply -k  .
```

8. Head over to the endpoint you have configured for gitlab `https://gitlab.example.com` and login.
   

* Note: 
   Default passwords
   > Gitlab 'root' user's password configured as secret
   ```bash
   LAwGTzCebner4Kvd23UMGEOFoGAgEHYDszrsSPfAp6lCW15S4fbvrVrubWsua9PI
   ```
   > Postgres password configured as secret
   ```bash
    ZDVhZDgxNWY2NmMzODAwMTliYjdkYjQxNWEwY2UwZGMK
   ```
