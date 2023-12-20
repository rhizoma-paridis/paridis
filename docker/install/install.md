# 安装 docker

https://docs.docker.com/engine/install/centos/

## yum update

```Bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

Install the yum-utils package (which provides the yum-config-manager utility) and set up the stable repository.

```Bash
sudo yum install -y yum-utils
```

```Bash
sudo yum-config-manager --add-repo [https://download.docker.com/linux/centos/docker-ce.repo](https://download.docker.com/linux/centos/docker-ce.repo)
```

## 阿里云yum源

```Bash
sudo yum-config-manager --add-repo  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
```

## INSTALL DOCKER ENGINE

```Bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

如果出错如下：

Error: 
 Problem: package docker-ce-3:19.03.13-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed


找合适的版本安装

```Bash
dnf install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.2.el7.x86_64.rpm
```

再安装 docker

```Bash
yum install docker-ce docker-ce-cli
```

Start Docker.

```Bash
systemctl start docker
```

设置开机启动

```Bash
 systemctl enable docker
```

# docker 镜像库

```Bash
 # 该配置文件及目录，在Docker安装后并不会自动创建
$ sudo mkdir -p /etc/docker

# 配置加速地址
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
   "registry-mirrors": ["https://wlo78bqa.mirror.aliyuncs.com"]
}
EOF
# 我自己的阿里云镜像库
# 重启服务
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

> 如果个人开发环境使用 win, 建议使用 wsl。