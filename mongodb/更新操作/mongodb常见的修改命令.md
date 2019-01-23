# 修改

## 替换符合条件的数据

替换`student`表中`name`字段等于zhangsan的内容为`{name:'zhangsan123',age:43}`
```
db.student.update({name:'zhangsan'},{name:'zhangsan123',age:43})
```

## 修改符合条件的内容

### 修改符合一个条件的记录且只修改它的一个字段

修改`student`表中`name`等于zhangsan的记录的`age`字段为25:
```
db.student.update({name:'zhangsan'},{$set:{age:25}})
```
### 修改符合一个条件的记录且只修改它的多个个字段

修改`student`表中`name`等于zhangsan的记录的`age`字段为25,`name`字段等于`zhangsan123`.
```
db.student.update({name:'zhangsan'},{$set:{age:25,name:'zhangsan123'}})
```



# 删除

## 删除数据库

__注意__:要想删除某个数据库必须进入该数据库中.

删除`shop`数据库:
```
db.dropDatabase();
```

## 删除数据表

删除`user`表:
```
db.user.drop()
```
删除成功返回`true`,失败返回`false`.

## 删除记录

### 删除符合条件的记录

删除`user`表中`age`字段为23的记录:
```
db.user.remove({age:23})
```

### 删除符合条件记录的第一条记录

删除`user`表`age`字段为23的第一条记录:
```
db.user.remove({age:23},{justOne:true})
```