#gitlab

## ci

### .gitlab-ci.yml

```yml
variables:  
  MAVEN_CLI_OPTS: "-Dmaven.repo.local=~/.m2/repository -Dmaven.test.skip=true"  
  HARBOR_URL: "harbor.test.com"  
  HARBOR_REPO: "harbor.test/docker/"  
  IMAGE: ${HARBOR_REPO}${CI_PROJECT_NAME}:${CI_PIPELINE_ID}  
stages:  
  - build  
  - build_image  
  - deploy_k8s  
  
cache:  
  paths:  
    - ~/.m2/  
  
build-jar:  
  image: maven:3.8.2-jdk-8  
  stage: build  
  script:  
    - echo '<settings xmlns="https://maven.apache.org/SETTINGS/1.2.0"  
            xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"  
            xsi:schemaLocation="https://maven.apache.org/SETTINGS/1.2.0/ https://maven.apache.org/xsd/settings-1.2.0.xsd">  
            <mirrors>  
                  <mirror>  
                        <id>aliyun</id>  
                        <mirrorOf>central</mirrorOf>  
                        <name>aliyun</name>  
                        <url>https://maven.aliyun.com/repository/central</url>  
                  </mirror>  
            </mirrors>  
            </settings>' > ~/.m2/settings.xml  
    - mvn  clean package $MAVEN_CLI_OPTS  
  artifacts:  
    paths:  
      - target/*.jar  
  
docker_image:  
  stage: build_image  
  script:  
    - docker build . -t ${IMAGE}  
    - docker login ${HARBOR_URL} -u worker -p Zaq1Xsw2  
    - docker push ${IMAGE}  
  
k8s_deploy:  
  image: alpine/k8s:1.21.13  
  stage: deploy_k8s  
  script:  
    - echo "41.215.189.110  lb.kubesphere.local" >> /etc/hosts  
    - kubectl config set-cluster cluster.local --server=https://lb.kubesphere.local:6443 --certificate-authority=${ca} --embed-certs=true  
    - kubectl config set-credentials 119k8s --client-certificate=${userc} --client-key=${userk} --embed-certs=true  
    - kubectl config set-context 119k8s@cluster.local --cluster=cluster.local --user=119k8s  
    - kubectl config use-context 119k8s@cluster.local  
    - kubectl apply -f ./k8s/*
```

## 变量配置

相关证书使用 `file` 类型变量，注意证书内容在粘贴过程中可能会出现错误。如结尾可能多一个回车。
