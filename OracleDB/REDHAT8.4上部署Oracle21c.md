# REDHAT8.4上部署Oracle21c

> 参考 ： https://blog.csdn.net/u012778985/article/details/119865618

## 虚拟机配置要点

1. 21c默认需要8G内存，否则[grid](https://so.csdn.net/so/search?q=grid&spm=1001.2101.3001.7020)安装自检会报错

2. asm磁盘需要配置单独scsi通道、永久.

```
disk.locking = "false"
disk.EnableUUID = "TRUE"
```



## 系统安装







## 操作系统准备

1. ip地址规划

   | 服务器主机名                                                 | ora21c       |
   | ------------------------------------------------------------ | ------------ |
   | 公共IP地址                                                   | 192.168.0.20 |
   | 集群实例名称                                                 | ora21c       |
   | 操作系统                                                     | Redhat8.4    |
   | 存储                                                         | vmdk         |
   | [Oracle](https://so.csdn.net/so/search?q=Oracle&spm=1001.2101.3001.7020)版本 | Oracle 21c   |
   | 升级版本                                                     | 21.0.0.0.0   |

2. 网络配置

   ```
   [root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens160
   TYPE=Ethernet
   PROXY_METHOD=none
   BROWSER_ONLY=no
   BOOTPROTO=static
   DEFROUTE=yes
   IPV4_FAILURE_FATAL=no
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   IPV6_DEFROUTE=yes
   IPV6_FAILURE_FATAL=no
   IPV6_ADDR_GEN_MODE=stable-privacy
   NAME=ens160
   UUID=296f5307-0bc8-4877-b80a-82f851475ae1
   DEVICE=ens160
   ONBOOT=yes
   IPADDR=192.168.0.20
   NETMASK=255.255.255.0
   ```

3. 修改hosts文件及hostname

   ```
   cat >> /etc/hosts <<EOF
   192.168.0.20 ora21c
   EOF
   cat /etc/hosts
   hostnamectl --static set-hostname ora21c
   hostname
   ora21c
   ```

4. 创建用户和组

   ```
   groupadd oinstall -g 1023
   groupadd dba -g 1024
   groupadd asmadmin -g 1025
   groupadd asmdba -g 1026
   groupadd asmoper -g 1027
   useradd -g oinstall -G asmadmin,asmdba,asmoper,asmoper,dba -u 1023 grid
   useradd -g oinstall -G dba,asmdba,asmoper -u 1024 oracle
   echo oracle | passwd --stdin grid
   echo oracle | passwd --stdin oracle
   ```

5. 创建目录

   ```
   mkdir -p /u01/app/21c/grid
   mkdir -p /u01/app/grid
   mkdir -p /u01/app/oracle/product/21c/dbhome_1
   chown -R grid:oinstall /u01
   chown -R grid:oinstall /u01/app/grid
   chown -R oracle:oinstall /u01/app/oracle
   ```

6. 关闭防火墙及selinux

   ```
   systemctl stop firewalld
   systemctl disable firewalld
   systemctl status firewalld
   systemctl start avahi-daemon.socket
    
   setenforce 0
   sed -i 's/enforcing/disabled/g' /etc/selinux/config
   getenforce
   ```

7. 配置用户环境变量

   **grid用户**

   ```
   cat >> /home/grid/.bash_profile <<EOF
   export ORACLE_BASE=/u01/app/grid
   export ORACLE_HOME=/u01/app/21c/grid
   export ORACLE_SID=+ASM
   export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
   export NLS_DATE_FORMAT="yyyy-mm-dd hh24:mi:ss" 
   export PATH=.:\${PATH}:\$HOME/bin:\$ORACLE_HOME/bin:\$ORACLE_HOME/OPatch 
   export PATH=\${PATH}:/usr/bin:/bin:/usr/bin/X11:/usr/local/bin 
   export PATH=\${PATH}:\$ORACLE_BASE/common/oracle/bin 
   export ORACLE_TERM=xterm
   export LD_LIBRARY_PATH=\$ORACLE_HOME/lib 
   export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:\$ORACLE_HOME/oracm/lib 
   export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:/lib:/usr/lib:/usr/local/lib 
   export CLASSPATH=\$ORACLE_HOME/JRE 
   export CLASSPATH=\${CLASSPATH}:\$ORACLE_HOME/jlib 
   export CLASSPATH=\${CLASSPATH}:\$ORACLE_HOME/rdbms/jlib 
   export CLASSPATH=\${CLASSPATH}:\$ORACLE_HOME/network/jlib 
   export THREADS_FLAG=native
   export TEMP=/tmp
   export TMPDIR=/tmp
   umask 022 
   EOF
   ```

   **oracle用户**

   ```
   cat >> /home/oracle/.bash_profile << EOF
   export ORACLE_BASE=/u01/app/oracle
   export ORACLE_HOME=\$ORACLE_BASE/product/21c/dbhome_1 
   export ORACLE_BASE_HOME=\$ORACLE_BASE/homes/OraDB21Home1
   export ORACLE_SID=orcl
   export LANG=en_US.UTF-8
   export NLS_LANG=american_america.AL32UTF8
   export NLS_DATE_FORMAT="yyyy-mm-dd hh24:mi:ss" 
   export PATH=.:\${PATH}:\$HOME/bin:\$ORACLE_HOME/bin:\$ORACLE_HOME/OPatch 
   export PATH=\${PATH}:/usr/bin:/bin:/usr/bin/X11:/usr/local/bin 
   export PATH=\${PATH}:\$ORACLE_BASE/common/oracle/bin:/home/oracle/run 
   export ORACLE_TERM=xterm
   export LD_LIBRARY_PATH=\$ORACLE_HOME/lib 
   export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:\$ORACLE_HOME/oracm/lib 
   export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:/lib:/usr/lib:/usr/local/lib 
   export CLASSPATH=\$ORACLE_HOME/JRE 
   export CLASSPATH=\${CLASSPATH}:\$ORACLE_HOME/jlib 
   export CLASSPATH=\${CLASSPATH}:\$ORACLE_HOME/rdbms/jlib 
   export CLASSPATH=\${CLASSPATH}:\$ORACLE_HOME/network/jlib 
   export THREADS_FLAG=native
   export TEMP=/tmp
   export TMPDIR=/tmp
   export GI_HOME=/u01/app/21c/grid
   export PATH=\${PATH}:\$GI_HOME/bin 
   export ORA_NLS10=\$GI_HOME/nls/data 
   umask 022 
   export TMOUT=0
   EOF
   ```

8. 配置内核参数

   ```
   cat >> /etc/sysctl.conf << EOF
   kernel.shmall = 2251799813685247
   kernel.shmall = 503316480 
   kernel.shmmni = 4096
   kernel.sem = 5010 641280 5010 128
   net.core.rmem_default = 4194304
   net.core.rmem_max = 4194304
   net.core.wmem_default = 262144
   fs.file-max = 6815744
   net.ipv4.ip_local_port_range = 9000 65500
   net.core.wmem_max = 1048576
   fs.aio-max-nr = 1048576
   EOF
   sysctl -p
   ```

9. 设置系统资源限制

   ```
   cat >> /etc/security/limits.conf << EOF
   oracle hard memlock 3145728
   oracle soft nproc 2047
   oracle hard nproc 16384 
   oracle soft nofile 4096 
   oracle hard nofile 65536 
   oracle soft stack 10240 
   oracle hard stack 32768
   grid soft nproc 2047
   grid hard nproc 16384 
   grid soft nofile 4096 
   grid hard nofile 65536 
   grid soft stack 10240 
   grid hard stack 32768
   EOF
   ```

   