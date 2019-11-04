---
title: macOS 搭建 LMNMP 环境
date: 2019-11-04 20:06:04
tags:
- LNMP
- mac
- PHP
categories:
- 随记
---

使用 `brew` 命令进行安装



## 安装 MySql

```shell
brew search mysql
```

搜索一下，看看有什么版本，大家可以根据自己的需要选择版本，我这选择 `5.7` 

```shell
brew install mysql@5.7
```

静静等待安装，安装完成之后启动服务

```shell
brew services start mysql
```

根据安装完之后的信息提示

```shell
We've installed your MySQL database without a root password. To secure it run:
    mysql_secure_installation
```

到 `/usr/local/Cellar/mysql@5.7/5.7.28/bin` 目录下执行

```bash
./mysql_secure_installation
```

按照信息提示一步一步完成。然后记得把 `mysql` 添加到环境变量中去

在 `~/.zshrc` 中添加

```bash
export PATH=$PATH:/usr/local/Cellar/mysql@5.7/5.7.28/bin
```

> 后面是自己 mysql 的安装目录

执行一下命令使其生效

```shell
source ~/.zshrc
```

> 因为我用的是 zsh，所以是在 `.zshrc` ，如果你是使用默认的 bash，那同样在 `~/.bash_profile` 添加环境变零，使其生效即可



## 安装 Nginx

```shell
brew install nginx
```

等待安装完成，编辑 `/usr/local/etc/nginx/nginx.conf` 配置文件，将默认监听的端口 `8080` 改成 `80` 

```bash
http {
    ............

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;
   	.......
```

启动服务

```shell
brew services start nginx
```

访问 `http://localhost` ，就可以看到 Nginx 的欢迎页面了



## 安装 PHP

同样使用

```shell
brew search php
```

找你所需要的版本安装

```shell
brew install php
```

这样，它会直接安装最新版本的 php。查看 php 版本发现还是系统自带的 7.1 版本，这个是因为还没把我们安装的最新版本加到环境变量中去。

```shell
### 使用自带的 bash
echo 'export PATH="/usr/local/opt/php@7.2/bin:$PATH"' >> ~/.bash_profile
echo 'export PATH="/usr/local/opt/php@7.2/sbin:$PATH"' >> ~/.bash_profile

source ~/.bash_profile

### 使用 zsh
echo 'export PATH="/usr/local/opt/php@7.2/bin:$PATH"' >> ~/.zshrc
echo 'export PATH="/usr/local/opt/php@7.2/sbin:$PATH"' >> ~/.zshrc

source ~/.zshrc
```



## 配置 PHP 和 Nginx

`/usr/local/etc/nginx/servers` 目录下添加 `default.conf` 

```bash
server {
        listen       80;
        server_name  default.me ;  ## 根据自己需要填写
        root   "/usr/local/var/www/default"; ## 根据自己的目录
        index index.html index.htm index.php;

        location / {
            #autoindex  on;
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php(.*)$  {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }
}
```

再次检查 `/usr/local/etc/nginx` 目录下是否存在 `fastcgi.conf` ，`nginx.conf`

如果没有的话，复制默认的

```bash
cp nginx.conf.default nginx.conf
cp fastcgi.conf.default fastcgi.conf
```

重启服务

```shell
_ php-fpm -d
nginx -s reload
```

就可以开心的玩耍了。



## 遇到的问题

访问时，网页一直显示 502 

```bash
[error] 38689#0: *6 kevent() reported that connect() failed (61: Connection refused) while connecting to upstream, client: 127.0.0.1, server: blog.me, request: "GET / HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "blog.me"
```

因为没有启动 php-fpm服务，启动就行了。



 ***不对之处，请多指教***

