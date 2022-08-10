title: CentOS7静默安装oracle11g
date: 2022-08-10 09:22:24
category: linux
tags: oracle

# CentOS7静默安装oracle11g

## 1、配置
### 1.1、配置环境
修改主机名
关闭selinux
略

### 1.2、安装库

```
yum -y install binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*i686 compat-libstdc++-33*.devel compat-libstdc++-33 compat-libstdc++-33*.devel gcc gcc-c++ glibc glibc*.i686 glibc-devel glibc-devel*.i686 ksh libaio libaio*.i686 libaio-devel libaio-devel*.devel libgcc libgcc*.i686 libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.devel libXi libXi*.i686 libXtst libXtst*.i686 make sysstat unixODBC unixODBC*.i686 unixODBC-devel unixODBC-devel*.i686

```
### 1.3、配置用户
创建oinstall和dba组
```
groupadd oinstall
groupadd dba
```
创建oracle用户
```
useradd -g oinstall -G dba oracle
```
设置oracle用户密码
```
passwd oracle
```

### 1.4、配置内核参数
```
[root@docker ~]# vim /etc/sysctl.conf 

# System default settings live in /usr/lib/sysctl.d/00-system.conf.
# To override those settings, enter new settings here, or in an /etc/sysctl.d/<name>.conf file
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912   #最低：536870912，最大值：比物理内存小1个字节的值，建议超过物理内存的一半
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```
执行
sysctl -p

### 1.5 修改用户限制
```
vim  /etc/security/limits.conf

#在末尾添加
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
oracle hard stack 10240
```

在/etc/pam.d/login 文件中，使用文本编辑器或vi命令增加或修改以下内容
```
session required /lib64/security/pam_limits.so
session required pam_limits.so
```
在/etc/profile 文件中，使用文本编辑器或vi命令增加或修改以下内容
```
if [ $USER = "oracle" ]; then
   if [ $SHELL = "/bin/ksh" ]; then
       ulimit -p 16384
       ulimit -n 65536
    else
       ulimit -u 16384 -n 65536
   fi
fi
```
source /etc/profile
## 2 安装前
### 2.1 创建目录
```
mkdir -p /home/data/app/
chown -R oracle:oinstall /home/data/app/
chmod -R 775 /home/data/app/
```
### 2.2 配置环境变量
```
[oracle@docker ~]$ vim ~/.bash_profile 

export ORACLE_BASE=/home/data/app/oracle
export ORACLE_SID=cpaasdb
```
执行 source ~/.bash_profile
上传ORALCE软件到/home/data下。解压到/home/data/database

