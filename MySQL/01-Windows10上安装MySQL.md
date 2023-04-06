# Windows10 上安装 MySQL

## 版本介绍

- MySQL Community Server 社区版本，开源免费，但不提供官方技术支持。 
- MySQL Enterprise Edition 企业版本，需付费，可以试用 30 天。 
- MySQL Cluster 集群版，开源免费。可将几个 MySQL Server 封装成一个 Server。
- MySQL Cluster CGE 高级集群版，需付费。 

本次尝试安装的是 CE 版本。

## 官方下载

- 官方网址

  http://www.mysql.com/

  跳转到 DOWNLOADS

  https://www.mysql.com/downloads/

  点击画面下部的 `MySQL Community (GPL) Downloads »`

  点击 `MySQL Installer for Windows` 链接（当前 2023.01.04 的最新版本：`MySQL Installer 8.0.31 `）？？？为啥找不到合适的版本？看来 CE 版本，没有64bit 的。

  https://dev.mysql.com/

  https://dev.mysql.com/downloads/mysql/

  https://dev.mysql.com/downloads/windows/installer/

  第一个安装包比较小, 第二个安装包比较大, 因为包含调试工具，我这里下载的是第一个 `Windows (x86, 64-bit), ZIP Archive`。
  
  选择下方的  `No thanks, just start my download.`，进入下载。
  
- 当前下载的最新版本安装文件：

  `mysql-installer-community-8.0.30.0.msi`

  `mysql-installer-community-8.0.31.0.msi`

## 软件安装 - CE版

- 双击 `mysql-installer-community-8.0.30.0.msi` 开始安装；

