执行 `docker build -t <imageName:imageTag> .`

Docker 客户端会将构建命令后面指定的路径(`.`)下的所有文件打包成一个 tar 包，发送给 Docker 服务端;

Docker 服务端收到客户端发送的 tar 包，然后解压，根据 Dockerfile 里面的指令进行镜像的分层构建；

**docker context 就是服务端上解压 tar 后的路径。它里面的内容跟 build 时指定的内容一样。**

在上面的命令中 `.` 路径会被打包发送到 server，所以解压后它就是 context。这也是为什么一些教程说 `.`是 context。但这条命令的 `.` 只是客户端的路径。

`docker build` 在不指定 dockerfile 时，默认从指定的 build 路径下查找 Dockerfile。所以如上命令就是从 `.` 中查找 Dockerfile。当然也可以使用 `-f` 参数指定 Dockerfile 的位置。

如下结构：

```text
helloworld-app
├── Dockerfile
└── docker
    ├── app-1.0-SNAPSHOT.jar
    ├── hello.txt
    └── html
        └── index.html

```

` docker build -f ./Dockerfile -t hello:v1 ./docker`

## 总结

为了简单就创建一个文件夹，把 Dockerfile 跟其他需要的文件都放到该文件夹下，在该目录下执行`docker build -t hell:v1 .` 

但在项目中一般不这样做，所以就添加 `.dockerignore` 文件屏蔽不需要的文件。

