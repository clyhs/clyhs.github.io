---
title: docker与jenkins搭建持续集成
date: 2019-06-26 10:36:39
category: jenkins
tags: [docker,jenkins]
---

### 环境

| 服务器       | 软件                        |
| ------------ | --------------------------- |
| 192.168.0.7  | docker,git,registry,jenkins |
| 192.168.0.16 | docker                      |

### 安装docker和git

**192.168.0.7,192.168.0.16**(先自行安装docker,git)

```
yum install docker
yum install git
```

离线安装git需要以下安装包

   perl-Error-1:0.17020-2.el7
   perl-TermReadKey-2.30-20.el7     
   perl-Git-1.8.3.1-19.el7          
   git-1.8.3.1-19.el7 

在192.168.0.7上创建git

```
#创建git用户
useradd git
passwd git
#创建mytest项目
su - git
mkdir mytest.git
cd mytest.git/
git --bare init
```

在192.168.0.16上执行，可以克隆

```
git clone git@192.168.0.7:/home/git/mytest.git
```

### jenkins安装  

在192.168.0.7上安装jenkins         

下载jenkins包

地址：http://mirrors.jenkins.io/war-stable/latest/jenkins.war 

部署到tomcat/webapps里面，清掉webapps里面自带目录，把jenkins.war 改成ROOT.war

如：/home/domains/tomcat/webapps/ROOT.war

或者直接DOCKER部署

 docker pull jenkins/jenkins

```
docker run -d --name jenkins -p 8888:8080 -p 50000:50000 -v /home/domains/jenkins_home:/var/jenkins_home jenkins/jenkins
```

查看日志

```shell
docker logs -f jenkins
```

