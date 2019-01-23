> The findAndModify command modifies and returns a single document. By default, the returned document does not include the modifications made on the update. To return the document with the modifications made on the update, use the new option.

findAndModify命令修改文档并且返回该单文档.默认情况下返回是query匹配到的文档而不是修改后后的文档,如果需要返回修改后的文档请使用new选项.

findAndModify命令Mongodb的命令它查询和更新结合到了一起或者叫查询后更新.类似于update的upsert选项它是原子操作.

将原本独立开来的两条语句合并到一起的好处就是避免了多线程的竟态条件,而且先查询后更新也是非常常见的需求,例如如下场景:

__获取符合规则X的ID然后查找该ID更新或者修改他的某个字段.__

这个逻辑中包含了两个操作:

1. find
2. update

在多线程的状态下很容易造成如下的问题,多个线程同时find在某个一线程update之前,然后对该id执行了多次update操作.

当然可以在update之前做一下判断或者通过其他手段来抑制或者hank这个问题,但是这不是最佳实践,一旦代码量增多,这将会是维护者的梦魇.

语法如下:

```json
{
  findAndModify: <collection-name>,
  query: <document>,
  sort: <document>,
  remove: <boolean>,
  update: <document>,
  new: <boolean>,
  fields: <document>,
  upsert: <boolean>,
  bypassDocumentValidation: <boolean>,
  writeConcern: <document>,
  collation: <document>,
  arrayFilters: <array>
}
```

| 属性名                   | 是否可选 | 描述                                                         |
| ------------------------ | -------- | ------------------------------------------------------------ |
| query                    | 是       | 标准的db.collection.find()操作,需要注意的是虽然可以查询多个文档但是最后修改的只是查询到的第一个. |
| sort                     | 是       | 当查询获取到多个文档的时候使用排序来获取第一个元素.          |
| remove                   | 否       | remove和update必须二选一,根据指定的查询来删除文档默认为false. |
| update                   | 否       | remove和update必须二选一,标准的更新操作可以运算符或者文档更新被选中的文档. |
| new                      | 是       | 如果为true则返回修改的后的文档而不是修改前的文档,删除的情况下无视这个选项,默认为false. |
| fields                   | 是       | 根据字段返回文档的子集.                                      |
| upsert                   | 是       | 和update的upsert选项作用相同,当匹配不到文档的时候就根据update中的内容创建,默认为false. |
| bypassDocumentValidation | 是       | 跳过文档验证,默认为false.                                    |
| writeConcern             | 是       | :joy:聚合操作?                                               |
| maxTimeMS                | 是       | 指定超时的最大时间.                                          |
| findAndModify            | 否       | 命令名称.                                                    |
| collation                | 是       | 为当前操作指定`collation`允许用户指定更加细致的特定语言规则用于字符比较.详细内容请查看`collation`. |
| arrayFilters             | 是       | 类似于`collation`用于定义更加细致的过滤数组的规则,详细内容请查询官方文档. |

**注意**:当在未对文档字段添加唯一索引的时高并发的情况下指定upsert字段会导致插入重复的文档.

> 纯属瞎扯:虽然findAndModify保证了原子操作但是那只是针对于Find和Update操作的原子化,并不是针对于数据的唯一.如果有两个线程同时执行了findAndModify未设置唯一索引的话会同时根据upsert设置数据内容.

__官方标准示例:__

下面的命令更新people集合中匹配查询的文档:

```javascript
db.runCommand(
   {
     findAndModify: "people",
     query: { name: "Tom", state: "active", rating: { $gt: 10 } },
     sort: { rating: 1 },
     update: { $inc: { score: 1 } }
   }
)
```

这条命令执行了如下的操作:

1. 获取所有名称为tom且状态为active且rating字段的值大于10的文档
2. 根据rating字段以升序的形式将获取到的文档进行排序
3. 获取排序后的文档集合中的第一个文档然后给它的score字段值加1

返回内容:

1. `lastErrorObject`提供了这条命令执行的细节信息
2. `value`字段返回未更改前使用query查询到的内容(使用new选项除外)

```json
{
  "lastErrorObject" : {
     "connectionId" : 1,
     "updatedExisting" : true,
     "n" : 1,
     "syncMillis" : 0,
     "writtenTo" : null,
     "err" : null,
     "ok" : 1
  },
  value" : {
    "_id" : ObjectId("54f62d2885e4be1f982b9c9c"),
    "name" : "Tom",
    "state" : "active",
    "rating" : 100,
    "score" : 5
  },
  "ok" : 1
}
```

如果查询没有匹配到任何内容则返回null:

```json
{ "value" : null, "ok" : 1 }
```

__官方使用upsert示例:__

```javascript
db.runCommand(
               {
                 findAndModify: "people",
                 query: { name: "Gus", state: "active", rating: 100 },
                 sort: { rating: 1 },
                 update: { $inc: { score: 1 } },
                 upsert: true
               }
             )
```

如果为查询到匹配的内容就插入查询的内容,返回的字段如下:

- `lastErrorObject`包含了名称操作的细节信息,提供了一个upserted字段该字段返回了插入后的内容的objectId
- `value`该字段值为null

```json
{
  "value" : null,
  "lastErrorObject" : {
     "updatedExisting" : false,
     "n" : 1,
     "upserted" : ObjectId("54f62c8bc85d4472eadea26f")
  },
  "ok" : 1
}
```

__官方同时使用upsert和new选项示例:___

如果同时使用了两个操作符例如:

```javascript
db.runCommand(
   {
     findAndModify: "people",
     query: { name: "Pascal", state: "active", rating: 25 },
     sort: { rating: 1 },
     update: { $inc: { score: 1 } },
     upsert: true,
     new: true
   }
)
```

则返回操作后的内容(匹配到内容)或者插入的内容(未匹配到内容).

返回的具体内容如下(未匹配到的):

```javascript
{
  "lastErrorObject" : {
     "connectionId" : 1,
     "updatedExisting" : false,
     "upserted" : ObjectId("54f62bbfc85d4472eadea26d"),
     "n" : 1,
     "syncMillis" : 0,
     "writtenTo" : null,
     "err" : null,
     "ok" : 1
  },
  "value" : {
     "_id" : ObjectId("54f62bbfc85d4472eadea26d"),
     "name" : "Pascal",
     "rating" : 25,
     "state" : "active",
     "score" : 1
  },
  "ok" : 1
}
```

__sort和remove联用:__

```javascript
db.runCommand(
   {
     findAndModify: "people",
     query: { state: "active" },
     sort: { rating: 1 },
     remove: true
   }
)
```

同时指定sort和remove将会删除排序后获取到的文档集合中的第一个文档.

返回结果如下:

```json
{
  "lastErrorObject" : {
     "connectionId" : 1,
     "n" : 1,
     "syncMillis" : 0,
     "writtenTo" : null,
     "err" : null,
     "ok" : 1
  },
  "value" : {
     "_id" : ObjectId("54f62a6785e4be1f982b9c9b"),
     "name" : "XYZ123",
     "score" : 1,
     "state" : "active",
     "rating" : 3
  },
  "ok" : 1
}
```

