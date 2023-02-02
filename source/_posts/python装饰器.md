---
title: python装饰器
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 14:44:21
password: d5e8139b6262895c208b9fd9e7a21ecba14eb00445638566fcb77bae14408691
summary: 装饰器
tags: basic
categories: Python
---
## 装饰器
 > 原理:将一个函数的引用传递到另一个闭包里面去

```python
def func_set(func(函数)):
    def func_start(参数):
        print("验证一")
        print("验证二")
        func()
    return func_start
```

> 使用方法一

```python
@func_set
def test(参数):
    print("test")
```

 > 使用方法二
 
 ```python
ret = func_set(test)
ret()
```

> 说明:装饰器的闭包中，外部函数用来接收函数，内部函数用来接收函数的参数，并且在没有调用函数之前只要有@就开始装饰,在装饰不定长的函数时

```python
def test(x,*args,**kwargs):
    print("test")
```
 
- 这种函数的装饰器的内部函数参数定义为(*args, **kwargs)就可以了，这种装饰器叫做通用装饰器,遇到有返回值的函数的时候，装饰器的内部函数也要有return，格式:return 函数（参数）



### 装饰器带参数
> 带参数的装饰器:@装饰器名称(参数)

```python
def 第一层(参数):                        # 第一层用来保存装饰器的参数
    def 第二层(函数):                    # 第二层用来保存被装的函数
        def 第三层(函数的参数):           # 第三层用来保存被装函数的参数，并且进行装饰函数，并且使用第一层保存的装饰器参数
            语句块
            [return] 函数
        return 第三层
    return 第二层
  ```

> 多个装饰器对同一个函数装饰:装饰的时候最下面的开始装,从下往上装


### 类装饰器
> 基本构架

```python
class Tt():
    def __init__(self, func):
        self.func = func

    def __call__(self,[*args,**kwargs]):
        代码
        [return] self.func

@Tt     #还可以点出更多东西以后会学到
def test():
    return "haha"

test()
```