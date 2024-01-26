---
title: 通过 GitHub Copilot 免费使用 gpt-4
math: false
date: 2024-01-26 15:45:59
updated: 2024-01-26 15:45:59
tags:
  - GitHub Copilot
  - gpt-4
categories:
  - [技术]
---

这次介绍项目可以把 OpenAI API 格式的请求转发到 GitHub Copilot 服务端，从而免费使用 gpt-4。当然前提是你有 GitHub Copilot。对于学生，可以通过 GitHub Education 免费使用。

先上项目链接: [CoGPT](https://github.com/Geniucker/CoGPT/)

## 原理介绍

通过修改 Copilot Chat 插件的 js 文件，我们可以 dump 出请求体和请求头，我们可以发现 GitHub Copilot 基本上就是一个提供了 gpt 模型的 API 服务，我们只需要把请求转发到 GitHub Copilot 服务端就可以。但是经过测试，Copilot 并不支持 OpenAPI 最新的模型，但是其包含的 gpt-3.5-turbo 和 gpt-4 基本上可以满足我们的需求。

## 项目发展过程

最开始使用的是 Python 的 FastAPI 作为服务器，因为 Python 语言本身较为简单，同时 FastAPI 提供了较为方便的请求体解析，异步的支持使得他的性能在 Python 的 http 框架中较为突出。如有兴趣，具体代码可以看 [py](https://github.com/Geniucker/CoGPT/tree/py) 分支。

然而，过程中遇到了一些困难：

如何返回流式响应：查找了文档，可以使用 `fastapi.responses.StreamingResponse` 函数，我们只需要写一个异步生成器，就可以返回流式响应。但是，如果在流式响应的过程中发生了错误，这个异常会被 FastAPI 捕获，导致我们无法自行处理异常。这个问题非常难以解决，所以最终我还是放弃了 Python 作为服务器的方案。

那么服务端剩下的选择基本上就是 Go, Java, Node.js 之类的语言了。由于我之前有接触过一点 Go，同时也给 [copilot-gpt4-service](https://github.com/aaamoon/copilot-gpt4-service) 项目贡献过了较多代码，所以我就选择了 Go 作为服务端的语言。这次我们使用了 [gin](https://github.com/gin-gonic/gin) 框架。

## 如何使用

### 警告

这个项目仅适合**个人**使用。并不适合访问量巨大的盈利项目。

最佳实践方式：

- 本机部署，仅自己使用（推荐）
- 部署在个人服务器上，仅自己使用，或和几个朋友共同使用（不公开）

不建议的方式：

- 提供公共服务
  - 在一个 ip 上使用了很多 token 容易被判定为异常行为
- 使用 Serverless 服务
  - 由于 Serverless 服务的 ip 不固定，所以很容易被判定为异常行为
- 用于盈利项目
  - 请求量过大，容易被判定为异常行为

<font size=5>**重要提示**</font>

请不要尝试上述任何一种不建议的方式，否则可能会导致 GitHub Copilot 账号，甚至 GitHub 账号被封禁。

### Docker 部署

Linux 上非常推荐使用 Docker 部署

```bash
git clone https://github.com/Geniucker/CoGPT.git && cd CoGPT
docker compose up -d
```

默认情况下，服务会监听在 `8080` 端口，如果需要修改，可以修改 `docker-compose.yml` 文件中的 `ports` 部分。例如，如果你想要监听在 `8081` 端口，可以把 `8080:8080` 改为 `8081:8080`。

### 本机部署

如果你不想使用 Docker，也可以直接在本机使用

从 [Release](https://github.com/Geniucker/CoGPT/releases) 页面下载对应平台的文件，解压后运行其中的 `cogpt-api` 文件即可。

默认情况下，服务会监听在 `localhost:8080`，如果需要修改，可以参考下文的 [配置](#配置) 部分。

### 配置

支持两种配置方式，一种是通过环境变量，一种是通过命令行参数

#### 环境变量

支持环境变量和 `.env` 文件，环境变量的优先级高于 `.env` 文件

下面是所有支持的环境变量和默认值

keys | default | description
--- | --- | ---
`HOST` | `localhost` | Host to listen on
`PORT` | `8080` | Port to listen on
`CACHE` | `true` | Whether to cache tokens in sqlite database. If false, tokens will be cached in memory
`CACHE_PATH` | `db/cache.sqlite3` | Path to sqlite database. Only used if `CACHE` is `true`
`DEBUG` | `false` | Whether to enable debug mode. If true, the service will print debug info
`LOG_LEVEL` | `info` | Log level. 

#### 命令行参数

可以通过 `./cogpt-api -h` 查看所有支持的命令行参数

### 使用

#### 获取 token

首先，你要确保你的 GitHub 账号有 GitHub Copilot 的权限。然后从 [Release](https://github.com/Geniucker/CoGPT/releases) 页面下载对应平台的文件，解压后运行其中的 `cogpt-get-apptoken` 文件即可，按照说明操作即可。

如有必要设置代理，运行`./cogpt-get-apptoken -h` 查看帮助

#### 使用

假设你部署的服务在 `localhost:8080`，你可以在原本应该填`https://api.openai.com`的地方替换为`http://localhost:8080`，然后在填写 token 的地方填写你刚刚获取的 token 即可。

## 许可申明

这个项目大体上基于 [copilot-gpt4-service](https://github.com/aaamoon/copilot-gpt4-service) 项目，但是我本身就是这个项目的 Collaborator，并且有很多代码都是我写的，同时 [copilot-gpt4-service](https://github.com/aaamoon/copilot-gpt4-service) 为 MIT 协议，所以没有许可问题。

由于 [copilot-gpt4-service](https://github.com/aaaMoon/copilot-gpt4-service) 的架构较为混乱，并且 PR 保持较多的状态，所以在原项目重构不太现实，所以我就自己写了一个。

当前 [CoGPT](https://github.com/Geniucker/CoGPT/) 项目为 [MPL-2.0](https://github.com/Geniucker/CoGPT/blob/main/LICENSE) 协议。