- 画面 `Choosing a Setup Type`

  `Developer Default`

  ![setuptype.png](https://storage.googleapis.com/zenn-user-upload/8d2f564caf09-20220309.png)

- 画面 `Check Requirements`

  ![CheckRequirements.png](https://storage.googleapis.com/zenn-user-upload/9c473e282ceb-20220309.png)

- 画面 `Installation`

  ![Installation.png](https://storage.googleapis.com/zenn-user-upload/5434a6a663d9-20220309.png)

  点击 **Execute** 按钮，开始安装，需要花费一定的时间。

  ![Installationfinish.png](https://storage.googleapis.com/zenn-user-upload/6faf436ee606-20220309.png)

- 画面 `Product Configuration`

  ![Product Configuration.png](https://storage.googleapis.com/zenn-user-upload/cc44edf5aef0-20220309.png)

- 画面 `Type and Networking`

  ![Networking.png](https://storage.googleapis.com/zenn-user-upload/ebc14efb5431-20220309.png)

- 画面 `Authentication Method`

  ![Authentication.png](https://storage.googleapis.com/zenn-user-upload/daf9244a85d8-20220309.png)

- 画面 `Accounts and Roles`

  `Qwer1234#`

  ![Accounts.png](https://storage.googleapis.com/zenn-user-upload/dc3dd1c5807c-20220309.png)

- 画面 `Windows Service`

  ![Service.png](https://storage.googleapis.com/zenn-user-upload/70684fd0b26a-20220309.png)

- 画面 `Apply Configuration画面`

  ![Configuration.png](https://storage.googleapis.com/zenn-user-upload/4f406906a123-20220309.png)

  点击 **Execute** 按钮

  ![Configuration.png](https://storage.googleapis.com/zenn-user-upload/570ac26a442e-20220309.png)

- 画面 `Product Configuration`

  ![Product.png](https://storage.googleapis.com/zenn-user-upload/2cb084a2b126-20220309.png)

- 画面 `MySQL Router Configuration`

  ![Router.png](https://storage.googleapis.com/zenn-user-upload/59a381696374-20220309.png)

- 画面 `Product Configuration`

  ![Product Configuration.png](https://storage.googleapis.com/zenn-user-upload/1220c0067f07-20220329.png)

- 画面 `Connect To Server`

  ![Connect To Server.png](https://storage.googleapis.com/zenn-user-upload/3b5738c75260-20220329.png)

- 画面 `Apply Configuration`

  ![Apply Configuration.png](https://storage.googleapis.com/zenn-user-upload/d9b38d3a15a4-20220329.png)

  点击 **Execute** 按钮

  ![Apply Configuration2.png](https://storage.googleapis.com/zenn-user-upload/3b4594a9b619-20220329.png)

- 画面 `Product Configuration`

  ![Product Configuration2.png](https://storage.googleapis.com/zenn-user-upload/8ac6895bdba4-20220329.png)

- 画面 `Installation Complete`

  ![Installation Complete.png](https://storage.googleapis.com/zenn-user-upload/4b35021b48a1-20220329.png)

  

### 安装参数

- 安装过程中，设置的各种参数：

  | 项目                 | 参数     | 备注           |
  | -------------------- | -------- | -------------- |
  | 网络协议             | TCP/IP   |                |
  | 开放端口             | 3306     | 默认           |
  | X Protocol Port      | 33060    | ？             |
  | Root Accout Password | P@ssw0rd | 管理用户：root |
  | Windows Service Name | MySQL80  | 默认           |



## ZIP方式 - CE版

安装版的mysql卸载起来太麻烦了，所以选择使用 ZIP 安装版的不失为一个好的选择，毕竟 ZIP 安装版的东西都在一个文件夹下，不要了直接删了文件夹就好。

- 把下载的 `mysql-8.0.31-winx64.zip` 文件，解压到 `C:\mysql-8.0.31-winx64` 目录下；

- 在 `C:\mysql-8.0.31-winx64` 目录下，添加一个 `my.ini` 文件，文件内容如下：

  ```ini
  [mysqld]
  basedir = C:\mysql-8.0.31-winx64       
  datadir = C:\mysql-8.0.31-winx64\data
  port = 3306
  lower_case_table_names = 2
  default_authentication_plugin=mysql_native_password
  ```

- 以管理员身份打开 PowerShell 进入mysql 的 bin 目录里，执行：

  ```bash
  .\mysqld --initialize --console
  ```

  执行内容：

  ```
  PS C:\mysql-8.0.31-winx64\bin> .\mysqld --initialize --console
  2023-01-07T12:10:49.553945Z 0 [Warning] [MY-010918] [Server] 'default_authentication_plugin' is deprecated and will be removed in a future release. Please use authentication_policy instead.
  2023-01-07T12:10:49.553990Z 0 [System] [MY-013169] [Server] C:\mysql-8.0.31-winx64\bin\mysqld.exe (mysqld 8.0.31) initializing of server in progress as process 6044
  2023-01-07T12:10:49.595646Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
  2023-01-07T12:10:50.231524Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
  2023-01-07T12:10:52.018868Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: fzqUt),z=9Ms
  PS C:\mysql-8.0.31-winx64\bin>
  ```

  其中 `zNmx!E:lg38<` 是为默认账号 root 生成的默认密码。

- 安装并启动服务：

  继续上一步的环境，执行如下指令，启动数据库服务。

  ```bash
  ## 之前安装过mysql服务但没有卸载干净的可能需要这一步
  .\mysqld --remove
  
  ## 安装服务     
  .\mysqld --install
  
  ## 启动服务
  net start MySQL
  ```

- 登录MySQL

  ```
  .\mysql -uroot -p
  ```

- 重置密码

  ```SQL
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'P@ssw0rd';
  ```

  把 root 用户密码设置为 `P@ssw0rd` 。

## 数据连接

### 本地连接

- MySQL Shell

  利用该工具，可以不需要登录直接进入 客户端工作台。

- MySQL 8.0 Command Line Client

  需要输入口令，进入 客户端工作台。
  
- A5M2 本地连接正常

### 远程连接

#### Windows 环境

注：如果是 ZIP 目录方式安装的话，需要在 防火墙 的 入站规则 里面，开放 3306 端口。Installer 方式安装的话，不需要这一步设置。

- 启动 `MySQL 8.0 Command Line Client`， 输入如下指令：

  ```SQL
  -- 切换数据库
  USE mysql;
  
  -- 查询用户授权信息
  SELECT USER,HOST FROM USER;
  
  -- 设置root用户对外开放
  UPDATE user SET host='%' WHERE user='root';
  
  -- 刷新授权
  FLUSH PRIVILEGES;
  ```

  用户表存在默认的`mysql`库里，`host`是能够使用该表连接到数据库的机器的host，默认是 `localhost`，也就是说默认只有本机能够连接，这里重新设成 '%'，也就是任意机器都行，那么就可以远程连接了（当然，这里只设置了`root`用户的远程连接权限，如果你想用其他用户远程连接，那么还是得给该用户进行这样的设置）。

- 其它授权指令参考

  高版本修改用户权限方法：

  ```SQL
  # 先创建远程用户
  CREATE USER 'root'@'%' IDENTIFIED BY  'password';
  
  # 授权
  GRANT ALL PRIVILEGES ON *.* to 'root'@'%' WITH GRANT OPTION;
  
  # 刷新授权
  FLUSH PRIVILEGES;
  ```

- 旧版本授权方式：

  你想user使用mypwd从任何主机连接到mysql服务器的话：
  
  ```SQL
  GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' IDENTIFIED BY 'mypwd' WITH GRANT OPTION;  
  FLUSH   PRIVILEGES;
  ```
  
  如果你想允许用户user从ip为192.168.1.4的主机连接到mysql服务器，并使用mypwd作为密码：
  
  ```SQL
   GRANT ALL PRIVILEGES ON *.* TO 'user'@'192.168.1.3' IDENTIFIED BY 'mypwd' WITH GRANT OPTION;   
   FLUSH   PRIVILEGES;
  ```



#### Linux 环境

- 参考文档：https://blog.csdn.net/Artisan_w/article/details/126359506
