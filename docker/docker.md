```dataview
list from "docker"
```

# 存储

查看当前存储路径

```Bash
docker info | grep "Docker Root Dir"
 Docker Root Dir: /var/lib/docker
```

停止 docker 服务

```Bash
[root@localhost ~]# systemctl stop docker
```

新建 docker 数据存储目录

```Bash
mkdir -p /data/docker/
```

迁移docker 数据

```Bash
rsync -avz /var/lib/docker /data/docker/
```

编辑 /etc/docker/daemon.json 文件修改存放位置，如果没有此文件就新建一个。

```Bash
[root@localhost ~]# vim /etc/docker/daemon.json 

{
  "data-root":"/data/docker"
}
```

重启 docker，查看结果

```Bash
[root@localhost ~]# systemctl restart docker
[root@localhost ~]# docker info | grep "Docker Root Dir"
 Docker Root Dir: /data/docker

```

删除原有数据

```Bash
[root@localhost ~]# rm -rf /var/lib/docker/
```
