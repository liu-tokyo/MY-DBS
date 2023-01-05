# Ubuntu 上安装 PostgreSQL

## 1. 安装数据库

### 1.1 官方安装指令

> https://www.postgresql.org/download/linux/ubuntu/

```bash
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql
```

### 1.2 参照安装指令

```bash
# 要安装 PostgreSQL，首先刷新服务器的本地包索引：
sudo apt update

## 然后，安装 Postgres 软件包以及添加了一些附加实用程序和功能的 -contrib 软件包：
sudo apt install postgresql postgresql-contrib

## 确保服务已启动：
sudo systemctl start postgresql.service

## 重新启动服务
sudo service postgresql restart
```



### 1.3 查询服务状态

- 查询数据库服务状态

  ```bash
  ## 查询服务状态
  sudo service postgresql status
  ```

  出现类似如下信息：

  ```
  vip@vip:~$ sudo service postgresql status
  [sudo] password for vip:
  ● postgresql.service - PostgreSQL RDBMS
       Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
       Active: active (exited) since Thu 2023-01-05 15:29:17 UTC; 16min ago
      Process: 884 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
     Main PID: 884 (code=exited, status=0/SUCCESS)
          CPU: 1ms
  
  Jan 05 15:29:17 vip systemd[1]: Starting PostgreSQL RDBMS...
  Jan 05 15:29:17 vip systemd[1]: Finished PostgreSQL RDBMS.
  ```

  

### 1.4 登录到数据库

使用 PostgreSQL 角色和数据库

- 安装过程创建了一个名为 postgres 的用户帐户，该帐户与默认的 Postgres 角色相关联。 有几种方法可以利用此帐户访问 Postgres。 一种方法是通过运行以下命令切换到服务器上的 postgres 帐户：

  ```bash
  sudo -i -u postgres
  ```

- 查看版本、状态

  ```bash
  # 查看postgresql版本
  sudo -u postgres psql -c "SELECT version();"
  
  # 查看postgresql服务是否启动
  service postgresql status
  ```


### 1.5 SSH 远程连接

- 类似于如下指令

  ```
  ssh -p 9022 vip@localhost
  ```

  **※** 虚拟机的 22 端口映射到 宿主机的 9022 端口。

## 2. 允许远程访问

