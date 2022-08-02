#airflow

## Dockerfile

```dockerfile
FROM apache/airflow:2.3.3-python3.9

MAINTAINER paridis

COPY ./camel_airflow_plugins/ /opt/airflow/plugins/

COPY ./template/ /opt/airflow/email_template/

LABEL version="0.3"

RUN pip config set global.index-url http://152.32.187.85:3724/repository/pypi-public/simple \
    && pip config set install.trusted-host 152.32.187.85:3724 \
    && pip install CamelLuigi \
        battle-log-parser \
        battle-log-pipe \
        camel-email \
        camel-gift-tag \
        camel-pytosql-x \
        camel-queries \
        camel-tableau-lib \
        camel-test \
        camel-utils-x \
        camel-k8s-config \
        camelluigi \
        camelopdaily \
        cameloperators \
        camelruleengine \
        dbx \
        dbxx \
        merge-server-x \
        tableau-api-lib \
        --no-cache-dir \ 
        -c "https://raw.githubusercontent.com/apache/airflow/constraints-2.3.3/constraints-3.9.txt" 
```

## build

```shell
docker build -t xupeiqi/airflow:2.3.3-python3.9-0.3 . 
```

## tag

```shell
docker tag xupeiqi/airflow:2.3.3-python3.9-0.3  harbor.xuwanzi.cn/camel-docker/airflow:2.3.3-python3.9-0.3 
```

## push

```shell
docker push harbor.xuwanzi.cn/camel-docker/airflow:2.3.3-python3.9-0.3
```
