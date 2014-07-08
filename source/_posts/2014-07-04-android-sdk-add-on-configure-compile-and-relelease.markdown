---
layout: post
title: "Android SDK Add-on Configure, Compile and Relelease"
date: 2014-07-04 10:07
comments: true
categories: android
---
转自：[http://www.poemcode.net/2010/09/android-sdk-addon/](http://www.poemcode.net/2010/09/android-sdk-addon/)


SDK Add-on 是一个比较小众的话题，一是通常厂商不会公开自己的add-on，二是即便公开了，应用开发者也很少使用。所以通常是厂商自己的技术团队自娱自乐，框架团队抽取公用的控件，制成一个 add-on，然后公布给自家的应用程序开发团队。

由于是小众的，所以网络中关于这方面的资料比较少。这几天由于工作原因，我有幸接触到 Add-on，因此把这方面的知识进行了汇总，整理成如下内容。这些知识来自于我对 sample 示例的理解，并经过项目实践验证，我竭力保证无误，但差错之处也可能存在，如发现，请指正。

以下内容将分成若个步骤：

1.  添加模块；
2.  添加资源文件；
3.  加入编译脚本；
4.  发布；

在阅读下面内容前，先假设是对 $ANDROID_HOME/device/sample 进行修改。

#### 一、添加模块（Add Module）

在自己 _product-name_ 目录下建立 frameworks 文件夹，从 sample 中拷贝 Android.mk 文件到 frameworks。

依照 sample 的方式，在 frameworks 路径下，建立 PlatformLibrary 文件夹，再次从 sample/frameworks 中拷贝 Android.mk 文件，并将其中的 LOCAL_MODULE 和 LOCAL_DROIDDOC_OPTIONS 修改为自己中意的名字。–这里值得注意的是，LOCAL_MODULE 变量被定义两次，第一次是 library，第二次是 document。

依照 sample/frameworks/com.example.android.platform_library.xml 的形式创建自己的 XML 文件，文件名称、XML 内容中 name 和 file 保持和 LOCAL_MODULE(library) 一致。

完成上述操作之后，建立 java 文件夹，并将自己创建的源文件（java）放入到其中，这里不再赘述。

#### 二、添加 Overlay(Add Resoruce)

代码中可能需要访问图片、字符串等资源，但是这些在原有的 Android 中没有，因此需要想办法自行加入，Android 对此提供了两种方式。

第一种就是直接在 $ANDROID_HOME/framework/base/core/res/res 下进行操作，添加文件，追加字符串。这种方式虽然简单，但是破坏了原有的“纯洁性”，因此不予推荐，在第三节修改编译脚本中我采用了第二种方式。

第二种方式就是使用 overlay。在 product-name 目录下建立 overlay 文件夹，依照 framework 的结构建立各级目录，例如想添加图片，可以先依次建立 base/core/res/res/drawable，然后将图片置于其中，同样可以建立 layout 等文件夹。

如果要添加字符串或者 style 资源，就要稍微复杂一点，需要使用 add-resource 来让系统“明白”新添加的内容。

```XML
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
    <add-resource type="string" name="numwheel_year" />
    <!-- Strings for number wheel -->
    <string name="numwheel_year">年</string>
</resources>

```

如果没有 add-resource，则会遇到这样的信息：

完了以后，就可以用使用 `com.android.internal.R` 来访问这些资源了。

#### 三、加入编译脚本（Add build script）

在自己 product-name 目录下建立 sdk_addon 文件夹，并依照 sample/sdk_addon 分别拷贝 manifest.xml 和 hardware.xml，注意 manifest.xml 中的内容必须要根据之前第一步的变更同步修改。

接下来最关键，也是最容易出错的环节到了，那就是修改自己 product 的 mk 文件，需要修改 PRODUCT_PACKAGES、PRODUCT_COPY_FILES、DEVICE_PACKAGE_OVERLAYS 的值，定义 PRODUCT_SDK_ADDON_NAME、PRODUCT_SDK_ADDON_COPY_FILES、PRODUCT_SDK_ADDON_COPY_MODULES和PRODUCT_SDK_ADDON_DOC_MODULE。修改和定义的区别就在于，前者使用”+=”，后者使用“:=”。这些变量的值可以参照sample示例。

由于 sample 的示例中没有出现 DEVICE_PACKAGE_OVERLAYS，所以我这里特别解释一下它的作用，除了刚才为 add-on 增加资源以外，它还起到的作用就是替换掉原来的framework中的资源，比如图片等等，惟一的要求就是必须建立和原来图片位置相同的路径。和 add-on 不同的是，如果你是替换原来资源，那么是不用 add-resource 的。DEVICE_PACKAGE_OVERLAYS 的值就是 overlay 文件夹的路径，不包含 $ANDROID_HOME，到 overlay 一级即可。

完成了修改以后，使用 make PRODUCT-product-name-sdk_addon 进行编译。当然也可以直接执行 make sdk_addon，但是需要注意在此之前，你需要使用 chooseproduct 指定 product-name。执行完毕以后，会得到一个 zip 文件，这个就是 add-on 了。

#### 四、发布（Relase）

完成上面的环节以后，剩下临门一脚了。所谓发布，就是通过诸如 HTTPD 一类的 HTTP SERVER 将资源公布出去。从 http://httpd.apache.org 下载 httpd，如何安装以及修改监听端口可参看其 document，在此不予赘述。

现在把刚才生成的 zip 文件，放置到 htdoc 目录下。并且创建一个 repository.xml，其内容和形式参看 
[https://dl-ssl.google.com/android/repository/repository.xml](https://dl-ssl.google.com/android/repository/repository.xml) ，我把它删减一下得到以下内容：

```XML
<sdk:sdk-repository
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:sdk="http://schemas.android.com/sdk/android/repository/2">
 
    <sdk:add-on>
        <sdk:name>Google APIs</sdk:name>
        <sdk:api-level>3</sdk:api-level>
        <sdk:vendor>Google Inc.</sdk:vendor>
        <sdk:revision>03</sdk:revision>
        <sdk:description>Android + Google APIs</sdk:description>
        <sdk:desc-url>http://developer.android.com/</sdk:desc-url>
        <sdk:uses-license ref="android-sdk-license" />
        <sdk:archives>
            <sdk:archive os="any">
                <sdk:size>34908058</sdk:size>
                <sdk:checksum type="sha1">1f92abf3a76be66ae8032257fc7620acbd2b2e3a</sdk:checksum>
                <sdk:url>google_apis-3-r03.zip</sdk:url>
            </sdk:archive>
        </sdk:archives>
        <sdk:libs>
            <sdk:lib>
                <sdk:name>com.google.android.maps</sdk:name>
                <sdk:description>API for Google Maps.</sdk:description>
            </sdk:lib>
        </sdk:libs>
    </sdk:add-on>
 
    <sdk:license type="text" id="android-sdk-license">
        This is the Android Software Development Kit License Agreement.
    </sdk:license>
</sdk:sdk-repository>

```


接下来的事情，就是替换掉里面的 Google 信息为自己，大部分可以从它们的名称中理解其含义，例如 size 是指文件大小，你可以使用ls –l来查看文件大小。checksum 则要麻烦一些，执行 sha1sum ZIP.zip，即使用 sha1 对刚才生成 zip 文件生成校验码。

修改完成以后，通过浏览器访问 `http://IP-address:port/repository.xml`，确认是否 OK。确认以后，打开 Android SDK and AVD Manager，点击左侧 Available Packages，选择 “Add Add-on Sites” 填入URL，接下来的安装就不费口舌了。完成以后，你可以在 sdk 下的 add-ons 目录下看到刚刚安装的 add-on。

捎带提一下，应用开发方面使用该 add-on 时，调整工程的 Project Build Target 设定即可。
