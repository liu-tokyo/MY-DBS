# Windows 上安装 Oracle 数据库



## 1. 安装 Oracle XE

### 1.1 官网下载

- https://www.oracle.com/database/technologies/xe-downloads.html

  当前（2023.01.06）的最新版本是 `Oracle Database 21c Express Edition` 。

  下载文件名称：`OracleXE213_Win64.zip`

即使是 XE 版本，文件大小也有将近 2GB 之大。

### 1.2 软件安装

- 解压缩之后，点击 `setup.exe` 开始安装；

- 默认安装位置：`C:\app\liu\product\21c\` ；

- 数据库登录口令：`qwer1234`

- 其余：按照安装向导，继续安装即可。

  配置数据库要花费很长时间，可以去休息一会儿，顺便抽根烟。

- Oracle XE 连接信息：

  | 连接方式         | 连接地址                  | 备注 |
  | ---------------- | ------------------------- | ---- |
  | 多租户容器数据库 | localhost:1521            |      |
  | 插接式数据库     | localhost:1521/XEPDB1     |      |
  | EM Express URL   | https://localhost:5500/em |      |

- 插接式数据库名称：

  <span style="color:red">XEPDB1</span>

## 2. 安装 Oracle EE

### 2.1 官网下载

- https://www.oracle.com/database/technologies/oracle-database-software-downloads.html

  当前（2022.01.06）最新版本 `Oracle Database 21c`

  下载文件名称：`OracleXE213_Win64.zip`

### 2.2 软件安装

- 解压缩之后，点击 `setup.exe` 开始安装；

  需要拷贝到本地硬盘 `C:\WINDOWS.X64_213000_db_home`，虚拟机共享方式无法安装？不支持 UNC 路径方式，这一点和 XE 版本有所不同。看来商业版本 和 XE版本面向的对象不一样，所以在安装方法上，差异还是不小的。

  这样的话，不需要宿主机解压，用 `7.zip` 直接解压到虚拟机的本地硬盘，速度更快。

- 安装参数如下，基本采用默认参数：

  | 项目             | 参数             | 备考                       |
  | ---------------- | ---------------- | -------------------------- |
  | Oracle 基目录    | C:\DB            | `C:\` 这个默认的路径不能用 |
  | 数据库文件位置   | C:\oradata       |                            |
  | 数据库版本       | 企业版           |                            |
  | 字符集           | Unicode          |                            |
  | 全局数据库名称   | orcl.localdomain |                            |
  | 可插入数据库名称 | orclpdb          |                            |
  | 密码             | qwer1234         |                            |

  Oracle Enterprise Manager Database Express URL：https://localhost:5500/em

- 插接式数据库名称：

  <span style="color:red">ORCLPDB</span>

## 3. 创建插接式数据库

一个容器数据库CDB由一个根容器（CDBSEED）和0个、1个或多个可插拔数据库（PDB）组成。其中，

- 根容器包含一组主数据文件和元数据；
- 种子容器是用于创建其它可插拔数据库的模板；
- 可插拔数据库包括它自己的数据文件和应用程序对象（用户、表，索引等）。

**※**：12c之前的数据库都是非容器数据库（non-CDB）。

### 3.1 设置环境变量

- 设置环境变量：

  ```sh
  # E:\Oracle 为我的安装目录
  ORACLE_HOME=C:\app\liu\product\21c\dbhomeXE
  ORACLE_SID=XE
  ```

  **※** 估计 Linux 环境需要设置，至少 Windows 环境下面，没有设置如上变量，操作没啥问题。是否设置以后有些指令里面的内容可以通用化？

### 3.2 创建数据库

- 登录数据库：

  ```SQL
  SQLPLUS / AS SYSDBA
  ```

- 执行如下 SQL，创建名称为 `jpaas_system` 的数据库：

  数据库的数据位置：`C:\app\liu\product\21c\oradata`

  ```SQL
  -- 创建数据库
  CREATE PLUGGABLE DATABASE vip_pdb admin user vip_pdb_admin identified by Passw0rd 
    STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 100M)
    DEFAULT TABLESPACE users datafile 'C:\app\liu\product\21c\oradata\XE\vip_pdb\vip_pdb.dbf' size 100M
    PATH_PREFIX ='C:\app\liu\product\21c\oradata\XE\vip_pdb\'
    FILE_NAME_CONVERT = ('C:\app\liu\product\21c\oradata\XE\PDBSEED\','C:\app\liu\product\21c\oradata\XE\vip_pdb\');
  
  ```

  上面的指令在 XE 版本，没有问题；但是在 EE 版本，需要把 `C:\app\liu\product\21c\oradata` ，根据实际环境，替换为 ` C:\DB\oradata\ORCL` 。

  ```SQL
  CREATE PLUGGABLE DATABASE vip_pdb admin user vip_pdb_admin identified by Passw0rd 
    STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 100M)
    DEFAULT TABLESPACE users datafile 'C:\DB\oradata\ORCL\vip_pdb\vip_pdb.dbf' size 100M
    PATH_PREFIX ='C:\DB\oradata\ORCL\vip_pdb\'
    FILE_NAME_CONVERT = ('C:\DB\oradata\ORCL\PDBSEED\','C:\DB\oradata\ORCL\vip_pdb\');
  
  ```

  简化版：

  ```SQL
  CREATE PLUGGABLE DATABASE vip_pdb admin user vip_pdb_admin identified by Passw0rd 
    FILE_NAME_CONVERT = ('C:\DB\oradata\ORCL\PDBSEED\','C:\DB\oradata\ORCL\vip_pdb\');
  
  ```

  Oracle数据库中，`vip_pdb` 会自动被转换为大写字符的 `VIP_PDB` 。

  ```SQL
  -- 查看数据库
  SHOW PDBS;
  
  -- 切换当前会话到某个pdb中,并授权给管理用户
  ALTER SESSION SET CONTAINER=VIP_PDB;
  
  -- 打开指定pdb
  ALTER PLUGGABLE DATABASE VIP_PDB OPEN;
  
  -- 授予所有权利
  GRANT ALL PRIVILEGES TO VIP_PDB_ADMIN;
  
  -- 关闭指定pdb
  ALTER PLUGGABLE DATABASE VIP_PDB CLOSE IMMEDIATE ;
  
  -- 重启数据库，容器数据库自动重启，保持当前的状态。
  -- ORA-65040: 不允许从插接式数据库内部执行该操作
  ALTER PLUGGABLE DATABASE ALL SAVE STATE;
  
  ```

- 其它指令：

  ```SQL
  -- 查询表空间
  SELECT TABLESPACE_NAME FROM DBA_TABLESPACES;
  
  -- 显示当前用户
  SHOW USER；
  
  -- 查看用户状态
  SELECT USERNAME,COMMON,CON_ID FROM CDB_USERS WHERE ACCOUNT_STATUS = 'OPEN';
  
  -- 查看数据文件
  SELECT FILE_NAME, TABLESPACE_NAME FROM CDB_DATA_FILES;
  
  ```

  

## 4. 创建 AQ 数据表

### 4.1 创建 AQ

- 批处理指令：

  ```SQL
  -- 更改会话对象 PDB
  ALTER SESSION SET container=VIP_PDB;
  
  -- 创建用户 （user1）
  CREATE USER user1 IDENTIFIED BY "password" DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
  
  GRANT EXECUTE ON DBMS_AQ TO user1;
  GRANT EXECUTE ON DBMS_AQADM TO user1;
  GRANT AQ_ADMINISTRATOR_ROLE TO user1;
  GRANT ADMINISTER DATABASE TRIGGER TO user1;
  
  -- 适当的权限设置（授予DBA权限以确认通信）
  GRANT CONNECT TO user1;
  GRANT RESOURCE to user1;
  GRANT DBA TO user1; 
  
  -- 创建队列表
  BEGIN
    DBMS_AQADM.CREATE_QUEUE_TABLE(
      QUEUE_TABLE => 'user1.VIP_QUEUE_TABLE',
      QUEUE_PAYLOAD_TYPE => 'SYS.AQ$_JMS_MESSAGE'
    );
  END;
  /
  
  -- 创建队列
  BEGIN
    DBMS_AQADM.CREATE_QUEUE(
      QUEUE_NAME =>'user1.VIP_QUEUE',
      QUEUE_TABLE => 'user1.VIP_QUEUE_TABLE'
    );
  END;
  /
  
  -- 启动队列
  BEGIN
    DBMS_AQADM.START_QUEUE(
      'user1.VIP_QUEUE'
    );
  END;
  /
  
  ```

### 4.2 查询 AQ

- 查询创建结果

  ```SQL
  -- 显示当前容器
  SHOW CON_NAME;
  -- SYS_CONTEXT 函数是Oracle提供的一个获取环境上下文信息的预定义函数。该函数用来返回一个指定namespace下的parameter值。该函数可以在SQL和PL/SQL语言中使用。
  SELECT SYS_CONTEXT ('USERENV', 'CON_NAME') FROM DUAL;
  
  -- 查询当前容器信息
  SELECT CON_ID,DBID,NAME,OPEN_MODE FROM V$PDBS;
  
  -- 切换容器
  -- ALTER SESSION SET container=ORCLPDB;
  ALTER SESSION SET CONTAINER=XEPDB1;
  
  -- 查询创建的 QUEUE
  SELECT * FROM DBA_TABLES WHERE OWNER='USER1';
  SELECT * FROM ALL_TABLES WHERE OWNER='USER1';
  SELECT table_name FROM DBA_TABLES WHERE OWNER='USER1';
  
  -- 查询表名中含有 VIP 字符的数据表
  SELECT * FROM DBA_TABLES WHERE TABLE_NAME LIKE '%VIP%';
  SELECT * FROM ALL_TABLES WHERE TABLE_NAME LIKE '%VIP%';
  SELECT * FROM USER_TABLES WHERE TABLE_NAME LIKE '%VIP%';
  
  ```
  
  通过 A5M2 连接数据库之后，可以看到 AQ 的创建效果。



## 补：相关参考网站

- https://www.cnblogs.com/lottu/p/14946192.html

  比较详细的操作过程

- http://www.manongjc.com/detail/63-ibfhaqgtattecsz.html

  码农教程

- https://web.baimiaoapp.com/

  在线图片识别转化为文字

### 官方创建数据库文档

Oracle 官方[网站](https://docs.oracle.com/database/121/SQLRF/statements_6010.htm#SQLRF55686)有如下说明：

- **Creating a PDB by Using the Seed: Example**

  The following statement creates a PDB `salespdb` by using the seed in the CDB as a template. The administrative user `salesadm` is created and granted the `dba` role. The default tablespace assigned to any non-`SYSTEM` users for whom no permanent tablespace is assigned is `sales`. File names for the new PDB will be constructed by replacing `/disk1/oracle/dbs/pdbseed/` in the file names in the seed with `/disk1/oracle/dbs/salespdb/`. All tablespaces that belong to `sales` must not exceed 2G. The location of all directory object paths associated with `salespdb` are restricted to the directory `/disk1/oracle/dbs/salespdb/`.

  ```SQL
  CREATE PLUGGABLE DATABASE salespdb
    ADMIN USER salesadm IDENTIFIED BY password
    ROLES = (dba)
    DEFAULT TABLESPACE sales
      DATAFILE '/disk1/oracle/dbs/salespdb/sales01.dbf' SIZE 250M AUTOEXTEND ON
    FILE_NAME_CONVERT = ('/disk1/oracle/dbs/pdbseed/',
                         '/disk1/oracle/dbs/salespdb/')
    STORAGE (MAXSIZE 2G)
    PATH_PREFIX = '/disk1/oracle/dbs/salespdb/';
  ```




## 学习小结

从 **12C** 开始，Oracle DB 的理念有所变更（为了适应云端？），表现的是，指令也会发生变化，使用中要注意这些变化。