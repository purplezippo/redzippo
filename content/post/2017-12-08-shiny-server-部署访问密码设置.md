---
title: shiny server 部署访问密码设置
author: 雨禾
date: '2017-12-08'
slug: shiny-server-部署访问密码设置
categories:
  - R
tags:
  - shiny
---

## 1. 安装Apache web server

```
apt-get install apache2
apt-get install aptitude
aptitude install -y build-essential libapache2-mod-proxy-html libxml2-dev
```
## 2. 安装proxy模块

```
a2enmod
ssl proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
```

## 3. 配置Reverse Proxy
```
vim /etc/apache2/sites-enabled/000-default.conf
```
设置为：

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf

        ProxyPreserveHost On
        ProxyPass / http://0.0.0.0:3838/
        ProxyPassReverse / http://0.0.0.0:3838/
        ServerName localhost

        <location />
        AuthType Basic
        AuthName 'Restricted Access - Authenticate'
        AuthUserFile /etc/httpd/htpasswd.users
        Require valid-user
        </location>
</VirtualHost>

```
其中``<location /></location>``部分设定apache通过基础授权的方式提供web服务


## 4.安装htpasswd

```
sudo apt-get install apache2-utils
```

## 5.添加账户密码
```
mkdir /etc/httpd
htpasswd -bc /etc/httpd/htpasswd.users muscle 123456
```
其中muscle为账户名，123456为密码。-c 命令htpassed创建新的用户文件，当用户文件已经创建时把c去掉任然调用上述语句可以追加新的用户。

## 6.重启服务

```
/etc/init.d/apache2 restart
```
此时访问网站将需要通过账号密码。


参考链接：

* http://ipub.com/shiny-https/
* http://ipub.com/shiny-password-protect/