### 2.3 复制响应文件模板
```
mkdir /home/data/etc
cp  /home/data/database/response/* /home/data/etc/
[oracle@docker ~]$ ls /home/data/etc
dbca.rsp  db_install.rsp  netca.rsp
```
设置响应文件权限
```
chmod 700 /home/data/etc/*.rsp
```
## 3.安装
### 3.1静默安装
su - oracle
修改安装Oracle软件的响应文件/home/data/etc/db_install.rsp
```
oracle.install.option=INSTALL_DB_SWONLY     // 安装类型
ORACLE_HOSTNAME=centosdb        // 主机名称（hostname查询）
UNIX_GROUP_NAME=oinstall     // 安装组
INVENTORY_LOCATION=/home/data/app/oracle/oraInventory   //INVENTORY目录（不填就是默认值）
SELECTED_LANGUAGES=en,zh_CN,zh_TW // 选择语言
ORACLE_HOME=/home/data/app/oracle/product/11.2.0/dbhome_1    //oracle_home
ORACLE_BASE=/home/data/app/oracle     //oracle_base
oracle.install.db.InstallEdition=EE 　　　　// oracle版本
oracle.install.db.isCustomInstall=true 　　//自定义安装，否，使用默认组件
oracle.install.db.DBA_GROUP=dba /　　/ dba用户组
oracle.install.db.OPER_GROUP=oinstall // oper用户组
oracle.install.db.config.starterdb.type= //数据库类型
oracle.install.db.config.starterdb.globalDBName=cpaasdb //globalDBName
oracle.install.db.config.starterdb.SID=cpaasdb      //SID
oracle.install.db.config.starterdb.memoryLimit=81920 //自动管理内存的内存(M)
oracle.install.db.config.starterdb.password.ALL=oracle //设定所有数据库用户使用同一个密码
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false         //（手动写了false）
DECLINE_SECURITY_UPDATES=true 　　//设置安全更新（貌似是有bug，这个一定要选true，否则会无限提醒邮件地址有问题，终止安装。PS：不管地址对不对）
```
开始安装
```
[oracle@docker database]$ /home/data/database/runInstaller -silent -responseFile /home/data/etc/db_install.rsp
```
查看日志
tail -f /home/data/app/oracle/inventory/logs/installActions2016-08-31_06-56-29PM.log
出现类似如下提示表示安装完成：
```
------------------------------------------------------------------------
The following configuration scripts need to be executed as the "root" user.
#!/bin/sh
#Root scripts to run

/u01/app/oraInventory/orainstRoot.sh  #不同的地址根据自己的实际
/u01/app/oracle/product/11.2.0/db_1/root.sh
To execute the configuration scripts:
1. Open a terminal window
2. Log in as "root"
3. Run the scripts
4. Return to this window and hit "Enter" key to continue

Successfully Setup Software.
```
使用root用户执行脚本,
su - root
```
/u01/app/oraInventory/orainstRoot.sh 
/u01/app/oracle/product/11.2.0/db_1/root.sh
```
注：不同的地址根据自己的实际

### 3.2 增加或修改oracle的环境变量
```
su  - oracle
vim ~/.bash_profile
export ORACLE_HOSTNAME=centosdb
export ORACLE_BASE=/home/data/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_SID=cpaasdb
export PATH=.:$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$ORACLE_HOME/jdk/bin:$PATH
export LC_ALL="en_US"
export LANG="en_US"
export NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"
export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"
```

### 3.3 配置监听程序

```
[oracle@docker ~]$ netca /silent /responsefile /home/data/etc/netca.rsp

Parsing command line arguments:
Parameter "silent" = true
Parameter "responsefile" = /home/data/etc/netca.rsp
Done parsing command line arguments.
Oracle Net Services Configuration:
Profile configuration complete.
Oracle Net Listener Startup:
Running Listener Control:
/u01/app/oracle/product/11.2.0/db_1/bin/lsnrctl start LISTENER
Listener Control complete.
Listener started successfully.
Listener configuration complete.
Oracle Net Services configuration successful. The exit code is 0
```
### 3.4 启动监控程序
```
lsnrctl start
```

## 4 静默dbca建库
### 4.1、复制一份文件
cp /home/data/etc/dbca.rsp /home/data/etc/dbca_xxx.rsp

修改dbca_xxx.rsp 的cpaasdb为cpaasxxx

切换到ORALCE用户


su - oracle
执行以下命令
dbca -silent -responseFile /home/data/etc/dbca_xxx.rsp
如下：
Copying database files
1% complete
3% complete
11% complete
18% complete
26% complete
37% complete
Creating and starting Oracle instance
40% complete
45% complete
50% complete
55% complete
56% complete
60% complete
62% complete
Completing Database Creation
66% complete
70% complete
73% complete
85% complete
96% complete
100% complete
Look at the log file "/home/data/app/oracle/cfgtoollogs/dbca/cpaasxxx/cpaasxxx.log" for further details.

完成后
添加用户
export ORACLE_SID=cpaasxxx
sqlplus / as sysdba
> create user cpaasxxx identified by cpaasxxx;
> grant dba,connect,resource to cpaasxxx;

## 5 导入导出
exp system/123456@213.234.12.32/mydb file=D:\example.dmp
imp system/123456@213.234.12.32/mydb file=D:\example.dmp full=y ignore=y

### 6 oracle

```
su - oracle
lsnrctl start
sqlplus /nolog
conn / as sysdba;
startup
```

