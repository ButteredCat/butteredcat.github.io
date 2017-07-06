---
layout: post
title: 利用图书馆远程登录服务访问知网
excerpt: "通过 VPN 访问图书馆购买的各种数据库资源"
modified: 2017-07-06
categories: articles
tags: []
comments: true
share: true
---

包括[上海图书馆](http://www.library.sh.cn/)在内的国内各大图书馆都提供远程登录服务。只要办理了读者证，不光可以入馆阅读或者外借图书，还可以通过 SSL VPN 远程登录，访问图书馆购买的各种数据库。

上海图书馆的读者证分三类，三类都可以远程访问图书馆资源。

| 读者证功能名称 | 押金    |
| :------ | :---- |
| 参考阅览功能  | 无     |
| 普通外借功能  | 100元  |
| 参考外借功能  | 1000元 |

上海图书馆远程登录的方便之处在于，其提供了浏览器和客户端两种访问方式，并且支持多平台的访问。

浏览器方式很简单。先用读者证号在图书馆主页登录（初始密码是办理读者证使用的证件号），然后点击右上角“我的图书馆”，打开左边栏最下方的“e卡通”，点击[链接](http://search1.library.sh.cn/mylibrary/application/eservice)进入使用。这种方式需要使用 Windows XP 以上操作系统及 IE8 以上版本的浏览器。

而客户端访问支持 macOS 和 Linux，适用范围更广泛。[下载页面](http://support.arraynetworks.com.cn/troubleshooting/index.html)提供了 MotionPro 和 Standalone 两种客户端，都可以使用，并且分别有 Windows、macOS 和 Linux 版本。macOS 建议安装 MotionPro，因为最新版的 macOS 10.12.3 已经无法安装 Standalone 独立客户端了。安装完成后，依照下表新建一个配置，就可以连接图书馆内网了。

| 项目                 | 填写内容                       |
| ------------------ | -------------------------- |
| Profile Name       | 配置文件名称，可以随意填写，比如ShLib      |
| SPX Host or IP     | 必须填写ra.digilib.sh.cn       |
| SPX Host User Name | 输入8位数字读者证卡号                |
| SPX Host Password  | 输入登录“我的图书馆”所需密码，默认为18位身份证号 |

可以访问的数据库列表都在[这里](http://db.idoc.sh.cn/Navigator.aspx?&_005=005002&_011=011001)了。有些后面打了红叉的数据库是只限馆内访问的，好在知网、维普以及人大复印报刊资料都可用。
