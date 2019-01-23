
# 插入

在进入数据库的状态下(使用`use xxx`命令).

## 命令一览

- insert
- insertOne
- insertMany

## 插入一条数据

向`user`集合中插入一条数据:
```
db.user.insert({"name":"zhangsan","age":20})
```

可能的输出:
```
WriteResult({ "nInserted" : 1 })
```

## 插入多条数据

向`user`集合中插入多个文档:
```
db.user.isnert([{"name":"zhangsan","age":20},{"name":"zhangsan","age":20}]);
```

可能的输出:
```
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
```


