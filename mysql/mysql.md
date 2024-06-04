### 创建用户

首先，您需要使用`CREATE USER`语句来创建一个新的MySQL用户帐户。这个语句的基本格式如下：

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

- `'username'` 是您想要创建的用户的名称。
- `'host'` 是用户可以连接的主机。可以使用 `'%'` 来表示任何主机，或者使用`'localhost'`表示本机连接，或者指定一个特定的IP地址。
- `'password'` 是用户的密码。出于安全考虑，建议使用强密码。

### 授权

创建用户后，您需要为该用户授权，以便他们可以访问和操作数据库。使用`GRANT`语句来授予权限：

```sql
GRANT privileges ON database.* TO 'username'@'host';
```

- `privileges` 是您想要授予的权限列表，例如`SELECT`, `INSERT`, `UPDATE`, `DELETE`等。
- `database.*` 表示授权的范围，可以是特定的数据库、数据表或所有数据库和表（使用`*.*`）。
- `'username'@'host'` 是您之前创建的用户的名称和主机。

如果您想要授予用户所有权限，可以使用`ALL PRIVILEGES`：

```sql
GRANT ALL PRIVILEGES ON *.* TO 'username'@'host' WITH GRANT OPTION;
```

`WITH GRANT OPTION`允许用户将他们的权限授予其他用户。

### 刷新权限

在授予权限后，您需要执行`FLUSH PRIVILEGES`命令来使更改立即生效：

```sql
FLUSH PRIVILEGES;
```
