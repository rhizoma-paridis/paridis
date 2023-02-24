#airflow

## 前置条件

[airflow config](airflow%20config.md)

[email](email.md)

[wecom](wecom.md)

[image](image.md)

[fornetkey](fornetkey.md)

## pvc

### local

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: airflow-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  claimRef:
    name: airflow-pvc
    namespace: airflow
  local:
    path: /data/kpv/airflow
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - "ip-10-1-3-115.us-west-2.compute.internal"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: airflow-pvc
  namespace: airflow
spec:
  storageClassName: "local-storage"
  volumeName: airflow-pv
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 5Gi

```

### efs

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: airflow-pvc
  namespace: airflow
spec:
  storageClassName: "efs-sc"
  # volumeName: airflow-pv
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 5Gi

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: airflow-db-info-pvc
  namespace: airflow
spec:
  storageClassName: "efs-sc"
  # volumeName: airflow-db-info-pv
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 10Mi
---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: airflow-files-pvc
  namespace: airflow
spec:
  storageClassName: "efs-sc"
  # volumeName: airflow-files-pv
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 1Gi
```

## secret

```yaml
apiVersion: v1
data:
  GIT_SYNC_PASSWORD: Z2hwX1ZsMFVISEpDQ3k0Q0VObDhWRlR4VGR1MXY2BdDRIQlFEQg==
  GIT_SYNC_USERNAME: Y2FtZWwtZGF0YS1ncm91cA==
kind: Secret
metadata:
  name: git-credentials
  namespace: airflow
type: Opaque
```

不过对于 token 这种东西，我觉得资源文件太麻烦，不如 cli 方便。使用资源文件的话得事先把数据加密，命令行的话明文就行，它自己会加密。注意申请的 token 的期限，到期的话换一个就行了。

```shell
kubectl create secret generic git-credentials \
--from-literal=GIT_SYNC_PASSWORD=test\
--from-literal=GIT_SYNC_USERNAME=test
```

## docker config

```shell
kubectl create secret docker-registry dockercfg \ 
--docker-server=harbor.xuwanzi.cn \
--docker-username=test \
--docker-password=CamelTest1234 \
--docker-email=xupeiqi@camel4u.com \
-n airflow
```

## helm values

```yaml


# Default airflow repository -- overrides all the specific images below
defaultAirflowRepository: harbor.in.xuwanzi.com/docker/airflow

# Default airflow tag to deploy
defaultAirflowTag: "2.3.3-python3.9-0.19"

# Airflow version (Used to make some decisions based on Airflow Version being deployed)
airflowVersion: "2.3.3"

# Select certain nodes for airflow pods.

affinity: 
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - preference:
        matchExpressions:
        - key: app
          operator: In
          values:
          - airflow
      weight: 1

tolerations: 
- key: app
  operator: Equal
  value: airflow
  effect: "NoSchedule"

# Ingress configuration
ingress:
  # Configs for the Ingress of the web Service
  web:
    # Enable web ingress resource
    enabled: true
    # The hostname for the web Ingress (Deprecated - renamed to `ingress.web.hosts`)
    hosts: 
      - name: "airflow.in.xuwanzi.com"
    # The Ingress Class for the web Ingress (used only with Kubernetes v1.19 and above)
    ingressClassName: "nginx"

executor: "KubernetesExecutor"

data:
  # Otherwise pass connection values in
  metadataConnection:
    user: "xxxx"
    pass: "xxxx"
    protocol: "mysql"
    host: "xxx.com"
    port: 3306
    db: k8s_airflow
    sslmode: disable

# Fernet key settings
# Note: fernetKey can only be set during install, not upgrade
fernetKey: "xxxxxxxxxxxxxx="

webserverSecretKey: "xxxxxxxxxxxxx"

workers:
  extraVolumes:
    - name: db-info
      persistentVolumeClaim:
        claimName: airflow-db-info-pvc
    - name: files
      persistentVolumeClaim:
        claimName: files-pvc
  extraVolumeMounts:
    - name: db-info
      mountPath: "/opt/airflow/db_info"
    - name: files
      mountPath: "/opt/airflow/files"
  resources: 
    limits:
      cpu: 1
      memory: 500Mi
    requests:
      cpu: 200m
      memory: 300Mi
# scheduler:
#   resources: 
#     limits:
#       cpu: 2
#       memory: 6Gi
#     requests:
#       cpu: 1
#       memory: 4Gi

webserver:
  defaultUser:
    password: xxxxxx
  extraVolumes:
    - name: db-info
      persistentVolumeClaim:
        claimName: db-info-pvc
    - name: files
      persistentVolumeClaim:
        claimName: files-pvc
  extraVolumeMounts:
    - name: db-info
      mountPath: "/opt/airflow/db_info"
    - name: files
      mountPath: "/opt/airflow/files"
#   resources: 
#     limits:
#       cpu: 300m
#       memory: 3Gi
#     requests:
#       cpu: 100m
#       memory: 1Gi

# triggerer:
#   resources: 
#     limits:
#       cpu: 300m
#       memory: 2Gi
#     requests:
#       cpu: 100m
#       memory: 500Mi

# statsd:
#   resources: 
#     limits:
#       cpu: 50m
#       memory: 200Mi
#     requests:
#       cpu: 50m
#       memory: 100Mi
# Auth secret for a private registry
# This is used if pulling airflow images from a private registry
registry:
  secretName: dockercfg


postgresql:
  enabled: false

config:
  core:
    plugins_folder: "/opt/airflow/plugins"
    parallelism: 150
    dagbag_import_timeout: 160.0
  email:
    subject_template: "/opt/airflow/email_template/subject_template.j2"
    html_content_template: "/opt/airflow/email_template/content_template.j2"
  smtp:
    smtp_host: smtp.exmail.qq.com
    smtp_starttls: False
    smtp_ssl: True
    smtp_user: data_public@xxx.com
    smtp_password: xxxx
    smtp_port: 465
    smtp_mail_from: data_public@xxx.com
    smtp_timeout: 30
    smtp_retry_limit: 5
  kubernetes:
    delete_worker_pods: True
    delete_worker_pods_on_failure: False
    enable_tcp_keepalive: True
    tcp_keep_idle: 120
    tcp_keep_intvl: 30
    tcp_keep_cnt: 6
  webserver:
    page_size: 50

dags:
  gitSync:
    enabled: true

    # git repo clone url
    # ssh examples ssh://git@github.com/apache/airflow.git
    # git@github.com:apache/airflow.git
    # https example: https://github.com/apache/airflow.git
    repo: "https://github.com/xxx-xxx/k8s-airflow.git"
    branch: main
    rev: HEAD
    depth: 1
    # the number of consecutive failures allowed before aborting
    maxFailures: 0
    # subpath within the repo where dags are located
    # should be "" if dags are at repo root
    subPath: ""
    # if your repo needs a user name password
    # you can load them to a k8s secret like the one below
    #   ---
    #   apiVersion: v1
    #   kind: Secret
    #   metadata:
    #     name: git-credentials
    #   data:
    #     GIT_SYNC_USERNAME: <base64_encoded_git_username>
    #     GIT_SYNC_PASSWORD: <base64_encoded_git_password>
    # and specify the name of the secret below
    #
    credentialsSecret: git-credentials

logs:
  persistence:
    # Enable persistent volume for storing logs
    enabled: true
    ## the name of an existing PVC to use
    existingClaim: airflow-pvc

```

## install

```shell
helm upgrade --install airflow apache-airflow/airflow -f value.yaml -n airflow --version 1.6.0 --description "update fornetkey"
```
