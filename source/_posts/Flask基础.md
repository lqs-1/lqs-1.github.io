---
title: Flask基础
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 16:04:52
password:
summary: flask学习笔记
tags: flask
categories: Python
---
# Flask
## `__name__`对象说明

`__name__`返回的就是当前的模块名字

`__name__`如果在本文件中使用的话，那么就是`__main__`

`__name__`如果本文件被导入到别的文件中使用的话，那么就是模块的名字
在文件中，被调用的时候，永远不可能等于`__main__`



## Flask
创建Flask对象: 名字 = Flask(一个魔法值(以这个为根目录)),括号里面的参数，就表示以这个参数所在的模块为根目录，默认这个目录中的static为静态目录，templates为模板目录

创建视图: 等于创建一个函数，要显示给浏览器的数据，直接return

路径配置: 给视图函数加装饰器,格式: app.route('路径')

运行flask: app.run(),flask的参数:
```python
 app = Flask(__name__,  # 创建flask的应用对象，__name__返回的就是当前的模块名字,表示以什么为根目录
    static_url_path='/python',  # 访问静态的url前缀，不设置的话默认值是static
    static_folder='static',  # 静态文件的目录，默认就是static
    template_folder='templates',  # 模板文件的目录，默认的是templates
    )
```
### flask的配置
 >   flask没有Django那种的专门的配置文件，但是可以通过以下的三种方式来配置
 > -   1、使用配置文件:
     -   1、创建一个配置文件xxx.cfg
     -  2、app.config.from_pyfile(文件的路径)
> -    2、使用配置对象:
     -   1、创建一个类，将配置写入
     -   2、app.config.from_object(类名)
> -    3、直接配置
    -    app.config['DEBUG'] = True
     -   app.config.update(key=v1,k2=v2):一次性配置多个，如果已经存在，那么表示更新

### 获取配置中的参数
>    在flask中，配置中除了可以存储flask的配置，也可以存储开发者放的一些数据
 > -   获取方式一: app.config.get('key')
> -    获取方式二: current_app.config.get('key'),这是一个模块，也在flask模块中导入，current_app等价于app,有的时候我们拿不到app，所以这个模块可以替代

### app.run()的使用
 >   app.run(host='x.x.x.x', port=xxx, debug=True(False)) ,app.run(host='0.0.0.0', port=xxx)是万能的

### 查看路由表
>  和Django相比，Flask没有一个直观的路由表，而Django却有urls.py文件可以查看路由表
>  
 >   Flask查看路由表: app.url_map，这个方法就可以查看flask的路由表，默认的有static的路由记录，还可以查看请求方式
 >   
  >  如果某些视图只能用某些请求方式来访问，那么方法就是:@app.route('路径', methods=[请求方式列表])
  >  
 >   当有多个视图对应同一个url地址的时候，解决方法是：要么修改url,要么修改请求方式
 >   
 >   当有多个url地址对应一个视图函数的时候(给视图函数多装几个装饰器)，那么就是说这多个url都可以访问这个视图
 >   
 >   重定向:从flask中导入redirect模块，就可以实现重定向(跳转，两次请求，重定向状态码302),用法:return redirect(url地址)
 >   
 >   反向解析: 从flask中导入url_for模块,就可以实现反向解析，用法: url = url_for('视图函数的名字', 参数列表)

