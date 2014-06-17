---
layout: post
title: "python批量修改火星坐标"
date: 2014-03-15 10:02
comments: true
categories: python
tags: [Octopress ,python,火星坐标]
---
科普：火星坐标系统--国家保密插件，也叫做加密插件或者加偏或者SM模组，其实就是对真实坐标系统进行人为的加偏处理，按照几行代码的算法，将真实的坐标加密成虚假的坐标，而这个加偏并不是线性的加偏，所以各地的偏移情况都会有所不同。而加密后的坐标也常被人称为火星坐标系统。<br>
API：  http://api.zdoz.net/interfaces.aspx<br>
使用方法： 同目录下，from.txt存放待转换GPS数据，经纬度以制表符分隔。<br>
输出：同目录下，生成done.txt为转换好的数据。<br>

```python
import httplib
import json
 
def transGps(origin):
    host="api.zdoz.net"
    path = '/transgps.aspx?lat='+ origin[0]+'&lng='+origin[-1]
    h = httplib.HTTPConnection(host,80)
    h.request('GET', path)
    result = h.getresponse().read()
    result = json.loads(result)
    transfered=[n for n in range(2)]
    if result.get('Lat')!=None :
        transfered[0]=str(result['Lat'])
    if result.get('Lng')!=None:
        transfered[1]=str(result['Lng'])
    return transfered
 
inputFile = 'from.txt'
outputFile = 'done.txt'
fin = open(inputFile, 'rU')
fout = open(outputFile,'w')
fout.write("lat\tlng\n")
for line in fin :
    transfered = transGps(line.replace("\n","").split("\t"))
    fout.write(transfered[0].encode('UTF-8'))
    fout.write('\t')
    fout.write(transfered[1].encode('UTF-8'))
    fout.write('\n')
    print transfered[0]
    print transfered[1]
    print ''
fin.close()
fout.close()
```
