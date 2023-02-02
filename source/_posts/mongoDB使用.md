---
title: mongoDB使用
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 16:39:14
password: d5e8139b6262895c208b9fd9e7a21ecba14eb00445638566fcb77bae14408691
summary: MongoDB基础
tags: mongo
categories: db
---
# Mongodb

> - 全局说明:文档=字典=map=dict=json 
> 
> - $是mongodb的专用字符

## 安装
```shell
sudo apt-get install -y mongodb-org
```
## 启动(服务器):
```shell
sudo service mongod start
sudo service mongod stop
sudo service mongod restart
```
## 启动客户端
```shell
mongo
```



##    服务环境:
```text
mongod://localhost:27017
redis://localhost:6379
mysql://localhost:3306
nginx://localhost:8888
tomcat://localhost:8080
flask://localhost:5000
django://localhost:8000
```


## mongodb的命令
```text
show dbs: 查看有哪些数据库
show database: 查看有哪些数据库
use 数据库: 切换到某个数据库,如果没有,直接创建(如果被使用的数据库是没有的,那么也是可以用的,因为mongodb是会自动创建的,但是,如果在创建完成之后没有存入任何数据,那么这个表用完之后,也不会保存)
db: 查看当前数据库(代表当前数据库)
db.dropDatabase(): 删除当前使用的数据库

mongodb没有表的概念 数据都存储在集合当中

集合也是不需要自己创建的,可以在存入数据的时候,mongodb自动创建集合

也可以手动创建集合
db.createCollection(name, options): name 是被创建的这个集合的名字,options是这个集合的一些属性约束,用一个字典来设置
db.createCollection("userinfo",{capped:true, size:10}):表示的就是,创建一个叫做userinfo的集合,capped表示限制,默认false,无上限,size是在capped为true的时候设置,表示数据超过10字节的时候,就会被新的数据覆盖,把之前的数据往外挤

show cllections: 查看所有的集合
db.集合名.drop(): 删除给定的集合
db.集合名: 使用指定的集合
db.集合名.find(): 查询指定的集合中的数据
db.集合名.insert({"name": "lqs","age":"12"}): 往指定的集合插入数据

这是一个Object类型(文档类型,字典),那么他会自动生成一个_id:ObjectId("xxxxxxx"),保证文档的唯一性
```
### 添加数据
```text
db.集合名.insert({})
db.集合名.save({})
两者的区别:
    insert()的方式来添加数据的时候,是绝对添加,如果添加一个_id已经存在的数据,那么它会直接报错,说,如果添加一个_id已经存在
    save()的方式来添加数据的时候,是相对添加,如果添加一个_id不存在的数据,就和insert()方法一样,如果添加一个_id已经存在的数据,那么就是替换这个数据
```
###    文档插入
```text
db.集合名.insert({_id:value,key:"value",key:"value",...}): key是可以不用引起来,也可以引起,value根据具体数数据类型,字符串就引起,数字就不引起,_id如果自己不设置,系统就会自动生成
```
### 查询数据
```text
美化查询:db.集合名.find().pretty():

db.集合名.find(): 查给定集合的所有数据

db.集合名.find(条件 根据这个条件查出对应的数据 字典格式, 返回的字段): 查询出给定集合里满足条件的所有数据
db.集合名.findOne(条件 根据这个条件查出对应的数据 字典格式): 查询出给定集合里满足条件的一条数据

比较运算符
    $lt: 小于
    $lte: 小于等于
    $gt: 大于
    $gte: 大于等于
    $ne: 不等
使用例子:
    db.collection.find({age:{$lt:18}}):查询出collection集合中的年龄小于18的所有数据


范围运算符:
    $in: 范围是指定的范围给在数组中
    $nin: 不在指定的范围的
使用例子:
    db.collection.find({age:{$in:[12,45,23]}}):查询出collection集合中的年龄是12,23,45的人


逻辑运算符:
    与: 没有直接的运算符,直接写

    $or       逻辑或
    $and      逻辑与
    $not      逻辑非
    $nor      逻辑or的取反
    $exists   存在逻辑
    $type     查询键的数据类型
使用例子:
    db.collection.find({$or:[{age:{$lt:18}},{name:"sc"}]}):查询出collection中的年龄小于18或者名字等于sc的


正则:
    用/正则表达式/或者$regex:"正则表达式"编写正则表达式
使用例子:
    db.collection.find({name:/^abc/}):查询名字是abc开头的
    db.collection.find({name:{$regex:"123$"}}):查询名字是123结尾的


limit和skip:
    limit: 选中多少个
    skip: 跳过多少个
    配合使用的时候,先skip再limit
使用例子:
    db.collection.find().skip(2).limit(2):跳过两个再选中两个


自定义查询(写js):
    db.collection.find({$where:function(){return this.age>30}})
    where可以执行一个函数,this就是表示当前collection中的数据,一条一条的来过this,类似于遍历
```


#### 查询之后返回字段的过滤
```text
db.collection.find({条件},{name:1,_id:0 参数就是返回的字段,1为返回,不写不返回,除了_id,如果不想返回_id,那么_id:0})
```
#### 排序
```
db.collection.find().sort({排序字段:1, 排序字段:-1}): sort表示排序,1为升序,-1为降序
```
#### 统计个数
```
db.collection.find({条件}).count()
db.collection.count({条件})
```