#### url带参数
```text
@app.route('/路径/<转换器类型:参数名>'):
    已经实现的转换器类型: int , float , path,如:@app.route('/index/<int:id>'),在访问的时候:127.0.0.1:5000://index/123就可以访问.
    不加转换器类型就是表示匹配普通的字符串（除了/）:@app.route('/路径/<参数名>')
    int转换器类型，匹配整数
    float转换器类型，匹配小数
    path转换器类型，匹配字符串，但是包括/
        @app.route('/goods/<int:id>')
        def goods_detail(id):
                '''定义商品视图函数'''
            return f'goods detail page{id}'

    自定义转换器(三步):
        1、定义自己的转换器:
            导入from werkzeug.routing import BaseConverter。
            定义一个转换器类，使其继承BaseConverter.
        2、将自定义的转换器添加到flask的应用中:
            app.url_map.converters['转换器名'] = 类名
        3、使用:
            @app.route('/路径/<转换器类型:参数名>')
```
### 定义万能转换器
```text
说明一点:转换器也可以跟参数（re）: @app.route('/路径/<转换器类型(参数):参数名>')
方式一、
    class ReConverter(BaseConverter):
        '''
            re万能转换器
            创建好自定义的转换器之后，我们就将这个自定义的转换器，添加到应用中
            在视图函数使用的时候，可以跟参数，一个正则
            工作原理:
                在访问网址的时候，给定参数，在服务器接收到数据时，会根据转换器的正则来判断时候是一个非法的数据
        '''
        # url_map是一个固定的参数,必须调用父类的初始化方法，也必须传入这个参数
        def __init__(self, url_map, regex):
            super().__init__(url_map)
            # self.regex是BaseConverter中的参数（接收正则）,他是专门来看看你自定义的转换器是什么样的一个转换器，接收参数（re）
            # 将正则表达式的参数保存到对象的属性中，flask会去使用这个属性来进行正则匹配
            self.regex = regex
    # 加到应用中
    app.url_map.converters['re'] = ReConverter
    @app.route("/goods/<re(r'1[34578]\d{9}'):id>")
    def goods_detail(id):
        '''定义商品视图函数'''
        return f'goods detail page{id}'
方式二、
    class ReConverter(BaseConverter):
        '''
            re万能转换器
            创建好自定义的转换器之后，我们就将这个自定义的转换器，添加到应用中
            在视图函数使用的时候，可以跟参数，一个正则
            工作原理:
                在访问网址的时候，给定参数，在服务器接收到数据时，会根据转换器的正则来判断时候是一个非法的数据
        '''
        # url_map是一个固定的参数,必须调用父类的初始化方法，也必须传入这个参数
        def __init__(self, url_map):
            super().__init__(url_map)
            # self.regex是BaseConverter中的参数（接收正则）,他是专门来看看你自定义的转换器是什么样的一个转换器，接收参数（re）
            # 将正则表达式的参数保存到对象的属性中，flask会去使用这个属性来进行正则匹配
            self.regex = r'1[34578]\d{9}'
    # 加到应用中
    app.url_map.converters['re'] = ReConverter
    @app.route("/goods/<re:id>")
    def goods_detail(id):
        '''定义商品视图函数'''
        return f'goods detail page{id}'
```

### 高级的转换器应用
```text
class ReConverter(BaseConverter):
    '''
        re万能转换器
        创建好自定义的转换器之后，我们就将这个自定义的转换器，添加到应用中
        在视图函数使用的时候，可以跟参数，一个正则
        工作原理:
            在访问网址的时候，给定参数，在服务器接收到数据时，会根据转换器的正则来判断时候是一个非法的数据
    '''

    # url_map是一个固定的参数,必须调用父类的初始化方法，也必须传入这个参数
    def __init__(self, url_map):
        super().__init__(url_map)
        # self.regex是BaseConverter中的参数（接收正则）,他是专门来看看你自定义的转换器是什么样的一个转换器，接收参数（re）
        # 将正则表达式的参数保存到对象的属性中，flask会去使用这个属性来进行正则匹配
        self.regex = r'1[34578]\d{9}'

    '''
        to_python和to_url都是父类中的方法:
            to_python:在匹配成功以后，请求网址带来的参数会交给to_python做出处理，返回的结果为视图函数的参数
            to_url:在使用重定向(url_for)的时候使用，url("视图名",参数)
            使用重定向的时候，被定向的视图函数如果有参数需要匹配，那么，在重定向的时候，在使用url_for()的时候，就会去匹配一次被定向视图使用的转换器，在匹配成功的同时调用to_url方法
            在to_url中处理以后，会交给to_python处理

    '''

    def to_python(self, value):
        return value

    def to_url(self, value):
        return '13443218888'


# 加到应用中
app.url_map.converters['re'] = ReConverter


@app.route("/goods/<re:id>")
def goods_detail(id):
    '''定义商品视图函数'''
    return f'goods detail page{id}'


@app.route("/index")
def index():
    url = url_for("goods_detail", id=13412344321)
    return redirect(url)
```

