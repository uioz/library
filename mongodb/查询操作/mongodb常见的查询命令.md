# 查询

## 查询数据库

```
show dbs
```

可能的输出:
```
admin   0.000GB
config  0.000GB
local   0.000GB
```

## 查询数据表

在进入数据库的状态下(使用`use xxx`命令).

显示数据表命令:
```
show collections
```

## 查询记录

### 查询所有的记录

在进入数据库的状态下(使用`use xxx`命令).

查询`user`表中所有的记录:
```
db.user.find()
```

### 查询匹配条件的记录

在进入数据库的状态下(使用`use xxx`命令).

查询`user`表中年龄为12的记录:
```
db.user.find({age:12})
```

### 查询时进行数值比较

在进入数据库的状态下(使用`use xxx`命令).

#### 大于某个数值

查询`user`表中年龄大于12的记录:

```
db.user.find({age:{$gt:12}})
```

#### 小于某个数值

查询`user`表中年龄小于12的记录:

```
db.user.find({age:{$lt:12}})
```
#### 大于等于某个数值

查询`user`表中年龄大于等于12的记录:

```
db.user.find({age:{$gte:12}})
```
#### 小于等于某个数值

查询`user`表中年龄小于等于12的记录:
```
db.user.find({age:{$lte:12}})
```

#### 不等于某个数值

查询`user`表中年龄不等于12的记录:
```
db.user.find({age:{$ne:12}})
```

> https://blog.csdn.net/u013457382/article/details/50855607

#### 在某个数值区间内

查询`user`表中年龄大于等于12且小于等于22的记录:
```
db.user.find({age:{$lte:22,$gte:12}})
```

### 多条件查询AND查询

查找姓名为`hello`且年龄为`22`的记录:
```
db.user.find({name:'hello',age:22})
```

### 多条件查询OR查询

查找`age`为22或者`age`为12的记录:
```
db.user.find({$or:[{age:22},{age:12}]})
```

### 在查询中使用正则表达式

#### 模糊搜索

查询`article`表中`content`字段含有`hello world`的所有的记录:
```
db.article.find({content:/hello world/})
```

#### 查询以某个字符开头的

查询`article`表中`title`字段以`hello`开头的所有的记录:
```
db.article.find({content:/^hello world/})
```

### 查询第一条数据

查询`user`中所有结果的第一条记录:
```
db.user.findOne()
```

### 过滤输出结果列

#### 只显示某一列的数据

查询`user`表所有的数据但是只显示`name`字段的信息:
```
db.user.find({},{name:1})
```
__此处的`name:1`中的1(相当于Javascript中的True)代表现实这个字段,如果为0(代表JavaScript中的False)则排除这个字段__.

> https://www.cnblogs.com/wuxiang/p/5553658.html

#### 指定列显示

查询`user`表所有的数据但是只显示`name`字段和`age`字段的信息:
```
db.user.find({},{name:1,age:1})
```

#### 使用条件查询后再过滤输出信息

查询`user`表中`age`大于等于20的用户的`name`字段:
```
db.user.find({age:{$gte:20}},{name:1})
```



### 查询后对于结果使用排序

#### 升序查询

查询`user`表所有用户的信息,使用`age`字段排序后输出:
```
db.user.find().sort({age:1})
```
#### 降序查询

查询`user`表所有用户的信息,使用`age`字段排序后输出:
```
db.user.find().sort({age:-1})
```

__在`sort`中1代表升序而-1代表降序.__


### 查询后对结果进行分页处理

#### 输出前n条数据

查询`user`表所有用户的信息,输出前两条:
```
db.user.find().limit(2)
```

#### 跳过n条结果输出

查询`user`表所有用户的信息,从第二条后输出:
```
db.user.find().skip(2)
```

#### 获取指定区间内的结果

查询`user`表所有用户的信息,从第二条后输出一条记录:
```
db.user.find().skip(2).limit(1)
```

# 查询额外信息

## executionStats信息

该指令是`explain`方法的字符串参数,利用他我们可以获取一次查询中的各种用于描述查询本身的信息,利用他提供的一次查询中所使用的时间我们可以用于优化查询.

查看查询一次`user`表提供的额外信息:
```
db.user.find().explain('executionStats')
```
