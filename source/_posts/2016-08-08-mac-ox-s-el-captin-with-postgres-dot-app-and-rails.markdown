---
layout: post
title: "在Mac OS X El Caption上用Postgres.app来搭建Rails 5.0.0开发环境"
date: 2016-08-08 17:41:14 +0800
comments: true
categories: 
---

最近研究一篇整合Angular、Bootstrap、Rails和Postgres的教程。里面推荐采用Postgres来做关系型数据库。网上有很多做法，但本着天真文科生的本性，决定用最简单的方式：一路图形界面的工具。

<!-- more -->

先是习惯性地上网一搜，结果发现各种对Postgres.app的吐槽。症状是Rails的应用不能连上数据库，吐槽的点很多。归结下来主要是两个问题：

1）一是Postgress.app采用的应该是MacOS X图形界面应用的部署规则，config文件不再出现在/etc的常规路径下面，gem安装时无法在默认的地方找到配置文件，导致安装不能成功。

2）二是装完连不上，用户名密码等问题。

以下是笔者的安装过程，基本还是比较顺利。

首先是找到Postgres.app的官方网站：[http://postgresapp.com](http://postgresapp.com/, "http://postgresapp.com")

<p class="no-indent">{% img /images/20160808/postgres_app_homepage.png %}</p>下载这个zip文件。解压后可以看到一个上图大象的文件。双击安装。按提示来这个基本应该没有问题。

中间的插曲是，Mac OS X会默认禁止没有做过验证的app运行。在System Preference里的Security & Privacy下修改这个设置，让这个程序能够运行。记得先用登录密码打开左下方的锁。然后选择Anywhere。

<p class="no-indent">{% img /images/20160808/security_and_privacy.png %}</p>
双击后会提示你将这个文件转到你的Application所在。没事，转吧。

然后，你应该能看到有个大象的标志在系统栏里，右上方，和我的Evernote相得益彰啊~
<p class="no-indent">{% img /images/20160808/postgres_on_the_task_bar.png %}</p>

右键下，看看数据放在哪里了…

<p class="no-indent">{% img /images/20160808/right_click_postgres.png %}</p>
接着装个图形界面的管理工具。连一下看看是不是可以用了。

https://eggerapps.at/postico/

Postico的作者“碰巧”也是Postgres核心开发组的成员。但不要直接去Apps Store里下哟，那可是实打实收费CNY300+，可以先在官网下一个evaluate版本，虽然有些限制，但先试用下。真觉得好就买吧。

<p class="no-indent">{% img /images/20160808/postico_homepage.png %}</p>
解压下载的zip文件，拷贝到Applications文件夹下，双击打开。

<p class="no-indent">{% img /images/20160808/postico_welcome_page.png %}</p>
按Connect。第一次会要求你填写一些信息。

如果作为开发机而已，没有其他的特别需要，只要填一个Nickname就可以了。其他留默认的即可。

Host就是localhost，User就是你安装时用的用户名，Datebase是和用户名同名的。我就全留白了。
<p class="no-indent">{% img /images/20160808/postico_connect_info.png %}</p>
连上数据库就是这样的界面。
<p class="no-indent">{% img /images/20160808/postico_connected_db.png %}</p>
再来张有表格的样子。
<p class="no-indent">{% img /images/20160808/postico_database_with_tables.png %}</p>
好了接着就是怎么配置和Rails的整合了。

在Postgres.app官网的文档里，有关于ruby的配置说明。首先需要安装对Command Line Tools（命令行工具）的支持。

[命令行工具安装](http://postgresapp.com/documentation/cli-tools.html, "命令行工具安装")
<p class="no-indent">{% img /images/20160808/postico_config_command_line_tools.png %}</p>
注意，这里做完了要source一下，或者在直接运行一下export。Mac OS X一般用的都是Bash，没必要设置Fish的。

用RVM做一个单独的gemset。

{% codeblock terminal lang:bash %}
# 假设已经安装了ruby 2.3.0，否则先运行
# rvm get stable
# rvm install 2.3
# rvm use 2.3
> rvm gemset create receta
> rvm use 2.3@receta
> gem install rails
{% endcodeblock %}

Ruby的Postgres适配器的安装（http://postgresapp.com/documentation/configuration-ruby.html）
<p class="no-indent">{% img /images/20160808/postico_config_ruby_gem_pg.png %}</p>
于是手动安装pg。

{% codeblock terminal lang:bash %}
# 先看一眼是不是在对的gemset下。
> rvm gemset list
# 如果正确。
> ARCHFLAGS="-arch x86_64" gem install pg
{% endcodeblock %}

现在可以新建一个rails测一下了。

{% codeblock terminal lang:bash %}
rails new receta -T -B --database=postgressql
{% endcodeblock %}

改一下gem的源，或者配置个vpn。笔者是直接用的vpn。

{% codeblock terminal lang:bash %}
> bundle install
{% endcodeblock %}

用Sublime Text 3打开这个项目，找到config/database.yml。我的机器上只要增加：host: localhost 这一行。

{% codeblock ~/config/database.yml lang:ruby %}
# ~/config/database.yml

default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: receta_development
  host: localhost
{% endcodeblock %}

然后，用rake db来验证下。
{% codeblock terminal lang:bash %}
> rake db:create
{% endcodeblock %}

再到Postico里看下，耶，出来了~
<p class="no-indent">{% img /images/20160808/postico_with_rails_db.png %}</p>
最后跑下rails s，看看欢迎界面出来了吗？
