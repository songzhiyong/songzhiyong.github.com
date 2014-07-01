---
layout: post
title: "redis-started"
date: 2014-06-23 23:41
comments: true
categories: redis
---

###Redis installation
#####1)Download
Go to the Redis Download page and find the latest stable version. Download it and unpack:

```
tar xzf redis-*
cd redis-*
```
#####2)Compile
```
make
sudo make install clean
```
#####3)Create config:
```
mkdir /etc/redis
cp redis.conf /etc/redis/redis.conf
```
#####4)Start server & Connect
```
redis-server /etc/redis/redis.conf

redis-cli
```

###PhpRedis Installation
#####1) Preparation
```
sudo apt-get install php5-dev
```

php5-dev provides the dev library as well as the phpize command which is required for the compiling step, if you dont install this package, you would got this error message when you start your step 2:

```
No command 'phpize' found, did you mean:
 Command 'phpize5' from package 'php5-dev' (main)
phpize: command not found
```
#####2) Get phpredis source code, should be pretty easy by running

```
git clone git://github.com/nicolasff/phpredis.git
```

#####3) Compile and install

```
cd phpredis
phpize
./configure
make
sudo -s make install
```

#####4) Enable the phpredis extension
```
echo "extension=redis.so">/etc/php5/conf.d/redis.ini
```
for ubuntu PHP5.5, you should create copy this file into  

```
/etc/php5/mods-available
```

#####5) Write a simple php script to test (running on cli would be fine if php5-cli is installed)

```php
<?php
        // phpredis_set.php
        $redis=new Redis() or die("Can'f load redis module.");
        $redis->connect('127.0.0.1',6379);
        $redis->set('test', 'hello world');
```
you can check phpredis with command:

```
 php -r "if (new Redis()==true){echo \"Redis is ready!\"}"
```
        
Prior to try phpredis I was using Rediska as the php redis client. I did some pretty quick and dirty benchmarking comparison and phpredis is clearly a winner here, not surprisingly because phpredis is a compiled extension written in C while Rediska is a pure php library.

```
time for i in `seq 1 1000`; do php phpredis_set.php; done
```
real 0m13.072s
user 0m6.560s
sys 0m3.620s

```
time for i in `seq 1 1000`; do php rediska_set.php; done
```
real 0m21.035s
user 0m12.150s
sys 0m5.050s

and the source code for rediska_set.php:

```php
<?php
        require_once 'Rediska/library/Rediska.php';
        $rediska=new Rediska();
        $rediska->set('set_testkey', 1);
```     


###Python-Redis installation

#####1)Install 
```
sudo pip install redis
```
or

```
sudo easy_install redis
```

or download the source and from source

```
sudo python setup.py install

```
#####2)Getting started

```python
>>> import redis
>>> r = redis.StrictRedis(host='localhost', port=6379, db=0)
>>> r.set('foo', 'bar')
True
>>> r.get('foo')
'bar'
```
<--unfinished-->
