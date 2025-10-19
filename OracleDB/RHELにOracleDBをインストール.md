# RHELにOracleDBをインストール

> https://github.com/steveswinsburg/oracle21c-docker
>
> https://hub.docker.com/r/gvenzl/oracle-xe
>
> https://qiita.com/shakiyam/items/c9e60fbf654a5cbe8567
>
> 公式サイト：
>
> - Oracle Database Enterprise Edition Installation Guide for Docker Containers on Oracle Linux
>
>   https://docs.oracle.com/en/database/oracle/oracle-database/21/deeck/index.html
>
> - About this Docker Image for Oracle Database
>
>   Review your deployment options for this image.
>
>   The Oracle Database Enterprise Edition Docker image contains Oracle Database 21c Enterprise Edition, with the option to deploy either Enterprise Edition or Standard Edition, running on Oracle Linux 7 (x86-64). This image contains a default database in a multitenant configuration, with one pluggable database.
>
>   https://docs.oracle.com/en/database/oracle/oracle-database/21/deeck/index.html#DEEDK-GUID-EDA557B2-B0D6-45E1-8FBD-C1D756803982
>
> - AA
>
>   ```
>   
>   ```
>
>   
>
>   https://docs.oracle.com/en/database/oracle/oracle-database/21/deeck/index.html#DEEDK-GUID-EDA557B2-B0D6-45E1-8FBD-C1D756803982

## スナップショット 1

系统安装结束

## スナップショット 2

Docker-engieer安装结束

### 安装过程

> 参考 ： https://www.server-world.info/query?os=CentOS_Stream_9&p=docker&f=1



1. `docker` が正常にインストールされたか確認するため、コンテナでテスト用イメージを実行します。

   > 参考 ： https://docs.docker.jp/engine/installation/linux/rhel.html#id7

   ```
   [liu@localhost ~]$ sudo docker run hello-world
   [sudo] liu 的密码：
   Unable to find image 'hello-world:latest' locally
   latest: Pulling from library/hello-world
   2db29710123e: Pull complete 
   Digest: sha256:c77be1d3a47d0caf71a82dd893ee61ce01f32fc758031a6ec4cf1389248bb833
   Status: Downloaded newer image for hello-world:latest
   
   Hello from Docker!
   This message shows that your installation appears to be working correctly.
   
   To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
       (amd64)
    3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
       to your terminal.
   
   To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash
   
   Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/
   
   For more examples and ideas, visit:
    https://docs.docker.com/get-started/
   
   ```

   

### docker グループの作成

1. `docker` グループを作成し、ユーザを追加します。

   ```
   [liu@localhost ~]$ sudo usermod -aG docker liu
   ```

2. ログアウトしてから、再度ログインします。

   しなければ、有効になりません。

3. `sudo` を使わずに `docker` が実行できることを確認します。

   ```
   [liu@localhost ~]$ docker run hello-world
   ```

   

### ブート時の Docker 開始設定

- Docker をブート時に起動するようにするには、次のように実行します。

  ```
  $ sudo chkconfig docker on
  ```

  这个指令有问题，可以找其它网页的设置方法作为参考。

- Docker デーモンを開始します。

  ```
  $ sudo service docker start
  ```

  

### `git` 安装

> https://www.server-world.info/query?os=CentOS_Stream_9&p=git&f=1

- Git をインストールします。

  ```
  dnf -y install git
  ```

  

## スナップショット 3

> 参考 ： https://zenn.dev/msksgm/articles/20211225-oracle-database-19c-docker

- Oracle Database 19c をダウンロード

  https://www.oracle.com/database/technologies/oracle-database-software-downloads.html

- リポジトリをクローン

  ```bash
  git clone https://github.com/oracle/docker-images.git
  cd docker-images
  ```

- ダウンロードしたファイルを配置する

  ダウンロードした zip ファイルを`OracleDatabase/SingleInstance/dockerfiles/21.3.0`に配置します。

  ```bash
  cd OracleDatabase/SingleInstance/dockerfiles/21.3.0
  mv ~/Downloads/LINUX.X64_213000_db_home.zip .
  ```

