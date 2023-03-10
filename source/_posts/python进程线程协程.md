---
title: python进程线程协程
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 14:20:10
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: python进程线程协程实现
tags: basic
categories: Python
---
## 进程
### multiprocessing下面
- 1、Process:创建一个进程,参数有target，args,args是一个元组作为形式参数
- 2、start:运行这个进程
- 3、Queue:是队列，先进先出，和栈相反，Queue(20)就是说可以存放20个数据，数据部分类型。用full来判断是否满，用empty来判断是否空了用put来往里面放数据,用get来拿数据
- 4、Pool:进程池，实现重复利用，进程任务数量太多或者任务数不确定的时候，最好使用进程池。如果有10000个进程，那么创建一个只有10个任务的进程池，就可以反复一万次来完成任务，避免系统崩溃,利用for循环来创建进程，apply_async(函数，(参数，参数,),创建之后立刻关闭close，并且紧跟其后用join，否则程序死掉'

```python
from multiprocessing import *
import time
import random

def info(i):
    t_start = time.time()
    time.sleep(random.randrange(1,2)*2)
    t_stop = time.time()
    print(f'{i}执行时长:{t_start-t_stop}')
    
def main():
    p = Pool(3)
    for i in range(100):
        p.apply_async(info, (i))
    p.close()
    p.join()

if __name__ == "__main__":
    main()
```

## 线程

### threading下面
-    1、Thread:创建一个线程对像，可以有两个参数 target= ，args=，args是一个元组
-    2、start：使用创建的线程
-    3、enumerate:用来查看有多少个线程在跑，用len来显示个数，用while来控制
-    4、Lock:互斥锁，可以实例出一个实力对象，require是上锁，release是解锁

> 说明一点:在用c语言编写的python解释器中执行python的线程的时候，会涉及到GIL锁，GIL锁就是同一时刻只有一个程序在执行，这是一种假的多任务，只有线程在有，解决GIL锁的方法
> - 1、换解释器
> - 2、换线程语言(因为python基本可以执行其他的所有语言，c、c++、java等)

### 线程共享全局变量
 -   1、如果在线程代码中对全局变量进行修改，要用global,也就是说要令全局变量'等于'一个什么东西的时候必须用global，如：i+=1，用变量的方法来修改的不用使用global，如list.append()
 -   2、如果没有要对全局变量进行修改，就不用global申明，只是使用全局变量也不用使用global
 -   3、全局变量在线程中也可以共享
 -   4、有可能在多线程中可能会出现资源争夺（用Local互斥锁来解决，互斥锁不要太多，用以产生死锁）

### Thread的实例对象还可以是一个类
 -   1、让这个类来继承threading.Thread这个类
 -   2、用的时候直接 对象名 = 这个类（）
 -  3、start的使用：这个类的对象名.start
 -  4、（最重要):这个类中只能进行多线程的就是run函数，所以如果要对类创建多线程，那么必须有run函数，类当中的其他函数只能通过 self.函数名 的形式申明在run函数中

```python
from threading import *
import time

class Prison(Thread):

    def info2(self):
        print("---info2----")

    def info3(self):
        print("---info3----")

    def run(self): # 特别重要:只会调用run
        print("---info1----")

def main():
    info = Prison()
    info.start()

if __name__ == "__main__":
    main() 
```

## 协程

### yield协程

```python
import time
def info1():
    while True:
        print('-----1-----')
        time.sleep(0.5)
        yield
def info2():
    while True:
        print('-----2-----')
        time.sleep(0.5)
        yield
def main():
    in1 = info1()
    in2 = info2()
    while True:  一定要用循环来切换
        next(in1)
        next(in2)
if __name__ == "__main__":
    main()
```

### greenlet协程
```python
from greenlet import *
import time
def info1():
    while True:
        print('-----1-----')
        t2.switch() 使用greenlet协程必须要使用switch来切换
        time.sleep(0.5)
def info2():
    while True:
        print('-----2-----')
        t1.switch()
        time.sleep(0.5)
t1 = greenlet(info1) 创建一个对象
t2 = greenlet(info2)
t1.switch() 开始调用
```

### gevent协程(最大的有点就是利用延时切换任务，充分利用cpu)

```
import gevent
def f1(n):
    for i in range(n):
        print("----f1----", i)
        gevent.sleep(0.3) # 用于延时，不用time.sleep(),而是用gevent.sleep()
def f2(n):
    for i in range(n):
        print("----f2----", i)
        gevent.sleep(0.3)
def f3(n):
    for i in range(n):
        print("----f3----", i)
        gevent.sleep(0.3)
def main():
    # 使用方法一、
    t1 = gevent.spawn(f1, 5)
    t2 = gevent.spawn(f2, 5)
    t3 = gevent.spawn(f3, 5)
    t1.join()
    t2.join()
    t3.join()

    # 使用方法二、
    gevent.joinall([
        gevent.spawn(f1, 5),
        gevent.spawn(f2, 5),
        gevent.spawn(f3, 5)
    ])


if __name__ == "__main__":
    main()
  ```