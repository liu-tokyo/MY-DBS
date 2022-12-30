# Ubuntu上安装 SQL Devlopment

> 作业环境：Ubuntu 22.04  
> 前提条件：安装 JDK

**Oracle SQL Developer** 是非常强悍的开源的SQL开发工具,面前市面上流行的数据库都支持连接，可以在 SQL Developer 里直接连接 Oracle 和 MySQL 。

除非特殊情况，请按照 **安装方法-2** 进行安装，相对简洁、简单。

## 1. 安装办法-1

安装不依赖 OS 平台的版本。

### 1.1 官方软件下载

- 官方下载网址：

  `https://www.oracle.com/database/sqldeveloper/technologies/download/`

  下载 `Other Platforms` 版本，该版本不依赖 OS，在安装了 JDK 的设备上都能正常运行，当然步骤稍微繁琐。

  当前（2022.12.31）的最新版本是 `sqldeveloper-22.2.1.234.1810-no-jre.zip` 。

- 解压下载文件：

  解压之后把文件夹中的 `sqldeveloper` 目录，拷贝到 **主目录** 的下面。这样放置到 `./sqldeveloper` 之后，有利于后续的指令编写。

### 1.2 修改 JDK 路径

如果电脑中安装了多个版本的 JDK 的话，需要指定启动有的 JDK 版本；如果电脑中只有一个版本的 JDK，则可以忽略该步骤。

- 查询 JSK 安装信息：

  ```
  sudo update-alternatives --config java
  ```

- 修改 JDK 路径

  ```
  sudo nano ./sqldeveloper/ide/bin/launcher.sh
  ```

  找到 `CheckJDK()` 的位置，将其中的 `APP_JAVA_HOME` 修改为自己希望的 JDK 的路径，大致如下：

  ```
  if [ "X$APP_JAVA_HOME"!="X" ]
      then
      if [ !-d ${APP_JAVA_HOME} ]
      then
      APP_JAVA_HOME="/home/xxx/jdk1.6.0_18"
      fi
  fi
  ```

- 增加可执行权限

  ```
  sudo chmod +x ~/sqldeveloper/sqldeveloper.sh
  ```

### 1.3 创建启动文件

- 点击 **显示应用程序**，找到 `启动应用程序` 图标，点击打开。

- 在 `启动应用程序首选项` 画面中，点击 **添加** 按钮；

- 在 `添加应用程序` 画面中，按照如下设置之后，点击 **添加** 按钮保存。

  名称：`SQL Devlopment`

  命令：`/home/liu/sqldeveloper/sqldeveloper.sh`

- `额外的启动程序` 列表中会出现 `SQL Devlopment` 项目，把该项目拖曳到桌面。

  拖曳到桌面上之后，文件名称为：`sqldeveloper.sh.desktop` ，后面增加了 `.desktop` 。

  拖曳到桌面上之后，`启动应用程序首选项` 画面中的该项目会自动消失，如果没有消失的话，需要手动删除，否则每次开机都会执行，会很麻烦。

- 该桌面图标上，点击鼠标右键，选择 `属性` 。

  在属性画面上，执行如下操作之后，关闭 属性 页面。

  - `基本` 页，修改图标；
  - `权限` 页，勾选 `允许执行文件` 。

- 该图标上，鼠标右键点击，点击下拉菜单中的 `允许运行` 项目。

  选择之后，桌面图标的错误表示消失了，原来设置的图标也正常能够显示了。

上述操作之后，双击该图标就能够启动 `SQL Devlopment ` 。

- 该图标用文本编辑器打开的话，真实内容如下：

  ```
  [Desktop Entry]
  Type=Application
  Exec=/home/liu/sqldeveloper/sqldeveloper.sh
  Hidden=false
  NoDisplay=false
  X-GNOME-Autostart-enabled=true
  Name[zh_CN]=SQL Devlopment
  Name=SQL Devlopment
  Comment[zh_CN]=
  Comment=
  ```

### 1.4 添加到收藏夹

为了方便操作，把桌面图标转移到 应用程序 页面上，并且添加到 收藏夹，这样随时点击很方便。

