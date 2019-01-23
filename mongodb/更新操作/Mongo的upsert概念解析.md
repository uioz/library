#  标题 upsert操作

upsert操作是 update+insert的简称,具体执行的逻辑如下:

1. 使用查询条件进行查询
2. 获取查询后的内容进行更新
3. 如果未获取到内容执行insert操作

insert操作会将所有匹配的内容替换成插入后的内容,在update中这不是我们想要见到的.

update操作可以指定更新的字段,但是insert不可以,所以引出如下的问题:

1. 不仅可以插入记录还要和更新操作不冲突
2. 已经存在的内容不修改

试想一下我们有多个结构相同的文档,我们所期待的是使用upsert操作会修改原有字段上的值,如果没有就创建他,或者针对已有的内容不操作,只对于不存在的记录插入这个字段.

要想实现第一个的功能需要配合使用`$set`运算符,第二个操作需要使用`$setOnInsert`运算符.

现在我们来模拟一下这种情况,首先创建一个空集合,然后执行如下的命令:

```javascript
db.products.update(
  { _id: 1 },
  {
     $set: { item: "apple" },
     $setOnInsert: { defaultQty: 100 }
  },
  { upsert: true }
)
```

输出

```json
{ "_id" : 1, "defaultQty" : 100, "item" : "apple" }
```

修改之前的命令继续执行:

````javascript
db.products.update(
  { _id: 1 },
  {
     $set: { item: "banana" },
     $setOnInsert: { defaultQty: 123 }
  },
  { upsert: true }
)
````

输出:

```json
{ "_id" : 1, "defaultQty" : 100, "item" : "banana" }
```

**注意**:实际上一个upsert操作需要进行考量操作,一旦传入了一个数据就会变成一个彻头彻尾的insert操作原有的数据就不会存在了.

例如接着使用上面例子这次我们传入如下的命令:

```javascript
db.products.update(
  { _id: 1 },
  {
     sex:'male'
  },
  { upsert: true }
)
```

原有的数据就会变为`{ "_id" : 1, "sex" : "male" }`了.

所以在upsert为true的情况下,不允许两者(运算符和字段)同时存在,否则会报错.