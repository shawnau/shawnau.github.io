# MongoDB+pymongo简介(持续更新中)


MongoDB 是一种文件导向的 NoSQL 数据库，由 C++ 撰写而成。 

<!--more-->

## MongoDB简介

 - 什么是NoSQL? 看看[这篇zhihu问题](https://www.zhihu.com/question/20059632)足矣. 我的粗浅理解就是NoSQL和SQL不同的地方是, SQL需要定义好一行的标签, 每行数据的维度都是固定的, 格式也是固定的,非常严格. 而NoSQL每个单元的数据都是长短可变的, 甚至每个key对应的value也都是可变的
 - 为什么用MongoDB? 对我而言只是因为他的document文档格式和python的dict以及json格式无缝衔接, 数据单元可伸缩性好, 入门门槛极低. 非常适合小规模的web开发.

## mongo shell 简介

[官方文档](https://docs.mongodb.com/getting-started/shell/)

 - mongo shell是个数据库控制台, 拿mysql简单理解就是登录mysql之后出现的控制台, 可以进行各种数据库操作, mongodb也一样有他自己的控制台, 即mongo shell.

 - 安装见[官方文档](https://docs.mongodb.com/getting-started/shell/installation/), 这里不多说了(有时间再补充吧)

### 0. mongodbd的数据结构

```
database -> collection1 -> document1
                        -> document2
                        -> document3
                        -> ...
         
         -> collection2 -> document1
                        -> document2
                        -> document3
                        -> ...
         -> ...
```

 - mongodb中的最小存储单元是文档(document), 文档组成集合(collection), 集合再组成数据库(database), 其中数据库和集合是可以命名的.
 - document的内容类似于JSON, 或者pyhton中的字典(dict), 由`field: value`的键值对组成.例子如下. 详细信息请参考[官方文档关于document1的说明](https://docs.mongodb.com/manual/core/document/)

```
{ 
    "_id": "oranges", 
    "qty": { "in stock": 8, "ordered": 12 },
    "tag": "food"
}
```

### 1. 启动/关闭mongod和mongo(OSX系统下,linux类似)

```bash
$ mongod           # 方法1: 启动mongod, 切换至另一个终端开启控制台(因为这个终端会用作打log信息)
$ mongod --fork --logpath /var/log/mongod.log # 方法2: 静默启动, 把log信息存入/var/log/mongod.log下
$ mongo <database> # 连接数据库database, 进入mongo shell
$ mongo --eval "db.getSiblingDB('admin').shutdownServer()" # 关闭数据库
$ sudo service mongod start # linux下的启动指令
```
 - `mongod`指令启动数据库, `mongo`指令登录数据库控制台. 这么说就好理解了, 所以要开两个终端, 要不就用第二种静默启动, 把log保存在文件里的方法继续在同一个终端下工作.

### 2. mongo shell下的基本操作

```bash
> db               # 显示现在使用的数据库
> show dbs         # 查看数据库
> use <database>   # 切换数据库
> show collections # 查看collections
> db.myCollection.insert( { x: 1 } ); # 新建一个名为myCollection的collection, 插入一个document. 注意末尾的分号

> db["3test"].find() # 使用find列出名为3test的collection的文档
> db.getCollection("3test").find().pretty() # 使用pretty优化输出
> quit()             # 退出, 也可以按CTRL+C
```

### 3. MongoDB 的CRUD操作

CURD: Create, Read, Update, Delete

1. 创建/插入文档
 - `db.collection.insertOne( { x: 1 } )`: 创建collection
 - `db.collection.insertMany( [{ x: 1 }, { x: 2 }] )`: 插入两个文档, 表示方式类似于python中的数组
 - `db.collection.insert()`: 以上两种方法都集成到了`insert()`中

2. 查询文档
 - `db.collection.find( { field1: <value>, field2: <value> ... } )`: 查询符合键/值对的文档
 - `db.collection.find().limit(5)`: 利用cursor modifier限制搜索范围为5个结果

3. 更新文档
 - `db.collection.update(<filter>, <update/replacment>, <options>)`
 - mongodb提供了更新操作符(update operator)来更新文档, 格式如下:
 ```
 {
    <operator1>: { <field1>: <value1>, ... },
    <operator2>: { <field2>: <value2>, ... },
    ...
 }
 ```
 - 操作符介绍

name          | function
--------------|-------------
$inc          | 增加值
$mul          | 乘以值
$rename       | 重命名
$setOnInsert  | 如果更新导致插入一个文档, 则set某个值. 如果更新只是修改某个已经存在的文档则不起作用
$set          | set文档的某个值
$unset        | 移除文档的某个field
$min          | 如果指定的值比已存在的值小, 就更新该值
$max          | 如果指定的值比已存在的值大, 就更新该值
$currentDate  | 将某个field的值设定为日期
 

 - 其他操作符见[官方文档](https://docs.mongodb.com/manual/reference/operator/update/)
 - 一个更新的例子
 
 ```
 db.inventory.update(
    { item: "paper" },
    {
      $set: { "size.uom": "cm", status: "P" },
      $currentDate: { lastModified: true }
    }
 )
 ```
  - 其中第一个dict是过滤器`<filter>`, 即需要更新的文档是满足item键的值为paper的第一个文档
  - 第二个dict为`<update/replacment>`, 使用`$set`操作符更新key为`"size.uom"`的值为`"cm"`, 更新key为`status`的值为`"p"`, 使用`$currentDate`更新key为`lastModified`的值为当前时间, 如果没有这个key就创建一个key

4. 删除文档
 ```
 db.collection.remove(
    <query>,
    <justOne>
 )
 ```
 - 第一个参数就是用来搜索符合条件的文档的, 例如`{x: 1}`会删除所有符合`{x: 1}`的文档, 第二个参数可选, 是设定只删除第一个`true`还是删除所有符合条件的文档的`false`
 - 删除collection使用`db.<collectionName>.drop()`指令
 - 删除数据库使用`db.dropDatabase();`(前提是在该数据库下, 即先使用`use <dbName>;`)
 - 或者直接在终端使用`mongo <dbname> --eval "db.dropDatabase()"`

---

## pymongo简介

pymongo是使用python操作mongodb的包, [官方文档](http://api.mongodb.com/python/current/)

### 1. 建立数据库和collection
```python
from pymongo import MongoClient
client = MongoClient('localhost', 27017)
```
 - 连接数据库, 省略参数之后会使用默认host和port, 如例子所示

```python
db = client.test_database
db = client['test-database']
```
 - 以上两句是等价的, 创建了一个名为`test_database`的数据库

```python
collection = db.test_collection
collection = db['test-collection']
```
 - 以上两句是等价的, 在`test_database`中创建了一个叫`test_collection`的collection. 
 - collection是数据库中的一组文档. 文档(document)是mongodb中数据的最小单位, 类似于python中的dict, 或者JSON格式

### 2. 插入一个document

```python
import datetime
post = {"author": "Mike",
        "text": "My first blog post!",
        "tags": ["mongodb", "python", "pymongo"],
        "date": datetime.datetime.utcnow()}

posts = db.posts
post_id = posts.insert_one(post).inserted_id
```
 - 以上语句封装了一个JSON格式的post, 并使用`insert_one()`方法插入了名为`posts`的collection.
 - 其中`post`中的`datetime.datetime.utcnow()`会被自动转换成`BSON`格式
 - 插入成功之后会返回一个`inserted_id`, 对于每个document是独一无二的.
 - 对于每个插入的document, mongodb会自动为其添加一个名为`_id`的key, 其value也为`inserted_id`

### 3. 使用`find_one()`查询文档

```python
>>> posts.find_one()
>>> {u'_id': ObjectId('...'),
... u'author': u'Mike',
... u'date': datetime.datetime(...),
... u'tags': [u'mongodb', u'python', u'pymongo'],
... u'text': u'My first blog post!'}
```
 - 直接调用`find_one()`会返回`posts`的第一个文档

```python
posts.find_one({"author": "Mike"})
posts.find_one({"_id": post_id})
```
 - 定制搜索条件. 其中如果按照`_id`搜索的话, 必须传入`ObjectId`类而不是`str`类, 比如例子中`post_id`为第2节中返回的`ObjectId()`
 - 另外, 如果想要自己初始化一个`_id`可以这么做:

```python
from bson.objectid import ObjectId
document = client.db.collection.find_one({'_id': ObjectId(post_id)})
```

### 4. 使用`find()`查询多个文档

```python
for post in posts.find():
   pprint.pprint(post)
```

 - `find()`返回一个`Cursor`实例, 它是一个可遍历实例, 可以像list一样进行遍历
 - 参数使用和`find_one()`类似, 可以定制搜索条件

### 5. 编码问题

> MongoDB stores data in BSON format. BSON strings are UTF-8 encoded so PyMongo must ensure that any strings it stores contain only valid UTF-8 data. Regular strings (<type ‘str’>) are validated and stored unaltered. Unicode strings (<type ‘unicode’>) are encoded UTF-8 first. The reason our example string is represented in the Python shell as u’Mike’ instead of ‘Mike’ is that PyMongo decodes each BSON string to a Python unicode string, not a regular str.

简单说就是mongodb会把Unicode自动转换成UTF-8存储, 对于已经encode的`str`则不作处理

### 6. 使用`count()`计数

```python
>>> posts.count()
3

>>> posts.find({"author": "Mike"}).count()
2
```

### 7. 高级搜索

```python
d = datetime.datetime(2009, 11, 12, 12)
for post in posts.find({"date": {"$lt": d}}).sort("author"):
    pprint.pprint(post)
```
 - 此处使用`$lt`搜索出了所有在`d`之前的所有posts, 并按照`"author"`关键词升序排列

### 8. 创建自己的编号(index)

```python
>>> result = db.profiles.create_index([('user_id', pymongo.ASCENDING)],
...                                   unique=True)
>>> sorted(list(db.profiles.index_information()))
[u'_id_', u'user_id_1']
```
 - 用`index_information()`查询一下, 发现现在我们有了两个index, 一个是插入文档时自创的`_id`, 还有一个是刚刚创建的`user_id`

```python
>>> user_profiles = [
...     {'user_id': 211, 'name': 'Luke'},
...     {'user_id': 212, 'name': 'Ziltoid'}]
>>> result = db.profiles.insert_many(user_profiles)
```
 - 使用`insert_many()`方法插入两个文档之后,再试试插入拥有相同`user_id`的文档试试

```python
>>> new_profile = {'user_id': 213, 'name': 'Drew'} 
>>> duplicate_profile = {'user_id': 212, 'name': 'Tommy'}
>>> result = db.profiles.insert_one(new_profile)  # 插入编号213, ok
>>> result = db.profiles.insert_one(duplicate_profile)  # 插入编号212, 已经存在, 抛出DuplicateKeyError
Traceback (most recent call last):
DuplicateKeyError: E11000 duplicate key error index: test_database.profiles.$user_id_1 dup key: { : 212 }
```

Refer:
https://docs.mongodb.com/manual/mongo/
https://docs.mongodb.com/manual/crud/
https://api.mongodb.com/python/current/tutorial.html