![img](https://clyhs.github.io/images/jenkins/01.png)

找到启动输入的password复制

打开http://{ip}:8888/ 输入password

![img](https://clyhs.github.io/images/jenkins/02.png)

选择推荐安装插件。

创建第一个管理员。

![img](https://clyhs.github.io/images/jenkins/03.png)

可以通过<http://{ip}:8888/login?from=%2F>           登录

登录后发现空白，可能是权限问题，解决方法：把映射的目录权限改为777，重启服务

```shell
docker stop jenkins
chmod -R 777 /home/domains/jenkins_home/
docker start jenkins
```

### 部署私有镜像仓库

在192.168.0.7上安装docker私有仓库    

```shell
docker run -d -v /home/domains/registry:/var/lib/registry -p 5000:5000 --restart=always --name registry registry
```

在192.168.0.16、192.168.0.7添加http信任

```
vi /etc/docker/daemon.json
#加入
{"insecure-registries":["192.168.0.7:5000"]}
```

重启docker:systemctl restart docker

然后找一个镜像，重新打一个与标签

这里用nginx:1.15.8

```shell
docker tag nginx:1.15.8 192.168.0.7:5000/nginx:1.15.8
docker push 192.168.0.7:5000/nginx:1.15.8
```

可以在192.168.0.7的/home/domains/registry看到生成一个docker目录，存放刚才push进去的镜像。

### Jenkins配置全局工具配置

系统管理->全局工具配置

先把maven，jdk复制到/home/domains/jenkins_home目录下，下面填写的jdk和maven用容器内部地址

比如jdk在/var/jenkins_home/jdk****目录下

![img](https://clyhs.github.io/images/jenkins/04.png)

![img](https://clyhs.github.io/images/jenkins/05.png)

![img](https://clyhs.github.io/images/jenkins/06.png)

添加凭证，可以远程通过ssh连接192.168.0.7

系统管理->凭据->系统

![img](https://clyhs.github.io/images/jenkins/07.png)

系统管理->系统设置->SSH remote hosts

![img](https://clyhs.github.io/images/jenkins/08.png)

### 上传JAVA项目到git仓库

```
git clone https://gitee.com/clyhs/mytest.git
git remote remove origin 
git remote add origin git@192.168.0.7:/home/git/mytest.git
git add .
git commit -m "init"
git tag 1.0.0
git push origin 1.0.0
```

如何获取

```
git clone git@192.168.0.7:/home/git/mytest.git
git checkout 1.0.0
```

### 创建jenkins项目

![img](https://clyhs.github.io/images/jenkins/09.png)

选择构建一个maven项目（如果没有这个选择，可以去插件管理进行安装）

#### 参数化构建过程

![img](https://clyhs.github.io/images/jenkins/10.png)

![img](https://clyhs.github.io/images/jenkins/11.png)

#### 源码管理

![img](https://clyhs.github.io/images/jenkins/12.png)

如果显示

```
Failed to connect to repository : Command "git ls-remote -h git@192.168.0.7:/home/git/mytest.git HEAD" returned status code 128:
stdout: 
stderr: Host key verification failed 
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

#### 创建一个jenkins

需要在192.168.0.7创建一个jenkins用户

```
useradd jenkins
passwd jenkins
su jenkins
ssh-keygen -t rsa
cat .ssh/id_rsa.pub #复制一下私钥
```

在jenkins里面添加一个jenkins用户凭证

![img](https://clyhs.github.io/images/jenkins/14.png)

![img](https://clyhs.github.io/images/jenkins/15.png)

配置maven

![img](https://clyhs.github.io/images/jenkins/16.png)

#### POST STEP执行shell

```
REPO=192.168.0.7:5000/mytest:${Tag}
cat > Dockerfile << EOF
FROM 192.168.0.7:5000/tomcat:7
run rm -rf /usr/local/tomcat/webapps/ROOT
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
CMD ["catalina.sh","run"]
EOF
docker build -t $REPO .
docker push $REPO
```

![img](https://clyhs.github.io/images/jenkins/17.png)

```
REPOSITORY=192.168.0.7:5000/mytest:${Tag}
# 部署
docker rm -f mytest |true
docker image rm $REPOSITORY |true
docker container run -d --name mytest -p 8111:8080 $REPOSITORY
```

![img](https://clyhs.github.io/images/jenkins/18.png)

### 开始构建

![img](https://clyhs.github.io/images/jenkins/19.png)

根据版本号参数进行构建

![img](https://clyhs.github.io/images/jenkins/20.png)

构建过程发现jenkins里面没有docker命令，为了之前配置jenkins容器的数据不丢失，把运行的jenkins容器，打成镜像**clyhs/jenkins:1.0**

```
docker commit containerId clyhs/jenkins:1.0
```

停止之前运行的容器

```
docker stop containerId
```

启动

```
docker run -d --name jenkins_my -p 8888:8080 -p 50000:50000 -v /home/domains/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker -v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 clyhs/jenkins:1.0
```

* 现没有/var/run/docker.sock权限，

* 缺少libltdl.so.7依赖

只能利用clyhs/jenkins:1.0来打新的镜像**clyhs/jenkins:2.0**



### 重建镜像

vi Dockerfile

```
FROM clyhs/jenkins:1.0
USER root
#清除了基础镜像设置的源，切换成阿里云的jessie源
RUN echo '' > /etc/apt/sources.list.d/jessie-backports.list \
  && echo "deb http://mirrors.aliyun.com/debian jessie main contrib non-free" > /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian jessie-updates main contrib non-free" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian-security jessie/updates main contrib non-free" >> /etc/apt/sources.list
#更新源并安装缺少的包
RUN apt-get update && apt-get install -y libltdl7

ARG dockerGid=999

RUN echo "docker:x:${dockerGid}:jenkins" >> /etc/group \
USER jenkins
```

```
docker build -t clyhs/jenkins:2.0 .
```

#### 重新启动

```
docker run -d --name jenkins_my2 -p 8888:8080 -p 50000:50000 -v /home/domains/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker  clyhs/jenkins:2.0
```

*注意：$(which docker)表示用宿主的docker*

终于构建成功

![img](https://clyhs.github.io/images/jenkins/21.png)

运行的容器如下：

![img](https://clyhs.github.io/images/jenkins/22.png)

用浏览器访问8111，出现正常的hello world!



[参考]: https://blog.51cto.com/lizhenliang/2159817

