---
title: MongoDB数据库的使用
tags:
  - python
  - mongodb
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-23 14:46:16
categories: 爬虫
pic:
---

## 准备工作

开始之前，请确保你已经安装好了`MongoDB`并启动了其服务，同时安装好了`Python`的`PyMongo`库。

安装好之后，我们需要把`MongoDB`服务启动起来。启动完成之后，它会默认在本地`localhost`的`27017`端口上运行。

接下来我们需要安装`PyMongo`这个库，它是`Python`用来操作`MongoDB`的第三方库，直接用`pip3`安装即可：

```python
pip3 install pymongo
```

## 连接MongoDB

连接`MongoDB`时，需要使用`PyMongo`库里面的`MongoClient`。一般来说，向其传入`MongoDB`的`IP`及端口即可，其中第一个参数为地址`host`，第二个参数为端口 `port`（如果不给它传递参数，则默认是`27017`）：

```python
import pymongo

client = pymongo.MongoClient(host='localhost', port=27017)
```

另外，`MongoClient`的第一个参数`host`还可以直接传入`MongoDB`的连接字符串，它以`mongodb`开头，例如：

```python
client = MongoClient('mongodb://localhost:27017/')
```

这样也可以达到同样的连接效果。

### 指定数据库

`MongoDB`中可以建立多个数据库，接下来指定操作其中一个数据库。这里以`test`数据库作为下一步需要在程序中指定使用的例子：

```python
db = client.test
```

这里调用`client`的`test`属性即可返回`test`数据库。当然，也可以这样指定：

```python
db = client['test']
```

这两种方式是等价的。

### 指定集合

`MongoDB`的每个数据库又包含许多集合`（collection）`，它们类似于关系型数据库中的表。

下一步需要指定要操作的集合，这里指定一个名称为`students`的集合。与指定数据库类似，指定集合也有两种方式：

```python
collection = db.students
```

或是

```python
collection = db['students']
```

这样便声明了一个`Collection`对象。

### 插入数据

接下来，便可以插入数据了。对`students`这个集合新建一条学生数据，这条数据以字典形式表示：

```python
student = {
    'id': '20170101',
    'name': 'Jordan',
    'age':'20',
    'gender': 'male'
}
```

新建的这条数据里指定了学生的学号、姓名、年龄和性别。直接调用`collection`的`insert`方法即可插入数据，代码如下：

```python
result = collection.insert(student)
print(result)
```

在`MongoDB`中，每条数据其实都有一个`_id`属性来唯一标识。如果没有显式指明该属性，`MongoDB`会自动产生一个`ObjectId`类型的`_id`属性。`insert()`方法会在执行后返回`_id`值。

运行结果如下：

```python
60aa06aca3aff156a9093404
```

当然，我们也可以同时插入多条数据，只需要以列表形式传递即可，示例如下：

```python
student1 = {
    'id': '20170102',
    'name': 'Jack',
    'age':'23',
    'gender': 'male'
}

student2 = {
    'id': '201701013',
    'name': 'Holy',
    'age':'25',
    'gender': 'male'
}
result = collection.insert([student1, student2])
print(result)
```

返回结果是对应的`_id`的集合：

```python
[ObjectId('60aa07d11755f43c2a5becf3'), ObjectId('60aa07d11755f43c2a5becf4')]
```

在`PyMongo`中，官方已经不推荐使用`insert`方法了。但是如果你要继续使用也没有什么问题。目前，官方推荐使用`insert_one`和`insert_many`方法来分别插入单条记录和多条记录，示例如下：

```python
student = {
    'id': '20170101',
    'name': 'Jordan',
    'age':'20',
    'gender': 'male'
}

result = collection.insert_one(student)
print(result)
print(result.inserted_id)
```

运行结果如下：

```python
<pymongo.results.InsertOneResult object at 0x00000267F7E9CD80>
60aa0884c1479c83fc0e07a6
```

与`insert`方法不同，返回的是`InsertOneResult`对象，可以调用其`inserted_id`属性获取`_id`。 对于`insert_many`方法，可以将数据以列表形式传递，示例如下：

```python
student1 = {
    'id': '20170102',
    'name': 'Jack',
    'age':'23',
    'gender': 'male'
}

student2 = {
    'id': '201701013',
    'name': 'Holy',
    'age':'25',
    'gender': 'male'
}
result = collection.insert_many([student1,student2])
print(result)
print(result.inserted_ids)
```

运行结果如下：

```python
<pymongo.results.InsertManyResult object at 0x0000021A0BA7A780>
[ObjectId('60aa0908ba9556afc2c8ebec'), ObjectId('60aa0908ba9556afc2c8ebed')]
```

该方法返回的类型是`InsertManyResult`，调用`inserted_ids`属性可以获取插入数据的`_id`列表。

### 查询数据

