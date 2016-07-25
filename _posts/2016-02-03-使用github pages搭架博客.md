---
layout:     post
title:      "使用github pages搭建博客"
subtitle:   ""
date:       2015-02-03 11:00:00
author:     "steven"
catalog:    true
tags:
    - blog
---

做为一个程序员，有时候需要记录一些解决问题的过程或一些想法，或者是新东西的学习，博客是个很好的解决方案。又可以分享帮助他人。但是又想做的不得不太一样，考察了下，最后选择了使用github pages来搭架。

github pages的特点：


1. 免费，可靠

2. 如果你经常使用github，可以提供很多便利

3. 很好的支持Markdown，可以使用大量的主题

4. 灵活的配置



搭架过程：

1.申请一个github账号
-------
   此过程比较简单，不再赘述。账号申请完成之后就可使用github pages了

2.在github中创建一个Repository
-------
   此步也比较简单

3.将创建好的Repository克隆到本地
------

   ```javascript
git clone xxxxxxxxxxx.git(创建的Repository的地址)
   ```
4.创建gh-pages分支
------
github pages规定，只有该分支的的页面，才会生成网页文件


   ```javascript
git checkout --orphan gh-pages
   ```
  以后的所有操作都在该分支下完成，请勿切换分支。

5.安装Jekyll
------
Jekyll是一个静态站点生成工具，它可以根据源码生成静态HTML页面。jekyll提供了模板，配置，变量，插件等功能。

  安装jekyll,首先要安装ruby。安装ruby请参考官网https://www.ruby-lang.org/en/downloads/
  并确保ruby的版本在2.0.0之上。
  查看ruby版本：

   ```javascript
ruby --version
   ```

   安装Bundler，Bundler是一个ruby项目管理工具，可以安装编译Ruby项目依赖的第三方工具或类库

  ```javascript
gem install bundler
   ```

   使用bundler安装Jekyll，要安装Jekyll首先要在项目文件夹中添加一个配置文件（Gemfile），用来声明用到的插件。文本内容：

  ```javascript
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins
  ```

   然后安装Jekyll

  ```javascript
bundle install
  ```

6.配置项目
--------

Jekyll安装完成之后，项目中会自动生成一些文件和文件夹。主要的有

  ```javascript
|--- _config.yml 项目配置文件
|--- _layouts    通用布局文件，包含各个页面的默认布局
|--- _includes   模板文件夹，包含默认的页面头部，底部和导航栏等的样式
|--- _posts      用来存放博客文章的文件夹
  ```

  _config.yml，用来配置站点的名称，url,作者信息，页面变量，markdown解析等，不同主题的配置不一样。以选择的主题配置为准。
  
* 基本配置
  
  baseurl: 博客的路径，可以配置为申请的域名
  markdown配置，例如：

  ```javascript
highlighter: rouge
markdown: redcarpet
kramdown:
nput: GFM
  ```
  其中redcarpet是一个markdown文件解析引擎，它可以把_posts下的.md文件解析为静态HTML页面。具体的安装和配置参考：
   https://jekyllrb.com/docs/configuration/
   
* 添加文章：
 
  将写好的文章（Markdown文件或HTML文件）放到 _posts文件下，github pages会自动解析，添加到页面中。文章名名称的格式是固定。按照”yyyy-MM-dd-文章名称“的格式命名，Jekyll会自动按照日期排序添加文章。

* 头部定义：
 
  按照Jekyll模板的定义，每个模板都要在头部定义一些变量，例如：

  ```javascript
---
layout:     post   # 指定使用的模板文件，“_layout” 目录下的模板文件名决定变量名
title:      title  # 文章的标题
date:       date   # 覆盖文章名中的日期
category:   blog   # 文章的类别
description: description
published:  true   # default true 设置 “false” 后，文章不会显示
permalink:  /:categories/:year/:month/:day/:title.html  # 覆盖全局变量设定的文章发布格式
---
  ```

 格式以 ---开始和结束，不同的主题可以配置不同的变量

* 变量

 在文章中可以使用声明的变量，官方说明：http://jekyllrb.com/docs/variables/
 
 
 使用格式，如使用标题：

  ```javascript
{{ page.title }} //{\\{ page.title }\\},其中的\\为转义符，使用时请去除
  ```

7.预览
-----

项目配置完成之后，可以在本地启动服务器进行预览。
编译并运行项目：

   ```javascript
bundle exec jekyll serve
   ```
运行该命令之后，Jekyll会编译整个项目，在项目文件中生成_site文件夹，里面就是根据配置生成的站点。
在浏览器器中打开 http://localhost:4000 就可以看到站点和生成的文章。


8.使用主题
-----

Jekyll支持模板，所以可以配置页面的主题，github上有大量的Jekyll主题。
这里推荐一个Jekyll主题网站：
http://jekyllthemes.org/

里面有海量的Jekyll主题，都是在github上开源的，可以直接使用，也可以自己再次修改自定义。
每个主题基本都有主题的配置方法说明，比较简单和方便。


9.更新配置和文章到github pages
-----

*   使用bundler

   ```javascript
bundle update github-pages
   ```
  或者
  
  ```javascript
bundle update
  ```
*  使用git

  ```javascript
git add . //添加所有文件到版本索引
git push origin github-pages //同步文件到远程仓库
  ```
  同步完成之后稍等一会，在浏览器中打开https://(github用户名).github.io/(Repository名称)就可看到刚刚搭建的博客了
