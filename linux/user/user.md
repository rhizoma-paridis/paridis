# group

# 添加

```shell
groupadd test
```

添加组 test。

## 删除

```shell
groupdel test
```

删除组 test。

# user

## 添加

```shell
useradd –G root test
```

添加用户 test。`-G` 附加用户到 root 组。

## 删除

```shell
userdel -r test
```

删除用户 test，`-r` 删除用户主目录。

## 修改

```shell
usermod -G momo test
```

给用户增加 momo 组。
