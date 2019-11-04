---
title: window_navicat连接远程服务器数据库
date: 2019-05-15 10:09:44
categories: 
- 随记
tags:
- Mysql
---

## window mysql管理器(Navicat Premium) 连接 服务器(linux)数据库

在服务端进入 `mysql` 之后执行以下命令

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'USER'@'IP' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
FLUSH PRIVILEGES;
use mysql;
select host, user from user;
exit;
```

> 其中， IP 和 PASSWORD 可以自定义，USER 是 mysql.user 中存在的用户。完成后：
>
> 重启 MySQL
>
> 防火墙放行 3306
>
> **注意：IP可以设置为%，%表示无限制主机**

```shell
// 放行3306端口
/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT

service iptables save
service iptables restart

```

