---
title: npm使用国内（淘宝）源
categories:
  - [技术]
tags:
  - npm
  - 实用技巧
date: 2020-03-31 10:50:36
updated: 2020-03-31 10:50:36
---
---
&emsp;&emsp;npm是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题。然而，npm默认是从国外源下载包的，这样国内用户使用速度就会很慢。但是，我们可以配置国内镜像（淘宝）来解决这个问题。这里只介绍一种方法（另一种方法是使用cnpm，但我在使用过程中发现了一些问题，所以不推荐）：
1. 在控制台执行：
   ```
   npm config set registry https://registry.npm.taobao.org
   ```
2. 验证命令：
   ```
   npm config get registry
   ```
   如果返回`https://registry.npm.taobao.org`，说明镜像配置成功。