## slim

仅安装运行特定工具所需的最少软件包,如果有空间限制并且不需要完整版本，请使用此tag,但是使用前需要经过完整测试。

以8-jre-slim这个tag为例，其中的slim表明当前的jre并非标准jre版本，而是headless版本，该版本的特点是去掉了UI、键盘、鼠标相关的库，因此更加精简，适合服务端应用使用，官方的建议是除非有明确的体积限制是再考虑使用该版本；

## alpine

alipine镜像基于alpine linux项目，是专门为容器内部使用而构建的操作系统。优点是尺寸小，缺点是不包含你可能需要的包，主要是用了一个更小的musl lib代替glibc，如果你的应用程序有特定libc需求，会遇到问题。

## buster/stretch/jessie

这三个是完整版镜像只不过对应的不同的 Debian代号

```text
buster:Debian 10
stretch:Debian 9
jessie:Debian 8
```

## jdk 相关

linux 发行版^[[https://zh.wikipedia.org/wiki/Linux发行版](https://zh.wikipedia.org/wiki/Linux发行版)]

### oracle

oracle, oraclelinux7表明镜像的操作系统是Oracle Linux 7

orclelinux8 则是基于 Oracle Linux 8.

## 总结

除了 slim 外，其他的都是基于一个 linux 发行版。

所以不在意体积就选 buster。最常用的话就是 alpine。