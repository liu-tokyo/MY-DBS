# `Ubuntu` 上安装 `Docker`

> [Ubuntu 20.04へのDockerのインストールおよび使用方法 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-ja)
>
> 

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

```
sudo apt-get update
sudo apt-get -y install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```



```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```



```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```



```
sudo apt-get update
```



```
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update
```



```
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```



```
sudo docker run hello-world
```



- 状态确认

  ```
  sudo systemctl status docker
  ```

- 无 `sudo` 使用 `Docker`

  把当前的用户名添加到 `docker` 组，以避免每次运行 `docker` 命令时都键入 `sudo`。

  ```
  sudo usermod -aG docker ${USER}
  ```

  要应用新的组成员资格，需要退出服务器并重新登录；或者键入如下指令（避免重新登录）：

  ```
  su - ${USER}
  ```

  通过键入以下内容验证用户是否已添加到 `docker` 组：

  ```
  id -nG
  ```

  如果您需要将用户添加到未登录的 `docker` 组，请使用以下方式显式声明该用户名：

  ```
  sudo usermod -aG docker username
  ```



### Docker安装批处理

- 批处理文件详细

  把如下内容保存到 `docker_install.sh` 的文件中：

  ```bash
  #!/bin/bash
  
  # remove old version
  sudo apt-get remove docker docker-engine docker.io containerd runc
  
  sudo apt-get update
  sudo apt-get -y install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
  
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
  sudo apt-get update
  
  sudo chmod a+r /etc/apt/keyrings/docker.gpg
  sudo apt-get update
  
  sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
  
  sudo usermod -aG docker ${USER}
  
  su - ${USER}
  
  id -nG
  
  echo "OK : normal finished!"
  ```

- 执行批处理：

  在保存文件的目录中，打开终端，输入如下指令：

  ```
  sudo bash ./docker_install.sh
  ```

  ※正常情况下，会顺利执行结束。

## OracleDB 21C的Image作成

- 参考网站

  |  #   | 网站                                                         | 简介                                     |
  | :--: | ------------------------------------------------------------ | ---------------------------------------- |
  |  1   | https://github.com/steveswinsburg/oracle21c-docker           | 介绍如何创建OracleDB 21C的Docker镜像     |
  |  2   | https://github.com/oracle/docker-images                      | Oracle官方：创建Oracle产品镜像的网站     |
  |  3   | https://zenn.dev/msksgm/articles/20211225-oracle-database-19c-docker | 介绍如何创建OracleDB 19C镜像的网站       |
  |  4   | https://docs.oracle.com/en/database/oracle/oracle-database/21/deeck/index.html | 如何在 Docker 上安装 Oracle Database 21c |
  |      | https://www.server-world.info/query?os=CentOS_Stream_9&p=podman&f=4 | Linux的各种安装                          |

  



```
git clone https://github.com/oracle/docker-images.git
cd docker-images
```



```
cd OracleDatabase/SingleInstance/dockerfiles/21.3.0
mv ~/Downloads/LINUX.X64_213000_db_home.zip .
```



```
cd ..
sudo ./buildContainerImage.sh -v 21.3.0 -e -i
```



```
  Oracle Database container image for 'ee' version 21.3.0 is ready to be extended: 
    
    --> oracle/database:21.3.0-ee

  Build completed in 678 seconds.
```

最快速的硬盘需要 11（678s） 分钟左右，外置 SSD 硬盘需要 2042秒（35分钟） 分钟；不知道为啥，外置的机械硬盘跑了 2 天才结束。



### 批处理

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

  ```
  sudo bash ./create_oracle_db_docker_image.sh ~/下载/LINUX.X64_213000_db_home.zip
  ```

  

## 启动数据库

### 启动容器

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

  

### 修改口令

- 注意：

  如果您没有设置 ORACLE_PWD 参数，请检查 `docker run` 输出的密码。

- 可以通过 `docker exec` 命令更改 SYS 帐户的密码。 **请注意，容器必须正在运行**。

  首先运行 docker ps 获取容器 ID。 然后运行：

  ```
  sudo docker exec <container id> ./setPassword.sh <new password>
  ```

  ```
  sudo docker exec 187f7110767c ./setPassword.sh qwer1234
  ```

  ※注意：最新的 `21.3.0` 的情况下，是为 `SYSTEM` 用户设置的口令。

### 进入容器执行

- 首先运行 `docker ps` 获取容器 ID。 然后运行：

  ```
  docker exec -it <container id> /bin/bash
  ```

- 或者作为 root：

  ```
  docker exec -u 0 -it <container id> /bin/bash
  ```

