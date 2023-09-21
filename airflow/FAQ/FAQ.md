## Please take a look at these docs to improve your DAG import time

这个问题是 airflow 在导入 dag 时超时。可以从两方面解决。

### 方案一

把  airflow 的导入 dag 超时时间加长。配置项是 `core - dagbag_import_timeout` 默认是 30s。改成160s。

### 方案二

优化 dag 代码。降低dag 的加载时间。相关资料可以参考[Best Practices — Airflow Documentation (apache.org)](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#reducing-dag-complexity) 里面也包含相关测试 dag 加载时间的测试。简单来说降低 dag 复杂度，降低代码复杂度。就是dag 越简单越好。
