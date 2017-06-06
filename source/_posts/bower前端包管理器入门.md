---
title: bower前端包管理器入门
date: 2017-04-15 11:52:53
category: bower
tags: [bower,js]
---
# 什么是bower
Bower是一个客户端技术的软件包管理器，它可用于搜索、安装和卸载如JavaScript、HTML、CSS之类的网络资源。其他一些建立在Bower基础之上的开发工具

## 准备工作
1.  安装npm
2.  安装nod.js
3.  安装bower

```
# npm install -g bower
```

### bower初始化
进入web项目静态目录如`static`或者`template`
```
# bower init
```
**注意：**在WINDOWS下如题用git窗口执行可以会报如下错误
```
$ bower init
bower ENOINT        Register requires an interactive shell

Additional error details:
Note that you can manually force an interactive shell with --config.interactive
```
记得切换到win的`CMD`命令下执行`bower init`
![img](https://clyhs.github.io/images/js/bower01.png)
些时目录下生成`bower.json`
```
{
  "name": "helloweb",
  "homepage": "index.html",
  "authors": [
    "xxx <xxx@126.com>"
  ],
  "description": "web",
  "main": "index.html",
  "license": "MIT",
  "private": true,
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ]
}
```
### 安装第三方JS如`jquery`
```
# bower install jquery --save
```
.
├── static
│   ├── bower_components
│   │     ├── jquery
│   └── bower.json
│   └──index.html
在`index.html`引用
```
<script type="text/javascript" src="bower_components/jquery/jquery.min.js"></script>
```