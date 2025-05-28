# 创建用户组：`groupadd`

**功能**：创建一个新的用户组。
**语法**：

```bash
groupadd [选项] 组名
```

**常用选项**：
`-g`：指定组的 GID（Group ID）。
`-r`：创建系统组（通常用于服务相关的组，GID 小于 1000）。
**示例**：

```bash
# 创建普通用户组 "dev"
groupadd dev

# 创建系统组 "sysadmin" 并指定 GID 为 1001
groupadd -g 1001 -r sysadmin
```

# 修改用户组属性：`groupmod`

**功能**：修改现有用户组的属性（如组名、GID 等）。
**语法**：

```bash
groupmod [选项] 组名
```

**常用选项**：
`-g`：修改组的 GID。
`-n`：修改组名（原组名需存在）。
**示例**：

```bash
# 将组 "dev" 的 GID 修改为 2000
groupmod -g 2000 dev

# 将组 "dev" 重命名为 "development"
groupmod -n development dev
```

# 删除用户组：`groupdel`

**功能**：删除一个不存在用户的用户组（若组内有用户，需先移除用户）。
**语法**：

```bash
groupdel 组名
```

**示例**：

```bash
# 删除用户组 "development"（需确保组内无用户）
groupdel development
```

# 管理组成员：`gpasswd`

**功能**：添加或删除组成员，设置组密码（较少使用）。
**语法**：

```bash
# 添加用户到组
gpasswd -a 用户名 组名

# 从组中删除用户
gpasswd -d 用户名 组名

# 设置组管理员（允许该用户管理组成员）
gpasswd -A 用户名1,用户名2 组名
```

**示例**：

```bash
# 将用户 "user1" 添加到组 "dev"
gpasswd -a user1 dev

# 从组 "dev" 中删除用户 "user1"
gpasswd -d user1 dev
```

# 查看用户组信息

**`cat /etc/group`**：查看所有用户组的详细信息（包含组名、GID、成员等）。

```bash
cat /etc/group
# 输出示例：
# dev:x:2000:user1,user2
# sysadmin:x:1001:
```

**`groups`**：查看当前用户所属的所有组。

```bash
groups  # 查看当前用户组
groups username  # 查看指定用户的组
```

# 批量管理用户组：`newgrp`

**功能**：切换当前用户的有效组（需用户属于该组）。
**语法**：

```bash
newgrp 组名
```

**示例**：

```bash
# 切换到 "dev" 组（需当前用户属于该组）
newgrp dev
```

# 注意事项

**删除含用户的组**：若用户组中存在成员，需先使用 `usermod -G` 修改用户的主组或次要组，或使用 `gpasswd -d` 移除所有成员，再删除组。
**系统组与普通组**：系统组（GID < 1000）通常由系统自动创建，用于服务或进程；普通组（GID ≥ 1000）用于普通用户管理。