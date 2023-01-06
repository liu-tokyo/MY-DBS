# Windows 上安装 Oracle 数据库



## 安装 Oracle XE

### 官网下载

- https://www.oracle.com/database/technologies/xe-downloads.html

  当前（2023.01.06）的最新版本是 `Oracle Database 21c Express Edition` 。

  下载文件名称：`OracleXE213_Win64.zip`

即使是 XE 版本，文件大小也有将近 2GB 之大。

### 软件安装

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

  

## 创建插接式数据库

一个容器数据库CDB由一个根容器（CDBSEED）和0个、1个或多个可插拔数据库（PDB）组成。其中，

- 根容器包含一组主数据文件和元数据；
- 种子容器是用于创建其它可插拔数据库的模板；
- 可插拔数据库包括它自己的数据文件和应用程序对象（用户、表，索引等）。

**※**：12c之前的数据库都是非容器数据库（non-CDB）。

### 设置环境变量

- 设置环境变量：

  ```sh
  # E:\Oracle 为我的安装目录
  ORACLE_HOME=C:\app\liu\product\21c\dbhomeXE
  ORACLE_SID=XE
  ```

### 创建数据库

- 登录数据库：

  ```SQL
  SQLPLUS / AS SYSDBA
  ```

- 执行如下 SQL，创建名称为 `jpaas_system` 的数据库：

  数据库的数据位置：`C:\app\liu\product\21c\oradata`

  ```SQL
  -- 创建数据库
  CREATE PLUGGABLE DATABASE vip_db admin user vip_db identified by vip_db 
    STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 100M)
    DEFAULT TABLESPACE users datafile 'C:\app\liu\product\21c\oradata\XE\vip_db\vip_db.dbf' size 100M
    PATH_PREFIX ='C:\app\liu\product\21c\oradata\XE\vip_db\'
    FILE_NAME_CONVERT = ('C:\app\liu\product\21c\oradata\XE\PDBSEED\','C:\app\liu\product\21c\oradata\XE\vip_db\');
  
  ```

  Oracle数据库中，`jpaas_system` 会自动被转换为大写字符的 `JPAAS_SYSTEM` 。

  ```SQL
  -- 查看数据库
  SHOW PDBS;
  
  -- 切换当前会话到某个pdb中,并授权给管理用户
  ALTER SESSION SET CONTAINER=VIP_DB;
  
  -- 打开指定pdb
  ALTER PLUGGABLE DATABASE VIP_DB OPEN;
  
  -- 授予所有权利
  GRANT ALL PRIVILEGES TO VIP_DB;
  
  -- 关闭指定pdb
  ALTER PLUGGABLE DATABASE VIP_DB CLOSE IMMEDIATE ;
  
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

  

## 创建 AQ 数据表

- 批处理指令：

  ```SQL
  -- 更改会话对象 PDB
  ALTER SESSION SET container=VIP_DB;
  
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

- 查询创建结果

  ```
  
  ```

  