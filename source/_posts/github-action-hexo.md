---
title: 使用 Github Actions 自动部署 Hexo
date: 2021-08-03 21:35:04
tags:
- Github
- Hexo
categories:
- 写BUG日常
- 教程
---
以后就不需要再自己部署博客了
<!--more-->
最近我一直在困恼一个问题，就是我如何在没有电脑的情况下如何部署我的博客呢？最近有好多想写的东西，并且博客有一些细节我想改动，然而单独购买一台 VPS 的价格让我望而却步，这个时候，我便想到了使用 Github Actions 进行部署。

## 准备工作

- Github Account
- 能联网的计算机

## 配置仓库

登录你的 Github Account。创建两个仓库，一个是 `blog`，用来储存博客源文件，另外一个是 `username.github.io` 用来存放部署玩的 HTML 文件。

将你的博客源文件 push 到仓库中（不会用 Hexo 的可以参考[官方文档](https://hexo.io/zh-cn/docs/index.html)）。在推送前检查以下项目

- _config.yml
   - 在 deploy 项中填写了 Github Pages 仓库地址（即 `username.github.io`），并且注意必须使用 ssh 的连接方式即 git@github:username/username.github.io.git，并且不能出现其他代码储存库如 Gitee 和 Coding。


## 生成密钥

你需要为 Github Actions 单独配置一个新的秘钥


``` bash 生成密钥
ssh-keygen -t rsa -f github-deploy-key
```

此时你得到了两个文件，`github-deploy-key` 和`github-deploy-key.pub`。首先复制 `github-deploy-key`的内容，在 Github 打开 blog 仓库，打开 `Settings->Secrets`，选择 `New Secrets`，Name 填 `HEXO_DEPLOY_PRI`，并将内容粘贴到 Value 里。再复制 `github-deploy-key.pub`的内容，在 Github 打开 username.github.io 仓库，打开 `Settings->Deploy Key`，选择 `Add deploy key`，Title 填 `HEXO_DEPLOY_PUB`，将内容粘贴到 Key 里

## 配置 Github Actions

打开 blog 仓库，打开 Actions 并创建一个新的 Actions，将内容修改为自己的真实情况并复制到 Actions 中。

``` bash Action>folded
name: CI

on:
  push:
    branches:
      - master

env:
  GIT_USER: Username
  GIT_EMAIL: user@example.com
  DEPLOY_REPO: example/username.github.io
  DEPLOY_BRANCH: master

jobs:
  build:
    name: Build on node ${{ matrix.node_version }} and ${{ matrix.os }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [ubuntu-latest]
        node_version: [12.x]

    steps:

      - name: Checkout
        uses: actions/checkout@v2

      - name: Checkout deploy repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.DEPLOY_REPO }}
          ref: ${{ env.DEPLOY_BRANCH }}
          path: .deploy_git

      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL

      - name: Install dependencies
        run: |
          npm install

      - name: Deploy hexo
        run: |
          npm run deploy
```
  
很好，你已经成功部署了一个 Github Actions，每当你对 blog 仓库进行 push 时都会自动激活这个 Action，它会自己部署 Hexo 并提交到你的 username.github.io，很方便是不是？
