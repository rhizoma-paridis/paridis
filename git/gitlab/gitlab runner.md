#gitlab

## docker install

```yaml
version: '3'
services:
  gitlab-runner:
    image: gitlab/gitlab-runner:ubuntu
    container_name: gitlab-runner
	volumes:
      - '/var/run/docker.sock:/var/run/docker.sock'
```

`volumes` 一定要如上挂载，[Use Docker to build Docker images | GitLab](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding) 这样避免使用 `dind`。因为 `dind` 必须设置 `privileged=true`。

## register

```shell
docker exec -it gitlab-runner gitlab-runner register

Runtime platform                                    arch=amd64 os=linux pid=63 revision=de104fcd version=14.5.1
Running in system-mode.

Enter the GitLab instance URL (for example, https://gitlab.com/):
# 填写 gitlab 地址
http://192.160.30.125

# 填写注册密钥
Enter the registration token:
xiFzX7Bz-yUajsUhSoTy

# 填写 desc
Enter a description for the runner:
[6fc9c9a97421]: public

# 填写 tag。使用时要与 stage tag 匹配
Enter tags for the runner (comma-separated):
public
Registering runner... succeeded                     runner=xiFzX7Bz

# executor
Enter an executor: custom, docker, docker-ssh, ssh, docker-ssh+machine, kubernetes, parallels, shell, virtualbox, docker+machine:
docker

# image
Enter the default Docker image (for example, ruby:2.6):
docker:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

当然也可以直接使用参数

```shell
gitlab-runner register -n \
   --url http://192.160.30.125\
   --registration-token xxxxxx \
   --executor docker \
   --description "guess_test" \
   --docker-image "docker:stable" \
   --docker-volumes /var/run/docker.sock:/var/run/docker.sock
   ## --docker-privileged
```

注册后的配置可以在 `/etc/gitlab-runner/config.toml` 里查看。