## Request对象
```text
从flask中导入request和Response
request中常用的属性:
    data:获取请求的数据，并转换为字符串，request.data
    form:获取请求中的表单数据,request.form.get()，获取一对一,request.form.getlist()，获取一对多的数据
    args:获取url中的参数(http:x.x.x.x:xxx/xxx?city=chengdu)，
    cookies:获取到cookie，request.cookies.get()
    headers:获取请求头中的报文:request.headers
    method:获取请求方式:request.method
    url:获取请求的url:request.url
    files:获取上传的文件:  if request.method == 'POST':
                            file = request.files.get('pic')
                            file.save(f'./img/{file.filename}')
    path: 拿到浏览器请求的地址（视图）
```

### abort
```
从flask中导入abort
无条件终止视图函数的执行，并返回给前端特定的信息
用法一、
    abort（状态码）
用法二、
    re = Response("login defete")
    abort(re)
```
### 自定义错误信息
```
用户访问的页面不错在的时候，定义一个独立的视图,用@app.errorheader(404)来装饰
固定写法:
    @app.errorhandler(404)
    def header_error(err):
         return '没有'
```
### 设置响应信息的方法
```
从flask中导入make_response
方法一:
    使用:
    rsp = make_response(显示的字符串)
    rsp.status = '999 lqs'  状态码
    rsp.headers['lqs'] = 'lqs'   头信息
    return rsp
方法二:
    return 显示字符串, 状态码, 头信息
 ```
#### json
```
方法一、import json
    dic = {"name": "lqs", "age": 12}
    json_str = json.dumps(dic)
    print(type(json_str))
    print(json_str)
    dic_str = json.loads(json_str)
    print(type(dic_str))
    print(dic_str)
json.dumps():字典转json字符串
json.loads():json字符串转字典

方法二、从flask中导入jsonify
    return jsonify(dic) ：将字典转成json字符串，并且自动设置头信息
```
### cookie的读写
```
读写和django一样，都是通过response写入，通过request读取
在有关cookie的操作中，都要 app.config["SECRET_KEY"] = xxxxxx
    导入:from flask import make_response
    实例化:resp = make_response(显示字符串)
    设置:resp.set_cookie("lqs", "Flask", max_age=3600)
    读取:request.cookies.get('lqs')
    删除:resp.delete_cookie('lqs')
```

### session的读写
```
导入: from flask import session
设置: session[key] = value
读取: session.get(key)
```

