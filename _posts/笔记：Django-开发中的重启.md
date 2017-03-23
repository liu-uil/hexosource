---
title: 笔记：Django 开发中的重启
date: 2015-11-05 16:31:10
tags:
- Django
- 原创
categories: 
- Python
---
在Django开发中，修改模板，修改模型，修改css等，修改无处不在，但是经常会碰到修改了没有生效的情况，其实是需要重启，以便重新加载这些源文件。很多时候BUG其实已经改好了，你差的只是一步重启，在这类问题上我浪费了一些时间，现在记录一下。 
服务器的部署环境是普通的Nginx+uWSGI。
#### 重启uWSGI
修改了view函数或者模型结构后，需要重启才能生效
#### 重启Nginx
修改了css文件、图片等static文件需要重新collectstatic一下，然后重启Nginx才能生效
#### 重启shell
在
```
python manage.py shell
```
中测试时，如果导入了模型，那么在修改模型后，首先要用migrate同步一下数据库，然后需要重新进一次shell才能正常使用模型
#### 重启celery worker
修改了tasks.py文件或者其他worker任务的源文件后，需要重新启动一次celery worker 
推荐用supervisor监控worker，这样重启就是
```
supervisorctl reload
```
比较方便，并且在监控下，worker如果死掉，能自动重启
