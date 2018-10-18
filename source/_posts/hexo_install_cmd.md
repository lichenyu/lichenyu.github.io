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

```
deploy:
  type: git
  repo: https://github.com/lichenyu/lichenyu.github.io.git
  branch: master
```

## [测试] ##

- hexo server
- http://localhost:4000/
- 在 _posts目录下新建Markdown文件
- hexo generate

## [部署] ##

安装 hexo-deployer-git 插件

```
npm install hexo-deployer-git --save
```

- hexo clean
- hexo generate
- hexo deploy

deploy步骤，如有必要需设置一下git的代理

## [添加Latex公式支持] ##

- npm install hexo-math --save
- npm install hexo-renderer-mathjax --save
- 配置 _config.yml 文件

内容如下

```
math:
  engine: 'mathjax' # or 'katex'
  mathjax:
src: "//cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
config:
  # MathJax config
  katex:
css: #custom_css_source
js: #custom_js_source # not used
config:
  # KaTeX config
```

支持行内公式

 - npm uninstall hexo-renderer-marked --save
 - npm install hexo-renderer-kramed --save
 - node_modules\kramed\lib\rules\inline.js，把第11、20行的escape变量的值做相应修改

内容如下

```
//escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
escape: /^\\([`*\[\]()#$+\-.!_>])/

//em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/
```

## [代码高亮] ##

```javascript
<link href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/styles/github.min.css" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

- 相应themes目录的head.ejs中添加highlight.js相关配置。
- (相应themes目录的highlight.styl，`highlight-background = #f6f8fa`。)
- (_config.yml中关闭highlight相关选项。)