安装PostgreSQL[**数据库**](http://lib.csdn.net/base/mysql)之后，默认是只接受本地访问连接。如果想在其他主机上访问PostgreSQL数据库服务器，就需要进行相应的配置。

### 2.1 配置 postgres 用户密码

- 由于postgresql在安装时会默认添加一个用户名为postgres的ubuntu系统用户，因此要先用其他账户删除 postgres 用户的密码，再配置属于自己的密码。

  ```bash
  # 删除postgres用户的密码
  sudo passwd -d postgres
  
  # 设置postgres用户的密码
  sudo -u postgres passwd
  ```

  **※** 新口令：<span style="color:red">qwer1234</span>

配置完密码后，配置远程连接 PostgreSQL 数据库的步骤很简单，只需要修改data目录下的 **pg_hba.conf** 和 **postgresql.conf** 配置文件。

### 2.2 修改 pg_hba.conf

- 配置对数据库的访问权限：

  ```bash
  sudo nano /etc/postgresql/15/main/pg_hba.conf 
  ```

  在pg_hba.conf文件末尾追加以下内容：

  ```
  host    all             all             0.0.0.0/0               scram-sha-256
  ```

  也可以用如下办法，相对限定 IP 范围：

  ```
   # TYPE DATABASE  USER    CIDR-ADDRESS     METHOD
   # "local" is for Unix domain socket connections only
   local all    all               trust
   # IPv4 local connections:
   host  all    all    127.0.0.1/32     trust
   host  all    all    192.168.1.0/24    md5
   # IPv6 local connections:
   host  all    all    ::1/128       trust
  ```

  其中，第7条是新添加的内容，表示允许网段`192.168.1.0`上的所有主机使用所有合法的数据库用户名访问数据库，并提供加密的密码验证。

  其中，数字 24 是子网掩码，表示允许`192.168.1.0--192.168.1.255`的计算机访问！

### 2.3 修改 postgresql.conf

- 配置PostgreSQL数据库服务器的相应的参数：

  ```bash
  sudo nano /etc/postgresql/15/main/postgresql.conf
  ```

  将数据库服务器的监听模式修改为监听所有主机发出的连接请求。

  定位到 `#listen_addresses = 'localhost'`。PostgreSQL安装完成后，默认只接受来自本机 localhost 的连接请求。

  将行开头都#去掉，将行内容修改为 `listen_addresses = '*'` 来允许数据库服务器监听来自任何主机的连接请求！

  ```
  listen_addresses = '*' 
  password_encryption = scram-sha-256
  ```

### 2.4 重启 postgresql 服务

- 添加完成后，保存退出，然后重启 postgresql 服务

  ```bash
  sudo service postgresql restart
  ```

  

## 3. 连接数据库

> https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart

**连接方法-1**

- **Linux** 系统可以直接切换到 postgres 用户来开启命令行工具：

  ```bash
  sudo -i -u postgres
  ```
  
- 然后您可以通过运行以下命令访问 Postgres 提示符：

  ```bash
  psql
  ```

  这将使您登录到 PostgreSQL 提示符，从这里您可以立即自由地与数据库管理系统进行交互。

- 要退出 PostgreSQL 提示符，请运行以下命令：

  ```bash
  \q
  ```

- 这将使您返回到 postgres Linux 命令提示符。 要返回到您的常规系统用户，请运行 exit 命令：

  ```bash
  exit
  ```

**连接方法-2**

- 连接到 Postgres 提示符的另一种方法是直接使用 sudo 作为 postgres 帐户运行 psql 命令：

  ```bash
  sudo -u postgres psql
  ```

  这将使您直接登录到 Postgres，而无需中间的 bash shell。

- 同样，您可以通过运行以下命令**退出**交互式 Postgres 会话：

  ```bash
  \q
  ```



## 4. 创建新角色

### 4.1 创建新用户

- 如果您以 postgres 帐户登录，则可以通过运行以下命令来创建新角色：

  ```postgresql
  create user --interactive
  ```

- 相反，如果您更喜欢在不从普通帐户切换的情况下对每个命令使用 sudo，请运行：

  ```bash
  sudo -u postgres create user --interactive
  ```

- 无论哪种方式，脚本都会提示您做出一些选择，并根据您的回答执行正确的 Postgres 命令以创建符合您规范的用户。

  ```
  Output
  Enter name of role to add: sammy
  Shall the new role be a superuser? (y/n) y
  ```

**创建用户实例**

- 通过运行以下信息进行用户创建

  ```postgresql
  -- 创建不带密码的账号-root
  create role root;
  -- 创建带密码的账号-root
  create user root with 'root123';
  -- 显示所有用户，并查看是否创建成功
  \du
  -- 更改postgresql密码,如修改root账号的密码
  alter user root with password 'root123#';
  ```

### 4.2 用户权限授权

- 创建完用户密码以后，可以用以下命令对用户进行权限授予：

  ```postgresql
  -- 给用户root赋予超级用户权限
  alter user root with superuser;
  -- 给用户root2赋予普通权限
  alter user  root2 with privileges; 
  -- 把数据库的所有权限给予用户root
  grant all privileges on database test to root;
  -- 把某schema下(ods)的所有表(test用户下)的所有权限给予用户root2
  grant all on all tables in schema ods to root2;
  -- 把某张表（test_table）的所有权限给予用户root2
  grant all on test_table to root2;
  -- 把某张表（test_table）的查询权限给予用户root2
  grant select on table test_table to root2;
  -- 撤销用户root2的某个数据库(test)权限；
  revoke all on database test from root2;
  -- 撤销用户root2某个schema(ods)下所有表的修改(update)权限；
  revoke update on all tables in schema ods from root2;
  -- 撤销用户root2某张表（test_table）的所有权限
  revoke privileges on test_table from root2;
  -- 将某张表（test_table）的所有者给root2
  alter table test_table owner to root2;
  -- 允许用户(root)登录
  ALTER ROLE root WITH LOGIN;
  -- 禁止用户(root)登录
  ALTER ROLE root WITH NOLOGIN;
  ```

  

## 5. 创建数据库

Postgres 身份验证系统默认做出的另一个假设是，对于用于登录的任何角色，该角色将拥有一个它可以访问的同名数据库。  
这意味着如果您在上一节中创建的用户名为 sammy，则该角色将尝试连接到默认情况下也称为“sammy”的数据库。 您可以使用 createdb 命令创建适当的数据库。

- 如果您以 postgres 帐户登录，您将键入如下内容：

  ```bash
  createdb sammy
  ```

- 相反，如果您更喜欢在不从普通帐户切换的情况下对每个命令使用 sudo，则可以运行：

  ```bash
  sudo -u postgres createdb sammy
  ```

**创建数据库实例**

- 创建数据库

  ```postgresql
  -- 创建数据库(test)
  create database test;
  -- 删除数据库(test)
  drop database test;
  -- 查询所有数据库
  \l
  -- 切换数据库test
  \c test
  ```



## 6. 创建新架构

- 创建schema

  ```postgresql
  -- 创建schema(ods)，以下写法均可
  create schema ods;
  create schema if not exists ods;
  -- 为某个用户(root2)创建market
  create schema market authorization roo2;
  -- 查看当前数据库下的所有schema
  select * from information_schema.schemata;
  -- 删除schema(ods)
  drop schema if exists ods;
  ```

  

## 7. 使用新角色

- 使用新角色打开 Postgres 提示符

  要使用基于身份的身份验证登录，您需要一个与您的 Postgres 角色和数据库同名的 Linux 用户。

  如果没有可用的匹配 Linux 用户，可以使用 adduser 命令创建一个。 您必须从具有 sudo 权限的非根帐户执行此操作（意思是，不是以 postgres 用户身份登录）：

  ```bash
  sudo adduser sammy
  ```

- 一旦这个新帐户可用，您可以通过运行以下命令切换并连接到数据库：

  ```bash
  sudo -i -u sammy
  psql
  ```

  或者，您可以内联执行此操作：

  ```bash
  sudo -u sammy psql
  ```

  假设所有组件都已正确配置，此命令将自动让您登录。

- 如果您希望您的用户连接到不同的数据库，您可以通过如下指定数据库来实现：

  ```postgresql
  psql -d postgres
  ```

- 登录后，您可以通过运行以下命令检查当前的连接信息：

  ```postgresql
  \conninfo
  ```

  ```
  Output
  You are connected to database "sammy" as user "sammy" via socket in "/var/run/postgresql" at port "5432".
  ```



## 8. 使用Navicat链接

`Navicat` 是一套可创建多个连接的数据库管理工具，用以方便管理 MySQL、Oracle、PostgreSQL、SQLite、SQL Server、MariaDB 和 MongoDB 等不同类型的数据库，它与阿里云、腾讯云、华为云、Amazon RDS、Amazon Aurora、Amazon Redshift、Microsoft Azure、Oracle Cloud 和 MongoDB Atlas等云数据库兼容。你可以创建、管理和维护数据库。Navicat 的功能足以满足专业开发人员的所有需求，但是对数据库服务器初学者来说又简单易操作。Navicat 的用户界面 (GUI) 设计良好，让你以安全且简单的方法创建、组织、访问和共享信息。



## 9. 卸载数据库

- 卸载指令

  ```bash
  # 移除postgresql
  sudo apt-get --purge remove postgresql\*
  # 移除配置信息
  sudo rm -r /etc/postgresql/
  sudo rm -r /etc/postgresql-common/
  sudo rm -r /var/lib/postgresql/
  sudo userdel -r postgres
  sudo groupdel postgres
  ```



## A. 错误对应

### A.1 建立连接出错

- 连接的时候出现如下错误：

  ```
  Failed to establish connection.
  password authentication failed for user "postgres"
  ```

- 修改文件：`pg_hba.conf`

  ```
  host    all             all             0.0.0.0/0               scram-sha-256
  ```

  修改为

  ```
  host    all             all             0.0.0.0/0               trust
  ```

- 修改后，需要重启数据库

  ```bash
  sudo service postgresql restart
  ```

- 查看端口是否启动

  ```bash
  ## 可能需要安装 net-tools 包
  # sudo apt install net-tools
  netstat -lantp
  ```

- 查看进程是否启动

  ```bash
  ps aux | grep postgres
  ```

  

## 补：常用命令

| 功能                 | 命令                                                         | 备注                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 列出当前数据库所有表 | \dt                                                          |                                                              |
| 列出表名             | SELECT tablename FROM pg_tables <br/>WHERE tablename NOT LIKE 'pg%'<br/>AND tablename NOT LIKE 'sql_%' <br/>ORDER BY tablename; |                                                              |
| 列出数据库名         | \l                                                           |                                                              |
|                      | SELECT datname FROM pg_database;                             |                                                              |
| 切换数据库           | \c 数据库名                                                  |                                                              |
| 通过命令行查询       | \d 数据库                                                    | 得到所有表的名字                                             |
|                      | \d 表名                                                      | 得到表结构                                                   |
| 通过SQL语句查询      | select * from pg_tables                                      | 得到当前db中所有表的信息（这里pg_tables是系统视图）          |
|                      | select tablename from pg_tables where schemaname='public'    | 得到所有用户自定义表的名字（这里"tablename"字段是表的名字，"schemaname"是schema的名字。用户自定义的表，如果未经特殊处理，默认都是放在名为public的schema下） |

