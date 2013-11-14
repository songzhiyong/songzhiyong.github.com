---
layout: post
title: "所谓的QQ空间“魔力”图片"
date: 2013-11-05 22:35
comments: true
categories: PHP
tags: [Web,PHP] 
---
最近，看到不少朋友在qq空间上转载一些奇怪的图片，上面显示我的昵称和头像，出于好奇，当然也纯属无聊，就自己试试看。
<!--more-->
首先，使用chrome审查元素发现这张图片来源于一个PHP文件：<br>
``` html Html代码 
    <img src="http://qq.sennvwu.com/qzone/do.php"
    onload="QZFL.media.reduceImage(0,400,300,{trueSrc:'http:\/\/qq.sennvwu.com\/qzone\/do.php',callback:function(img,type,ew,eh,o){var _h = Math.floor(o.oh/o.k),_w = Math.floor(o.ow/o.k);if(_w<=ew && _h>=eh){var p=img.parentNode;p.style.width=_w+'px';p.style.height=_h+'px';}}})"
    width="400">
```
而且这个文件不是QQ空间自带的，这就把问题搞的简单了，肯定是php服务器动态生成图片，大致有了
头绪就顺藤摸瓜吧,但很奇怪，基本这种日志源头都找不到，就算找到也不是那种效果，或者根本没有图片，仅在“个人中心”看到别人转载的图片才能看到那个效果。这是为什么呢？由于对PHP了解甚少，只能再往上搜索一些资料:
<br>
<strong>PHP中使用$_SERVER['HTTP_REFERER'] 获取前一页面的 URL 地址</strong>
<br>
这就对了，个人中心的url地址格式都为：
```html URL地址
    http://user.qzone.qq.com/QQ/infocenter
```
号码自然很简单就能冲HTTP_REFERER中拿到了。
那么后台是如何取到QQ头像、昵称等信息的呢？
我Google到了一个<strong>腾讯的WebService接口：</strong>
```html 腾讯的WebService接口
http://base.qzone.qq.com/fcg-bin/cgi_get_portrait.fcg?uins=QQ
```
不需要任何凭证信息即可获取uins指定的QQ号码的头像、昵称信息，返回的格式为JSON,不过编码格式为GBK，这个后面可能会再提到。另外上面的图片还有一个显示地理位置和，这个就比较常见了，我用的就是下面这个。
```html 根据IP获取位置信息
http://ip.taobao.com/service/getIpInfo.php?ip=127.0.0.1
```
格式同样也是JSON，这里就是UTF-8编码了。

然后，往事具备，只欠东风，要着手写php代码并配置服务器了，代码就直接加完注释
