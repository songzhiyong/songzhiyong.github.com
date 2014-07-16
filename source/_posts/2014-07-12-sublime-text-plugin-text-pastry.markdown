---
layout: post
title: "Sublime Text 2 Plugin - Text Pastry"
date: 2014-07-12 16:27
comments: true
categories: tricks
---
新技能get

关于Sublime Text2 的简洁优雅，不做赘述。

[Text-Pastry](https://github.com/duydao/Text-Pastry)让Sublime Text 2 如虎添翼。

###DEMO1
####From:
```
aaa
bbb
ccc
ddd
eee
```
####To:
```
1. aaa
2. bbb
3. ccc
4. ddd
5. eee
```
####TODO
（快捷键基于Mac OS，Windows上略有区别）

1. 选中多行

2. `CMD`+`SHIFT`+`L`进入多行编辑模式

3. 把多个光标同时移动到每行的最前面

4. `CMD`+`ALT`+`T`  进入Text Pastry命令输入模式

5. 选择 \i 回车

6. 1-5的标号已经加上，输入. (空格)

7. DONE



###DEMO2
####From:
```json
{
  "apps": [
    { 
      "bundle_id": "com.htc.fm",
      "app_name": "FM 收音机"
    },
    {
      "bundle_id": "com.htc.android.mail",
      "app_name": "邮件"
    },
    {
      "bundle_id": "com.qq.qcloud",
      "app_name": "微云"
    }
  ]
}
```
####To:
```json
{
  "apps": [
    { 
      "uuid": "c9e968cc-d122-4162-a502-874cdcfa470c",
      "bundle_id": "com.htc.fm",
      "app_name": "FM 收音机"
    },
    { 
      "uuid": "4fba9716-29bf-4c06-b22e-75d59622703f",
      "bundle_id": "com.htc.android.mail",
      "app_name": "邮件"
    },
    { 
      "uuid": "6c3f911a-318b-4aad-82d6-c5f183d8ca00",
      "bundle_id": "com.qq.qcloud",
      "app_name": "微云"
    }
  ]
}
```
####TODO

目测在每个数据里面加个uuid这种需求不会在代码外去实现 :）
但类似的应该会有，如何秒杀是关键。如果够熟练，估计只需十几秒，即使有上百条数据

1. `CMD`+`F` 搜索 "bundle_id"，少量也可`CMD`+`D`

2. `ALT`+`ENTER` 选中所有”bundle_id“，并进入多行同时编辑模式

3. `CMD`+`SHIFT`+`ENTER` 在改行上面插入一空行，多行同时输入"uuid": "",让光标在""中

4. `CMD`+`ALT`+`T`  进入Text Pastry命令输入模式

5. 选择 \uuid(old)或uuid(new) 回车

6. uuid已经加上，一直末尾,输入,

7. DONE




[More Info](https://github.com/duydao/Text-Pastry#readme)