- ビルドする

  バージョンは 19.3.0 なので `-v 21.3.0`を指定、Enterprise Edition なので `-e`を指定しました。`-i`は MD5 のチェックサムを無視するらしいです、これがないとうまく Oracle を起動できなかったので、指定しました。

  ```bash
  cd ..
  ./buildContainerImage.sh -v 21.3.0 -e -i
  ```

  以下のメッセージがでたら成功です。

  ```
  Oracle Database container image for 'ee' version 19.3.0 is ready to be extended:
  
  --> oracle/database:19.3.0-ee
  
  Build completed in 663 seconds.
  ```

- docker-compose.yaml を作成

  `/docker-images/OracleDatabase/SingleInstance/dockerfiles`に`docker-compose.yaml`を作成します（後述する`oradata`と同じ階層にあれば、どこのディレクトリでも問題ないです）。

  `docker-compose.yaml`：

  ```yaml
  version: "3.9"
  services:
    db:
      image: oracle/database:21.3.0-ee
      ports:
        - 1521:1521
        - 5500:5500
      volumes:
        - ./oradata:/opt/oracle/oradata
      environment:
        - ORACLE_PWD=Oracle21
        - ORACLE_PDB=oracle
  ```

  データを保存しておくための、ディレクトリ（`oradata`）を作成します。

  ```bash
  mkdir oradata
  chmod 777 orada
  ```

- コンテナを作成

  コンテナを作成します。

  ```
  docker compose up
  ```

  docker 起動ログに以下の表示がされたら成功です。

  ```bash
  #########################
  DATABASE IS READY TO USE!
  #########################
  ```

- Oracle に接続する

  以下のコマンドで、Oracle に入ります。

  ```bash
  docker compose exec db sqlplus SYSTEM/Oracle21@ORCLCDB
  ```

  以下のように、接続できたら成功です。

  ```bash
  SQL*Plus: Release 19.0.0.0.0 - Production on Sat Dec 25 08:13:50 2021
  Version 19.3.0.0.0
  
  Copyright (c) 1982, 2019, Oracle.  All rights reserved.
  
  Last Successful login time: Sat Dec 25 2021 08:07:58 +00:00
  
  Connected to:
  Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
  Version 19.3.0.0.0
  
  SQL>
  ```

