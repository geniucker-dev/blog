---
title: Python pip 使用国内源
categories:
  - [技术]
tags:
  - Python
  - pip
  - 实用技巧
date: 2020-03-25 18:32:11
updated: 2020-03-25 18:32:11
---

---
&emsp;&emsp;众所周知，Python使用pip安装第三方包时，会从 <https://pypi.org/> 资源库中下载，然而在国内，会面临下载速度慢，甚至无法下载的尴尬。这时，配置一个国内源就很重要。下面介绍方法：  
## 方法一：临时使用国内镜像安装
安装时加上参数`-i <国内镜像地址>`（文末会给出几个国内镜像地址），例如：

    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package

## 方式二：永久使用国内pypi镜像安装
1. 升级 pip 到最新的版本 (>=10.0.0) 后进行配置：
   ```
   pip install pip -U
   ```
   若使用默认源较慢可临时使用国内镜像
   ```
   pip install -i <国内镜像地址> pip -U
   ```
2. 进行配置：
   ```
   pip config set global.index-url <国内镜像地址>
   ```

## 推荐几个国内镜像
    清华大学：https://pypi.tuna.tsinghua.edu.cn/simple/
    阿里云：https://mirrors.aliyun.com/pypi/simple/
    豆瓣：https://pypi.douban.com/simple/