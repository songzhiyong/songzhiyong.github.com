---
layout: post
title: "所谓的QQ空间“魔力”图片"
date: 2013-11-05 22:35
comments: true
categories: php
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

然后，往事具备，只欠东风，要着手写php代码并配置服务器了，代码就直接加贴上了

```

<?php
//禁用错误报告
error_reporting ( 0 );
////报告运行时错误
//error_reporting(E_ERROR | E_WARNING | E_PARSE);
//
////报告所有错误
//error_reporting(E_ALL);
ob_start ();
header ( 'Content-Type: image/png' );

define ( 'IMG_NO', "fall_bg.png" ); #刚开始显示的提示信息
define ( 'IMG_BACKGROUND', "fall.png" );
define ( 'IMG_WIDTH', 1200 );
define ( 'IMG_HEIGHT', 300 );
define ( 'FONT_NAME', "YaHei.Consolas.1.11b.ttf" ); #字体文件名
define ( 'CACHE_PATH', rtrim ( realpath ( dirname ( __FILE__ ) ), '/' ) . '/cache/' ); #缓存目录
define ( 'CACHE_EXPIRE', 60 * 60 ); #缓存时间，单位秒


#(!is_dir(CACHE_PATH) && is_writable(CACHE_PATH)) || die;


/*
    $remote: 远程URL
    $local: 本地缓存路径
    $expire: 过期时间。为-1时，永久不更新缓存
*/
function load_from_cache($remote, $local, $expire = CACHE_EXPIRE, $as_path = false) {
    #过滤潜在的危险字符
    $local = preg_replace ( "/[.\/\\\?\*\'\"\|\:\<\>]/", "_", $local );
    $cache = CACHE_PATH . $local;
    #查找缓存
    if (file_exists ( $cache ) && ($expire = - 1 || filemtime ( $cache ) - time () < $expire))
        return $as_path ? $cache : file_get_contents ( $cache );
    
        #文件不存在或缓存过期，重新下载
    $content = file_get_contents ( $remote );
    file_put_contents ( $cache, $content );
    return $as_path ? $cache : $content;
}

/*
    返回客户端信息。
*/
function client_info() {
    $url = "http://ip.taobao.com/service/getIpInfo.php?ip=";
    $ip = getIp ();
    $info = explode ( '"', load_from_cache ( $url . $ip, $ip, - 1 ) );
    $string = $info [31];
    return json_decode ( '"' . $string . '"' );
}
//获取ip
function getIp() {
    if (getenv ( "HTTP_CLIENT_IP" ) && strcasecmp ( getenv ( "HTTP_CLIENT_IP" ), "unknown" )) {
        $ip = getenv ( "HTTP_CLIENT_IP" );
    } else if (getenv ( "HTTP_X_FORWARDED_FOR" ) && strcasecmp ( getenv ( "HTTP_X_FORWARDED_FOR" ), "unknown" )) {
        $ip = getenv ( "HTTP_X_FORWARDED_FOR" );
    } else if (getenv ( "REMOTE_ADDR" ) && strcasecmp ( getenv ( "REMOTE_ADDR" ), "unknown" )) {
        $ip = getenv ( "REMOTE_ADDR" );
    } else if (isset ( $_SERVER ['REMOTE_ADDR'] ) && $_SERVER ['REMOTE_ADDR'] && strcasecmp ( $_SERVER ['REMOTE_ADDR'], "unknown" )) {
        $ip = $_SERVER ['REMOTE_ADDR'];
    } else {
        $ip = "unknown";
    }
    //bae 的ip要特殊处理
    if (strpos ( $ip, ',' )) {
        $ipArr = explode ( ',', $ip );
        $ip = $ipArr [0];
    }
    return ($ip);
}

function getWishMess() {
    $time = date ( "H" );
    $mess = "你好~";
    if (($time >= 0) && ($time < 4)) {
        $mess = "夜深了，该睡了";
    } else if (($time >= 4) && ($time < 7)) {
        $mess = "起这么早？早安";
    } else if (($time >= 7) && ($time < 12)) {
        $mess = "上午好";
    } else if (($time >= 12) && ($time < 13)) {
        $mess = "吃午饭咯~";
    } else if (($time >= 13) && ($time < 17)) {
        $mess = "该午休了~";
    } else if (($time >= 17) && ($time < 18)) {
        $mess = "下班回家~";
    } else if (($time >= 18) && ($time < 22)) {
        $mess = "晚上好";
    } else if (($time >= 22) && ($time <= 23)) {
        $mess = "晚安，好梦~" ;
    }
    return ($mess);
}
$referer = $_SERVER ['HTTP_REFERER'];
#$referer = "http://user.qzone.qq.com/123456789/infocenter";

$pattern = "/http:\/\/user.qzone.qq.com\/(\d+)\/infocenter/";
if (preg_match ( $pattern, $referer, $matches )) {
    #获取QQ号码
    $uin = $matches [1];
    $info = explode ( '"', load_from_cache ( "http://base.qzone.qq.com/fcg-bin/cgi_get_portrait.fcg?uins=" . $uin, $uin ) );
    $avatar = $info [3];
    $nickname = $info [5];
    $client = client_info ();
    $client=iconv("utf-8","gbk",$client);
    $copyright = "Just for fun, by Jerome.";
    $weibo = "http://weibo.com/zhiyongs";
    
    #重点来了，生成图片
    try {
        $im = imagecreatefrompng ( IMG_BACKGROUND );
        
        #绘制头像
        $avatar_file = load_from_cache ( $avatar, $uin . ".jpg", 60 * 60 * 24, true );
        $im_avatar = imagecreatefromjpeg ( $avatar_file );
        imagecopymerge ( $im, $im_avatar, 14, 14, 0, 0, 100, 100, 100 );
        imagedestroy ( $im_avatar );
        
        #绘制文字
        $blue = imagecolorallocate ( $im, 0, 0x33, 0xFF );
        $white = imagecolorallocate ( $im, 0xFF, 0xFF, 0xFF );
        $texts = array (
            array (14, 140, 30, $blue, "To:" . $uin ),
            array (16, 170, 60, $blue, $nickname . "," . getWishMess () ),
            array (12, 340, 80, $blue, $client ), 
            array (10, 320, 100, $blue, $copyright ), 
            array (8, 320, 115, $blue, $weibo ) );
        
        foreach ( $texts as $key => $value ) {
            imagettftext ( $im, $value [0], 0, $value [1], $value [2], $value [3], FONT_NAME, mb_convert_encoding ( $value [4], "html-entities", "gbk" ) ); #解决乱码问题
        }
        
        imagepng ( $im );
        imagedestroy ( $im );
        
        header ( "Content-Length: " . ob_get_length () );
        ob_end_flush ();
    } catch ( Exception $e ) {
        
        #die($e->getMessage());
        $error = true;
    }
} else {
    $error = true;
}

if ($error) {
    header ( 'Content-Length: ' . filesize ( IMG_NO ) );
    echo file_get_contents ( IMG_NO );
}

```

一直困扰我的最后一个问题就是，刚发布是有效的，可一段时间后，腾讯就把你的图片缓存到它的服务器了，然后就不动态生成图片，魔力图片也随之失效。