### 钩子
```
类似于django中的中间件
在请求的时候使用，不是任何请求都会调用，是调用到视图函数的时候，在之前或者之后进行操作
@app.before_first_request : 第一次请求之前调用
@app.before_request : 每次请求之前调用
@app.after_request : 每次请求之后调用（视图函数没有出错的情况）
@app.teardown_request : 每次请求之后调用（不论是否出错都会调用）

@app.after_request : 对应的视图函数必须带一个参数，用来接收视图函数的返回值，也就是接收在页面上现实的东西
@app.teardown_request : 对应的视图函数必须带一个参数，用来接收视图函数的返回值，也就是接收在页面上现实的东西
```
### g对象
```
from flask import g
g对象中保存的数据，在一次请求中的，多个函数中可以使用
使用:
    g.key = value:存
    xx = g.key:取
```
### flask_script扩展(实现shell代码),这样子可以和django一样，runserver
```
pip install Flask-Script
from flask_script import Manager
manager = Manager(flask应用名)
manager.run()
```
### flask使用模板文件
```
from flask import render_template
return render_template("html模板", key=value, key=value)  或者    return render_template("html模板", **字典名（组织之后的上下文）)
在html模板文件中，可以使用过滤器:使用方法:{{value | 过滤器}}如safe就表示转义
```
### flask中html模板转义
```
使用方法: {{ 处理对象 | 过滤器 }}
常用过滤器:
    safe : 禁用转义
    capitalize : 变量值的首字母转换成大写其余小写
    lower : 全部转换为小写
    upper : 全部转换为大写
    title : 每个单词首字母大写
    trim  : 去除首尾空格
    reverse : 字符串反转
    format : 格式化输出
    striptags : 渲染之前把所有的html标签去掉
    first : 取出列表中的第一个元素
    last : 取出列表中的最后一个元素
    length : 获取列表的长度
    sum : 列表数据求和
    sort : 列表排序
自定义过滤器:
    在flask代码中定义
    第一种方式(函数):
        定义一个过滤器函数
        返回值就是过滤后的数据
        app.add_template_filter(函数名, 过滤器名)就可以注册这个过滤器
        然后在html模板中使用
    第二种方式(装饰器:
        定义一个过滤器函数
        返回值就是过滤后的数据
        加装饰器:@app.template_filter(过滤器名)
        然后在html模板中使用

```
### flask使用Flask-WTF
```
可以在服务器端(创建表单模型)校验表单数据，可以设置csrf攻击的关闭
pip install Flask-WTF
from flask_wtf import FlaskForm : 这个模块让类继承之后就可以实现创建表单模型
from wtforms import StringField, PasswordField, SubmitField : 这个wtforms模块中的方法都是关于表单中的各个控件的类型，这里没有写全面
from wtforms.validators import DataRequired, EqualTo : wtforms下面的validators模块中的方法就是一些校验规则

说明: 在flask中定义的表单模型作用有两个,首先，第一个作用就是不用用户再去自己打表单，可以将表单模型传到模板中生成表单，第二个作用就是验证数据，两个作用是区分开的

定义表单模型类:
    class LoginForm(FlaskForm):
    '''
        自定义的注册表单抽象类
            label标签表示设置名称,用中文就使用u"xx"
            validators表示验证数据，是一个列表类型
            DataRequired表示必须输入，有一个参数，表示验证失败的时候显示文本
            EqualTo表示和谁一样，有连个参数，第一个填写和谁一样，第二个填写验证失败的显示文本
    '''
        user_name = StringField(label=u"用户名", validators=[DataRequired("用户名不能为空")])
        password = PasswordField(label=r"密码", validators=[DataRequired("密码不能为空")])
        password2 = PasswordField(label=r"确认密码", validators=[DataRequired("确认密码不能为空"), EqualTo("password", "两次密码不一致")])
        submit = SubmitField(label=u"提交")


        @app.route("/login", methods=["POST", "GET"])
        def login():
            '''创建表单对象'''
            form = LoginForm()
            # 判断form中的数据是否合理
            # 如果form中的数据完全满足验证，则返回真，否则返回假
            if form.validate_on_submit():
                uname = form.user_name.data
                pwd = form.password.data
                pwd2 = form.password2.data
                print(uname, pwd, pwd2)
                session["username"] = uname
                url = url_for("index")
                return redirect(url)
            return render_template("login.html", form=form)


在html模板中生成表单:
     <form  method="post">
        {{ form.csrf_token}}
        {{ form.user_name.label }} {{ form.user_name }}<br>
        {% for msg in form.user_name.errors %}
        <p>{{ msg }}</p>
        {% endfor %}

        {{ form.password.label }} {{ form.password }}<br>
        {% for msg in form.password.errors %}
        <p>{{ msg }}</p>
        {% endfor %}

        {{ form.password2.label }} {{ form.password2 }}<br>
        {% for msg in form.password2.errors %}
        <p>{{ msg }}</p>
        {% endfor %}

        <p>{{ form.submit }}</p>

     </form>

```
### 宏的定义和使用
```
在同一个html中定义和使用:
    没有参数:
        定义:
        {% macro input() %}
         <input type="text" name="" value="" size="30">
        {% endmacro %}
        使用:
        {{ input() }}
    有参数:
    定义:
    {% macro input(type,name,size) %}
     <input type={{type}} name={{name}} value="" size={{size}}>
    {% endmacro %}
    使用:
    {{ input('text', 'username', 40) }}

在不同html中定义和使用:
    没有参数:
        定义在macro2.html中:
        {% macro input() %}
         <input type="text" name="" value="" size="30">
        {% endmacro %}
        在macro。html使用:
        {% import "macro2.html" as 别名 %}
        {{ 别名.input() }}

```
### 模板继承
```
和django中的模板继承一样: {% extend "模板名" %}  使用 {% block xx %} {% endblock xx %}
```

