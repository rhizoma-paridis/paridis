#airflow

## 邮件模板

模板内容可以参考 `git@github.com:xpeiqi/k8sAirflow.git` 中的 emailTemplate。

## 邮件账号配置

官方不建议在 `airflow.cfg` 文件中配置账号密码。所以使用 connection 配置。如下图配置即可。

![smtp_default](attachments/Pasted%20image%2020220802175625.png)

默认 connection_id 使用 smtp_default 即可。
