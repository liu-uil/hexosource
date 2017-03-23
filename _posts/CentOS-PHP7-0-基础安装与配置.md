---
title: CentOS PHP7.0 基础安装与配置
date: 2016-04-02 16:38:19
tags:
- 环境部署
- 配置
- 原创
categories:
- PHP

---

> 官方文档见[Unix 系统下的 Nginx 1.4.x][1]
官方文档有2点问题
1. 命令参数没更新，
2. 关于配置也不够详细，详见后文

#### 1 下载

```shell
    wget http://cn2.php.net/distributions/php-7.0.5.tar.gz
```
#### 2 解压缩
```shell
    tar zxf php-x.x.x
```
#### 3 编译
```shell
cd ../php-x.x.x
./configure --enable-fpm --with-mysqli  #此处应该是mysqli，而非mysql
make -j8
sudo make install
```
#### 4 将配置文件复制到正确的位置
```shell
sudo cp php.ini-development /usr/local/php/php.ini
sudo cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
sudo cp sapi/fpm/php-fpm /usr/local/bin
```
#### 5 将 php.ini 文件中的配置项 cgi.fix_pathinfo 设置为 0 。
如果文件不存在，则阻止 Nginx 将请求发送到后端的 PHP-FPM 模块， 以避免遭受恶意脚本注入的攻击。
打开 php.ini:
```shell
vim /usr/local/php/php.ini
#定位到 cgi.fix_pathinfo= 并将其修改为如下所示：
cgi.fix_pathinfo=0
```
#### 6 修改php-fpm.conf 配置文件
```shell
#默认是无效路径，需修改
include=/usr/local/etc/php-fpm.d/*.conf
#复制一份默认配置文件，需以.conf结尾
sudo cp /usr/local/etc/php-fpm.d/www.conf.default /usr/local/etc/php-fpm.d/myphp.conf
#修改用户为
user = www-data
group = www-data
#需新建用户www-data
```
#### 7 Nginx配置
```shell
#修改nginx配置
#vim /usr/local/nginx/conf/nginx.conf
server{
    listen       your_port; 
    server_name  your_ip;
    access_log /home/liu-uil/logs/php_access.log;   
    error_log /home/liu-uil/logs/php_error.log;  
    location / {
        root   html;
        index  index.php index.html index.htm;
    }
    
    location ~* \.php$ {
        fastcgi_index   index.php;
        fastcgi_pass    127.0.0.1:9000;
        include         fastcgi_params;
        fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
    }
}
#sudo mv /usr/local/nginx-1.5.6/html/index.html /usr/local/nginx-1.5.6/html/index.html.bak
#sudo vim /usr/local/nginx-1.5.6/html/index.php
<?php phpinfo(); ?>
```
#### 8 重启Nginx
```shell
sudo /usr/local/nginx/sbin/nginx -s stop
sudo /usr/local/nginx/sbin/nginx
```
访问your_ip:your_port就可以看到php的提示信息了。
  [1]: http://php.net/manual/zh/install.unix.nginx.php
