---
title: 免费部署自己的ChatGPT API
math: false
date: 2023-06-22 11:41:17
updated: 2023-06-22 12:47:17
tags:
  - ChatGPT
  - API
  - 部署
categories:
  - [技术]
---
知周所众，ChatGPT有两种用法：  
1. 网页版: 频率限制宽，免费，但是有 Cloudflare 检测，不稳定  
2. API：兼容各种服务，稳定，但免费版频率限制严，不适合像我这样的穷逼使用  

然而，例如 Openai Translator 这样的服务，需要使用 API ，所以我一直在等待一个能把网页版转换成 API 的项目。之前一段时间我终于等到了这个项目 [ChatGPT to API](https://github.com/acheong08/ChatGPT-to-API)。  
但是实践之后发现这个项目的文档不是十分完整，对于小白来说还是有一定的难度，所以我写了一个脚本来帮助大家快速部署。这个脚本可以帮你部署在 Docker 上，也可以帮你部署在宿主机上，同时我还通过一些方法绕过了 Cloudflare，所以不必担心出现 Access Denied 的情况。  

## 步骤
### 安装依赖
- git  
- Python3  
- Docker (如果你想部署在 Docker 上)  
- golang (如果你想部署在宿主机上)  
- 至少一个 ChatGPT 账号 (多个账号可以提高频率限制)  

### 克隆项目
你需要先把本项目克隆到你的电脑上，然后进入项目目录。假设目录是 `~/ChatGPT-to-API`  

### 修改配置
打开 `run.py`，修改如下变量：  
- `proxy`：你的代理地址 (只写ip或域名及端口)，如果不需要代理，改为空字符串`""`  
- `proxy_type`：你的代理类型，`socks5` 或 `http`
- `accounts`：这是一个包含账号信息的字典，你可以在这里添加账号，支持一个或多个账号 (不支持谷歌或微软登录)，格式如下：  
```python
{
    "账号1": "密码1",
    "账号2": "密码2",
    "账号3": "密码3",
    ...
}
```
- `access_tokens`：如果你一定要使用谷歌或微软账号登录，这可能是目前唯一的办法，获得的办法可以网上查，注意，这是**不推荐**的办法，因为通过这种方法access token到期之后无法自动刷新。  
- `server_host`：监听的地址，如果你想部署在 Docker 上，改为`0.0.0.0`。如果你想部署在宿主机上且只想让本机访问，改为`127.0.0.1`或`localhost`  
- `server_port`：监听的端口，如果你想部署在 Docker 上，改为`8080`  

### 安装 Python 依赖
- 如果部署在 Docker 中，忽略这一步  
- 如果部署在宿主机上，运行 `pip3 install -r requirements.txt`

### 构建
用 python 运行 `build.py`，并按照提示操作。**请注意**，无论是否成功构建，最后都会输出`Build finished. Press enter to exit...`，所以请自行观察这句话前面的输出，如果有错误，大概率是代理设置有问题，可以尝试把`socks5`改为`http`。  

### 运行
- 如果部署在 Docker 中，运行 `docker compose up -d`，若要查看是否成功，可查看`chatgpttoapi`容器的日志  
- 如果部署在宿主机上，Windows 用户运行 `python run.py`，Linux 用户运行 `python3 run.py`，查看输出即可判断是否成功  

### 使用
这个项目实现了 /v1/chat/completions 接口，其余接口未实现，所以语音识别等功能无法使用
调用方法和官方 API 一致，所以请查阅 OpenAI API 的文档  

## FAQ
### 如何更新
更新脚本：`git pull`  
更新 ChatGPT to API：重新运行`build.py`构建 (会自动拉取 ChatGPT to API)，然后运行  

## Enjoy~