插入数据后，可以利用`find_one`或`find`方法进行查询，其中`find_one`查询得到的是单个结果，`find`则返回一个生成器对象。示例如下：

```python
result = collection.find_one({'name': 'Holy'})
print(result)
```

这里我们查询`name`为`Holy`的数据，它的返回结果是字典类型，运行结果如下：

查询结果：

```python
{'_id': ObjectId('60aa07d11755f43c2a5becf4'), 'id': '201701013', 'name': 'Holy', 'age': '25', 'gender': 'male'}
```

可以发现，它多了`_id`属性，这就是`MongoDB`在插入过程中自动添加的。此外，也可以根据`ObjectId`来查询，此时需要调用`bson`库里面的`objectid`：

```python
from bson.objectid import ObjectId

result = collection.find_one({'_id': ObjectId('60aa07d11755f43c2a5becf4')})
print(result)
```

查询结果：

```python
{'_id': ObjectId('60aa07d11755f43c2a5becf4'), 'id': '201701013', 'name': 'Holy', 'age': '25', 'gender': 'male'}
```

对于多条数据的查询，可以使用`find`方法。例如，这里查找性别为`male`的数据，示例如下：

```python
results = collection.find({'age': 20})
print(results)
for result in results:
    print(result)
```

运行结果如下：

```python
{'_id': ObjectId('60aa065e109b00a7eee5447b'), 'id': '20170101', 'name': 'Jordan', 'age': '20', 'gender': 'male'}
{'_id': ObjectId('60aa0686802c3967b97d7791'), 'id': '20170101', 'name': 'Jordan', 'age': '20', 'gender': 'male'}
{'_id': ObjectId('60aa06aca3aff156a9093404'), 'id': '20170101', 'name': 'Jordan', 'age': '20', 'gender': 'male'}
{'_id': ObjectId('60aa076374369a6a0801fb4a'), 'id': '20170102', 'name': 'Jack', 'age': '23', 'gender': 'male'}
{'_id': ObjectId('60aa077162e23647877e4213'), 'id': '20170102', 'name': 'Jack', 'age': '23', 'gender': 'male'}
{'_id': ObjectId('60aa07d11755f43c2a5becf3'), 'id': '20170102', 'name': 'Jack', 'age': '23', 'gender': 'male'}
{'_id': ObjectId('60aa07d11755f43c2a5becf4'), 'id': '201701013', 'name': 'Holy', 'age': '25', 'gender': 'male'}
{'_id': ObjectId('60aa0884c1479c83fc0e07a6'), 'id': '20170101', 'name': 'Jordan', 'age': '20', 'gender': 'male'}
{'_id': ObjectId('60aa08f67cd6ac92cb505552'), 'id': '20170102', 'name': 'Jack', 'age': '23', 'gender': 'male'}
{'_id': ObjectId('60aa08f67cd6ac92cb505553'), 'id': '201701013', 'name': 'Holy', 'age': '25', 'gender': 'male'}
{'_id': ObjectId('60aa0908ba9556afc2c8ebec'), 'id': '20170102', 'name': 'Jack', 'age': '23', 'gender': 'male'}
{'_id': ObjectId('60aa0908ba9556afc2c8ebed'), 'id': '201701013', 'name': 'Holy', 'age': '25', 'gender': 'male'}
```

返回结果是`Cursor`类型，它相当于一个生成器，需要遍历获取的所有结果，其中每个结果都是字典类型。

如果要查询年龄大于`20`的数据，则写法如下：

```python
results = collection.find({'age': {'$gt': 20}})
```

查询的条件键值已经不是单纯的数字了，而是一个字典，其键名为比较符号`$gt`，意思是**大于**，键值为`20`。

比较符号归纳如下：

| 符号 | 含义 | 示例 |
| :---: | :---: | :---: |
| $lt | 小于 | {'age':{'$lt':20}} |
| $gt | 大于 | {'age':{'$gt':20}} |
| $lte | 小于或等于 | {'age':{'$lte':20}} |
| $gte | 大于或等于 | {'age':{'$gte':20}} |
| $ne | 不等于 | {'age':{'$ne':20}} |
| $in | 在范围内 | {'age':{'$in':[20,23]}} |
| $nin | 不在范围内 | {'age':{'$nin':[20, 23]}} |

