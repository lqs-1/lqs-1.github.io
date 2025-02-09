---
title: Django基础
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 15:42:38
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: django学习笔记
tags: django
categories: Python
---
## Django
### 创建项目
django-admin startproject 项目名

### 创建应用
python manage.py startapp 应用名
### 运行web服务器
python manage.py runserver




```python
python3 -m http.server [port] # 可以启动一个web服务，供局域网内用户下载文件
```

### 将固定的路径加入python可以搜索的目录
import sys

sys.path.insert(位置（从0开始），要加入搜索的目录)
```python
python manage.py shell # 进入django终端
```
### ORM框架
-    1、对象和数据库映射
-    2、根据设计的模型生成数据库中的表:
    -  1、生长牵引文件:python manage.py makemigrations
    - 2、执行牵引文件:python manage.py migrate

### 模型类
> 在models.py中编写模型类

  -  1、生长迁移文件:python manage.py makemigrations
    -    注意如果有ForeignKey的话一定要加上on_delete = models.CASCADE
  - 2、执行迁移文件:python manage.py migrate
    
> python manage.py shell
```text
首先:导入定义好的模型类
其次:实例化一个对象
插入 :对象.属性名=值
保存:对像.save()
拿到数据:变量 = 类名.objects.get(条件)
    变量.属性：直接查看
    变量.属性=值  ——> 变量.save()  :修改
    变量.delete():删除
```

### 后台管理（在admin.py中编写后台管理类）
```
本地化: LANGUAGE_CODE = 'zh-hans' :使用中文
       TIME_ZONE = 'Asia/Shanghai' :使用中国时间

创建管理员:python manage.py createsuperuser   (第一次新建管理员的时候必须要执行一下牵引文件，可以不生成，但必须执行)

自己创建的管理员是默认是放在数据库中的一个单独的表中的，但是可以使用（在settings.py中）：
# django认证系统使用的模型类
AUTH_USER_MODEL='应用.类名'

这个来规定生成数据放在什么表，在实际开发中是特别的重要的，一般和用户信息放在一起

使用这个必须导入(models.py中)from django.contrib.auth.models import AbstractUser

加入自己定义的数据库或者数据库模型: 1、导入 2、admin.site.register(类)

规定自定义的数据库或者数据库模型的显示: list_display('','')
```

### 使用视图（在views中编写）
```
使用模块: from django.http import HttpResponse,HttpResponseRedirect(重定向)
使用模块: from django.shortcuts import render,redirect(也是重定向)
使用 return HttpResponse('返回值') 返回结果
使用模块: from django.conf.urls import include, url
url的使用方法:url(r'正则表达式',处理字符串)
            url(r'正则表达式',include(处理源))

实现过程: views.py 中创建视图（使用模块第一个） --->  创建 urls.py (使用第二个模块,并且导入views.py)  --->  项目中的urls.py
中添加创建的urls.py(使用url添加)
```


### 模板（项目文件夹中创建 模板文件夹templates）
```
在项目文件夹中的setting中设置TEMPLATES中的DIRS，表示模板位置，一般拼接
使用模板:
    导入模块:from django.template import loader
    1、使用模板文件: temp = loader.get_template('文件名可以带路径')
    2、定义上下文（个模板传参数）: context = {字典参数}
    3、渲染模板:产生HTML代码:res_html = temp.render(context)
    4、返回浏览器:return HttpResponse(res_html)

最简单的使用方法: return render(request,'文件可以带路径',{传参数})

参数在html代码中的使用方法:
    普通变量:{{key}}
    for循环:{% fro x in xx %}
            {%endfor%}

```

## 关于django和数据库的操作
```
1、创models文件 class xxx(models.Model)
2、迁移文件
3、创建views文件 def xxx(request)，必须有返回值,一般返回值用HttpResponse,使用模板的返回值用render,需要重定向的话使用redirect
4、新建应用urls.py文件,用于定义访问时用的url地址
5、取项目中的urls文件中添加应用的url文件 ：url(r'^',include(xxx.urls))
6、在项目中创建templates模板文件,在到seting中配置一下，templates中的文件就是html文档
```


