# Oracle数据库性能监控

## 作业环境

- Ubuntu 22.04
- Docker （Docker version 20.10.22）
- Git （version 2.34.1）



## 1. 启动 OracleDB 数据库

主要用于测试，所以这次使用的是 Oracle XE 版本，该版本有如下限制：

- CPU上限：无论把数据库安装在多少核的服务器上，都只会提供一个CPU核心的运算能力；
- 安装和执行限制：只能安装一个实例且只能运行一个实例；
- 用户数据上限：最大11G的用户数据；
- 内存使用上限：最多1G；
- XE版本不支持分区表。

### 1.1 制作 OracleDB 镜像

- 启动终端，克隆制作 OracleDB 容器用的工程

  ```bash
  git clone https://github.com/oracle/docker-images.git
  ```

- 进入如下目录

  ```bash
  cd docker-images/OracleDatabase/SingleInstance/dockerfiles
  ```

- 制作 OracleDB-XE 的容器

  ```bash
  ./buildContainerImage.sh -v 21.3.0 -x
  ```

### 1.2 启动 OracleDB 镜像

- 调查生成的 Oracle 数据库容器的ID名称、ID

  ```
  docker images
  ```

  结果如下

  ```
  REPOSITORY        TAG         IMAGE ID       CREATED             SIZE
  oracle/database   21.3.0-xe   b1ecd1b49ce2   About an hour ago   6.53GB
  ```

- 启动数据库镜像

  ```
  sudo docker run \
  --name oracle21c \
  -p 1521:1521 \
  -p 5500:5500 \
  -e ORACLE_PDB=orcl \
  -e ORACLE_PWD=password \
  -e INIT_SGA_SIZE=3000 \
  -e INIT_PGA_SIZE=1000 \
  -v /opt/oracle/oradata \
  -d \
  oracle/database:21.3.0-xe
  ```

- 数据库启动状态确认

  ```
  docker ps -a
  ```

  `Up 3 minutes (health: starting)` 表明依然在启动，`Up 22 minutes (healthy)` 表明数据库已经启动结束，处于服务状态。

### 1.3 其它 OracleDB 镜像

- 网上有各种准备好的 Oracle 数据库版本，例如，可以用如下指令启动数据库：

  ```
  sudo docker run -d --name oracle -p 1521:1521 wnameless/oracle-xe-11g-r2:18.04-apex
  ```

  参考网站：`https://github.com/iamseth/oracledb_exporter`

  数据库口令应该为 ：`oracle`

## 2. 启动 oralcedb_exporter 

### 2.1 制作 oralcedb_exporter

- 参考网站

  https://github.com/iamseth/oracledb_exporter

  这是 Prometheus 官方推荐的网址，虽然是个人的网址，具有一定的可信性。

  该网站已经制作了多个版本的 `oracledb_exporter`，可以直接查询后，拉取相应的镜像：

  ```
  docker pull iamseth/oracledb_exporter
  ```

  当然您可以选择自己尝试构筑自己的 `oracledb_exporter` 镜像。

- 克隆如下网址：

  ```
  git clone https://github.com/iamseth/oracledb_exporter.git
  ```

- 构筑 oralcedb_exporter 

  ```
  docker build ./ -t oralcedb_exporter
  ```

- 查看构筑结果

  ```
  docker images
  ```

### 2.2 启动 oralcedb_exporter

- 指令

  ```
  docker run -d --name oracledb_exporter --link=oracle -p 9161:9161 -e DATA_SOURCE_NAME=system/oracle@oracle/xe iamseth/oracledb_exporter
  ```

  Since 0.2.1, the exporter image exist with Alpine flavor. Watch out for their use. It is for the moment a test.

  ```
  docker run -d --name oracledb_exporter --link=oracle -p 9161:9161 -e DATA_SOURCE_NAME=system/oracle@oracle/xe iamseth/oracledb_exporter:alpine
  ```

### 2.3 Oracle的指标数据

- 查看oracle的指标数据

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



## 3. Prometheus监控

### 3.1 配置Prometheus

在Prometheus的配置文件(Prometheus.yaml)的最后面，添加体下面内容。

\# oracle数据库性能监控

– job_name: ‘oracledb’

static_configs:

– targets: [‘10.0.39.203:9161’]

### 3.2 配置验证

在浏览器的地址栏访问http://{prometheus}/targets，将会看到新配置的oracle。

![51f263ee1a55e875ebead6fb93f66c08.png](https://img-blog.csdnimg.cn/img_convert/51f263ee1a55e875ebead6fb93f66c08.png)

## 4 Grafana监控

### 4.1 配置mysql监控dashboard

下载mysql_exporter的dashboard(mysql-overview_rev5.json),在grafana中导入dashboard：mysql-overview_rev5.json。

![cac5e1cedfafa2234e473d47a6b485ea.png](https://img-blog.csdnimg.cn/img_convert/cac5e1cedfafa2234e473d47a6b485ea.png)

参考材料