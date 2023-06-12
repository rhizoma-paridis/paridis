生成密钥

```Bash
ssh-keygen -t rsa

# 此处可写文件名，不写默认为 id
Enter file in which to save the key (/home/xupeiqi/.ssh/id_rsa): 
Created directory '/home/xxx/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/xupeiqi/.ssh/id_rsa.
Your public key has been saved in /home/xupeiqi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ofgu+8sSP5WDJQJ1qqR+nvSU63aGMCeNYXLSFE6eKu4 xupeiqi@10-8-51-20
The key's randomart image is:
+---[RSA 3072]----+
|   +o .          |
|  =..o           |
|  +=.   .        |
| =.*......       |
|o.* =..+S.       |
|+  =.+o +        |
| o o==o. .       |
|. + *=+o         |
| E o+OBo         |
+----[SHA256]-----+
 
```

复制密钥到 authorized_keys

```Bash
ssh-copy-id -i ./data_x.pub xupeiqi@192.32.187.85
```

win 中没有上个命令，可以直接把公钥内容复制到 `~/.ssh/authorized_keys` 文件中。

1. 确保 `authorized_keys` 权限为 600。
2. 保证 `.ssh` 文件夹权限为 700。

```Bash
 chmod 700 ~/.ssh
 chmod 600 ~/.ssh/authorized_keys
```

## 登录权限

```shell
sudo vim /etc/ssh/sshd_config
```

### 密码登录

修改如下配置：

```bash
PasswordAuthentication yes
ChallengeResponseAuthentication no
```

### 密钥登录

```bash
PermitRootLogin no
PubkeyAuthentication yes
GSSAPIAuthentication yes
GSSAPICleanupCredentials no
UsePAM yes
```

重启服务

```shell
systemctl restart sshd
```

## 代理设置

文件位置 `~/.ssh/config`

```text
Host 136
    HostName 1.1.1.136
    User datamanager
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/136

Host 119
    HostName 1.1.1.119
    User user
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/data
    ProxyCommand C:\Windows\System32\OpenSSH\ssh.exe -W %h:%p data_85
```

> 此处 ProxyCommand 里不能写 ssh 必须写路径名，或是：ssh.exe。否则总是报错。

## github

```JavaScript
Host github.com
    HostName github.com
    User xupeiqi
    IdentityFile ~/.ssh/xpq
```

此处 host 与 hostname 一定是 [github.com](http://github.com)

> 测试：ssh -T <git@github.com>
