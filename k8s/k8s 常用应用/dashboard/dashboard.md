安装 `dashboard` 前先安装 `metrics-server`。要不然相关指标看不到。

# 创建 ns

```bash
kubectl create ns k8s-dashboard
```

# TLS 证书

这里使用自签证书。

```shell
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=kdb.xuwanzi.cn"
```

`kdb.xuwanzi.cn` 就是最终要使用的域名。

创建 secret。

```shell
kubectl create secret tls k8s-dashboard-tls --key tls.key --cert tls.crt -n k8s-dashboard
```

这个证书是用来做 https 访问用的。

# install

## 添加仓库

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

helm repo update
```

## install

准备 `values.yaml`

```yaml

ingress:
  # 启用 ingress
  enabled: true
  annotations:
    # alb.ingress.kubernetes.io/scheme: internet-facing
    # alb.ingress.kubernetes.io/target-type: ip
    # alb.ingress.kubernetes.io/group.name: alb-group
    kubernetes.io/tls-acme: 'true'
    # 根据文档提示选择如下 annotation 
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"

  className: "nginx"
  paths:
    - /
  customPaths: []
  hosts:
    - kdb.xuwanzi.cn
  tls:
    - secretName: dashboard-cert
      hosts:
        - kdb.xuwanzi.cn
        
extraArgs:
  - --token-ttl=86400

metricsScraper:
  ## Wether to enable dashboard-metrics-scraper
  enabled: true
  args:
    - --log-level=info
    - --logtostderr=true

```

安装命令。

```shell
helm install k8s-dashboard k8s-dashboard/kubernetes-dashboard -f values.yaml --version 6.0.6 -n k8s-dashboard
```

# 添加用户

> k8s 1.24 之后不再给 serviceAccount 自动生成 secret。所以这里得自动生成 token。

user 文件。

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: k8s-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: k8s-dashboard

```

```shell
kubectl apply -f user.yaml
```

集群里一般有默认的 `cluster-admin` 角色，如果没有就自己建一个。

secret 文件。

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: admin-user-token
  annotations:
    kubernetes.io/service-account.name: admin-user
```

> 注意 `kubernetes.io/service-account.name: admin-user` 这里让 secret 跟 `admin-user` 用户绑定。

```shell
kubectl apply -f token.yaml -n k8s-dashboard
```

## 查看结果

```shell
kubectl describe sa admin-user -n k8s-dashboard

Name:                admin-user
Namespace:           k8s-dashboard
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              admin-user-token
Events:              <none>
```

可以看到  `Tokens` 一项里已经跟 admin-user-token 绑定。

## 获取 Token

```shell
kubectl get secret admin-user-token -n k8s-dashboard -o jsonpath='{$.data.token}' | base64 -d | sed $'s/$/\\\n/g'
```

或者。

```shell
kubectl describe secret admin-user-token -n k8s-dashboard
```