- 在桌面上打开终端，把该桌面图标文件，拷贝到 `/usr/share/applications` 中：

  ```
  sudo mv sqldeveloper.sh.desktop /usr/share/applications/.
  ```

- 点击 显示应用程序 图标，该程序已经能够显示在 应用程序 列表中。

  鼠标右键点击，选择 `添加到收藏夹`，这样每次都能从左侧的任务条中点击该图标启动应用程序。

- 用文本编辑器打开 `sqldeveloper.sh.desktop` ，添加如下 1 行：

  ```
  Icon=/home/liu/sqldeveloper/icon.png
  ```

- 最终没有解决图标问题，显示的还是通用图标。尝试如下指令：

  ```
  sudo update-icon-caches /usr/share/icons/*
  ```

重新启动电脑之后，图标给正确显示了。

- 如果直接要把图标添加到 `收藏夹` 的话，可以用如下指令尝试：

  ```
  sudo mv sqldeveloper.sh.desktop ~/Desktop/
  ```

## 2. 安装方法-2

安装 Linux 平台专用版本。

### 2.1 官方软件下载

- 下载网址

  `https://www.oracle.com/database/sqldeveloper/technologies/download/`

  下载 Linux 版本，当前文件名称为 `sqldeveloper-22.2.1.234.1810.noarch.rpm` 。

### 2.2 构筑专用版本

利用下载 RPM 文件，构筑 Ubuntu 的专用版本。

- 安装编辑工具

  ```
  sudo apt-get install alien
  ```

  如果加压工具没有安装的话，也需要安装，不过新版的Ubuntu都已经安装，可以忽略；

  ```
  sudo apt-get install tar
  ```

  都执行 1 次，如果已经安装，会提示已经是最新版本了说明。

- 编译专用版本

  找到 `sqldeveloper-22.2.1.234.1810.noarch.rpm` 的文件路径，打开终端，执行如下指令：

  ```
  sudo alien -cv sqldeveloper-22.2.1.234.1810.noarch.rpm
  ```

  这个编译的时间比较长，需要 10 分钟以上，结果是编出来一个 `sqldeveloper_22.2.1-235.181_all.deb` 文件。

### 2.3 安装专用版本

- 指令安装方式

  ```
  sudo dpkg -i sqldeveloper_22.2.1-235.181_all.deb
  ```

- 自动安装方式

  我都是比较懒惰的，在 DEB 文件上鼠标右键，选择 `使用其它程序打开`，然后选择 `软件安装`， 让系统自己安装。



## 3. 安装方法-3

最新版的 Ubuntu 已经内置了 SQL Devlopment 的安装指令，如上折腾纯粹是老版本 Ubuntu 的祸。毕竟 Ubuntu 是亲民的 Linux 版本，常用软件的安装基本都是内置了。

- 查找
  软件源内是否有 SQL Devlopment ？

  ```
  apt search sqldev
  ```

  得到的结果：

  ```
  liu@liu-virtual-machine:~$ apt search sqldev
  正在排序... 完成
  全文搜索... 完成  
  sqldeveloper-package/jammy,jammy 0.5.4 all
    Oracle SQL Developer Debian package builder
  ```

- 安装

  根据查找结果，执行如下指令：

  ```
  sudo apt install sqldeveloper-package/jammy
  ```

居然安装了一个图像编辑的工具软件？？？

据说需要如下步骤，不是一般的繁琐：

> 参考：`https://blog.csdn.net/ficksong/article/details/6984804`

- 下列指令

  ```
  sudo apt-get install tofrodos
  
  sudo ln -s /usr/bin/fromdos /usr/bin/dos2unix
  sudo ln -s /usr/bin/todos /usr/bin/unix2dos
  
  make-sqldeveloper-package -b BUILD_LOCATION LOCATION_OF_ZIP_FILE
  make-sqldeveloper-package -b /tmp/ORA/ ~/Desktop/sqldeveloper-2.1.0.63.73-no-jre.zip
  make-sqldeveloper-package: Building sqldeveloper package in "/tmp".
  ```

不再折腾这个办法了，感觉很麻烦。
