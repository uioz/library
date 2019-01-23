# 索引

和其他操作一样使用索引操作的时候也需要使用`use`命令打开数据库.

## 建立一个普通索引 

给`user`表使用`username`建立索引:
```
db.user.ensureIndex({username:1})
```
__数字1表示该索引依赖升序建立,反之使用-1表示使用降序建立.__

## 建立复合索引

所谓的复合索引指的就是给一个数据表中添加字段索引.

给`user`表的`username`和`age`建立索引:
```
db.user.ensureIndex({"username":1, "age":-1}) 
```

### 复合索引的使用规则

上面我们提到使用`username`和`age`建立了索引.

但是在使用中只有:
- {username:xxx}
- {username:xxx},{age:xxx}
- {age:xxx},{username:xxx}

这种情况才会触发使用索引.

单数使用`{age:xxx}`来查找的时候是不会使用索引的.

## 命名索引

格式如下:
```
db.xxxx.ensureIndex({[索引]},{[前面的索引命名]})
```

例子:
```
db.user.ensureIndex({"username":1},{"name":"userindex"}) 
```

## 建立唯一索引

将`user`表中的`username`字段设置为唯一索引:
```
db.user.ensureIndex({username:1},{unique:true})
```
__和不同索引不同的是,唯一索引的建立需要表中一定有该字段才可以,建立普通索引不需要表中有索引约束的字段存在.__

__其次唯一索引要保证所有记录的该字段都不应该是重复的.__

## 查看索引

查看`user`表已经注册的索引:
```
db.user.getIndexes()
```

## 删除索引

### 使用键值对

删除`user`表中使用`username`字段建立的索引:
```
db.user.dropIndex({username:1})
```

## 后台创建索引

当我们有大量的数据的时候创建索引的时候,会需要等待很长时间,原因很简单这一个同步的操作,建立索引的时候可以让这个过程在后台完成而不阻塞我们当前的进程.

```
db.user.ensureIndex({username:1},{backgroud:true})
```

### 使用索引名称

__这种情况需要知道索引的名称.__

删除`user`表中索引名称为`userIndex`的索引:
```
db.user.dropIndex('userIndex')
```