[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>

另外，还可以进行正则匹配查询。例如，查询名字以`H`开头的学生数据，示例如下：

```python
results = collection.find({'name': {'$regex': '^H.*'}})
```

这里使用`$regex`来指定正则匹配，`^M.*`代表以`M`开头的正则表达式。

一些功能符号归类为下表：

| 符号 | 含义 | 示例 |
| :---: | :---: | :---: |
| $regrx | 匹配正则表达式 | {'name':{'$regex':'^M.*'}} |
| $exists | 属性是否存在 | {'name':{'$exists':True}} |
| $type | 类型判断 | {'age':{'$type':'int'}} |
| $mod | 数字模操作 | {'age':{'$mod':[5,0]}} |
| $text | 文本查询 | {'$text':{'$search':'Holy'}} |
| $where | 高级条件查询 | {'$where':'obj.fans_count==obj.follows_count'} |

### 计数

要统计查询结果有多少条数据，可以调用`count`方法。以统计所有数据条数为例：

```python
count = collection.find().count()
print(count)
```

我们还可以统计符合某个条件的数据：

```python
count = collection.find({'age': 20}).count() 
print(count)
```

运行结果是一个数值，即符合条件的数据条数。

### 排序

排序时，可以直接调用`sort`方法，并在其中传入排序的字段及升降序标志。示例如下：

```python
results = collection.find().sort('name', pymongo.ASCENDING)
print([result['name'] for result in results])
```

运行结果如下：

```python
['Holy', 'Holy', 'Holy', 'Holy', 'Jack', 'Jack', 'Jack', 'Jack', 'Jack', 'Jack', 'Jordan', 'Jordan', 'Jordan', 'Jordan']
```

调用`pymongo.ASCENDING`指定升序。如果要降序排列，可以传入`pymongo.DESCENDING`。

### 偏移

只需要取某几个元素，可以利用`skip`方法偏移几个位置，比如偏移2，就代表忽略前两个元素，得到第3个及以后的元素：

```python
results = collection.find().sort('name', pymongo.ASCENDING).skip(3)
print([result['name'] for result in results])
```

运行结果如下：

```python
['Holy', 'Jack', 'Jack', 'Jack', 'Jack', 'Jack', 'Jack', 'Jordan', 'Jordan', 'Jordan', 'Jordan']
```

还可以用`limit`方法指定要取的结果个数，示例如下：

```python
results = collection.find().sort('name', pymongo.ASCENDING).limit(1)
print([result['name'] for result in results])
```

运行结果如下：

```python
['Holy']
```

值得注意的是，在数据量非常庞大的时候，比如在查询千万、亿级别的数据库时，最好不要使用大的偏移量，因为这样很可能导致内存溢出。

### 更新

数据更新，可以使用`update`方法，指定更新的条件和更新后的数据即可。例如：

```python
condition = {'name': 'Holy'}
student = collection.find_one(condition)
student['age'] = 30
result = collection.update(condition, student)
print(result)
```

更新`name`为`Holy`的数据的年龄：首先指定查询条件，然后将数据查询出来，修改年龄后调用`update`方法将原条件和修改后的数据传入。

运行结果如下：

```python
{'n': 1, 'nModified': 0, 'ok': 1.0, 'updatedExisting': True}
```

返回结果是字典形式，`ok`代表执行成功，`nModified`代表影响的数据条数。

也可以使用`$set`操作符对数据进行更新，代码如下：

```python
result = collection.update(condition, {'$set': student})
```

这样可以只更新`student`字典内存在的字段。如果原先还有其他字段，则不会更新，也不会删除。而如果不用`$set`的话，则会把之前的数据全部用`student`字典替换；如果原本存在其他字段，则会被删除。

`update`方法其实也是官方不推荐使用的方法。这里也分为`update_one`方法和`update_many`方法，用法更加严格，它们的第2个参数需要使用`$`类型操作符作为字典的键名，示例如下:

```python
condition = {'name': 'Kevin'} 
student = collection.find_one(condition)
student['age'] = 26
result = collection.update_one(condition, {'$set': student})
print(result) print(result.matched_count, result.modified_count)
```

上面的例子中调用了`update_one`方法，使得第2个参数不能再直接传入修改后的字典，而是需要使用`{'$set':student}`这样的形式，其返回结果是`UpdateResult`类型。然后分别调用`matched_count`和`modified_count`属性，可以获得匹配的数据条数和影响的数据条数。

运行结果如下：

```python
<pymongo.results.UpdateResult object at 0x10d17b678>
1 0
```

## 删除

删除操作比较简单，直接调用`remove`方法指定删除的条件即可，此时符合条件的所有数据均会被删除。

示例如下：

```python
result = collection.remove({'name': 'Kevin'})
print(result)
```

运行结果如下：

```python
{'ok': 1, 'n': 1}
```

另外，这里依然存在两个新的推荐方法 ——`delete_one`和`delete_many`，示例如下：

```python
result = collection.delete_one({'name': 'Kevin'})
print(result) 
print(result.deleted_count)
result = collection.delete_many({'age': {'$lt': 25}})
print(result.deleted_count)
```

运行结果如下：

```python
<pymongo.results.DeleteResult object at 0x10e6ba4c8> 
14
```

`delete_one`删除第一条符合条件的数据，`delete_many`即删除所有符合条件的数据。它们的返回结果都是`DeleteResult`类型，可以调用`deleted_count`属性获取删除的数据条数。
