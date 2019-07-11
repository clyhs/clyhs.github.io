---
title: jenkins利用testng自动化测试
date: 2019-07-10 16:27:34
category: jenkins
tags: jenkins
---

### 自动化测试

#### Eclipse+TestNG

TestNG插件安装，直接通过eclipse marketplace查找

![img](https://clyhs.github.io/images/jenkins/23.png)

**在工程中新建TestNG类**

![img](https://clyhs.github.io/images/jenkins/24.png)

生成TestNG.xml，**NewTest.java**如下

![img](https://clyhs.github.io/images/jenkins/30.png)

提交到git

```
git add .
git commit -m "2.0.0"
git tag 2.0.0
git push origin 2.0.0
```



#### jenkins安装xUnit plugin和TestNG Result

进入系统管理->插件管理

**安装xUnit plugin**

![img](https://clyhs.github.io/images/jenkins/25.png)

**安装TestNG Result**

![img](https://clyhs.github.io/images/jenkins/28.png)

#### 修改mytest配置

*之前配置看上一篇<a href="http://abigfish.net/2019/06/26/docker%E4%B8%8Ejenkins%E6%90%AD%E5%BB%BA%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90" >docker与jenkins搭建持续集成</a>*

**构件**

![img](https://clyhs.github.io/images/jenkins/27.png)

**构建后操作**

选择publish TestNG Results

![img](https://clyhs.github.io/images/jenkins/29.png)

#### 构建

选择tag为2.0.0

构建完成如下：

![img](https://clyhs.github.io/images/jenkins/31.png)

多了一个testng Results

![img](https://clyhs.github.io/images/jenkins/32.png)

进去

![img](https://clyhs.github.io/images/jenkins/34.png)

#### 查看服务器

![img](https://clyhs.github.io/images/jenkins/33.png)