### 第一大点:models,模型,注册模型类
django操作数据库的相关知识
```
应用下的models.py文件中定义的所有模型，都要继承一个抽象基类:
创建抽象基类:在项目中创建一个pythonPakage,名字叫db,在这个包下面创建一个base_model.py文件，在文件中编写一个抽像基类
```
```python
from django.db import models
class BaseModel(models.Model):
        '''模型抽象基类'''
        create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    update_time = models.DateTimeField(auto_now=True, verbose_name='更新时间')
        is_delete = models.BooleanField(default=False, verbose_name='删除标记')
    class Meta:
        '''说明是一个抽象模型类'''
        abstract = True
```

 #####   约束选项在字段类型中使用，字段类型在models模块下使用
字段类型:
```
1、AutoField:自动编号
2、BooleanField:布尔类型
3、NullBooleanField:Null，True，False三种值
4、CharField（max_length=）:字符串,必须指定max_length
5、TextField:大文本类型
6、IntegerField:整数
7、DecimalField(max_digits=,decimal_places=):浮点数，第一个参数表示总位数，第二个参数表示小数位数
8、FloatField(max_digits=,decimal_places=):浮点数，参数同上
9、DateField（[auto_now=,auto_now_add=]）:日期类型，有两个可选参数，用的时候只能用一个，第一个表示更新时间，第二个表示第一次创建的的时间
10、TimeField（[auto_now=,auto_now_add=]）:时分秒
11、DateTimeField（[auto_now=,auto_now_add=]）:年月日时分秒
12、FileField:文件上传
13、ImageField:有效图片

14、ForeignKey:一对多
15、ManyToManyField:多对多
16、OneToOneField:一对一

约束选项:
1、default:默认值
2、primary_key:主键
3、unique:不重复
4、db_index:索引，db_index=True/False
5、db_column:指定表字段的名字，db_column='名字'
6、null:空
7、blank:是否为空，在后台使用，默认True，在django管理页面有用
8、chioces：chioces选项,等于一个元组，格式 choices = ((xx,xxx),(xx,xxx)……),网页显示为一个列表框
9、verbose_name：制定在后台的显示名称 ，字段和表名都可以用
```
#### 富文本
```
django使用tinymce：
    pip install django-tinymce
    from tinymce.models import HTMLField
    xx = HTMLField(verbose_name = 'xx')
    在settings.py中注册这个富文本编辑器，INSTALLED_APPS中添加tinymce
    在settings.py中设置富文本编辑器的属性：
        TINYMCE_DEFAULT_CONFIG = {
            'theme': 'advanced',
            'width': 编辑器的宽,
            'height': 编辑器的高,
        }
    在项目urls中添加: url(r'^tinymce/', include('tinymce.urls'))

```
####	查询
```
说明一下:exclude,filter,get三个都时带条件的--->模型名.objects.查询方式(属性名__条件名=值)
条件名:
exact:判等 ，可以不用直接用 =
contains:模糊查询，包含
endswith\startswith:模糊查询，结尾\开头
isnull:空查询,布尔值
in:范围查询，属性名__in = 列表或者元组
gt,lt,gte,lte:大于，小于，大于等于，小于等于
day/month/year:使用方法:日期查询

get():返回满足条件的一条记录
all():返回所有记录，查询集
filter():返回满足条件的所有记录，查询集
exclude():返回不满足条件的所有记录，查询集
order_by(‘属性名1’,’属性名2'……):升序
order_by(‘-属性名1’,’-属性名2'……):降序，order_by()是将查出来的数据进行排序，如：BookInfo.objects.all().order_by('id','-btitle')


Q对象:用于查询时候的多个条件的  ’与或非‘  表示
    from django.db.models import Q
    例子:
        BookInfo.object.filter(Q(id = 2) & Q(btitle = 'hello'))   与
        BookInfo.object.filter(Q(id = 2) | Q(btitle = 'hello'))   或
        BookInfo.object.filter(~Q(id = 2))                        非
F对象:条件中用于属性比较,还可以进行算数运算
    from django.db.models import F
    例子:
        BookInfo.objects.filter(id__gt = F('bid'))
        BookInfo.objects.filter(id__gt = F('bid')*3)



聚合函数:sum,count,avg,max,min,在django中通过aggregates来使用
    from django.db.models import Sum,Count,Avg,Max,Min
    例子:
        BookInfo.objects.all().aggregate(Count('id')) : 返回值是字典
        BookInfo.objects.all().aggregate(Sum('bread') : 总和


插入、更新、删除:
    save():插入更新
    delete():删除

关联查询:
    一对多:对象.多类的名字__set.all()
    多对一:对象.关系属性


模型管理器:
    也是在models中编写，
    格式:
        class xxx (models.Manager):
            可以定义方法或者定义查询（all）


元选项(指定表名):
    class Meta:
        db_table = 'xxx'
    verbose_name = 'xxx'
    verbose_name_plural = verbose_name
    每个需要自定义表名的模型下方都要写这个代码，否则用系统给的表明,后两行代码代表指定在后台显示什么名字
```




