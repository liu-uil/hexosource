---
title: Django 1.8 环境下 xadmin 调试部署
date: 2015-11-21 16:31:41
tags:
- Django
- 原创
categories: 
- Python
---
Django是一个功能很强大的web框架，因为发展的很快，所以相近的版本间很多函数不同是很正常的事情。这是好事，但是给Django的第三方开发者带来了很大的负担，需要同步更新自己的代码，这确实是很麻烦...不管怎么样，现在如果想在Django 1.8 环境上部署git上的原版[xadmin](https://github.com/sshwsfc/django-xadmin)还是要修改不少地方的，也可以尝试安装1.8的branch，不过可能也会有问题。下面就Django 1.8 版本下部署xadmin做了一些说明。
部署的大致步骤是：
#### 1，pip install django-xadmin 安装
#### 2，修改install_app中的代码如下:
```python
INSTALLED_APPS = (
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.sites',
'django.contrib.messages',
'django.contrib.staticfiles',
#'django.contrib.admin', # 这个可以去掉
'xadmin',
'crispy_forms',
# 'reversion', # 需要pip install django-reversion
)
```
#### 3，在urls.py里添加:
```python
import xadmin
xadmin.autodiscover()
```
这里有个注意事项，你需要把admin的配置去掉，这俩admin可能存在冲突，会导致出错。
#### 4，修改urlpatterns配置如下：
```python
urlpatterns = patterns('',
url(r'^xadmin/', include(xadmin.site.urls), name='xadmin'),
#...
)
```
#### 5，如果对admin进行了简单的配置, 你需要做的就是替换到自定义Admin的继承为 object ，替换admin.site.register为xadmin.site.register。
可以参考https://github.com/sshwsfc/django-xadmin/blob/master/demo_app/app/adminx.py
一个简单的示例：
```python
#coding:utf-8
from django.contrib import admin
from django.core import urlresolvers
from .models import Post
#class PostAdmin(admin.ModelAdmin):
class PostAdmin(object): # 一处替换
search_fields = ('title', 'alias')
fields = ('content', 'summary', 'title', 'alias', 'tags', 'status',
'category', 'is_top', 'is_old', 'pub_time')
list_display = ('preview', 'title', 'category', 'is_top', 'pub_time')
ordering = ('-pub_time', )
save_on_top = True
def preview(self, obj):
# 第二处替换： 'xadmin:blog_post_change'
#url_edit = urlresolvers.reverse('admin:blog_post_change', args=(obj.id,))
url_edit = urlresolvers.reverse('xadmin:blog_post_change', args=(obj.id,))
return u'''
<span><a href="/%s.html" target="_blank">预览</a></span>
<span><a href="%s" target="_blank">编辑</a></span>
''' % (obj.alias, url_edit)
preview.short_description = u'操作'
preview.allow_tags = True
#admin.site.register(Post, PostAdmin)
# 第三处替换
xadmin.site.register(Post, PostAdmin)
```
这样走下来就是部署完了，如果是旧版的Django（低于1.6吧）应该可以访问xadmin了，但是如果是1.7版本或以后的可能还有别的问题，下面是需要做的修改，
首先是改下版本的范围
```python
xadmin/models.py
if 4 < django.VERSION[1] < 7:
AUTH_USER_MODEL = django.contrib.auth.get_user_model()
else:
AUTH_USER_MODEL = getattr(settings, 'AUTH_USER_MODEL', 'auth.User')
```
否则启动不了
改了可以启动了，但是就会开始各种报错，原因是很多原先用的Django组件都过时了，需要逐一处理，如果用sed批量替换需把条件设置严格一点，以免误换了不想关的地方又麻烦
```python
from django.db.models.related import RelatedObject
from django.db.models.fields.related import ForeignObjectRel
```
文件中用到RelatedObject 都要改成ForeignObjectRel，下面transaction的commit_on_success也过时了，改成atomic
```python
@transaction.commit_on_success
@transaction.atomic
```
formtools 和django-contrib-comments安装
将https://github.com/django/django-formtools 这里的formtools文件夹拷贝到/usr/local/python2.7.3/lib/python2.7/site-packages/Django-1.8.3-py2.7.egg/django/contrib/（也就是contrib的目录下）即可
```python
pip install django-contrib-comments
and use latest code
from django.contrib.comments.models import Comment # old
from django_comments.models import Comment # new
```
opts的module_name属性也过时了，需要修改
'Options' object has no attribute 'module_name'
改成model_name
如果更改了代码，错误没有变化，需要重启一下服务器（我的是部署在Nginx+uwsgi上，所以就是重启Nginx+uwsgi，可以写成脚本，每次重启直接执行一下，1s就ok），终于可以看到登陆界面了（可能还有别的问题，可以参考上面的办法解决）
登陆的时候由于要检查cookie，这里又会报错，需要把相关的test注释掉
```python
#self.check_for_test_cookie()
```
此外get_query_set在django 1.8中做了如下改动
```python
Rename "get_query_set" for "get_queryset" #307
#(isinstance(field, models.related.RelatedObject) and改成如下
isinstance(field, models.fields.related.ForeignObjectRel) and
/usr/local/python2.7.3/lib/python2.7/site-packages/xadmin/views/list.py in get_context, line 375
self.get_model_method_fields()改成
tuple(self.get_model_method_fields()))
```
此外可能还有个别的问题，也是上面的那几类问题了，我的网站简单，当这些都解决后，xadmin界面就出来了。
![xadmin登录界面][1]
补充问题：
```python
Remove widget.is_hidden=True for Django 1.7
/usr/local/python2.7.3/lib/python2.7/site-packages/xadmin/plugins/quickform.py 中的
#self.is_hidden = widget.is_hidden 需要注释掉
```
[1]: http://chuantu.biz/t5/25/1470305890x3738746547.png
