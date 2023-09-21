## 普通安装

### install

```Bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
```

当然也可以使用其他方式，如：helm。

具体可以参考官方文档：

[Installation Guide - NGINX Ingress Controller (kubernetes.github.io)](https://kubernetes.github.io/ingress-nginx/deploy/)

### config

#### 前置条件

创建workload, service。

#### ingressClass

因为 ingressController 有多个实现，也可以在集群中安装多个，所以使用 ingress 中使用这个参数控制使用哪个 ingressController。

安装文件中 ingressClass 如下：

```YAML
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.0
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

ingressClass 默认是集群资源，当然也可以设置也 ns 范围的资源。这里不关注这个，nginx ingressClass 没有把自己设置为默认 ingressClass。

所以可以设置一下，省的在 ingress 中再设置。

```YAML

apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.0
  name: nginx
  # 设置为默认 ingressClass
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```

### 使用 ingress

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ing
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  # ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /ng
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```

如上配置，如果有默认的 ingressClass这里就不用设置，如果没有就得加上 `ingressClassName`

### 常用annotation 示例

[Rewrite - NGINX Ingress Controller (kubernetes.github.io)](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)

### Ingress高可用

Ingress高可用，我们可以通过修改deployment的副本数来实现高可用，但是由于ingress承载着整个集群流量的接入，所以生产环境中，建议把ingress通过DaemonSet的方式部署集群中，而且该节点打上污点不允许业务pod进行调度，以避免业务应用与Ingress服务发生资源争抢。然后通过SLB把ingress节点主机添为后端服务器，进行流量转发。

```YAML
# 修改mandatory.yaml
# 主要修改pod相关
apiVersion: extensions/v1beta1 
#修改为DaemonSet
kind: DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      # 添加该字段让docker使用物理机网络，在物理机暴露服务端口(80)，注意物理机80端口提前不能被占用
      hostNetwork: true
      # 使用hostNetwork后容器会使用物理机网络包括DNS，会无法解析内部service，使用此参数让容器使用K8S的DNS
      dnsPolicy: ClusterFirstWithHostNet
      nodeSelector:
        vanje/ingress-controller-ready: "true"
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Equal"
        value: ""
        effect: "NoSchedule"
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
```

修改参数如下：

1. kind: Deployment #修改为DaemonSet
2. replicas: 1 #注销此行，DaemonSet不需要此参数
3. hostNetwork: true #添加该字段让docker使用物理机网络，在物理机暴露服务端口(80)，注意物理机80端口提前不能被占用
4. dnsPolicy: ClusterFirstWithHostNet #使用hostNetwork后容器会使用物理机网络包括DNS，会无法解析内部service，使用此参数让容器使用K8S的DNS
5. nodeSelector:vanje/ingress-controller-ready: "true" #添加节点标签
6. tolerations: 添加对指定节点容忍度

这里我在2台master节点部署(生产环境不要使用master节点，应该部署在独立的节点上)，因为我们采用DaemonSet的方式，所以我们需要对2个节点打标签以及容忍度。

```Bash
# 给节点打标签
kubectl label nodes k8s-master02 vanje/ingress-controller-ready=true
kubectl label nodes k8s-master03 vanje/ingress-controller-ready=true
# 节点打污点
# master节点我之前已经打过污点，如果你没有打污点，执行下面2条命令。此污点名称需要与yaml文件中pod的容忍污点对应
kubectl taint nodes k8s-master02 node-role.kubernetes.io/master=:NoSchedule
kubectl taint nodes k8s-master03 node-role.kubernetes.io/master=:NoSchedule
```

### 参考文档：

[k8s配置ingress - 简书 (jianshu.com)](https://www.jianshu.com/p/c726ed03562a)

## 使用 helm 安装

### 配置仓库

```Bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### values

```YAML
controller:
  # -- Optionally change this to ClusterFirstWithHostNet in case you have 'hostNetwork: true'.
  # By default, while using host network, name resolution uses the host's DNS. If you wish nginx-controller
  # to keep resolving names inside the k8s network, use ClusterFirstWithHostNet.
  dnsPolicy: ClusterFirstWithHostNet

  # -- Required for use with CNI based kubernetes installations (such as ones set up by kubeadm),
  # since CNI and hostport don't mix yet. Can be deprecated once https://github.com/kubernetes/kubernetes/issues/23920
  # is merged
  hostNetwork: true

  ## This section refers to the creation of the IngressClass resource
  ## IngressClass resources are supported since k8s >= 1.18 and required since k8s >= 1.19
  ingressClassResource:
    # -- Is this the default ingressClass for the cluster
    default: true

  # -- Use a `DaemonSet` or `Deployment`
  kind: DaemonSet
  service:
    annotations: 
      # 在 eks 中默认会添加 classic elb，但是这个要被废弃了，所以添加如下注解，使用 nlb
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "3600"
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
      service.beta.kubernetes.io/aws-load-balancer-type: nlb


```



### 安装

```Bash
kubectl create ns ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -f ing-nginx/value-ing.yaml -n ingress-nginx

```

## aws load balancer contrller

在 aws 中使用 ingress-nginx aws 会自行创建一个 elb。但是 elb 是 classic 的，要废弃了。也无法迁移。

所以线上得使用 alb。使用 alb 的话就得使用这个控制器。

### install

[https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/)

### 使用

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    alb.ingress.kubernetes.io/group.name: alb-group
    # alb.ingress.kubernetes.io/load-balancer-name: bigdata-http
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
  name: hello-ing
  namespace: hello
spec:
  ingressClassName: alb
  rules:
  - host: af.xuwanzi.cn
    http:
      paths:
      - backend:
          service:
            name: hello-service
            port:
              number: 80
        path: /
        pathType: Prefix
```

## metallb

[https://github.com/metallb/metallb](https://github.com/metallb/metallb)

[https://zhuanlan.zhihu.com/p/146085109](https://zhuanlan.zhihu.com/p/146085109)

一个本地的 loadbanlence.

ingress 安装后对外暴露的话也得有一个  service。使用在云服务上的话serviceType可以使用 loadbanlence。但是私有机群的话要么使用 nodePort。这样的话做域名映射的话如果单独设置一台node 的 ip。node 挂了就出问题，那么就得在 k8s 前面再有一台 nginx 的服务器去接管 k8s 整个集群的流量，相当于 nginx 对接 ingress。这样的话也很麻烦。所以使用 metallb 使本地拥有 lb 功能。

