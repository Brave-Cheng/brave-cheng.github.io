---
layout: post
title: "CentOS配置Memcache应用层缓存"
date: 2015-09-11
image: '/assets/img/'
description: 'CentOS配置Memcache应用层缓存'
tags:
- CentOS
- Memcache
- 应用缓存
categories:
- Memcache
twitter_text: 'CentOS配置Memcache应用层缓存'
---

> 当功能代码逻辑开发完成后，你会尝试完善数据库设计，SQL查询优化，重构代码寻求高效的计算结果，通过不同的优化手段来提升HTTP响应请求时间，比如，合并CSS，压缩JS，图片压缩等前端优化手段，从运维架构方面，可以先做CDN缓存，将你的代码部署到离用户最近的机房，以提高用户响应时间，服务端部署反向代理，作分布式缓存，提升服务性能，也可以加快应用访问时间。

> 我们现在的产品业务范围比较小，功能也不多，在功能代码开发完成后，在代码层面优化比较好的前提下，产品还是会遇到访问速度慢等性能问题。在硬件与带宽配置未升级时，针对我们的性能问题，暂使用缓存解决HTTP响应时间长问题，而缓存方面，我们选择了Memcache。

配置环境与附属包

-   CentOS6.3
-   [libevent2.0.22](http://libevent.org/)   
-   [Memcached](http://memcached.org/)
-   [memcached extension](http://pecl.php.net/package/memcache)

##下载需要的软件包
服务器安装Memcache缓存系统服务，需要下载上面的三个安装包，在Linux中可以直接wget url 下载。附上我的下载

![Alt text](/assets/img/memcache/下载libevent.png)
Libevent下载

![Alt text](/assets/img/memcache/C95FFE56-6F70-4143-B637-550B60D81AF4.png)

Memcached下载

![Alt text](/assets/img/memcache/安装memcache php 扩展.png)

php-memcache扩展下载

##开始编译安装软件

###编译安装Libevent

> 
- tar xzvf libevent-2.0.22-stable.tar.gz #解压包
- cd libevent-2.0.22-stable #进入到解压的目录
- /assets/img/memcache/configure --prefix=/usr && make && make install #编译前配置，生成Makefile文件，路径可自行更改

![Alt text](/assets/img/memcache/编译libevent.png)

编译完成后，查看包是否存在

![Alt text](/assets/img/memcache/查看libevent是否安装.png)

###编译安装memcached服务器软件

>
+ tar xzvf memcached-1.4.24.tar.gz #解压包
+ cd memcached-1.4.24 #进入到解压的目录
+ /assets/img/memcache/configure –with-libevent=/usr && make && make install #编译前配置，生成Makefile文件，路径必须与libevent中一致

![Alt text](/assets/img/memcache/编译memcache.png)

查看是否安装成功

![Alt text](/assets/img/memcache/查看memcache是否安装.png)

###编译安装PHP-Memcache扩展

> 
- tar zxvf memcache-2.2.7.tgz #解压包
- cd memcache-2.2.7 #进入到解压的目录
- /alidata/server/php-5.5.7/bin/phpize #动态为php添加扩展。phpize路径可能不一致，请根据自己的实际情况
- /assets/img/memcache/configure –enable-memcache –with-php-config=/alidata/server/php-5.5.7/bin/php-config –with-zlib-dir #php-config请根据自己环境情况填写
- make; make install #编译+安装

![Alt text](/assets/img/memcache/安装php-memcache扩展.png)

编译安装后，会提示安装扩展路径。

![Alt text](/assets/img/memcache/complete.png)

###为PHP.INI添加Memcache扩展

上面安装完成后，需要为编译安装好的Memcache扩展写入PHP配置文件中，找到php.ini文件，修改其内容如下：

>
+ extension_dir = "/alidata/server/php/lib/php/extensions/no-debug-non-zts-20121212/" #修改extension_dir路径
+ extension=memcache.so #在php.ini末尾增加扩展

保存退出，然后重启WEB服务器和PHP进程。需要注意的是，有的PHP-FPM编译环境不一致，重启php-fpm需要使用另外的方式进行，大家可以参考下面的资源。
![Alt text](/assets/img/memcache/重启服务器.png)

使用`ps aux | grep php-fpm`查看php-fpm的配置文件，然后`KILL -USR2 进程号`重启

###启动Memcached守护进程

![Alt text](/assets/img/memcache/重启memcached.png)

`/usr/local/bin/memcached -d -m 10 -u root -l 127.0.0.1 -p 12000 -c 256 -P /tmp/memcached.pid`

> 
+ -d选项是启动一个守护进程，
+ -m是分配给Memcache使用的内存数量，单位是MB，我这里是10MB，
+ -u是运行Memcache的用户，我这里是root，
+ -l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.0.200，
+ -p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口，
+ -c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定，
+ -P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid，

查看Memcache进程情况

![Alt text](/assets/img/memcache/启动memcached.png)


##安装配置综合了下面相关资源
+ [Linux下的Memcache安装](http://www.ccvita.com/257.html)
+ [php5.4 的 php-fpm 重启教程](http://column.iresearch.cn/u/abcedseo/678047.shtml)
+ [CentOS安装memcached及配置php的memcache扩展](https://app.yinxiang.com/shard/s5/nl/5189175/56813af9-de75-4c8e-8168-be53c0cc207a/) 