## 第二大点:views,视图的使用
```
1、项目中的urls中加入应用中的urls中的所有，表示项目启动以后可以使用应用的链接配置
2、应用中的urls用于配置浏览器页面输入的地址，一般只配置‘/’以后的
3、编写views文件，写清楚每个url的动作，定义在函数中,需要返回值

4、views文件中所写的代码就是写应该在网页中现实的东西
5、如果要使用模板，需要在项目文件夹下创建一个templates文件夹，并且配置好路径

6、return HttpResponse(xx),不用模板
   return HttpResponseRedirect(地址),redirect(地址)，两个功能一样都是重定向，在数据库添加后及时显示时候用
   return render(request, '模板文件位置',{参数})

重点:关于数据库的操作:导入 models  (类)
    关于网页内容操作:导入 views   (函数)

seting 中的设置，要改一起改
    setting 中的DEBUG属性(调试模式):默认是True，改成False后可以显示标准的报错页面
    setting 中的ALLOWED_HOSTS属性(允许访问):默认注释了的，是一个列表，ALLOWED_HOSTS = ['*']表示所有用户都可访问

给视图传参数: 在urls中配置url的时候，给一个正则的分组就表示给视图传参数:
            第一种:直接分组:views中的形参可以随便定义 :url(r'^index(\d+)$',views.show)-->def show(request,a)
            第二种:分组命名:views长的形参必须和分组的名字一致  :url(r'^index(?p<num>\d+)$',views.show)-->def show(request,num)
request参数:
    def index (request)中使用,躲在用于登陆的时候使用
        request.POST:保存的是网页中用post方式提交的参数，参数保存在请求头中
        request.GET:保存的是网页中用get方式提交的参数，参数保存在url中
    使用方法:
        xx = request.POST.get('xxx')
        xx = request.GET.get('xxx')


 重点:
    如果是网页模板，那么需要创建一个templates文件夹在项目文件夹下并且setting中配置路径，上面也提到过
    如果是静态的文件，js，css，image这些文件的话，需要在项目文件夹下创建一个static文件夹，平且在setting中配置STATIC_URL = '/static/' 和 STATICFILES_DIR = [os.path.join(BASE_DIR,'刚刚创建的文件夹static')]
```

### Cookie（多用于记住用户名）
```
Cookie的设置:需要一个HttpResponse的对象或者他的子类的对象，set_cookie
取出cookie:request对象的COOKIES中

设置:
    xxx  =  HttpResponse() 实例化HttpResponse的对象
    xxx.set_cookie('key',value,max_age = x) max_age是这只cookie的生存周期的，单位为秒
    xxx.set_cookie('key2',value2,max_age = x)
    return xxx
取出:
    xxx = request.COOKIES['key']
    return HttpResponse(xxx)

```
### session(多用于银行卡,登录状态)
```
session是一个特殊的Cookie,他的安全性更高，只给浏览器一个cookie编号,更重要的是，session可以记住用户的登录状态

读取设置都在request
设置session:
    request.session['key'] = value
    request.session['key2'] = value2
    request.set_expiry(xx) 设置存活时间，整数，秒
    return HttpResponse(request)

获取:
    xxx = request.session['xx']
    xxx = request.session['xx']
    xxx = request.session.get('xx')
    return HttpResponse(xx)
登录状态:
    判断有没有这个键:request.session.has_key['']

```


