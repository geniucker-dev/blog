---
title: 通过 GitHub Copilot 免费使用 gpt-4
math: false
date: 2024-01-26 15:45:59
updated: 2024-01-31 17:24:00
tags:
  - GitHub Copilot
  - gpt-4
categories:
  - [技术]
---

这次介绍项目可以把 OpenAI API 格式的请求转发到 GitHub Copilot 服务端，从而免费使用 gpt-4。当然前提是你有 GitHub Copilot。对于学生，可以通过 GitHub Education 免费使用。

先上项目链接: <https://github.com/Geniucker/CoGPT>

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

在一个目录下创建一个 `docker-compose.yml` 文件，内容如下

```yaml
version: '3'

services:
  cogpt-api:
    image: ghcr.io/geniucker/cogpt:latest
    environment:
      - HOST=0.0.0.0
    ports:
      - 8080:8080
    volumes:
      - ./db:/app/db
      - ./log:/app/log
    restart: unless-stopped
    container_name: cogpt-api
```

如果你想要使用开发版，将 `ghcr.io/geniucker/cogpt:latest` 替换为 `ghcr.io/geniucker/cogpt:dev`。

默认情况下，服务会监听在 `8080` 端口，如果需要修改，可以修改 `docker-compose.yml` 文件中的 `ports` 部分。例如，如果你想要监听在 `8081` 端口，可以把 `8080:8080` 改为 `8081:8080`。

其他配置也可以在 `environment` 部分修改。或者更方便的是，你可以修改 `.env` 文件（你可以将 `.env.example` 复制为 `.env` 并修改它）。**注意** `db` 和 `log` 的配置应该在 `docker-compose.yml` 的 `volumes` 部分修改。

所有的配置选项都在 [配置](#配置) 部分列出。

然后运行 `docker compose up -d` 来启动服务。

### 本机部署

如果你不想使用 Docker，也可以直接在本机使用

从 [Release](https://github.com/Geniucker/CoGPT/releases) 页面下载对应平台的文件，解压后运行其中的 `cogpt-api` 文件即可。

默认情况下，服务会监听在 `localhost:8080`，如果需要修改，可以参考下文的 [配置](#配置) 部分。

### 作为服务运行

#### Linux

对于基于 systemd 的 Linux，你可以按照下面的步骤操作。

首先，从 [Release](https://github.com/Geniucker/CoGPT/releases) 页面下载最新的 Linux 版本。解压后将 `cogpt-api` 移动到 `/opt/cogpt/` 并赋予可执行权限。

然后将 [cogpt-api.service](https://github.com/Geniucker/CoGPT/blob/main/examples/cogpt-api.service) 的内容复制到 `/etc/systemd/system/cogpt-api.service`。

如果你需要修改配置，你应该编辑 `/opt/cogpt-api/.env` 文件。

最后，运行下面的命令来启用并启动服务。

```bash
sudo systemctl enable cogpt-api
sudo systemctl start cogpt-api
```

运行 `sudo systemctl stop cogpt-api` 来停止服务。

运行 `sudo systemctl disable cogpt-api` 来禁用服务。

#### MacOS

对于 MacOS，服务基于 `launchd`。

首先，从 [Release](https://github.com/Geniucker/CoGPT/releases) 页面下载最新的 MacOS 版本。解压后将 `cogpt-api` 移动到 `/opt/cogpt/` 并赋予可执行权限。

然后将 [com.cogpt-api.plist](https://github.com/Geniucker/CoGPT/blob/main/examples/cogpt-api-service.ps1) 的内容复制到 `/Library/LaunchDaemons/com.cogpt-api.plist`。

如果你需要修改配置，你应该编辑 `/opt/cogpt-api/.env` 文件。

最后，运行 `sudo launchctl load /Library/LaunchDaemons/com.cogpt-api.plist` 来启动服务。

运行 `sudo launchctl unload /Library/LaunchDaemons/com.cogpt-api.plist` 来停止服务。

#### Windows

对于 Windows，我们可以使用计划任务。你可以按照下面的步骤操作。

首先，从 [Release](https://github.com/Geniucker/CoGPT/releases) 页面下载最新的 Windows 版本。解压到一个目录。假设是 `C:\CoGPT\`。

然后在 `C:\CoGPT\` 下创建一个 `cogpt-api-service.ps1` 文件，并把 [cogpt-api-service.ps1](https://github.com/Geniucker/CoGPT/blob/main/examples/cogpt-api-service.ps1) 的内容复制到 `cogpt-api-service.ps1`。

以管理员权限打开 PowerShell 并运行下面的命令。

```powershell
cd C:\CoGPT\
.\cogpt-api-service.ps1 enable
```

下面是所有可以使用的命令。所有命令都应该在 PowerShell **以管理员权限** 运行。

```powershell
./cogpt-api-service.ps1 enable  # 启用并启动服务
./copgt-api-service.ps1 disable # 停止并禁用服务
./cogpt-api-service.ps1 start   # 启动服务
./cogpt-api-service.ps1 stop    # 停止服务
./cogpt-api-service.ps1 restart # 重启服务
./cogpt-api-service.ps1 status  # 检查服务状态
```

### Share Token

如果你想要和朋友共享这个服务，直接共享你的 GitHub app token 是不安全的。这个功能就是为了这种情况设计的。你可以创建一个映射，将所谓的 share token 映射到真实的 GitHub app token。

一种方式是设置环境变量或修改 `.env` 文件。你应该将 `SHARE_TOKEN` 设置为一个字符串，例如 `share-xxxxxxx1:ghu_xxxxxxx1,share-xxxxxxx2:ghu_xxxxxxx2`。格式为 `share-token:real-token,share-token:real-token`。你可以添加一个或多个映射。

另一种方式是使用命令行参数。你可以运行 `./cogpt-api -share-token share-xxxxxxx1:ghu_xxxxxxx1,share-xxxxxxx2:ghu_xxxxxxx2` 来启动服务。你可以添加一个或多个映射。

**注意** share token 必须以 `share-` 开头。不以 `share-` 开头的映射将会被忽略。

为了生成一个随机的 share token，你可以下载最新的 release 文件。解压后运行 `./gen-share-token`。

### 配置

支持两种配置方式，一种是通过环境变量，一种是通过命令行参数。优先级：命令行参数 > 环境变量 > `.env` 文件

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
`SHARE_TOKEN` | `""` | Maps of share-token and real token. For example, `SHARE_TOKEN=share-xxxxxxx1:ghu_xxxxxxx1,share-xxxxxxx2:ghu_xxxxxxx2`.

#### 命令行参数

可以通过 `./cogpt-api -h` 查看所有支持的命令行参数

### 服务端代理配置

与普通配置相同，支持环境变量和命令行参数

环境变量：`ALL_PROXY`, `HTTPS_PROXY`, `HTTP_PROXY`，优先级：`ALL_PROXY` > `HTTPS_PROXY` > `HTTP_PROXY`

命令行参数可以通过 `./cogpt-api -h` 查看

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
