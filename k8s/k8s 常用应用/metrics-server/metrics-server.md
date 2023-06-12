# repo

```shell
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

helm repo update
```

# install

```yaml
args:
  - --kubelet-insecure-tls
```

安装。

```shell
helm install metrics-server metrics-server/metrics-server --version 3.9.0 -n kube-system
```