---
title: Hexo + GitHub 搭建个人博客, 并实现远程备份
tags:
- Blog
- Git
categories:
- Blog
---

<!-- ![](http://img06.tooopen.com/images/20170106/tooopen_sy_195886579867.jpg) -->

<img src="http://img07.tooopen.com/images/20170630/tooopen_sy_215144649617.jpg">

之前按照网上的一般教程搭建了Hexo博客，但是在更换电脑想改博客的时候发现很难操作。因此又基于git重新搭建了一个新的博客，在这里做一个总结。

这篇博客搭建教程会比传统教程多出好几步，但是可以实现在各台电脑上都可以写博客了，而且可以防止数据丢失，相当于你的Hexo文件架在了GitHub上，而不只是在本地电脑中。读者可能需要一些基本的git操作的知识。

> 此篇文章下依赖的系统环境为Ubuntu 16.04

> 这里假设你的GitHub的用户名叫admin，读者根据自己的实际用户名做相应替换,以下凡是出现admin的地方皆换为自己的用户名

> [Hexo](https://hexo.io/zh-cn/docs/)官方文档

## Hexo 安装

在安装前，电脑中必须已装好两项应用：
- [Node.js](https://nodejs.org/en/)
- [Git](https://git-scm.com/downloads)

如果您的电脑上已经装有上述应用，那么可以直接跳过此两项安装过程，如果未安装，安装过程如下

#### 安装 Node.js

安装 Node.js 的最佳方式是使用 nvm。

``` bash
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
$ nvm install stable
```

#### 安装 Git

``` bash
$ sudo apt-get install git-core
```

### 安装 Hexo

``` bash
$ npm install -g hexo-cli
```

## 建站
安装 Hexo完成后，在指定文件夹下执行下列命令
``` bash
$ hexo init admin.github.io
$ cd admin.github.io
$ npm install
$ git init
```

建站后文件目录如下：

![](http://7xpqdb.com1.z0.glb.clouddn.com/%E5%88%9D%E5%A7%8B%E5%8C%96hexo.jpg)

建站后基础配置可参考[官方文档](https://hexo.io/zh-cn/docs/configuration.html)

## 新建GitHub仓库

在你的GitHub上新建一个仓库，名为admin.github.io

## Hexo主题

Hexo主题的配置网上可以找到很多方法，这里的方法略有不同，主要是为了之后Hexo文件push到GitHub上后可以与这里的主题项目良好的兼容，因为除非自己编写，否则这些主题都是别人的项目，但在我的git项目中又需要引用这个主题项目。这里就牵涉到git的子模块了。以下不做非常详细的关于子模块的说明，但是读者必须要按照以下的顺序来执行。

使用这种方法是为了以后自己修改主题的话也可以push到远程。

> 这里以next主题为例，读者可以根据自己找到的主题，将主题名称替换以下的next,并将git地址替换以下的地址

首先找到你想要的主题的GitHub,如https://github.com/iissnan/hexo-theme-next ，fork到自己的GitHub上。

然后执行

``` bash
$ cd admin.github.io
$ git submodule add https://github.com/admin/hexo-theme-next.git themes/next
```

之后在站点的_config.yml文件下找到theme字段，将其值改为你的主题名称
如：
``` yml
theme: next
```

> 具体主题的配置搜索相关主题的说明

## 与远程的连接

按照以上步骤操作下来，相当于已经初始化了一个选有主题的Hexo文件了，下面可以先将这个文件push一下,注意这里一定要新建一个分支，因为master要拿来做部署用，假设分支叫hexo

``` dash
$ git checkout -b hexo
$ git add .
$ git commit -m "new hexo"
$ git remote add origin git@github.com:admin/admin.github.io.git
$ git push origin hexo
```

可以看一下GitHub上是否已经成功传入了代码，注意分支是否正确。

## 本地修改博客

当你在本地修改了博客，或者修改了主题之后，需要重新push你的新代码，这里需要注意主题是一个子模块。

如果修改了主题，那么需要先将主题文件push上去，即push到之前fork下来的项目。

在theme/next目录下
``` bash
$ git add .
$ git commit -m "modified theme next"
$ git push origin master
```

如果你还想同步更新主题作者的更新，那么可以再使用fork同步的方法，这个不是本文的重点不赘述，读者可自行查阅方法。

然后再提交修改过的博客，即先保证admin.github.io目录下为hexo分支，然后执行
``` bash
$ git add .
$ git commit -m "modified blog"
$ git push origin hexo
```

这样即实现了主题和博客的独立修改，且主题可保持与作者的更新。虽然操作较麻烦，但是这样会让自己的配置结构更优，可以避免很多问题。

## 异地修改博客

当更换电脑，或原来的Hexo文件意外丢失之后，就可以用以下方法从GitHub上获取到Hexo文件，然后进行修改。

如果是新电脑，先安装好git, nodejs和hexo

执行
``` bash
$ git clone -b hexo git@github.com:admin/admin.github.io.git
```

这时会发现clone下来的文件中theme/next为空，因为这时一个子模块，还需要执行：

``` bash
$ cd admin.github.io
$ git submodule init
$ git submodule update
```

此时Hexo文件就完整了，可以正常修改,修改好后按上述方法提交。

此时你就有了一个稳定的博客系统，除非GitHub崩溃，否则你的博客不用担心任何数据丢失和迁移问题了。


## 本地测试
``` dash
$ hexo s #启动本地服务器测试
```
之后在浏览器中输入http://localhost:4000 ，即可在本地看到当前架设的网站页面

## 部署
在站点的_config.yml文件中加入以下字段:
``` yml
deploy:
  type: git
  repository: https://github.com/admin/admin.github.io.git
  branch: master
  plugins: -hexo-generator-feed
```
站点文件下输入命令:
``` bash
$ npm install hexo-deployer-git --save
$ hexo clean
$ hexo g
$ hexo d
```
完成部署
在浏览器中输入 **admin.github.io** 即可看到页面