- SQL Developer で接続する

  以下から、SQL Developer をダウンロードします。

  https://www.oracle.com/jp/database/sqldeveloper/technologies/download/

  ダウンロードできたら、左上の緑色の「＋」マークをクリックすることで、接続の設定がでてきます。
  以下のように入力した後に「接続」ボタンをクリックすると、接続できます。

  | 項目                 | 入力内容              |
  | -------------------- | --------------------- |
  | Name                 | Oracle docker（任意） |
  | データベースのタイプ | Oracle                |
  | ユーザー名           | system                |
  | パスワード           | Oracle19              |
  | ホスト名             | localhost             |
  | ポート               | 1521                  |
  | SID                  | ORCLCDB               |

  ![img](https://res.cloudinary.com/zenn/image/fetch/s--r3gwZqDR--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/89074e6869f11926968f4991.png%3Fsha%3Da50f62e436ba303a189781c7128b34eb713c6c8f)





## Oracle 公式サイトの説明

> https://docs.oracle.com/en/database/oracle/oracle-database/21/deeck/index.html#DEEDK-GUID-6589D4A1-14F3-42B3-8461-C9A7B840D148

```
docker run -d --name container_name \
 -p host_port:1521 -p host_port:5500 \
 -e ORACLE_SID=cdb-system-identifer \
 -e ORACLE_PDB=pdb-name \
 -e ORACLE_PWD=oracle-user-password \
 -e INIT_SGA_SIZE=cdb-database-sga-memory-in-mb \
 -e INIT_PGA_SIZE=cdb-database-pga-memory-in-mb \
 -e ORACLE_EDITION=ee-or-se-database-edition \ 
 -e ORACLE_CHARACTERSET=character-set \
 -e ENABLE_ARCHIVELOG=[true|false]
 -v [host-mount-point:]/opt/oracle/oradata \
container-registry.oracle.com/database/enterprise:21.3.0
```

```
docker run -d --name container_name \
 -p host_port:1521 -p host_port:5500 \
 -e ORACLE_SID=cdb-system-identifer \
 -e ORACLE_PDB=pdb-name \
 -e ORACLE_PWD=oracle-user-password \
 -e INIT_SGA_SIZE=cdb-database-sga-memory-in-mb \
 -e INIT_PGA_SIZE=cdb-database-pga-memory-in-mb \
 -e ORACLE_EDITION=ee-or-se-database-edition \ 
 -e ORACLE_CHARACTERSET=character-set \
 -e ENABLE_ARCHIVELOG=[true|false]
 -v [host-mount-point:]/opt/oracle/oradata \
container-registry.oracle.com/database/enterprise:21.3.0

Parameters:
 --name:                 The name of the container. (Default: auto-generated
 -p:                     The port mapping of the host port to the container port.
                         Two ports are exposed: 1521 (Oracle Listener), 5500 (OEM Express)
 -e ORACLE_SID:          The Oracle Database SID that should be used.(Default:ORCLCDB)
 -e ORACLE_PDB:          The Oracle Database PDB name that should be used. (Default: ORCLPDB1)
 -e ORACLE_PWD:          The Oracle Database SYS, SYSTEM and PDBADMIN password. (Default: auto-generated)
 -e INIT_SGA_SIZE:       The total memory in MB that should be used for all SGA components (Optional)
 -e INIT_PGA_SIZE:       The target aggregate PGA memory in MB that should be used for all server processes attached to the instance (Optional)
 -e ORACLE_EDITION:      The Oracle Database Edition [enterprise|standard]. (Default: enterprise)
 -e ORACLE_CHARACTERSET: The character set that you want used when creating the database. (Default: AL32UTF8)
 -e ENABLE_ARCHIVELOG    The ARCHIVELOG mode. By default, set to false. 
                         If set to true, then ARCHIVLOG mode is enabled in the database (for fresh database creation only)
 -v /opt/oracle/oradata
                         The data volume that you want used for the database. Must be writable by the oracle user (uid: 54321) inside the container
                         If omitted, then the database will not be persisted over container recreation.
 -v /opt/oracle/scripts/startup | /docker-entrypoint-initdb.d/startup
                         Optional: A volume with custom scripts to be run after database startup.
                         For further details see the section "Running scripts after setup and on
                         startup" section below.
 -v /opt/oracle/scripts/setup | /docker-entrypoint-initdb.d/setup
                         Optional: A volume with custom scripts that you want run after database setup.
                         For further details see the "Running scripts after setup and on startup" section below.
```



```
$ docker run -d --name oracle-db \
  container-registry.oracle.com/database/enterprise:21.3.0.0
```



### インストール手順

https://docs.oracle.com/en/database/oracle/oracle-database/21/ladbi/running-oracle-universal-installer-to-install-oracle-database.html#GUID-DD4800E9-C651-4B08-A6AC-E5ECCC6512B9

- [Downloading the Installation Archive Files from the Oracle Database Website](https://docs.oracle.com/en/database/oracle/oracle-database/21/ladbi/downloading-the-installation-archive-files-from-otn.html#GUID-698509F0-8BB5-494D-B80F-C752F38AA255)

  打鍵端末Windows利用して、ダウンロードできる。

- [Downloading the Software from Oracle Software Delivery Cloud Portal](https://docs.oracle.com/en/database/oracle/oracle-database/21/ladbi/downloading-the-software-from-oracle-software-delivery-cloud-portal.html#GUID-34A70DC4-170F-41CD-8C7A-F4EE75B0EDAB)

  対象RHEL8.4は画面がないで、処理が進めることできない。



### ダウンロードサイト

- https://www.oracle.com/database/technologies/oracle-database-software-downloads.html
