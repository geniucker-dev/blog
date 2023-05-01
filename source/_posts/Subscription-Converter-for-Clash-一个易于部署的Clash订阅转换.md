---
title: Subscription Converter for Clash - 一个易于部署的Clash订阅转换
tags:
  - Clash
  - 订阅转换
  - subconverter
  - subscription
categories:
  - [技术]
math: false
date: 2023-05-01 11:21:31
update:
---

自己的项目别人都发完博客了结果自己还没发过233，还是稍微记录一下  

## 项目缘起
~~高考前两个月太空~~  
一直在担心公共订阅转换的安全性问题。已开始我看到了 [subconverter](https://github.com/tindy2013/subconverter) ，但是当时身无分文，只能部署在 [Vercel](https://vercel.com/) 这样的 Serverless Function 里。但是由于 subconverter 是用 C++ 写的，无法在 Vercel 上部署（曾经可以部署可执行文件，但是我死活弄不出来），所以就有了自己写订阅转换的想法。  

## 初始想法
1. 我需要用 Linux，我需要解决节点自动更新的问题，这个可以用Clash的proxy-provider解决  
2. 暂时只要支持我当时所用的一元只因厂就可以了，如果做一个通用型的Clash订阅转换太复杂了(即使只是Clash到Clash的转换)，所以我硬编码了大部分内容，连地区分组都是硬编码的，这会导致问题，之后会说  

## 初步实现
Vercel 的 Serverless Function 支持 Node.js, Go, Python, Ruby，显然我只会 Python，而且刚好之前摸过一点点 [Flask](https://flask.palletsprojects.com/) ，所以就用了 Flask。  
初代的文件结构主要是这样的
```shell
.
│  base.py                  #包含了拉取规则的函数
│  head.py                  #硬编码的配置文件组成部分，包括了地区分组等 
│  index.py                 #这个api的主程序
│  requirements.txt         #python模块依赖列表
│  ruleList.py              #规则来源列表
│  vercel.json              #vercel配置文件
│
├─static                    #静态网页的js，用来渲染Markdown
│  └─js
│          markdeep.min.js
│
└─templates                 #静态网页，一个简单的首页，用于引导
        index.html
```
当时的流程非常简单：用户发起get请求，服务端拉取规则，生成proxy-provider的部分，再和其他硬编码的部分进行拼接，就可以返回给用户了。  

## 问题
可以欣赏一下我硬编码的屎山代码：<https://github.com/Geniucker/sub-conv/tree/a4b4ed09956f5c55442a4e365d7f58192d5059b6>，这个代码简单到你一看就知道我在干嘛(特别是`head.py`文件)  
最大的问题就在于我硬编码了大部分内容，这会导致很多问题：
1. 虽然开始的时候一元的节点配置很稳定，节点的地区基本不变，但是到后来他频繁的增删节点，例如美国节点时有时无。在美国节点消失的时候，由于地区组是通过provider的filter匹配出来的，如果匹配不到就会报错。而我只能通过改代码来解决这个问题。  
2. 我有使用多个只因厂的需求，在硬编码的情况下，这极难实现  
3. 维护困难  

## 解决
要解决这些问题，只有一个办法：重构除了拉取规则以外的几乎所有部分<https://github.com/Geniucker/sub-conv/commit/e48bbc64257be11c77abcc8b06853888bcf4a98f> (这次提交好像还是有bug的)  
于是我直接做了一个通用型的Clash订阅转换  
主要的工作流程是这样的：用户发起get请求->服务端获取节点信息，生成地区分组->获取规则->生成其他部分并拼接->返回给用户  
那么到这里之后其实功能已经实现的大差不差了，但是还是没有实现多只因厂，也发现了别的问题：调整规则组还是要修改代码，对用户不友好;用户每次请求接口都需要拉取一次规则，很长的时间花在网络上，所有就想着弄缓存。  
于是，我事先了如下内容：
- 多只因厂支持  
- 配置文件  
- 使用 GitHub Action 定时拉取一次规则然后缓存到仓库里（这会导致一个问题，fork仓库的自动更新规则产生的提交会和我都仓库的提交产生冲突导致无法拉取上游更新，我还不知道怎么解决。***要是你知道如何解决请务必联系我***）

之后还遇到了 Vercel 用量告急的问题，这是因为请求太多了。在早先的实现里，我为每个地区的组单独开了provider，每个provider都会分别发出请求以拉取节点更新。在得知meta内核支持组内filter之后，提供了meta内核的选项，这样就可以每个只因厂只用一个provider然后在分组里面再匹配出对应地区的节点。  

## 当前支持的功能
综上，目前我实现了如下功能
- 一个可以勉强能看的订阅转换 Web-UI (感谢 [@Musanico](https://github.com/musanico))  
- 大体基于 ACL 的规则（包括了ZJU专用规则）  
- 基于 Provider 的节点自动更新  
- 对ZJU Connect的支持  
- 多只因厂支持  
- 剩余流量和总流量的显示（单机场的时候才有用，需要你的机场和你用的Clash同时支持，已知Clash for Windows, Clash Verge, Stash, Clash Meta for Android等已支持）  
- 实现了 clash 订阅转换 proxy-provider 的 api（以前是使用别人的api）  
- 支持配置文件 (`config.py`)

## 部署
请查阅 [Wiki](https://github.com/geniucker/sub-conv/wiki)  
虽然使用英文写的，但是我懒得用中文再写一遍了  