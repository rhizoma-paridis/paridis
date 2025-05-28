# 1. 使用 `useradd` 命令创建用户

```bash
sudo useradd [选项] 用户名
```

常用选项：

- `-m`：创建用户的同时创建家目录（推荐使用，否则用户登录时可能没有家目录）。
- `-g`：指定用户的主用户组（默认为与用户名同名的组）。
- `-G`：添加用户到附加用户组（多个组用逗号分隔，如 `-G sudo,users`）。
- `-s`：指定用户的默认 shell（如 `-s /bin/bash`，默认为 `/bin/sh`）。
- `-d`：指定用户的家目录路径（默认 `/home/用户名`）。

**示例**：创建名为 `john` 的用户并生成家目录：

```bash
sudo useradd -m john
```

**创建用户时指定主组和附加组**：

```bash
sudo useradd -m -g developers -G sudo,adm john
```

主组为 `developers`，附加组为 `sudo`（允许使用 `sudo` 命令）和 `adm`。

## 最佳实践

使用默认主组，添加 docker, sudo 组，创建用户目录。

```bash
sudo useradd -m -G sudo,docker john
```

# 2. 为用户设置密码

用户创建后需要设置密码才能登录：

```bash
sudo passwd john
```

执行后按提示输入新密码（输入时不会显示字符），并重复确认一次。

# 3. 查看用户信息

查看用户详细信息（存储在 `/etc/passwd`）：

```bash
cat /etc/passwd | grep john
```

查看用户组信息（存储在 `/etc/group`）：

```bash
cat /etc/group | grep john
```

# 4. 修改用户组和权限

**修改已有用户的属性**（如添加附加组、更改家目录）：

```bash
sudo usermod -G sudo john          # 添加到 sudo 组
sudo usermod -d /home/new_john john  # 更改家目录
```

# 5. 删除用户

删除用户但保留家目录：

```bash
sudo userdel john
```

删除用户并删除家目录：

```bash
sudo userdel -r john
```