### flask中一些特殊的方法
```
有些方法和变量不需要传递也可以在模板中直接使用
config中的方法
request中的方法
url_for中的方法
模板的闪现:
    闪现就是表示有的信息，只是出现一次
    在flask中:from flask import flash
            在视图中使用flash(xxx)来添加信息，可以添加很多条
    在模板中:
        {% for x in get_flashed_messages() %}
        {{ x }}
        {% endfor %}
```
### flask链接数据库
```
pip install flask-sqlalchemy  模型类到sql语句的转换，再将结果转换为模型类对象
pip install flask-mysqldb   数据库驱动
from flask_sqlalchemy import SQLAlchemy  可以创建数据库对象
配置:
    app.config['SQLALCHEMY_DATABASE_URI'] = "mysql://root:123456@127.0.0.1:3306/msfood"
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True   # 配置跟踪,数据库和模型类同步
创建数据库sqlalchemy工具对象,就可以使用数据库了
    db = SQLAlchemy(app)


创建模型类
class Role(db.Model):
    '''身份表'''
    __tablename__ = "tbl_roles"  # 指明表名
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(32), unique=True)
    users = db.relationship("User", backref="role")
     def __repr__(self):
        '''补充方法,可以让显示对象的时候更直观,类似于__str__方法'''
        return f'{self.users}'
class User(db.Model):
    '''用户表'''
    __tablename__ = "tbl_users"  # 指明数据库的表名
    id = db.Column(db.Integer, primary_key=True)  # 主键，自动递增
    name = db.Column(db.String(64), unique=True)
    email = db.Column(db.String(128), unique=True)
    password = db.Column(db.String(128))
    role_id = db.Column(db.Integer, db.ForeignKey("tbl_roles.id"))

生成数据表:
    *第一次生成的时候清除所有数据:
         db.drop_all()
    创建所有表
        db.create_all()

说明:
    创建表的时候，和底层相关，只要是表中的一行，都是有关底层的，用的都是真实的表的名字
    users = db.relationship("User", backref="role"): 这个语句就是表示和底层无关，只和flask中的对象有关，User就是一个模型对象
        users表和roles表是有关联的（外键）,那么db.relationship就可以让两个表相互拿东西，表示从User模型类中拿出和Role模型有关联的数据
        backref表示没有定义relationship的对方表中，也可以反推拿出和User模型有关的Role模型的数据
        使用方法:
            在Role模型:role.users
            在user模型中:user.role

添加数据:
    创建对象:xx = 模型(属性=值，属性=值)
    session记录对象任务:db.session.add(xx),添加一条数据，db.session.add_all(),添加多条数据
    提交任务到数据库:db.session.commit()
```
### sqlalchemy查询
```
用模型类来查询
模型类.query.all():查询所有数据，返回的是一个列表，可以取出某一个对象，然后读取属性值
模型类.query.first():返回第一条记录
模型类.query.get(主键的id):根据主键id获取对象
模型类.query.filter_by(字段=xx,字段==xx).all():获取满足条件的所有数据,条件是与
模型类.query.filter(模型类.字段==xx,模型类.字段==xx).all():拿出满足条件的所有数据，条件是与
模型类.query.filter(or_(模型类.字段==xx,模型类.字段.endswith(xxx))).all():拿出满足条件的所有数据，条件是或，使用或必须导入from sqlalchemy import or_
模型类.query.filter(条件).offset(2).limit(3).order_by(模型类.字段.desc()).all():返回满足条件并且忽略前两条数据，取出三条数据然后降序排列的数据，每一个部分都可以单独使用
db.session.query(模型类.字段, func.count(模型类.字段)).group_by(模型类.字段).all():分组查询，from sqlalchemy import func,func.聚合函数(属性),就可以查出来
```

#### sqlalchemy修改
```
模型类.query.filter(条件).update({字段名:值,字段名:值,字段名:值})
db.session.commit()
```
#### sqlalchemy删除
```
 xx = 模型类.query.get(x)
 db.session.delete(xx)
 db.session.commit()
```

