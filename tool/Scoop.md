#tool

win 上目前也有好几个包管理工具了，像 choco，scoop, wget。但是 scoop 有点不同的是，它可以安装一些 linux 上的工具，比如 grep, fd 等等。

官方地址：[Scoop](https://scoop.sh/)

## install

默认安装的话按官方说明即可。scoop 安装软件的默认路径：个人用户路径 `~\scoop\` , 全局安装路径 `C:\ProgramData\scoop`。我觉得这两个路径没有问题，所以我个人不喜欢重置这两个路径。当然在安装时你可以自定义这两个路径。

```powershell
# 开启当前用户执行远程脚本权限
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
$env:SCOOP='D:\Applications\Scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
# 设置全局变量需要管理员权限
$env:SCOOP_GLOBAL='F:\GlobalScoopApps'
[Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', $env:SCOOP_GLOBAL, 'Machine')
irm get.scoop.sh | iex
```

## 使用

最基础的使用方法和其它包管理器类似，命令列表：

- `scoop search <app>` - 搜索软件
- `scoop install <app>` - 安装软件
- `scoop info <app>` - 查看软件详细信息
- `scoop list` - 查看已安装软件
- `scoop uninstall <app>` - 卸载软件，`-p`删除配置文件。
- `scoop update` - 更新 scoop 本体和软件列表
- `scoop update <app>` - 更新指定软件
- `scoop update *` - 更新所有已安装的软件
- `scoop checkup` - 检查 scoop 的问题并给出解决问题的建议
- `scoop help` - 查看命令列表
- `scoop help <command>` - 查看命令帮助说明

## bucket

所有的包管理器都会有相应的软件仓库 ，而 bucket 就是 Scoop 中的软件仓库。细心的你可能会发现 `scoop` 翻译为中文是 “舀”，而 `bucket` 是 “水桶”，所以安装软件可以理解为从水桶里舀水，似乎很形象的说。

Scoop 默认软件仓库（main bucket）软件数量是有限的，但是可以进行额外的添加。通过 `scoop bucket known` 命令可以查看官方认可的 bucket：

```shell
$ scoop bucket known
main
extras
versions
nightlies
nirsoft
php
nerd-fonts
nonportable
java
games
jetbrains
```

### 添加 bucket

```shell
scoop bucket add main
```

### 查看 bucket

```shell
scoop bucket list

Name   Source                                   Updated          Manifests
----   ------                                   -------          ---------
extras https://github.com/ScoopInstaller/Extras 2022/9/7 8:33:55      1675
java   https://github.com/ScoopInstaller/Java   2022/9/7 4:44:05       226
main   https://github.com/ScoopInstaller/Main   2022/9/7 8:35:03      1073
```

## 清理安装包缓存

Scoop 会保留下载的安装包，对于卸载后又想再安装的情况，不需要重复下载。但长期累积会占用大量的磁盘空间，如果用不到就成了垃圾。这时可以使用 `scoop cache` 命令来清理。

- `scoop cache show` - 显示安装包缓存
- `scoop cache rm <app>` - 删除指定应用的安装包缓存
- `scoop cache rm *` - 删除所有的安装包缓存

如果你不希望安装和更新软件时保留安装包缓存，可以加上 `-k` 或 `--no-cache` 选项来禁用缓存：

- `scoop install -k <app>`
- `scoop update -k *`

## 删除旧版本软件

当软件被更新后 Scoop 还会保留软件的旧版本，更新软件后可以通过 `scoop cleanup` 命令进行删除。

- `scoop cleanup <app>` - 删除指定软件的旧版本
- `scoop cleanup *` - 删除所有软件的旧版本

与安装软件一样，删除旧版本软件的同时也可以清理安装包缓存，同样是加上 `-k` 选项。

- `scoop cleanup -k <app>` - 删除指定软件的旧版本并清除安装包缓存
- `scoop cleanup -k *` - 删除所有软件的旧版本并清除安装包缓存

## 全局安装

全局安装就是给系统中的所有用户都安装，且环境变量是系统变量.

使用 `scoop install <app>` 命令加上 `-g` 或 `--global` 选项可对软件进行全局安装，全局安装需要管理员权限，所以需要提前以管理员权限运行的 Pow­er­Shell 。更简单的方式是先安装 `sudo`，然后用 `sudo` 命令来提权执行：

```none
scoop install sudo
sudo scoop install -g <app>
```

此外对于全局软件的更新和卸载等其它操作，都需要加上 `-g` 选项：

- `sudo scoop update -g *` - 更新所有软件（且包含全局软件）
- `sudo scoop uninstall -g <app>` - 卸载全局软件
- `sudo scoop uninstall -gp <app>` - 卸载全局软件（并删除配置文件）
- `sudo scoop cleanup -g *` - 删除所有全局软件的旧版本
- `sudo scoop cleanup -gk *` - 删除所有全局软件的旧版本（并清除安装包包缓存）

## 代理设置

Scoop 默认使用的是系统代理，如果你想手动指定代理，可以输入下面的命令。需要注意的是只支持 http 协议。配置文件位置 `C:\Users\xupeiqi\.config\scoop\config.json`。

```none
scoop config proxy 127.0.0.1:7890
```

> 设置完可以通过`scoop config proxy`查看。但是 scoop 很多东西需要从 github 中下载，所以要对 github 设置代理，参考[github proxy](../git/github/github.md#proxy)

如果你想取消代理，那么输入下面的命令，这将会恢复使用系统代理。

```none
scoop config rm proxy
```

## 开启多线程下载

使用 Scoop 安装 Aria2 后，Scoop 会自动调用 Aria2 进行多线程加速下载。

```none
scoop install aria2
```

使用 `scoop config` 命令可以对 Aria2 进行设置，比如 `scoop config aria2-enabled false` 可以禁止调用 Aria2 下载。以下是与 Aria2 有关的设置选项：

- `aria2-enabled`: 开启 Aria2 下载，默认`true`
- `aria2-retry-wait`: 重试等待秒数，默认`2`
- `aria2-split`: 单任务最大连接数，默认`5`
- `aria2-max-connection-per-server`: 单服务器最大连接数，默认`5` ，最大`16`
- `aria2-min-split-size`: 最小文件分片大小，默认`5M`

博主在这里推荐以下优化设置，单任务最大连接数设置为 `32`，单服务器最大连接数设置为 `16`，最小文件分片大小设置为 `1M`

```none
scoop config aria2-split 32
scoop config aria2-max-connection-per-server 16
scoop config aria2-min-split-size 1M
```

## 切换软件版本

像 java, python 这些软件会装多个版本。使用 scoop 的话可以方便切换。

```shell
scoop list
Installed apps:

Name            Version      Source Updated             Info
----            -------      ------ -------             ----
7zip            22.01        main   2022-09-07 13:48:15
fzf             0.32.1       main   2022-08-09 18:10:20
genact          0.12.0       main   2022-09-07 12:24:15
grep            3.7          main   2022-09-07 13:48:21
openjdk11       11.0.2-9     java   2022-08-30 18:39:29
openjdk8-redhat 8u342-b07    java   2022-08-30 18:50:46
sudo            0.2020.01.26 main   2022-08-09 18:03:33
fd              8.4.0        main   2022-08-17 17:27:05 Global install
```

上面有多个 Openjdk ，切换如下。

```shell
$ scoop reset openjdk11
Resetting openjdk11 (11.0.2-9).
Linking ~\scoop\apps\openjdk11\current => ~\scoop\apps\openjdk11\11.0.2-9

$ java -version
openjdk version "11.0.2" 2019-01-15
OpenJDK Runtime Environment 18.9 (build 11.0.2+9)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.2+9, mixed mode)
```

它其实就是重新设置了下环境变量。把旧的删除，新的加上。

## 问题

Couldn't find manifest for '7zip'? 

也会是缺少其他软件，只是因为 scoop 找不到相关软件，所以添加该软件所在的 bucket 。然后再执行安装。

## 常用软件

- sudo
- grep
- fd
- java
