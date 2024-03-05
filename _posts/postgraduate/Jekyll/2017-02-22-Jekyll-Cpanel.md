---
layout: post
title: "基于Jekyll静态框架的Github站点设计"
data: 2017-02-22 19:27:00 +0800
categories: 研究生涯
tag: Jekyll
---
* content
{:toc}


这里主要是讲述这个博客搭建过程中的心酸历程，主要涉及的是`Jekyll`、`Github`、`Cpanel`等。也有搭建站点的一些内容。仅作后来人以及本人回顾。错误之处，请留言指正。

<!-- more -->

## Jekyll相关

Jekyll可以将纯文本转化为静态网站。[`Github page`](http://pages.github.com/) 可以直接运行`Jekyll`是它极大的特点，我也是因为这个才关注到这个开源模板的。

有关Jekyll的具体内容参考：

+ 中文官方主页： [http://jekyll.com.cn/](http://jekyll.com.cn/)
+ 英文官方主页： [http://jekyllrb.com/](http://jekyllrb.com/)

因为`Jekyll`是开源项目，所有也可以参考其`Github`主页：

+ `Github主页` : [http://github.com/jekyll/jekyll](http://github.com/jekyll/jekyll)

介绍和使用放在`jekyll/doc/_doc`中。

> 因为自己也还在摸索和探索过程，所以此处不会详细探讨细节。可能在未来的某天会以另外一篇博文的形式放在博客上。

此处主要说明以下`Jekyll`的安装过程。

### 安装Ruby 和 RubyGems

[`Ruby`](http://www.ruby-lang.org/zh_cn/downloads/)是一门语言，不用多解释。而[`RubyGems`](http://rubygems.org/pages/download)其实就是`Ruby`的包管理工具。相当`pip`之于`Python`，`npm`之于`Node`中。

因为`Jekyll`是用`Ruby`语言编写的，所以我们需要安装他们。

> 您最好将它加入**系统环境变量**，这样就可以使用**命令行**玩耍了。当然了，`Windows`自带的命令行实在是渣，当然我们有替代工具--[`cmder`](http://cmder.net/)。[官网](http://cmder.net/)你就算用了吃奶的劲也保不定能下下来。这个时候，不要放弃，你可以考虑[`softpedia.com`](http://www.softpedia.com/get/Programming/Other-Programming-Files/Cmder.shtml)或者[`CSDN`](http://download.csdn.net/tag/Cmder)或者向<a href="mailto:lowols@outlook.com">我</a>索要一份。

### 安装Jekyll

打开终端，输入以下命令：

```ruby
$ gem install jekyll
```

这就就ok了。

### 选择Jekyll模板

在终端输入：

```jekyll
$ jekyll server
```
或者

```jekyll
$ jekyll s
```
打开浏览器，输入`http://127.0.0.1:4000/`，就可以看到`Jekyll`为你生成的页面。真的，非常简陋。

这个时候，你有以下的方法改善：

+ 从头到尾自己设计
+ 选择一个模板

此处我选择较为简单的方式，选择一个**模板**。那么你可能会问，模板从哪里来。因为`Jekyll`开源，所以我们很容易获得很多资源。

+ 当然专门收集模板的网站：[http://jekyllthemes.org/](http://jekyllthemes.org/)
+ 在`Google`上搜索`Jekyll`和`Github`，在人家的仓库中`download`别人的模板(要注意人家是否允许拷贝哦)。

选择了模板后要根据自己的需求更改，而且主要是在`_config.yml`文件中。此处不予详细讨论，以后另有博文详细说明。

**这里，我假设你已经根据自己的要求搭建好了自己的`Jekyll`博客**。

---------------

## Github仓库

将仓库命名为`yourname.github.io`，然后通过`Git`命令将文件整个博客文件上传至`Github`。

这样，在你的浏览器中输入：`http://yourname.github.io`就可以访问你的这个静态博客了。

> ❓其实我上传后`Github pages`显示出来的界面跟你使用`$ jekyll server`生成的界面是一样的，但是有些功能无法实现，比如没办法适应`Bootstrap`的页面变化之内的。后来考虑了是不是只上传`Jekyll build`的`_site`文件夹就可以，因为`Github`可以自动解析`Jekyll`。朋友们可以自己试一下，然后给我反馈。

-----------------

## 搭建自己的站点

有以下几个步骤：

### 购买域名

可以在很多地方购买到域名，这里你们可以自己鉴别。

我是在[`GoDaddy`](http://hk.godaddy.com/en/)上购买的，开始本来想用优惠码，可是选择优惠码之后没有**支付宝**付款的选项。又想着法子说用`Visa`卡来付款，可以提交订单也不能成功，后来咨询客服，说是要绑定`Visa`卡，我当然是拒绝的。所以就没有使用优惠码购买域名。

> 我买了**两年**的`.com`域名，花了`￥108`，大家可以参考以下。

### 购买虚拟主机

购买虚拟主机也有很多渠道，不过我还是选择了[`GoDaddy`](http://hk.godaddy.com/en/)😅，说实话，有点后悔，国外的网速太渣了，简直没办法。

你可以在我的产品中找到你购买的域名和服务器，并进行管理。

### Cpanel

现在该到了`Cpanel`，它是一个**虚拟主机管理系统**，使用开始会让你重新注册账号并关联域名，因为都是在`GoDaddy`买的，所以会自动关联。

进入到`Cpanel`控制界面，选择**文件管理器**，选择`Web Root(public_html/www)`，就进入**文件管理器**。

![进入文件管理器]({{ '/styles/images/jekyll/2017-02-23.png' | prepend: site.baseurl }})

然后关键的步骤是：

+ 在终端使用命令`$jekyll build`命令生成`_site`文件夹。
+ 将`_site`文件夹里的文件打包成`.zip`(不要打包成`.rar`格式)。

![_site文件夹中的文件]({{ '/styles/images/jekyll/2017-02-23-01.png' | prepend: site.baseurl }})

+ 3.上传到`Cpanel`文件管理器的`public_html`目录下，解压所有文件。

![解压你的文件]({{ '/styles/images/jekyll/2017-02-23-02.png' | prepend: site.baseurl }})

打开你的网站，你就可以享受你的博客了。

**那么，问题来了！**

#### 为什么上传到`public_html`目录下

因为`public_html`是`Cpanel`默认的首页目录，现实页面时，服务器会自动在该目录下搜寻`index.html`文件。

#### 为什么只需要上传`_site`文件夹内的文件

因为`_site`文件中的文件才是`Jekyll`生成的可执行的网页文件。

#### 有没有什么办法不需要每次都这样打包上传

当然是有的。请使用`ftp`功能。

+ 1. 首先你需要一个`ftp`客户端。这里的建议是[`FileZilla`](http://filezilla-project.org/)，真的非常好用。请下载一个`FileZilla Client`版本的。
+ 2. 在`Cpanel`主页点击进入`FTP`账户，按照实际情况配置。最为主要的是将`FTP`目录设置为`/home/xxxx/public_html`。

![FTP设置]({{ '/styles/images/jekyll/2017-02-23-03.png' | prepend: site.baseurl }})

+ 3. 打开`FileZilla`，选择**文件**`->`**站点管理器**`->`**新建站点**。

![FileZilla]({{ '/styles/images/jekyll/2017-02-23-04.png' | prepend: site.baseurl }})

其中，**主机**是你的**域名**或**IP**。**端口**选择**21**。**用户**就是你填写的`xxx@yourwebsite.com`。

> 注意：如果你使用了中文字符的话，可能会出现乱码。这个时候，往往是因为**字符集**出错，在设置中可以更改相应的字符集。算是个小技巧。

+ 4. 将`_site`文件夹的内容通过`FTP`上传到虚拟主机。

-------------------------------

## 将自己的站点与Github相连

在你的文档中增加一个`CNAME`命名没有后缀的文件。比如我的网站。

![CNAME设置]({{ '/styles/images/jekyll/2017-02-23-05.png' | prepend: site.baseurl }})

然后，将此文件上传到你的仓库中，即可。

> 至于网上很多教程说要利用[`DnsPod`](http://www.dnspod.cn/)进行`DNS`解析。我折腾了很久，按照别人提供的方法，会导致手机端无法正确定向。所以，我就使用原有的解析，**发现也能正确定向**。

如果大家有更改`DNS`需求，可以在你`GoDaddy`网站的**域名管理**中更改`DNS`服务器。

![DNS服务器]({{ '/styles/images/jekyll/2017-02-23-06.png' | prepend: site.baseurl }})

------------------------------------

## 可供参考的网址

+ 在`Hosting`上运行`Jekyll` [http://blog.timowens.io/running-a-jekyll-site-on-reclaim-hosting/](http://blog.timowens.io/running-a-jekyll-site-on-reclaim-hosting/)
+ 我的博客是如何搭建的(`github pages` + `HEXO` + 域名绑定) [http://www.jianshu.com/p/834d7cc0668d](http://www.jianshu.com/p/834d7cc0668d)
+ 搭建一个免费的，无限流量的`Blog`--`github Pages`和`Jekyll`入门[http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)

----
转载请署名，请勿非法转载。