#### flask中和django一样使用迁移的方式
```
pip install flask-migrate
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand

db = SQLAlchemy(app)

创建flask脚本管理工具
manager = Manager(app)
创建数据库迁移工具对象
Migrate(app, db)
向manager对象中添加数据的操作命令
manager.add_command("操作的名字", MigrateCommand)

main中:
    manager.run()
终端中:
    python3 xxx.py 操作的名字 init  : 让他初始化一下，自动创文件夹
    python3 xxx.py 操作的名字 migrate -m "版本说明信息" : 生成迁移文件（版本说明可以不要），同价与django中的makemigrations
    多次migrate之后需要更新数据库:python3 xxx.py 操作的名字 upgrade，可以升级同步
    python3 xxx.py 操作的名字 history : 查看历史修改
    python3 xxx.py 操作的名字 downgrade 版本编号 : 版本回退

```

### flask发送邮件
```
pip install flask-mail
from flask_mail import Mail, Message
app = Flask(__name__)

app.config.update(
    DEBUG=True,
    MAIL_SERVER='smtp.qq.com',
    MAIL_PROT=465,
    MAIL_USE_TLS=True,
    MAIL_USERNAME='749062870@qq.com',
    MAIL_PASSWORD='kttumcufpqasbcii',
)

 # sender发送方,recipients接收方列表
    mail = Mail() : 创建mail对象
    mail.init_app(app) : 初始化mail对象
    msg = Message("傻逼", sender="749062870@qq.com", recipients=["2807175480@qq.com"]) : 设置邮件消息的邮件头
    msg.body = "傻逼"  : 设置邮件的正文
    mail.send(msg) : 发送邮件
```
### 蓝图（Blueprint）
```
之前写的都在一个py文件中
蓝图就是分割各个块
将视图写在其他的py文件中，那么不用再给写在外面的视图加装饰器，可以在有app的py文件中导入，然后通过app.route("路径")(视图函数)

蓝图的正解:
    '''
        蓝图和django中的各个应用差不多,一个蓝图自成一块，在主窗口中注册进app里面就可以使用
        使用方法:
            创建一个应用目录，再添加一个__init__.py让这个目录成为一个可以调动的包，再__init__.py中定义好蓝图,再到视图中导入这个蓝图，就可以使用，最后再视图使用之后，再到__init__.py中
            导入视图中使用了蓝图的视图函数。
            蓝图再使用模板文件和静态文件的时候，和主app不同，主app有默认的配置，而蓝图中没有，需要再定义蓝图的时候手动设置，和主app的设置方法是一样的
            蓝图在没有定义静态文件目录和模板文件目录的时候，默认是使用主app的，就算配置可静态目录和模板目录，蓝图也是先去主app的静态目录和模板目录查找，没有找到在回到自己的里面去找，这个时候，就是主app的优先级大于蓝图
            在视图中使用蓝图:
                1、在__init__.py中from flask import Blueprint
                2、在__init__.py中定义蓝图,xx = Blueprint("蓝图名",__name__)
                3、导入蓝图
                4、用蓝图装饰，和主app装饰视图一摸一样的
                5、在__init__.py中导入使用了蓝图的视图函数（导入到蓝图之下）
                6、在主app的文件中注册蓝图:app.register_blueprint(蓝图, url_prefix="/lqs")，url_prefix表示前缀
    '''
```
### 单元测试
```
在flask中使用单元测试的时候，千万不要动源代码，重新写个测试文件
import unittest
导入要测试的flask代码文件
创建测试类，继承unittest.TestCase
定义好测试函数
import unittest
import json
from flaskDay14 import Test

class Index(unittest.TestCase):
    def setUp(self):
        # 设置flask工作在测试模式下，在测试出错的时候会给出具体错误行
        Test.app.testing = True
        # 模拟flask客户端,setUp中的代码会在所有函数之前调用
        self.client = Test.app.test_client()
    def test_index(self):
        # 模拟请求返回数据
        ret = self.client.post("/index", data={"username":"lqs", "password":"lqs"})
        # 拿到数据
        resp = ret.data
        # 装成字典
        resp = json.loads(resp)
        # 断言
        self.assertIn("code", resp)
        self.assertEqual(resp["code"], 2)


if __name__ == '__main__':
    # 测试
    unittest.main()

对所有的测试都可以使用
```