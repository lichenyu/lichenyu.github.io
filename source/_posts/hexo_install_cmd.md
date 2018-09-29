title: Install Hexo
date: 2018/09/26
categories:
- Test
tags:
- Hexo
---


## [Github Pages] ##

创建Repo，名称为username.github.io。

## [Git] ##

安装

## [NodeJs] ##

安装

## [Hexo] ##

管理员打开cmd

- （如需代理）npm config set proxy http://l00429910:password@proxycn2.huawei.com:8080
- （国内加速）npm config set registry "https://registry.npm.taobao.org"
- npm install -g hexo-cli

进入本地username.github.io目录

- （如需代理）git config --global http.proxy http://l00429910:password@proxycn2.huawei.com:8080
- （如需代理）git config --global http.sslverify false
- hexo init
- npm install

## [配置] ##

配置 _config.yml 文件

    deploy:
      type: git
      repo: https://github.com/lichenyu/lichenyu.github.io.git
      branch: master

## [测试] ##

- hexo server
- http://localhost:4000/
- 在 _posts目录下新建Markdown文件
- hexo generate

## [部署] ##

安装 hexo-deployer-git 插件

	npm install hexo-deployer-git --save

- hexo clean
- hexo generate
- hexo deploy

deploy步骤，如有必要需设置一下git的代理
