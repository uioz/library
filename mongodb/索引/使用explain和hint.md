# 使用explain和hint

> - Deprecated in the `mongo` Shell since v3.0
>
>   Starting in v3.2, the [`$explain`](https://docs.mongodb.com/manual/reference/operator/meta/explain/#metaOp._S_explain) operator is deprecated in the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell. In the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell, use [`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain) or [`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) instead.

## 简介

- explain用于分析操作
- hint用于强制使用指定的索引

根据官方文档`explain`的使用方式应该是这个样子的:

```
db.collection.explain().<method(...)>
```

## explain

最常见的`explain`输出有两种类型:

1. 使用索引
2. 未使用索引

参数(开启多输出):

1.  `queryPlanner` - default
2. `executionStats`
3. `allPlansExecution`
4. 为了向后兼容,任何true值将被视为`allPlansExecution`反之则视为`executionStats`

一个常见的查询计划如下:

```json
"executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 999999,
                "executionTimeMillis" : 539,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 999999,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "nReturned" : 999999,
                        "executionTimeMillisEstimate" : 465,
                        "works" : 1000001,
                        "advanced" : 999999,
                        "needTime" : 1,
                        "needYield" : 0,
                        "saveState" : 7825,
                        "restoreState" : 7825,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "direction" : "forward",
                        "docsExamined" : 999999
                }
        },
```

知识点:

- `executionTimeMillis`执行查询总的花费时间(并非实际时间)
- `nReturned`返回文档的数量(不能表示查询的次数)
- `totalKeysExamined`索引扫描条目
- `totalDocsExamined`文档扫描条目

## hint

向hint传入`{$natural:1}`可以强制让数据库做全表扫描.

副作用:

- 返回的结果是按照磁盘顺序排列的,对于活跃的集合这没有意义MongoDB会移动文档在磁盘上

建议使用:

- 对于只插入的文档
  - 依赖磁盘顺序
- 结果集相对于原数据比例较大的时候

TODO 略过这些内容,记得firefox上收藏的那篇文件文章