## 第三块:模板templates
```
html里面的模板标签:
    `{{变量}}`
    `{% xx %}`  ` {% endxx %}`
    {# 注释内容 #}
   ` {% comment %}` 多行注释 `{% endcomment %}`

过滤器:
    格式: 模板变量 | 过滤器:参数
    date:改变日期的显示格式
    length:求字符串和列表的长度
    default:设置不符合要求时候的默认值

自定义过滤器:
    在应用下创建一个templatetags包
    在templatetags中创建过滤器xxx.py
    在xxx.py中导入 from django.template import Library
    实例化Library对象，xx = Library()
    定义一个函数，和python一样的，有返回值的
    装饰一下，@xx.xxx

    在html中加载，在最前面:`{% load xxx %}`
    使用过滤器:变量 | xx
    如果xx函数中有参数，竖线前面的变量就是参数，如果有两个以上的参数那么只用传n-1个参数

模板继承:
    被继承的html模板不动，在子html模板中删除html代码，只写一句`{% extends '父模板地址' %}` 地址相对于templates

    在父模板中预留位置:子模板就可以重写父模板中预留的位置，从而保证有些不同 在父模板中`{% block 块名 %}` 可以写可以不写 `{% endblock 块名 %}`，子模板中也写`{% block 块名 %} `可以写可以不写 `{% endblock 块名 %}`，两边的两个标签需要一样块名一样，内容可以不一样
    既要使用父模板中的预留内容，也要自定义预留内容:`{% block 块名 %}` `{{ block.super }}`可以写可以不写 `{% endblock 块名 %}`

    html转义:
        在views.py 中render(request,'html 路径', {'content':'<h1>hello</h1>'})
        这样传过去的用的时候{{content}}，html并不会把传过来的<h1>当作标签，而是转义成了字符
        解决方法:
            {{ content | safe }}

csrf伪造攻击:
    主要原因:
        1、你在正常登录网站之后，浏览器保存了你的sessionid,而且你并没有退出
        2、在没有退出的情况下，访问了其他的网站做了一些不好的操作，间接的修改你的密码
    django默认启用了csrf防护，并且只针对post提交的数据防护，并且对包括自己的所有人防护
    解决办法:
        在html文件中有post提交的表单下方第一行输入`{% csrf_token %}`这样自己就可以正常使用了

验证码(防止暴力请求):
    用闭包的方式验证登录，有些界面只有登陆了才能使用
    工作的时候网上取down


url 反向解析(给地址取个名字,然后直接用地址就可以动态的生成网页地址，不管网页地址怎么变化):
    在项目中的urls中的链接应用url地址的地方添加第三个参数:url(r'^', include('应用文件夹.urls'), namespace='应用文件夹名字')  ,namespace='一般用应用名,也可以自定义'
    然后给应用中urls中的每一个自定义的链接取名:url(r'^xxx$',views.xx, name = '取名')

    在html中使用:
        1、<a href =` {% url 'namespace:name' %}`  url地址中没有参数  url(r'^xx$', views.xx, name = 'name')
        2、<a href =` {% url 'namespace:name' 1%}`  url地址中有一个参数   url(r'^xx/(\d+)$', views.xx, name = 'name')
        3、<a href = `{% url 'namespace:name' 1 3%} ` url地址中有两个参数   url(r'^xx/(\d+)/(\d+)$', views.xx, name = 'name')
        *4、<a href = `{% url 'namespace:name' a=1 b=3%}`  url地址中有两个被取名的参数   url(r'^xx/(?P<a>\d+)/(?P<b>\d+)$', views.xx, name = 'name')
    在views中使用:
        from django.core.urlresolvers import reverse
        1、reverse('namespace:name') url地址中没有参数  url(r'^xx$', views.xx, name = 'name')
        2、reverse('namespace:name', args = (1)) url地址中有一个参数   url(r'^xx/(\d+)$', views.xx, name = 'name')
        3、reverse('namespace:name', args = (1,3) url地址中有两个参数   url(r'^xx/(\d+)/(\d+)$', views.xx, name = 'name')
        *4、reverse('namespace:name', kwargs = {'a':1, 'b':2}) url地址中有两个被取名的参数   url(r'^xx/(?P<a>\d+)/(?P<b>\d+)$', views.xx, name = 'name')
```

