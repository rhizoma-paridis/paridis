k8s 对于初学都来说有一个很奇怪的点，就是用户，这个概念有点难理解。因为常规的系统用户都是系统内部管理的，但是 k8s 不是，它内部只有 serviceAccount。用户是外部系统管理的，可以用来链接 k8s，调用 k8s 功能。

所以 k8s 的用户是外部系统用户的 ID。对于 k8s 来说只是一个字符串，没有任务特殊性。那么这个 ID 要访问 k8s 的话，就需要 k8s 对它做一些权限限制，这时就用到 role, rolebinding 这些资源。那认证呢，认证就依赖 k8s 颁发的证书。对该用户 ID 生成一个 key，使用 k8s 的证书对该 key 颁发一个证书。这样就解决了认证问题。

## cluster

```shell
kubectl config --kubeconfig=config-demo set-cluster development --server=https://1.2.3.4 --certificate-authority=fake-ca-file
kubectl config --kubeconfig=config-demo set-cluster scratch --server=https://5.6.7.8 --insecure-skip-tls-verify
```

certificate-authority 的证书是集群的 ca.crt。

## user 与 group

```shell
# 生成 key
openssl genrsa -out my.key 2048
# 生成 key 的 csr 文件
openssl req -new -key my.key -subj "/CN=my/O=119" -out my.csr
# 使用 k8s 证书生成用户证书
openssl x509 -req -in my.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out my.crt -days 2
```

> 在 k8s 中证书的 subj 里  `CN` 用来表示用户名，`O` 用来表示用户组。跟一般数字证书中稍有不同。

经过上面步骤认证的材料已经准备好了。怎么跟 k8s 集群通信呢？通过 `kubectl` 。安装 `kubectl` 然后配置它。配置 `kubectl` 要访问的集群，使用的账号，使用的 `context`。

```shell
# 给 kubectl 配置要使用的用户
kubectl config set-credentials my --client-certificate=./my.crt --client-key=./my.key --embed-certs=true
# 配置要使用的 context，这里可以加上 --namespace=dev 参数限定用户的 namespace
kubectl config set-context my@cluster.local --cluster=cluster.local --user=my
# 切换 context
kubectl config use-context my@cluster.local
```

这时候使用 `kubectl` 已经不仅仅是一个用户 ID 了，它有证书了。而且这个证书是 k8s 集群颁发的。所以肯定是能认证通过的。

那这个用户认证过了之后就是授权，所以还需要在 k8s 中给这个用户授权。这就要用 role，rolebinding。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: exview
subjects:
- kind: User
  name: my
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: exview
subjects:
- kind: Group
  name: "119"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

上面给 user, group 授权都可以，使用的 role 是集群默认的 view。只有查看权限。

## serviceAccount

#todo

## 删除 config

--kubeconfig 是指定配置文件。配置分开管理是合理的，但是一般偷懒就用默认的 `~/.kube/config`。

-   要删除用户，可以运行 `kubectl --kubeconfig=config-demo config unset users.<name>`
-   要删除集群，可以运行 `kubectl --kubeconfig=config-demo config unset clusters.<name>`
-   要删除上下文，可以运行 `kubectl --kubeconfig=config-demo config unset contexts.<name>`

## 参考资料

- [配置对多集群的访问 | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
- [Kubernetes集群添加用户 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1825482)
- [Kubernetes（k8s）权限管理RBAC详解 - 掘金 (juejin.cn)](https://juejin.cn/post/7116104973644988446#heading-0)