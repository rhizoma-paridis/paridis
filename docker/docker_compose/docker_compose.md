## install

[Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)

```Bash
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```

## option

`-f` 指定文件

如文件 `test.yml` 启动：`docker-compse -f ./test.yml  up -d`

> 不指定文件时，docker-compose 默认会在当前文件夹下查找如下文件  `docker-compose.yml`, `docker-compose.yaml`,  `compose.yml`, `compose.yaml` 。**docker-compose 所有命令都依赖于当前使用的配置文件。**

## 常用命令

> 所有命令参数都是服务名，即 yaml 文件中的 serviceName。无参数时则是对配置中的所有服务操作。

```bash
# 命令格式 
docker-compose [-f <arg>...] [--profile <name>...] [options] [COMMAND] [ARGS...]
# 示例
docker-compose up -d nginx
docker-compose down

```

| command               | option | desc          |
| --------------------- | ------ | ------------- |
| up                    | -d     | 构建/pull，启动容器  |
| down                  |        | 删除容器，image    |
| logs                  | -f     | 查看日志          |
| start/stop/restart/rm |        | 启动/停止/重启/删除容器 |

## 常用服务

```yaml
version: '3'
services:
  mysql:
    hostname: mysql
    image: mysql:5.7
    # 如果需要容器使用宿主机IP(内网IP)，则可以配置此项，一般注释掉
    # network_mode: "host" 
    # 指定容器名称，如果不设置此参数，则由系统根据当前所在文件夹自动生成，建议设置。
    container_name: mysql 
    restart: always # 设置容器自启模式
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci 
    # 依赖服务
    depends_on: 
      - hello
    environment:
      - TZ=Asia/Shanghai # 设置容器时区与宿主机保持一致
      - MYSQL_ROOT_PASSWORD=123456 # 设置root密码
    volumes:
      # 卷挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式 （HOST:CONTAINER:ro）。
       - /etc/localtime:/etc/localtime:ro 
       - /docker/mysql-redis/mysql:/var/lib/mysql/data 
       - /docker/mysql-redis/my.cnf:/etc/mysql/my.cnf
    ports:
        # 暴露端口信息。
        # 使用宿主：容器 （HOST:CONTAINER）格式或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。
        # – “3000” 
        # – “8000:8000″ 
        # – “127.0.0.1:8001:8001″ 
        # 注：当使用 HOST:CONTAINER 格式来映射端口时，如果你使用的容器端口小于 60 
        # 你可能会得到错误得结果，因为 YAML 将会解析 xx:yy 这种数字格式为 60 进制。
        # 所以建议采用字符串格式。
        - "3306:3306"
```

### mysql

[mysql](mysql.md)

### mongo

[mongo](mongo.md)
