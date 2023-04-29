---
title: Chrome小技巧：让新标签页的搜索框变成真的
categories:
  - [技术]
tags:
  - Chrome
  - 实用技巧
date: 2020-03-29 16:16:22
updated:
---
---
&emsp;&emsp;使用Chrome的用户都知道，Chrome的新标签页的搜索框是“假的”，当你定位到搜索框开始输入时光标会立即跳到地址栏。这样的设计似乎有点反人类（仅个人意见，觉得不对请轻喷），尤其对于刚开始使用Chrome的用户来说需要花一段时间才能适应。其实，在Chrome的flags里有选项让搜索框变成“真的”。下面给出步骤：

<!-- more -->

#### 步骤一：
在Chrome的地址栏内输入`chrome://flags`进入如下页面：
![pic1](https://source.geniucker.top/20200329-1-1.jpg)

#### 步骤二：
在搜索框中输入关键词`real search box`，将得到如下结果：
![pic2](https://source.geniucker.top/20200329-1-2.jpg)
`Real search box in New Tab Page`：把新标签页的搜索框变成真的  
`Zero Suggestions in real search box on New Tab Page`：作用是单击新标签页中的搜索框能显示搜索记录  
`Make the New Tab page real search box match the omnibox's theme colors`：让新标签页的搜索框颜色和地址栏一样  
把需要的功能右边的选择框中的`Default`改为`Enabled`并按照提示重启浏览器即可（重启可能导致其他标签页的数据丢失）。
