---
title: gulp使用笔记
date: 2017-04-15 19:26:39
category: gulp
tags: [js,gulp]
---
# gulp简介
gulp是前端开发过程中对代码进行构建的工具，是自动化项目的构建利器；她不仅能对网站资源进行优化，而且在开发过程中很多重复的任务能够使用正确的工具自动完成；使用她，我们不仅可以很愉快的编写代码，而且大大提高我们的工作效率。

gulp是基于Nodejs的自动任务运行器， 她能自动化地完成 javascript/coffee/sass/less/html/image/css 等文件的的测试、检查、合并、压缩、格式化、浏览器自动刷新、部署文件生成，并监听文件在改动后重复指定的这些步骤。在实现上，她借鉴了Unix操作系统的管道（pipe）思想，前一级的输出，直接变成后一级的输入，使得在操作上非常简单。通过本文，我们将学习如何使用Gulp来改变开发流程，从而使开发更加快速高效。

gulp 和 grunt 非常类似，但相比于 grunt 的频繁 IO 操作，gulp 的流操作，能更快地更便捷地完成构建工作。

_ _ _

## 安装 gulp
```
# npm install -g gulp
# gulp -v
[19:20:24] CLI version 3.9.1

```
## 新建 package.json
```
# npm init
{
  "name": "helloweb",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
## 新建gulpfile.js文件
手动创建就可以
修改`package.json`的`"main":"gulpfile.js"`

## 安装semantic.json
```
# npm install semantic-ui

> Automatic (Use defaults locations and all components)
  Express (Set components and output folder)
  Custom (Customize all src/dist values)
```
选择一项按回车键安装semantic
在目录下生成`semantic.json`与`semantic`目录


