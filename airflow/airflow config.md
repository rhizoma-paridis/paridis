airflow 配置文件是 airflow.cfg。使用 `airflow info` 会生成默认配置文件。更详细的解释可以参考[官方文档](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html)。
常用配置项如下。

```ini
[core]
# dags 文件位置
dags_folder = /data/airflow/dags
# 配置执行器，因为没有太多脚本所以没有用分布式 Celery Executor
executor = LocalExecutor
# 配置 sql 链接
sql_alchemy_conn = mysql+pymysql://user:password@host:port/db
# 不加载示例文件
load_examples = False
# False:执行任务前重新加载 plugins
lazy_load_plugins = False
[webserver]
# 邮件中访问连接，设置为访问地址
base_url = http://54.189.249.136:8080

[email]
# 配置邮件内容模板
html_content_template = /data/airflow/email_template/content_template.j2
subject_template = /data/airflow/email_template/subject_template.j2
[smtp]
# 这些根据情况配置
smtp_host = smtp.exmail.qq.com
smtp_starttls = False
smtp_ssl = True
# 切记与 smtp_mail_from 一样，都配置为发件人邮箱
smtp_user = data_public@gmail.com
# 邮箱第三方授权码
smtp_password = xxxx
smtp_port = 465
smtp_mail_from = data_public@gamil.com
smtp_timeout = 30
smtp_retry_limit = 5
```

^481e37

这里的配置并不是使用 helm 安装时使用的配置。所以在 k8s 中使用 airflow 的配置，可以参考 values.yaml 文件中的配置项。