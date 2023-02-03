#gitlab

## install

```yaml
version: '3'
services:
  web:
    image: 'gitlab/gitlab-ee:15.8.0-ee.0'
    hostname: 'gitlab.com'
    container_name: gitlab-server
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.160.30.125'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    # volumes:
    #   - '$GITLAB_HOME/config:/etc/gitlab'
    #   - '$GITLAB_HOME/logs:/var/log/gitlab'
    #   - '$GITLAB_HOME/data:/var/opt/gitlab'
    # shm_size: '256m'
```

测试使用，使用不挂载 volumes。gitlab 运行时占用内存较大 shm_size 不要设置。如果出现 502 错误，把 docker 内存占用调大。

`external_url` 尽量使用 Ip/域名。哪怕是本地部署也不要使用 localhost ，否则容易网络不通。

初始用户是 `root` ，密码在 `/etc/gitlab/initial_root_password` 文件中。登录后修改密码 `user Setting -> Password`。