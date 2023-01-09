# 制作 OracleDB 容器 - Oracle DB 21c

制作 Oracle DB 21.3.0 版本的容器，容器通过 Docker 启动。

> 制作环境：Ubuntu 22.04.1 LST



## 1. 安装 Docker

> [Ubuntu 20.04へのDockerのインストールおよび使用方法 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-ja)

- 在尝试安装新版本之前卸载任何此类旧版本：

  ```bash
  sudo apt-get remove docker docker-engine docker.io containerd runc
  ```

安装有多种方式，参照 Docker [官网](https://docs.docker.com/engine/install/ubuntu/)的说明进行即可。

### 1.1 使用存储库安装

- 设置存储库

  ```bash
  sudo apt-get update
  sudo apt-get install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
  ```

- 添加 Docker 的官方 GPG 密钥：

  ```bash
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```

- 使用以下命令设置存储库：

  ```bash
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```

- 更新 apt 包索引：

  ```bash
  sudo apt-get update
  ```

  您的默认 umask 可能配置不正确，导致无法检测存储库公钥文件。 在更新包索引之前尝试授予 Docker 公钥文件的读取权限：

  ```bash
  sudo chmod a+r /etc/apt/keyrings/docker.gpg
  sudo apt-get update
  ```

- 要安装最新版本，请运行：

  ```bash
  sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
  ```

- 通过运行 hello-world 镜像验证 Docker Engine 安装是否成功：

  ```bash
  sudo docker run hello-world
  ```

### 1.2 使用软件包安装

如果您不能使用 Docker 的 apt 存储库来安装 Docker Engine，您可以下载您的版本的 deb 文件并手动安装。 每次升级 Docker Engine 时都需要下载一个新文件。

- 官网下载

  https://download.docker.com/linux/ubuntu/dists/

  在列表中选择您的 Ubuntu 版本。

  到 `pool/stable` 下面，并选择适用的架构 (amd64, armhf, arm64, or s390x)

- 为 Docker Engine、CLI、containerd 和 Docker Compose 包下载以下 deb 文件：

  - `containerd.io_<version>_<arch>.deb`
  - `docker-ce_<version>_<arch>.deb`
  - `docker-ce-cli_<version>_<arch>.deb`
  - `docker-compose-plugin_<version>_<arch>.deb`

- 安装 .deb 包。 将以下示例中的路径更新为您下载 Docker 包的位置。

  ```bash
  sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
    ./docker-ce_<version>_<arch>.deb \
    ./docker-ce-cli_<version>_<arch>.deb \
    ./docker-compose-plugin_<version>_<arch>.deb
  ```

  Docker 守护进程自动启动。

- 通过运行 hello-world 镜像验证 Docker Engine 安装是否成功：

  ```bash
  sudo docker run hello-world
  ```

  此命令下载测试图像并在容器中运行它。 当容器运行时，它会打印一条确认消息并退出。

您现在已经成功安装并启动了 Docker 引擎。 docker 用户组存在但不包含任何用户，这就是为什么您需要使用 sudo 来运行 Docker 命令的原因。 继续 Linux 后安装，以允许非特权用户运行 Docker 命令和其他可选配置步骤。

### 1.3 卸载 Docker 引擎

- 卸载 Docker Engine、CLI、containerd 和 Docker Compose 包：

  ```bash
  sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-compose-plugin
  ```

- 主机上的图像、容器、卷或自定义配置文件不会自动删除。 删除所有镜像、容器和卷：

  ```bash
  sudo rm -rf /var/lib/docker
  sudo rm -rf /var/lib/containerd
  ```

- 您必须手动删除任何已编辑的配置文件。

### 1.4 设置运行权限

- 状态确认

  ```bash
  sudo systemctl status docker
  ```

- 无 `sudo` 使用 `Docker`

  > 官网参考：https://docs.docker.com/engine/install/linux-postinstall/

  把当前的用户名添加到 `docker` 组，以避免每次运行 `docker` 命令时都键入 `sudo`。
  
  ```bash
  sudo usermod -aG docker ${USER}
  ```

  要应用新的组成员资格，需要退出服务器并重新登录；或者键入如下指令（避免重新登录）：
  
  ```bash
  ## su - ${USER}
  ## 您还可以运行以下命令来激活对组的更改：
  newgrp docker
  # 该指令在批处理里面执行之后，会阻挡继续执行；综合考虑，在安装之后，再次启动电脑，应该是更加简单的。
  ```
  
  通过键入以下内容验证用户是否已添加到 `docker` 组：
  
  ```bash
  id -nG
  ```
  
  如果您需要将用户添加到未登录的 `docker` 组，请使用以下方式显式声明该用户名：
  
  ```bash
  sudo usermod -aG docker username
  ```



### 1.5 安装 - 批处理

逐个命令执行，非常繁琐，主要用于熟悉指令；通过批处理方式，更为简便。

- Docker 安装部分

  `docker_setup.sh`：

  ```bash
  #!/bin/bash
  
  ## 卸载任何此类旧版本
  sudo apt-get remove docker docker-engine docker.io containerd runc
  
  ## 设置存储库
  sudo apt-get update
  sudo apt-get -y install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
  
  ## 添加 Docker 的官方 GPG 密钥
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  
  ## 设置存储库
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
  ## 更新 apt 包索引
  sudo apt-get update
  
  ## 授予 Docker 公钥文件的读取权限
  sudo chmod a+r /etc/apt/keyrings/docker.gpg
  sudo apt-get update
  
  ## 安装最新版本
  sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
  
  ## 权限设置
  sudo gpasswd -a $USER docker
  sudo usermod -aG docker ${USER}
  
  ## 清除垃圾软件包
  sudo apt autoclean
  sudo apt clean
  sudo apt autoremove
  
  echo "OK : normal finished!"
  ```

- 批处理执行：

  ```bash
  sudo chmod 777 docker_setup.sh
  ./docker_setup.sh
  ```
  
  ※正常情况下，会顺利执行结束；需要重新启动（注销、登录）之后，才能保证 `无SUDO` 执行有效。

## 2. 制作数据库镜像

- 参考网站

  |  #   | 网站                                                         | 简介                                     |
  | :--: | ------------------------------------------------------------ | ---------------------------------------- |
  |  1   | https://github.com/steveswinsburg/oracle21c-docker           | 介绍如何创建OracleDB 21C的Docker镜像     |
  |  2   | https://github.com/oracle/docker-images                      | Oracle官方：创建Oracle产品镜像的网站     |
  |  3   | https://zenn.dev/msksgm/articles/20211225-oracle-database-19c-docker | 介绍如何创建OracleDB 19C镜像的网站       |
  |  4   | https://docs.oracle.com/en/database/oracle/oracle-database/21/deeck/index.html | 如何在 Docker 上安装 Oracle Database 21c |
  |      | https://www.server-world.info/query?os=CentOS_Stream_9&p=podman&f=4 | Linux的各种安装                          |

  

### 2.1 镜像制作 - 指令

- 在合适的目录内，克隆如下网站：

  ```bash
  git clone https://github.com/oracle/docker-images.git
  cd docker-images
  ```

- 把下载的 `LINUX.X64_213000_db_home.zip` 文件，转移到如下位置：

  ```bash
  cd OracleDatabase/SingleInstance/dockerfiles/21.3.0
  mv ~/Downloads/LINUX.X64_213000_db_home.zip .
  ```

- 执行创建镜像指令 （Oracle EE）：

  ```bash
  cd ..
  sudo ./buildContainerImage.sh -v 21.3.0 -e -i
  ```

  比较花费时间，出现如下信息，证明创建镜像成功：

  ```bash
    Oracle Database container image for 'ee' version 21.3.0 is ready to be extended: 
      
      --> oracle/database:21.3.0-ee
  
    Build completed in 678 seconds.
  ```

  最快速的硬盘需要 11（678s） 分钟左右，外置 SSD 硬盘需要 2042秒（35分钟） 分钟；不知道为啥，外置的机械硬盘跑了 2 天才结束。

- 创建镜像的参数：

  ```bash
  Parameters:
     -v: version to build
         Choose one of: 11.2.0.2  12.1.0.2  12.2.0.1  18.3.0  18.4.0  19.3.0  21.3.0
     -t: image_name:tag for the generated docker image
     -e: creates image based on 'Enterprise Edition'
     -s: creates image based on 'Standard Edition 2'
     -x: creates image based on 'Express Edition'
     -i: ignores the MD5 checksums
     -o: passes on container build option
  
  * select one edition only: -e, -s, or -x
  ```

  例如创建 Oracle XE 版本的镜像，就不需要去下载 ZIP 文件，直接创建即可：

  ```bash
  sudo ./buildContainerImage.sh -v 21.3.0 -x -i
  ```

  

### 2.2 镜像制作 - 批处理

- 批处理文件（`create_oracle_db_docker_image.sh`）：

  ```bash
  #!/bin/bash
  
  ## 批处理参数检查，OracleDB安装ZIP文件作为第一个单数。
  if [ $# -eq 0 ]; then
  	echo "参数错误，请设置至少至少一个参数（OracleDB安装ZIP文件）";
  	exit 1
  fi
  
  ## 参数1：检查该文件是否存在
  if [ -e $1 ]; then
  	echo "参数1：$1"
  else
  	echo "参数1：该文件不存在"
  	exit 2
  fi
  
  ## 临时作业用目录
  if [ ! -e ~/temp ]; then
  	sudo mkdir ~/temp
  fi
  sudo chmod 777 ~/temp
  cd ~/temp
  
  git clone https://github.com/oracle/docker-images.git
  cd docker-images
  
  ## 把下载的OracleDB文件转移到本目录
  cd OracleDatabase/SingleInstance/dockerfiles/21.3.0
  mv $1 .
  
  ## 构筑IMAGE，并登录到Docker本地库
  cd ..
  sudo ./buildContainerImage.sh -v 21.3.0 -e -i
  
  ## 删除临时作业目录
  
  echo "OK : normal finished!"
  ```

- 执行该批处理指令

  参数1：下载的 `LINUX.X64_213000_db_home.zip` 文件的全路径，如果实在 `下载` 目录内，指令如下：

  ```bash
  sudo bash ./create_oracle_db_docker_image.sh ~/下载/LINUX.X64_213000_db_home.zip
  ```

  

## 3. 启动数据库

### 3.1 启动容器

- 启动指令1：

  ```bash
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
  oracle/database:21.3.0-ee
  ```

  首次运行时，将为您创建和设置数据库。 这大约需要 10-15 分钟。 打开 `Docker Dashboard` 并观察进度。 然后就可以连接了。

- 启动指令2：

  或者，您可以使用以下运行命令在本地计算机上安装一个额外的目录，这有助于避免在您逐渐将越来越多的数据插入数据库时出现“无磁盘空间”问题。 确保为您的系统适当地设置 -v 命令。

  ```bash
  sudo docker run \
  	--name oracle21c \
  	-p 1521:1521 -p 5500:5500 \
  	-e ORACLE_PDB=orcl \
  	-e ORACLE_PWD=password \
  	-e INIT_SGA_SIZE=3000 \
  	-e INIT_PGA_SIZE=1000 \
  	-v /path/to/store/db/files/:/opt/oracle/oradata \
  	-d \
  oracle/database:21.3.0-ee
  ```


- 和安装版本的 Oracle EE 版有所区别，容器方式的话，商业版 `Oracle EE` 创建的可插接式 PDB 的名称：

  <span style="Color:red">ORCL</span>

### 3.2 查询启动状态

- 指令

  ```bash
  docker ps -a
  ```

  `health: starting` 状态，属于启动中；`health`属于启动结束，处于正常服务状态。

### 3.3 修改口令

- 注意：

  如果您没有设置 ORACLE_PWD 参数，请检查 `docker run` 输出的密码。

- 可以通过 `docker exec` 命令更改 SYS 帐户的密码。 **请注意，容器必须正在运行**。

  首先运行 docker ps 获取容器 ID。 然后运行：

  ```bash
  sudo docker exec <container id> ./setPassword.sh <new password>
  ```

  ```bash
  sudo docker exec 187f7110767c ./setPassword.sh qwer1234
  ```

  ※注意：最新的 `21.3.0` 的情况下，是为 `SYSTEM` 用户设置的口令。

### 3.4 进入容器执行

- 首先运行 `docker ps` 获取容器 ID。 然后运行：

  ```bash
  docker exec -it <container id> /bin/bash
  ```

- 或者作为 root：

  ```bash
  docker exec -u 0 -it <container id> /bin/bash
  ```

### 3.5 远程连接

这一点和 Windows 上的安装版有所不同，不需要为了 远程连接 作特别的修改。毕竟容器方式本身必须对外提供服务，才有意义。



## 4. 学习小结

其它数据库，因为基本都是可以免费使用，所以都可以直接下载安装即可。只有 Oracle DB 的官方网站，想要下载，必须注册才可以，所以安装的时候，手续稍微繁琐。

即使是可以免费使用的 XE 版本，官方也没有提供可供随便使用的 Docker镜像。不过第三方提供的有类似的镜像。个人可以随便使用，但是公司开发的话，还是用 Oracle官方 提供的工具自我创建的镜像文件，更为靠谱。
