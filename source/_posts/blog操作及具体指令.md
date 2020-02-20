---
title: blog操作及具体指令
date: 2019-03-13 10:52:45
tags:
  - 环境搭建
  - blog
toc: true
categories:
  - 环境搭建
  - blog
comments: true
---

那个文件太长了 新起一个文件 总结换电脑之后的命令操作和图片等问题 ,  还是有点麻烦的 所以记一下。

<!--more-->

### 图片添加方式

```
修改_config.yml配置文件post_asset_folder项为true。
创建博客是使用命令创建：
hexo new [layout] <title>
其中的layout项可以省略，例如：
hexo new "这是一个新的博客"
使用完命令之后，在source/_post文件夹里面就会出现一个“这是一个新的博客.md”的文件和一个“这是一个新的博客”的文件夹。
安装插件
npm install hexo-asset-image --save
将想要上传的图片先扔到文件夹下，然后在博客中使用markdown的格式引入图片：
![你想要输入的替代文字](xxxx/图片名.jpg)
```

#### Typora设置如下

![1552463003251](blog操作及具体指令/1552463003251.png)



### 本地搜索功能开启

```
#添加插件
npm install hexo-generator-searchdb --save
修改blog下的_config.yml文件，进行编辑。
search:
    path: search.xml
    field: post
    format: html
    limit: 10000
    
修改主题配置文件
self_search: true
```

### 新设备命令

直接撸,打开我的电脑-->d: -->新建文件夹 (my_blog) -->右键git bash

```
git clone -b blog_file https://github.com/ejeonline/ejeonline.github.io.git my_blog/
npm install hexo-cli -g
cd my_blog
npm install
npm install hexo-deployer-git --save
npm install hexo-renderer-pug --save
npm install hexo-renderer-sass --save
npm install hexo-asset-image --save
npm install hexo-generator-searchdb --save   #本地搜索功能插件
hexo clean
hexo g
hexo server
hexo clean
hexo g
hexo d

```

### 博客文件提交及日常使用

```
#查看分支
$ git branch -a
* blog_file  #目前所在分支
  remotes/origin/HEAD -> origin/master
  remotes/origin/blog_file
  remotes/origin/master
$ git pull   #更新
$ git status  #状态
$ git reset --hard  #还原到默认版本
$ git add --all #添加文件
$ git commit -m "提交源文件到分支" 提交到本地仓库
$ git push --set-upstream origin blog_file #提交远程仓库

#重新发布
hexo clean
hexo g
hexo d
```

