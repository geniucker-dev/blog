---
title: 再记Hexo博客部署
mathjax: false
date: 2023-03-22 14:20:10
tags:
  - 建站
categories:
  - [技术]
update:
---
我们都知道，Hexo提供了`hexo d`这样一键部署的命令，这对于把博客放在本地的用户来说确实非常方便。但是由于一些原因我想要把Hexo的原始文件托管到GitHub上，这就需要想一个部署方案，既能本地推送部署，也能直接网页在线修改后部署。所以这肯定要用到CI来实现push之后自动发布网页。这里记录一下两个方案。

## 方案一：Vercel
Vercel 真的是一个非常好的平台，给开发者提供了免费的网页托管和 Serverless Functions 等一些非常有用的东西。如果你要通过 Vercel 实现上述的操作，可以看 [vercel的文档](https://vercel.com/guides/deploying-hexo-with-vercel), 这里我就不赘述了。由于我 Vercel 的用量有点大了，所以我就使用了下面的这种办法  

## 方案二：GitHub Actions
众所周知，GitHub为公共仓库提供了无限的CI时间，~~所以怎么能不白嫖呢~~/doge  
下面分成两个部分来介绍如何使用GitHub的CI来部署GitHub Page  

### `hexo g -d`的时候发生了什么
在开始写GitHub的CI之前，我们先要知道hexo原本的部署流程是怎么样的，比较典型的用法是`hexo g -d`，也就是生成并部署，期间发生了如下的事情：  
1. `hexo g`生成博客的静态文件到`public`目录  
2. 将`public`目录的内容同步到`.deploy_git`内的git仓库  
3. 强制推送到远程仓库  
4. GitHub Pages部署的CI自动运行，完成网页的部署  

### 我们如何实现
如上文分析，我们需要的博客的静态文件实际上在`public`文件夹里，所以我们只需要在Action里执行`hexo g`然后把`publish`文件夹的内容跑GitHub Pages的workflow（这可以看GitHub的文档）就可以了。  
然后我们注意到hexo博客目录下有一个`node_modules`目录比较大，它是博客所需的依赖，但实际上依赖信息都已经被记录到了博客目录下的`yarn.lock`或`package.json`这样的文件中了，我已我们应该把这个文件夹添加到`.gitignore`里，我们只要在Action中执行`npm install` (npm)或`yarn install` (yarn)即可。
下面是最终的workflow配置文件 (它应该是仓库的`.github/workflows/`目录里的某一个`yaml`文件，比如我的是`.github/workflows/deploy.yaml`)，我用的是yarn，如果你使用npm可以把第44和49行的`yarn`换成`npm`  
最终结果是每次push之后，都会自动运行这个workflow，并把网页部署到这个仓库的GitHub Pages(不会同时生成一个新的分支)  
```yaml
# This is a basic workflow to help you get started with Actions

name: deploy

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the "main" branch
  push:
    branches: [ "main" ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3
      - name: Setup Node.js environment
        uses: actions/setup-node@v3.6.0
        with:
          # Version Spec of the version to use. Examples: 12.x, 10.15.1, >=10.15.0.
          node-version: 18.15.0
      - name: generate
        run: |
          yarn install
          yarn hexo g
          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: public
          
  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1

```