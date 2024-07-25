# Windows 上安装 PostgreSQL

## 1. 安装数据库服务

### 1.1 数据库安装

官网下载，双击安装即可。

### 1.2 安装语言附件

没有语言附件的话，用 `pgAdmin 4` 管理数据库的时候，尤其是错误信息码，可能是乱码。可以选择安装 `EDB Language V4.2-1` 附件，当然也可以选择不安装，因为服务器端的操作比较少。

- 语言附件

  `edb_languagepack_4.exe`

### 1.3 数据库语言

当前，PostgreSQL 在服务器端，仅仅支持 UTF-8，不能选择用 `GB18030` 或 `GBK` 内码，来创建数据库。
 代码开发的时候，必须处理好 UTF8 和 ANSI 字符之间的变换。

## 2. 创建数据库

## 3. 创建数据表

## 4. 开放对外服务

- 默认只对本机开放，修改 `C:\Program Files\PostgreSQL\16\data\pg_hba.conf` ，修改之后需

  要重启数据库的相关服务。

  找到如下内容（`IPv4 local connections`）：

  ```
  # TYPE  DATABASE        USER            ADDRESS                 METHOD
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            scram-sha-256
  ```

  上述内容后面追加如下行：

  ```
  ## 允许 IP 地址从 192.168.0.1 到 192.168.0.254 的主机（客户端）。
  host all all 192.168.0.0/24 scram-sha-256
  ```

  本人虚拟机的实际情况（需要根据实际网络情况，进行设置）：

  ```
  ## C类网络
  host all all 192.168.114.0/24 scram-sha-256
  ## B类网络
  host all all 192.168.0.0/16 scram-sha-256
  ## A类网络（基本用不到）
  host all all 192.0.0.0/8 scram-sha-256
  ```

- 对外开放数据库端口

  打开防火墙，在 `入站规则` 增加 `PostgreSQL` 的端口号（默认`5432`，货检系统采用 `15432`）。

注意：如果有 2 个网卡的话，需要把 2 个IP的 掩码都添加，否则会造成有的访问正常，其它访问不正常的现象。