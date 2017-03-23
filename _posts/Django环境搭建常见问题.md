---
title: Django环境搭建常见问题
date: 2015-08-25 16:25:25
tags:
- Django
- 原创
categories: 
- Python
---
>前言
>
>开始我尝试了在windows下做django的开发，但是问题很多，越做越多，诸如编码、斜杠之类的问题，解决了也意义不大，试过的同学都懂，所以建议尽量linux下搭建吧..

---
#### 1. 虚拟机共享文件配置，hgfs下没有文件
由于我开始时在windows下开发的，所以有些包下好了，一时不好找到源，所以干脆共享给虚拟机，但是虚拟机系统的hgfs文件夹下就是看不到共享的文件，更新了vmware-tools后重启也不行，需要完全卸载vmware-tools后，重启虚拟机然后重新安装vmware-tools，如果没有看到，再重启一下，多试几次就会出现了。
#### 2. django 手动安装步骤
```python
wget https://www.djangoproject.com/m/releases/1.8/Django-1.8.3.tar.gz
#1） 安装
$ tar zxf Django-1.5.8.tar.gz
$ cd Django-1.5.8
$ sudo python setup.py install
#安装完成后
$ which django-admin.py
/usr/local/bin/django-admin.py
#表示成功
```
#### 3. python从系统自带的2.6.6升级到2.7.x的方法，重装
```python
#系统自带的还是会用到，所以不要卸载
#首先下载源tar包
#可利用linux自带下载工具wget下载，如下所示：
wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tgz
#下载完成后到下载目录下，解压
tar -zxvf Python-2.7.3.tgz
#进入解压缩后的文件夹
cd Python-2.7.3
#在编译前先在/usr/local建一个文件夹python27（作为python的安装路径，以免覆盖老的版本）
mkdir /usr/local/python2.7.3
#在解压缩后的目录下编译安装
./configure --prefix=/usr/local/python2.7.3
make
make install
#此时没有覆盖老版本，再将原来/usr/bin/python链接改为别的名字
mv /usr/bin/python /usr/bin/python_old
#再建立新版本python的链接
ln -s /usr/local/python2.7.3/bin/python2.7 /usr/bin/python
#这个时候输入
python
#就会显示出python的新版本信息
Python 2.7.3 (default, Sep 29 2013, 11:05:02)
[GCC 4.1.2 20080704 (Red Hat 4.1.2-54)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
#### 4. no module named yum解决办法
用3里面的步骤安装python后，会出现这个问题
解决办法:
修改yum文件
```python
vi /usr/bin/yum
#将 #!/usr/bin/python 修改为 #!/usr/bin/python2.4（自己的python版本）
```
#### 5. django安装时报错，invalid syntax，python版本不对
Linux自带的python版本不对，需按照3,4步骤进行升级
#### 6.django安装时出现缺少setuptools，需要安装setuptools
```python
Wget https://pypi.python.org/packages/source/s/setuptools/setuptools-18.0.1.tar.gz#md5=cecd172c9ff7fd5f2e16b2fcc88bba51
https://pypi.python.org/pypi/setuptools 最下方下载
```
#### 7. 安装setuptools，报错：Compression requires the (missing) zlib module
```python
yum install zlib
yum install zlib-devel（注意不要漏这一步，虽然不是zlib）
#安装完成后，重新编译和安装 python2.7
Make & make install
#执行完后，返回setuptools文件夹，python setup.py install 即可
```
#### 8. 错误：django was: No module named mysql.base
```python
'ENGINE': 'mysql'
```
改为
```python
'ENGINE': 'django.db.backends.mysql'
```
#### 9. 错误No module named MySQLdb
```python
yum install MySQL-python #注意大小写
#或者
wget http://sourceforge.net/projects/mysql-python/files/mysql-python/1.2.3/MySQL-python-1.2.3.tar.gz
#手动安装
tar zxvf MySQL-python-1.2.3.tar.gz
cd MySQL-python-1.2.3
python setup.py build
python setup.py install
```
#### 10. centos 安装mysql （yum install msql） 后不能连接数据库
安装全套
```python
yum install mysql
yum install mysql-server
yum install mysql-devel（后面两个不要忘）
#启动mysql 服务
service mysqld start
```
#### 11.mysqladmin 修改密码
1.例如你的 root用户现在没有密码，你希望的密码修改为123456，那么命令是：
```python
mysqladmin -u root password 123456
```
2.如果你的root现在有密码了（123456），那么修改密码为abcdef的命令是：
```python
mysqladmin -u root -p password abcdef
```
#### 12. mysql 加入中文出现乱码的解决办法
1.创建数据库时
修改 /etc/mysql 下的 my.cnf 文件
```python
[mysql]
 default-character-set=utf8
create database MyNewDatabase default character set utf8;
```
2.已有数据库
```python
ALTER TABLE blog_blogpost MODIFY COLUMN body VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL;
```
改变出乱码的TABLE 的COLUMN
#### 13.出现CSRF认证问题错误
在 view.py 中的 render_to_response 中，使用 RequestContext 来代替默认的 Context 。
```python
return render_to_response('contact_form.html', {
                          'errors': errors,
                          'subject': request.POST.get('subject', ''),
                          'message': request.POST.get('message', ''),
                          'email': request.POST.get('email', ''),
                          },context_instance=RequestContext(request))
```
在模板文件中的 form 表单内添加csrf_token。
```xml
<form action="/contact/" method="post">
    <br />{% csrf_token %}<br />
    <p>Subject: <input type="text" name="subject" value="{{ subject }}"></p>
    <p>Your e-mail (optional): <input type="text" name="email" value="{{ email }}"></p>
    <p>Message: <textarea name="message" rows="10" cols="50">{{ message }}</textarea></p>
    <input type="submit" value="Submit">
  </form>
```
解决方案二：不使用 CSRF 验证
```python
MIDDLEWARE_CLASSES = (
  'django.contrib.sessions.middleware.SessionMiddleware',
  'django.middleware.common.CommonMiddleware',
  #'django.middleware.csrf.CsrfViewMiddleware',
  'django.contrib.auth.middleware.AuthenticationMiddleware',
  'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
  'django.contrib.messages.middleware.MessageMiddleware',
  'django.middleware.clickjacking.XFrameOptionsMiddleware',
  'django.middleware.security.SecurityMiddleware',
  'django.middleware.locale.LocaleMiddleware',
)
```
#### 14. 在mysql中注入中文乱码的问题
如果数据库中存储的数据包含中文，那么在python的控制台打印查询结果时可能会出现乱码，下面是解决方案：
在python脚本开始设置编码方案为utf-8，即在脚本文件开头加上一行：
```python
#encoding=utf-8
```
连接MySQL数据库时设置编码集为utf8：
```python
conn = MySQLdb.connect(host = "localhost",
                 user = "user",
                 passwd = "passwd",
                 db = "pythontest",
                 charset = "utf8")
```
（可选，不一定需要）在脚本文件中设置Python的默认编码为utf-8：
```python
Import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```
（可选，不一定需要）修改数据库中的编码方案为utf-8。如果使用Navicat，直接从“数据库属性”中即可修改数据库编码方案；如果是Linux下的MySQL数据库，那么需要修改MySQL的配置文件，即设置 MySQL 的 my.cnf 文件，在 [client]/[mysqld]部分都设置默认的字符集（通常在/etc/mysql/my.cnf)：
```python
[client]
default-character-set = utf8
[mysqld]
default-character-set = utf8
```
一般前面两步就行了
