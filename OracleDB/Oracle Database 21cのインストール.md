# Oracle Database 21cのインストール



## 事前準備

Red Hat Enterprise Linux の場合はインストール・ガイドからインストール要件を確認して設定を行います。必要なパッケージ等の要件は Oracle Database 19c とほとんど変化がありません。



## インストール

インストール方法は、Oracle Home ディレクトリに zip ファイルを展開するだけになっています。

```
$ export ORACLE_HOME=/u01/app/oracle/product/21.3.0/dbhome_1
$ mkdir -p $ORACLE_HOME
$ cd $ORACLE_HOME
$ unzip ~/LINUX.X64_213000_db_home.zip
```

展開が終わったら GUI 環境でインストーラーを起動します。

```
$ cd $ORACLE_HOME
$ ./runInstaller
```

インストール後に DBCA を実行するかを設定します。ここではインストールのみ行います。

### Select Configuration Option

![Select Configuration Option](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2Fedd4efe4-2b3b-9fb2-c2cf-420a2fd774bf.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=273dbbc0f5c82b5d3b407b2f009197c8)



### Select Database Edition

次の画面は、シングル環境か Real Application Clusters 環境かを指定します。次はエディションの指定です。Standard Edition 2 (SE2) または Enterprise Edition を選択します。

![Select Database Edition](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2Fc6b27251-d120-e5e8-e5a5-b531ef9ebe11.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=763c9b41e57dde4089dd442fef0ae638)



### Privileged Operating System groups

次にインストール先（ORACLE_BASE)、インベントリの保存先を指定する画面を経て、管理グループを指定する画面が表示されます。こちらもOracle Database 19c から変更されていません。

![Privileged Operating System Groups](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2Fffd95b5e-cb9a-a475-bed7-c39d5e3343b5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6930888f98df23cb48d130ba908c2055)



### Perform Prerequesite Checks

自動実行スクリプトの画面を経て、次の画面は事前チェックの画面です。画面はわざとカーネル・パラメーター設定をせずに検証しているため、警告が表示されています。

![Perform Prerequisite Checks](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2Fcdf62f2e-8232-9568-96ec-df194eca88cf.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=57ad159d824be71228556069e0a12df3)



### AAA

最後に確認画面とrootユーザーでスクリプトを実行することを促す画面が表示されてインストールは完了します。



## データベースの作成

データベースの作成も従来とかわらず DBCA を使います。

### Select Database Operation

![Select Database Operation](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2F490c77c6-18e1-f02d-2b5a-ceac8cd79db9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=43f79a83ac79cce853beccf4a2fefa38)



### Specify Database Identification Details

DBCA の画面は Oracle Database 19c とほぼ変わりませんが、Oracle Database 21cからマルチテナント環境が強制されます。下記はデータベース名を決める画面ですが、コンテナ・データベースの作成オプションが変更できなくなっています。またPDB名のデフォルト値がCDB名を含む名前になっています。

![Specify Database Identification Details](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2Fe86314c9-5253-3ce4-4cb7-96b1bdeec0cd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c3aab4b3d1ca583d34b1325e77d97f14)



### Select Database Options

データベース・オプションの画面からはOracle Application Express (APEX) の選択が消えています。

![Select Database Options](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2F3ffa3a3a-2c62-788f-4a35-86fbc6d52443.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=501904a6d3a3e2c9fa8210a74e0c4527)



### Specify Configuration Options

環境変数LANG=en_US.UTF8の環境で実行したところ、「Use OS character set」のデフォルトが「WE8MSWIN1252」でした。

![Specify Configuration Options](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F181284%2F4bfac259-6daf-88f3-23c9-bde7c305d506.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a2f864751c17b92c9fda8e054ea1d0c4)

データベースの作成が完了しました。

```
$ sqlplus / as sysdba

SQL*Plus: Release 21.0.0.0.0 - Production on 土 8月 14 13:45:42 2021
Version 21.3.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.

Oracle Database 21c Enterprise Edition Release 21.0.0.0.0 - Production
Version 21.3.0.0.0
に接続されました。
SQL>
SQL> SELECT * FROM v$version;

BANNER
--------------------------------------------------------------------------------
BANNER_FULL
------------------------------------------------------------------------------------------
BANNER_LEGACY
--------------------------------------------------------------------------------
    CON_ID
----------
Oracle Database 21c Enterprise Edition Release 21.0.0.0.0 - Production
Oracle Database 21c Enterprise Edition Release 21.0.0.0.0 - Production Version 21.3.0.0.0
Oracle Database 21c Enterprise Edition Release 21.0.0.0.0 - Production
```

