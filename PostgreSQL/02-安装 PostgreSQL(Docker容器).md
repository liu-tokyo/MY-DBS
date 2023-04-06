# 安装 PostgreSQL - Docker容器

> Docker官方网站：https://hub.docker.com/_/postgres



## 指令启动方式

### 最简单方式

- 最简单指令

  ```bash
  docker run --name vip-postgres -e POSTGRES_PASSWORD='password' -d postgres
  ```

  但是，这么简单的启动方式，端口没有对外开放，所以只能内部访问；下面的启动方式，至少外面都可以访问：

  ```bash
  docker run -d --name postgres --restart always -e POSTGRES_USER='postgres' -e POSTGRES_PASSWORD='password' -e ALLOW_IP_RANGE=0.0.0.0/0 -p 5432:5432 -t postgres
  ```

  考虑到数据情况的话，下面的启动方式更为合适：

  ```bash
  docker run -d --name postgres --restart always -e POSTGRES_USER='postgres' -e POSTGRES_PASSWORD='password' -e ALLOW_IP_RANGE=0.0.0.0/0 -v /home/postgres/data:/var/lib/postgresql -p 5432:5432 -t postgres
  ```

  | 参数                        | 说明                                                       | 备注 |
  | --------------------------- | ---------------------------------------------------------- | ---- |
  | -e ALLOW_IP_RANGE=0.0.0.0/0 | 这个表示允许所有 IP 访问，如果不加，则非本机 IP 无法访问。 |      |
  | -e POSTGRES_USER=postgres   | 用户名                                                     |      |
  | -e POSTGRES_PASS=‘password’ | 指定密码                                                   |      |

  


### 相关操作指令

- 进入postgres容器

  ```bash
  docker exec -it 容器ID bash
  ```

- 修改编码格式

  ```SQL
  update pg_database set encoding = pg_char_to_encoding('UTF8') where datname = 'basemap'
  ```

- 查看pg版本

  ```SQL
  show server_version;
  # 或者
  select version();
  ```

- 尝试登录

  ```SQL
  # 登录数据库
  psql -U postgres -W
  ```

  

## **报错 psql**

<span style="Color:red">**重点：报错 psql: FATAL: Peer authentication failed for user "postgres"**</span>

- **#peer(不可信)，trust(可信)，md5(加密)**

  修改 /etc/postgresql/10/main/pg_hba.conf 文件

  找到下面这行

  ```
  local   all             postgres                                peer
  ```

  修改成md5（加密） （或改成 trust（可信））

  ```
  local   all             postgres                                md5
  ```

- **切换操作用户**

  ```bash
  # 切换成postgres用户
  su postgres
  ```

  尝试登录，成功。

- 重启容器

  ```
  docker restart 容器name
  ```

**完成！**