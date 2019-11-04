---
title: 修改 php 代码的时候总是需要等一分多钟才能生效，可能是 Zend OPcache的锅
date: 2019-04-28 17:11:56
category:
- PHP
tags:
- PHP
---



**背景：** 一直在 oneinstack  的集成环境下开发，每次修改 php 代码的时候总是需要等一分多钟才能生效。一分多钟啊！！！！！

无论怎么强制刷新浏览器，还有清除浏览器的访问记录，开启 Chrome 的隐身模式，一切能删除清楚缓存的方式都尝试了一遍。

发现都没有用没有用没有用！！



今天，终于受不了，google 了大半天，都是修改 nginx 服务，但都没有用！

终于在一个大神的提醒下，发现了 **Zend OPcache** 这个大坑，原来一直都是这个东西在搞鬼

官方简介：OPcache 通过将 PHP 脚本预编译的字节码存储到共享内存中来提升 PHP 的性能， 存储预编译字节码的好处就是 省去了每次加载和解析 PHP 脚本的开销

存错到内存里去内存里去啊啊啊啊，难怪怎么清楚缓存都没用

在调试应用阶段，不需要使用 PHP 的缓存，因为提交修改后要1分钟后才生效

它默认是关掉的，不知道是不是 oneinstack 这个锅，其他集成环境不会出现这情况

所以立马就把这个 ×× 东西给关掉

打开 `phpinfo()`  ，在输入内容中查看opcache部分，可以看到它的配置文件，我的是在 `/usr/local/php/etc/php.d/ext-opcache.ini`  

```
[opcache]
zend_extension=opcache.so
opcache.enable=0
opcache.memory_consumption=384
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.save_comments=0
opcache.fast_shutdown=1
opcache.enable_cli=0
;opcache.optimization_level=0

```

把 `opcache.enable`  、`opcache.enable_cli`  的值改成 0 就行

重启apache即可。如果你用的是nginx + php-fpm ,那么你就重启你的php-fpm



终于不用再傻傻得等一分多钟了！！！！！！