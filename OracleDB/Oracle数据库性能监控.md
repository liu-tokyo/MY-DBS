# Oracle数据库性能监控

## 1、安装和配置oralcedb_exporter

### 1.1 使用[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)安装oralcedb_exporter

这里以容器的方式提供oracledb_exporter，所使用的镜像为iamseth/oracledb_exporter，以守护进程方式运行此容器。对外暴露9161端口，通过DATA_SOURCE_NAME环境变量指定要监控的[oracle](https://so.csdn.net/so/search?q=oracle&spm=1001.2101.3001.7020)数据库。

$ docker run -d –name oracle -p 1521:1521 wnameless/oracle-xe-11g:16.04

$ docker run -d –name oracledb_exporter –link=oracle -p 9161:9161 -e DATA_SOURCE_NAME=system/oracle@oracle/xe iamseth/oracledb_exporter

### 1.2 查看oracle的指标数据

```
oracledb_exporter_last_scrape_duration_seconds
oracledb_exporter_last_scrape_error
oracledb_exporter_scrapes_total
oracledb_up
oracledb_activity_execute_count
oracledb_activity_parse_count_total
oracledb_activity_user_commits
oracledb_activity_user_rollbacks
oracledb_sessions_activity
oracledb_wait_time_application：
oracledb_wait_time_commit
oracledb_wait_time_concurrency
oracledb_wait_time_configuration
oracledb_wait_time_network
oracledb_wait_time_other
oracledb_wait_time_scheduler
oracledb_wait_time_system_io
oracledb_wait_time_user_io
oracledb_tablespace_bytes
oracledb_tablespace_max_bytes
oracledb_tablespace_bytes_free：
oracledb_process_count：oralce进程数
oracledb_resource_current_utilization：
oracledb_resource_limit_value：
```

![4507d31cdb6073df1c0817aa298345c1.png](https://img-blog.csdnimg.cn/img_convert/4507d31cdb6073df1c0817aa298345c1.png)

## 2、Prometheus监控

### 2.1 配置Prometheus

在Prometheus的配置文件(Prometheus.yaml)的最后面，添加体下面内容。

\# oracle数据库性能监控

– job_name: ‘oracledb’

static_configs:

– targets: [‘10.0.39.203:9161’]

### 2.2 配置验证

在浏览器的地址栏访问http://{prometheus}/targets，将会看到新配置的oracle。

![51f263ee1a55e875ebead6fb93f66c08.png](https://img-blog.csdnimg.cn/img_convert/51f263ee1a55e875ebead6fb93f66c08.png)

## 3 Grafana监控

### 3.1 配置mysql监控dashboard

下载mysql_exporter的dashboard(mysql-overview_rev5.json),在grafana中导入dashboard：mysql-overview_rev5.json。

![cac5e1cedfafa2234e473d47a6b485ea.png](https://img-blog.csdnimg.cn/img_convert/cac5e1cedfafa2234e473d47a6b485ea.png)

参考材料