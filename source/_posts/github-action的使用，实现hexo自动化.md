---
title: github action的使用，实现hexo自动化
date: 2021-01-14 11:39:35
tags:
	- github action
categories: 自动化
---

# 前言

github Action是一个github提供的实现持续集成的工具，能够方便的分享和使用他人分享的持续集成的脚本，且github提供一些机器的运行环境来方便的打包运行代码，从而完成我们想要的操作，比方说hexo博客，可以实现推送到指定分支时，就将md编译成html并推送到指定的github page上，当然也可以附加一些其他的内容，比方将所有的md中的图片上传到图床等。

# 目的

实现一个github action能够自动实现当推送新的post到指定分支时，能够自动将图片上传图床并替换md中的连接为图床链接，以及将md内容编译成html然后传到github page上，实现更新。

## 替换图片

写了一个简单的脚本，使用python实现，可以将md中的图片批量上传到图床（目前仅支持路过图床），然后将md文件中的图片链接替换成图床的链接，

项目地址：https://github.com/ggqshr/replace_img_md

# 实现

最后版本的action脚本如下：

```yaml
# This is a basic workflow to help you get started with Actions

name: push_to_page

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ doc,build ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
        with: 
          ref: doc
        

      # Runs a single command using the runners shell
      - name: setup_nodejs
        uses: actions/setup-node@v2.1.4
        with: 
          node-version: '12'

      - name: setup_python
        uses: actions/setup-python@v2
        with:
          python-version: '3.6'

      - name: replace_all_images
        env: 
          PASS: ${{ secrets.LUGUO_PASSWD }}
          USER: ${{ secrets.LUGUO_USER }}
          PROXY: ${{ secrets.PROXY_STRING }}
        run: |
          python -m pip install pqi
          pqi use pypi
          python -m pip install replace_md_img==0.0.10
          export HTTP_PROXY=$PROXY
          export HTTPS_PROXY=$PROXY
          replace_img process -u $USER -p $PASS source/_posts
      
      - name: update_doc_branch
        run: |
          git config --global user.email "action_bot@noreply.com"
          git config --global user.name "publish_action"
          git add .
          git commit -m "upload images[bot]"
          git push https://ggqshr:$GITHUB_TOKEN@github.com/ggqshr/ggqshr.github.io.git doc:doc
        continue-on-error: true

      - name: generate_html
        run: |
          npm install
          npm install -g hexo 
          hexo g
          
      # Runs a set of commands using the runners shell
      - name: publish
        env: 
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          export TZ='Asia/Shanghai'
          git clone -b main https://ggqshr:$GITHUB_TOKEN@github.com/ggqshr/ggqshr.github.io.git .deploy
          cp -r public/* .deploy/
          cd .deploy
          git add .
          date +"%Y-%m-%d %H:%M:%S" | git commit -F -
          git push https://ggqshr:$GITHUB_TOKEN@github.com/ggqshr/ggqshr.github.io.git main:main
      
```

可以看到action的文件中有一个Jobs，主要分为以下几个工作

[![github-action的使用，实现hexo自动化_0](https://s3.ax1x.com/2021/01/14/sas5z4.png)](https://imgchr.com/i/sas5z4)

其中有几个点需要注意，

1. 在推送代码和拉去代码时，可以使用以下的形式来，即token认证的形式来验证身份，而不用重复登陆

```bash
git clone https://username:token@github.com/username/repo.git
```

2. 可以使用github的secrets功能，来保存一些私密的信息，比如用户名和密码等，因为这个脚本是要放到github项目目录下的，是公开的，如果直接明文的写容易有安全问题，另外一方面，硬编码也不够灵活，可以通过以下步骤来设置

   [![github-action的使用，实现hexo自动化_1](https://s3.ax1x.com/2021/01/14/sasoQJ.png)](https://imgchr.com/i/sasoQJ)

   按照图上1,2,3的位置分别设置，然后就能在4的位置看到设置的值。这样设置之后就可以在action脚本当中取值，使用`${{ secrets.SERETS_NAME }}`

按照以上的步骤即可完成需求，实现自动化。