### 静态文件(css\js\image)
```
在项目文件夹中创建static文件夹
在setting中配置:
    STATIC_URL = '/定义网址以什么开头/'  一般都是static
        这里的名字如果想要随便改，然后在html中动态生成的话，在html中加载 `{% load staticfiles %} ` 在要使用的地方用`{% static '文件路径' %}`这样进行拼接
    STATICFILES_DIR = [拼接文件夹位置]
静态文件的使用实在html中的链接
```
###    中间件
```
1、获取浏览器端的ip地址: request.META['REMOTE_ADDR'],可以设置某些ip访问网站
2、中间件的左右就是在每一个视图函数调用之前都会先执行中间件，可以判断某些用户或者界面不能使用，如判断是否登录，判断ip是否被禁止
3、定义中间件的方法:
    在应用中新建一个middleware.py 的python文件
    中间件的构成:
        class xxxx(object):   中间件类
            xxx
            def process_view(self, request, view_func, *view_args, **view_kwargs)  中间件函数
                xxx
                return xxx
4、定义了中间件以后，我们需要在setting中注册：找到MIDDLEWARE_CLASSES，然后'应用名.middleware.中间类
5、中间件函数是内置的，只能使用内置的那么几个:
    process_view(self, request, view_func, *view_args, **view_kwargs) : url匹配之后，视图调用之前
    process_request(self, request) : 产生request对象之后， url匹配之前调用
    __init__(self) : 服务器响应第一个请求的时候调用
    process_response(self, request, response) : 视图调用之后，内容返回浏览器之前,需要return response
    process_exception(self, request, exception) : 视图函数出现异常的时候调用，如果有多个exception，则按照注册的从后往前执行
```

## 第四模块Admin后台管理
```
1、创建用户:python manage.py createsuperuser
2、登录用户:网址/admin
3、注册模型类:在models.py中定义模型（将数据库中的数据加载到后台管理页面），在admin.py中去注册这个模型，admin.site.register(模型名字)


上传图片:
    在static文件夹中新建一个media文件夹，用于保存用户的头像等
    在setting中设置MEDIA_ROOT = 拼接文件夹,MEDIA_URL = '/media/'

    后台管理界面上传图片:创建一个数据库模型，定义一个字段，类型为ImageField(upload_to = 'media中的某个文件夹')

浏览器上传的照片:
    表单的提交方式和编码方式必须如下
    <form method="post" enctype="multipart/form-data" action="/upload_down">
        `{% csrf_token %}`
        <input type="file" name="pic"><br/>
        <input type="submit" value="上传">
    </form>
        读取照片页面信息并保存：
    def upload(request):
        return render(request, 'upload.html')


    def upload_down(request):
        pic = request.FILES['pic']
        pic_path = f'{settings.MEDIA_ROOT}/user_admin/{pic.name}'
        with open(pic_path, 'wb') as pf:
        for content in pic.chunks():
            pf.write(content)
        PicInfo.objects.create(pic_super=f'user_admin{pic.name}')
        return HttpResponse('ok')
浏览器上传的图片：
    pic = request.FILES['xx']的对象：pic.name获取文件名,pic.chunks()分块读取内容进行保存，pic.size获取上传文件的大小都是常用的



分页（ 在view.py中和html代码中使用）:
    需要包：from django.core.paginator import Paginator
    Paginator属性:num_pages:返回分页的总页数,page_range:返回分页的页码列表
    Paginator方法:page(self,number)：返回低number页的Page实例对象
    Page的属性:number:返回当前页的页码，object_list：返回包含当前页的数据的查询集，paginator:返回对应的Paginator对象
    page的方法:has_previous:判断是否有前一页,has_next:判断是否有下一页,previous_page_number:放回前一页,next_page_unmber:返回下一页

```

## 第五模块(url匹配)

```
可以用path，url
from django.urls import path,include或者from django.conf.urls import url,include
在project的urls.py中，使用反向解析用到include,在include中添加参数namespace空间命名，可以进行反向解析，在应用中的urls.py中的urlpatterns上面添加一句app_name='xx'这个名字和namespace的名字相同。
在应用的urls.py中，想使用反向解析，不需要include,可以直接在最后添加参数，name='xx'
```




## 数据加密
```
pip install itsdangerous
from itsdangerous import TimedJSONWebSignatureSerializer
创建一个TimedJSONWebSignatureSerializer对象: xx = TimedJSONWebSignatureSerializer('加密秘钥'，过期时间)

加密：加密结果 = TimedJSONWebSignatureSerializer对象.dumps(数据)
解密: 解密结果 = TimedJSONWebSignatureSerializer对象.loads(加密结果)
```

## 异步发邮件
```
pip install celery
项目中新建一个任务包，在这个包中使用celery（from celery import Celery）
创建Celery的实例对象：xx = Celery('名字'，broker='redis://127.0.0.1:6379/1'),broker代表中间人
定义需要异步处理的函数，用@xx.task装饰
在需要用到异步处理的应用中使用：
    导入过来的函数.delay(参数)
再将整个项目复制一份，进入复制的项目中的celery的包中：
    import os
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dalifresh.settings')
    django.setup()
然后再终端输入命令:celery -A 取好的名字 worker -l info来运行这个异步处理
```