title: linux db2安装
date: 2022-08-10 09:22:25
category: linux
tags: db2

# linux db2安装

## 1 db2安装
解压db2安装包,进入server目录下，执行安装检查
```
tar -zxvf v9.7_linuxx64_server.tar.gz
cd server
./db2prereqcheck
```
运行安装程序
```
[root@server]./db2_install
```
创建DB2运行所需要的用户组和用户
```
groupadd -g 901 db2iadm1
groupadd -g 902 db2fadm1
groupadd -g 903 dasadm1
useradd -g db2iadm1 -u 801 -d /home/db2inst1 -m  db2inst1
useradd -g db2fadm1 -u 802 -d /home/db2fenc1 -m  db2fenc1
useradd -g dasadm1 -u 803 -d /home/dasadm1 -m  dasusr1
```
为db2inst1创建密码
```
passwd db2inst1
passwd db2fenc1
passwd dasusr1
```
进入/opt/ibm/db2/V9.7/instance目录
创建实例
```
[root@server]#cd /opt/ibm/db2/V9.7/instance

[root@instance]#./dascrt -u dasusr1

SQL4406W  The DB2 Administration Server was started successfully.

DBI1070I  Program dascrt completed successfully.


[root@instance]#./db2icrt -u db2inst1 db2inst1

DBI1070I  Program db2icrt completed successfully.
```
启动db2实例
```
[root@instance]#su - dasusr1
[dasusr1@db2]$db2admin start

[dasusr1@db2]$su - db2inst1
[db2inst1@db2]$db2start
```
关闭、启动数据库
```
[db2inst1@db2]$db2stop

[db2inst1@db2]$db2 force applications all

[db2inst1@db2]$db2start


```

创建样本库

```
[db2inst1@db2]$cd /opt/ibm/db2/V9.7/bin

[db2inst1@db2]$./db2sampl
```
设置DB2自启动
```
[root@db2]#cd /opt/ibm/db2/V9.7/instance

[root@instance]#./db2iauto -on db2inst1
```
配置TCPIP
```
[root@instance]#su - db2inst1

[db2inst1@db2]$db2set DB2COMM=TCPIP
```
创建数据库实例
db2 "create db cpaasm using codeset UTF-8 territory CN pagesize 8192 catalog tablespace managed by database using (file '/home/db2inst1/cpaasmts' 131072 ) user tablespace managed by database using (file '/home/db2inst1/cpaasuserspace' 131072 ) temporary tablespace managed by system using ('/home/db2inst1/cpaasmtempspace')"

创建额外BufferPool及表空间
```
db2 => connect to cpaasm

   Database Connection Information

 Database server        = DB2/LINUXX8664 9.7.7
 SQL authorization ID   = DB2IFFCS
 Local database alias   = cpaasm

db2 => create bufferpool buffer_assp8k size 25600 pagesize 8k
DB20000I  The SQL command completed successfully.
db2 => create bufferpool buffer_idx8k size 25600 pagesize 8k
DB20000I  The SQL command completed successfully.
db2 =>  create bufferpool buffer_clob32k size 6400 pagesize 32k
DB20000I  The SQL command completed successfully.
db2 => create tablespace assp pagesize 8192 managed by database using (file '/home/db2inst1/asspspace_c1' 131072) bufferpool buffer_assp8k autoresize no 
DB20000I  The SQL command completed successfully.
db2 => create tablespace assp_index pagesize 8192 managed by database using (file '/home/db2inst1/idxspace_c1' 131072) bufferpool buffer_idx8k autoresize no
DB20000I  The SQL command completed successfully.
db2 => create tablespace assp_clob pagesize 32768 managed by database using (file '/home/db2inst1/clobspace_c1' 32768) bufferpool buffer_clob32k
DB20000I  The SQL command completed successfully.
```

为数据库应用用户授权
```
db2 => grant connect on database to user cpaasdb
grant bindadd on database to user cpaasdb
grant createtab on database to user cpaasdb
DB20000I  The SQL command completed successfully.
db2 => DB20000I  The SQL command completed successfully.
db2 => DB20000I  The SQL command completed successfully.
db2 => grant implicit_schema on database to user cpaasdb
DB20000I  The SQL command completed successfully.
db2 => grant load on database to user cpaasdb
DB20000I  The SQL command completed successfully.
db2 => grant use of tablespace assp to user cpaasdb
DB20000I  The SQL command completed successfully.
db2 => grant use of tablespace assp_index to user cpaasdb
DB20000I  The SQL command completed successfully.
db2 => grant use of tablespace assp_clob to user cpaasdb
DB20000I  The SQL command completed successfully.
```
#
https://blog.csdn.net/zhlh_xt/article/details/40344709
删除数据库
db2 drop database cpaasm