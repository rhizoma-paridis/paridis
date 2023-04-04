#git #github

## user

```Bash
# 查看所有的配置以及它们所在的文件
git config --list --show-origin
# 查看配置
git config --list
# 配置用户名邮箱
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com 
```

## proxy

全局设置

```shell
#使用http代理 
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890
#使用socks5代理
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890
```

不建议这么做，因为这样会把所有使用 git 的链接都使用代理，如果使用 gitee 就反倒麻烦。

所以建议只对 github 做代理。

```bash
#使用socks5代理（推荐）
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890
#使用http代理（不推荐）
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

**取消代理**

当你不需要使用代理时，可以取消之前设置的代理。

```bash
git config --global --unset http.proxy 
git config --global --unset https.proxy
```