#### 去重
```
db.collection.distinct("字段名字",{条件}): 返回的是一个列表,列表里面是不重复的
```
#### 聚合查询
```
管道:上层的结果给下层用
$sum:总和,统计个数,$sum:1,以一倍计数
$avg:平均
$min:最小值
$max:最大值
$push:结果文档插入到一个数组中，
$addToSet:结果文档插入到一个数组中,但不创建副本
$first:根据资源文档排序获取第一个文档数据,
$last:根据资源文档排序获取最后一个文档数据,
$limit:显示多少个
$skip:跳过多少个
$sort:排序
$project:修改输入文档的结构,如重命名\添加\删除字段,创建计算结果
$unwind:将数组类型的字段进行拆分

$match:相当于mysql中where
$group:相当于mysql中group by

db.collection.aggregate(pipeline, options):语法:pipeline,管道,必须是一个数组
格式:db.collection.aggregate([{管道:{表达式}},{管道:{表达式}}])

管道中使用字段:
    不加$:在明确的条件中字段不用加$如{name:"lqs"}:这种表示name是lqs,冒号前面不加
    加$:在不明确的使用值的时候加$如{name2:$name}:这种表示获取name的值,冒号后面加

例子:
    db.collection.aggregate([{$match:{name:"lqs"}}]):查出名字是lqs的
    db.collection.aggregate([{$match:{_id:{"$gt":2}}},{$group:{_id:$name,aavg:{$avg:$salary}}}]):查出_id大于2的所有数据,在上面查出的数据中,再按照name分组,求出salary的平均,_id表示分组字段的key,分组字段必须加上$,在这里面出现的变量多用$修饰,aavg是别名这种有多个操作的,都是衔接的,必须下面能用
    db.collection.aggregate([{$group:{_id:$name,maxM:{$max:$salary},minM:{$min:$salary}}}]):分组查出最大最小
    db.collection.aggregate([{$group:{_id:$sex,count:{$sum:1}}}]):分组统计男女生的总个数
    db.collection.aggregate([{$group:{_id:null,count:{$sum:1}}}]):表示查出这个文档的个数,_id=null的时候,表示整个文档为一组
    上面的各种结果中都一定有一个_id字段,id表示分组依据
    db.collection.aggregate([{$group:{_id:$gender,count:{$sum:1}}},{$[project:{gender:$_id,count:1,_id:0}]}]):表示创建一个gender字段,值为上一个管道的_id的值,count:1表示显示这个字段,_id:0,表示不显示这个字段,也可以使用count:$count来显示,个之前的投影一样的1\0操作

    db.collection.aggregate([{$sort:{name:1}}]):按照姓名升序排序
    db.collection.aggregate([
        {$group:{_id:$gender,count:{$sum:1}}},
        {$project:{_id:0,gender:$_id.gender,count:1}},
        {$sort:{count:1}}
    ]) 按照男女生总数升序排列
    db.cllection.aggregate([
        {$skip:2},
        {$limit:2}
    ]) 跳过两条显示两条

    原数据:{"_id":1, "name":"map", "size":[1,2,3]}
    db.collection.aggregate([{$unwind:$size}]):将上面的数据中的size对应的列表拆分成三个,这个管道只对列表有用
    拆分后:
        {"_id":1, "name":"map", "size":1}
        {"_id":1, "name":"map", "size":2}
        {"_id":1, "name":"map", "size":3}


    高级使用:
    db.collection.aggregate([
        {$group:{_id:{conutry:$conutry,prevent:$prevent,userid:$userid}}}, 将不同国家的不同省份的用户分出来userid用于去重
        {$group:{_id:{conutry:$_id.conutry,prevent:$_id.prevent},conunt:{$sum:1}}}, 在将以上分好的用国家和省份分组统计出总个数
        {$project:{_id:0,conutry:$_id.conutry,prevent:$_id.prevent,conunt:1}} 显示某个国家某个地区的总人数
    ])
```



### 更新数据
```
db.集合名.update({条件 根据这个条件查出对应的数据 字典格式},{把查到的数据更新成什么 字典格式},{multi Boolean 如果是true那么就是把查到的所有符合的数据都更新,false的话就只是更新查到的一个符合条件的数据})
    这种方式不是简单的更新,这种属于是替换
db.集合名.update({条件 根据这个条件查出对应的数据 字典格式},{$set:把查到的数据更新成什么 字典格式},{multi Boolean 如果是true那么就是把查到的所有符合的数据都更新,false的话就只是更新查到的一个符合条件的数据})
    这种方式就是更新,不会把本不该更新的键值对也替换了
```

### 删除数据
```
db.集合名.remove({条件 根据这个条件查出对应的数据 字典格式},{justOne:true})

    justOne表示是否只删除一条,如果是false,那么删除满足条件的所有数据,默认false
```
    




### 数据类型
```
Object ID: 文档ID
String: 字符串,最常用,必须是utf-8
Boolean: 存储一个布尔,true或者false
Integer: 整数,可以是32位\64位
Double: 浮点数
Arrays: 数组或者列表,多个值存入一个键
Object: 用于嵌入式的文档,即一个值为一个文档
Null: 存储Null值
Timestamp: 时间戳,表示从1970-1-1到现在的总秒数
Data: 存储当前日期或者时间的UNIX时间格式,在mongodb中可以通过new Date(日期)来使用,存入的是年月日


说明:
    文档:要存入数据库中的字典,可以是各种嵌套的字典

    每一个文档都有一个固定的属性 _id 会自动生成(12字节的十六进制数),多数时间自己设置_id,为了保证唯一性
```




### 索引
```
创建索引:
    db.collection.ensureIndex({"索引字段",排序方式1升序-1降序},{"索引字段","取得名字"}):索引的创建\取名
查询索引:
    db.collection.getIndexes():查询所有的索引
删除索引：
    db.collection.dorpIndex("索引名")
```