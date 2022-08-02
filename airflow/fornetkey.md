在 airflow 中加密 connection 密码。不使用的话在数据库表 `connection` 的 password 字段是明文存储。所以最好使用。

airflow 默认安装会提供一个。但是如果数据库里添加了 connection，重装后因为 fornetkey 改变会导致在页面无法查看 connections。这里只用把 connection 表清空，再重新添加即可。

fornetkey 有一定的要求，获取如下

```Python
pip install cryptography

python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"

```

## 配置

在传统 airflow 中的话，配置文件中配置如下。

```Bash
fernet_key = 81HqDtbqAywKSOumSha3BhWNOdQ26slT6K0YaZeZyPs=
```

在 helm 中的话。

```YAML
fernetKey: "poIO1LOlpixrUHwaR0byB448DVQEK2Gsx2c3H1VMVdQ="

```

当然直接配置环境变量也是可以的。

```Bash
AIRFLOW__CORE__FERNET_KEY='81HqDtbqAywKSOumSha3BhWNOdQ26slT6K0YaZeZyPs='
```

## rotate

Once connection credentials and variables have been encrypted using a fernet key, changing the key will cause decryption of existing credentials to fail. To rotate the fernet key without invalidating existing encrypted values, prepend the new key to the `fernet_key` setting, run `airflow rotate-fernet-key`, and then drop the original key from `fernet_key`:

1. Set `fernet_key` to `new_fernet_key,old_fernet_key`
2. Run `airflow rotate-fernet-key` to re-encrypt existing credentials with the new fernet key
3. Set `fernet_key` to `new_fernet_key`

官网文档一如即往的水平稳定，看看就行。说实话没看懂怎么操作。不过 stackoverflow 上有像样的操作。我简化一下。

1. Set the value of the fernet key (through your configmap or whatever method you use to pass the `AIRFLOW__CORE__FERNET_KEY` env value) as `ThisIsObviouslyAFakeNEWFernetKeyUJ_65Hgdjsmg=,ThisIsObviouslyAFakeOLDFernetKeyOP_78KufGSd3=` (new_fernet_key,old_fernet_key).
2. Relaunch you scheduler or deploy the whole Airflow.
3. Ssh into the scheduler pod and run `airflow rotate-fernet-key`.
4. Exit the pod and bring down your scheduler again.
5. Set the value of `AIRFLOW__CORE__FERNET_KEY` to `ThisIsObviouslyAFakeNEWFernetKeyUJ_65Hgdjsmg=` (only the new fernet key).
6. Relaunch your scheduler service.

helm 中的airflow 并没有直接在配置文件中配置 fornetkey 。所以它是把 configMap 中的数据放到环境变量中了。那么改 configMap （airflow_fornet_key）就可以了。data 如下。注意是 new_key 在前。

```YAML
kind: Secret
apiVersion: v1
metadata:
  name: airflow-fernet-key
  namespace: airflow
data:
  fernet-key: ZFd4YWNVcEJRMDlKZWpCS1pVSXdjbWhpY0ZNME1YZ3pNWFJoZFdweE5saz0=, asdfasdNVcEJRMDlKZWpCS1pVSXdjbWhpY0ZNME1YZ3pNWFJoZFdweE5saz0=
type: Opaque
```

当然不会使用上面方式。

```shell
kubectl create secret generic airflow-fernet-key \ 
--from-literal=fernet-key=ZFd4YWNVcEJRMDlKZWpCS1pVSXdjbWhpY0ZNME1YZ3pNWFJoZFdweE5saz0=, asdfasdNVcEJRMDlKZWpCS1pVSXdjbWhpY0ZNME1YZ3pNWFJoZFdweE5saz0=
```
