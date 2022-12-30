# Ubuntu20.04系统上安装Java并配置环境变量

## 1. 确认安装版本

首先依然需要打开终端。当然我们要确认一下自己的Ubuntu系统电脑里是否安装了Java。

- 版本确认指令：

  ```bash
  java --version
  ```

  如果安装了Java，终端里会显示Java的版本。

  而如果没有安装Java，终端就会提示如下信息：

  ```bash
  liu@liu-virtual-machine:~$ java --version
  Command 'java' not found, but can be installed with:
  sudo apt install openjdk-11-jre-headless  # version 11.0.17+8-1ubuntu2~22.04, or
  sudo apt install default-jre              # version 2:1.11-72build2
  sudo apt install openjdk-17-jre-headless  # version 17.0.5+8-2ubuntu1~22.04
  sudo apt install openjdk-18-jre-headless  # version 18.0.2+9-2~22.04
  sudo apt install openjdk-19-jre-headless  # version 19.0.1+10-1ubuntu1~22.04
  sudo apt install openjdk-8-jre-headless   # version 8u352-ga-1~22.04
  ```

  不仅为我们提示了这台电脑没有安装Java，还贴心地提示了输入哪些命令可以安装这个平台环境，而这几个命令只是不同的版本而已。第一个是11版。



## 2. 安装 JDK

- 根据提示，我们可以直接输入如下指令进行安装：

  ```bash
  sudo apt install openjdk-11-jre-headless
  ```

  然后就是提示输入密码，输入完就是读取分析，之后就是Y确认了。完成后，我们再用查询的命令看下，可以看到`openjdk`安装成功。

- 仅仅安装 JRE 的话：

  当然由于这里是headless也就是最小的Java运行环境。由于`jre`是被包含在`jdk`软件包中的，所以安装了`jdk`就是安装了`jre`。如果是仅仅安装JRE环境的话，请使用如下指令：

  ```bash
  sudo apt update
  sudo apt install default-jre
  ```

- 安装旧版本的 JDK

  有些应用是运行在java8上的，这里你就要安装Java8了（提示里面的最后 1 行）。

  ```bash
  sudo apt update
  sudo apt install openjdk-8-jre-headless
  ```

  也可以用简化命令，同样能够安装：

  ```bash
  sudo apt install openjdk-8-jre
  ```

  

## 3. 设置环境变量

- 查询安装路径：

  要设置JAVA_HOME环境变量，用如下指令查询 JDK 的安装路径：

  ```bash
  sudo update-alternatives --config java
  ```

  这个命令来查找Java安装的路径，这个其实是在系统里安装了多个Java环境后配置使用的，我们安装了一个Java环境，所以终端提示我们无需配置，下面就是我们要用的安装路径。

  ```
  liu@liu-virtual-machine:~$ sudo update-alternatives --config java
  链接组 java (提供 /usr/bin/java)中只有一个候选项：/usr/lib/jvm/java-11-openjdk-amd64/bin/java
  无需配置。
  ```

- 设置环境变量

  ```bash
  sudo nano /etc/environment
  ```

  把查询到的 JDK 安装路径，保存到该文件中，对于我当前的电脑设备，大致如下：

  ```
  JAVA_HOME=”/usr/lib/jvm/java-11-openjdk-amd64/bin/java”
  ```

  