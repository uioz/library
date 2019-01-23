
本文参考链接:
> http://mongodb.github.io/node-mongodb-native/3.1/tutorials/crud/#findoneandupdate-findoneanddelete-and-findoneandreplace

# 安装

## 安装mongodb

基本操作`npm install mongodb --save`

## 启动mongodb服务器

运行示例:
```
const mongodbConnect = require('mongodb');

// 服务器地址
const mongodbServerUrl = 'mongodb://localhost:27017';

// 数据库名称
const mongodbName = 'demo';

mongodbConnect.connect(mongodbServerUrl,(error,client)=>{

    if(error){
        return console.log(error);
    }

    const database = client.db(mongodbName);

    console.log('Connected succssfully to server');

    client.close();
});
```

## 补充说明

1. 实际上mongodb API有两个版本,一个是传统的回调版本,这也是接下来的例子中使用的,另外一个是Promise版本.
2. 当直接使用`require('mongodb').connect()`方法的时候实际上使用的是`require('mongodb').MongoClient.connect()`方法,而`MongoClient`对象接受一个url,所以我们可以同时创建多个client对象来连接多个不同的mongo服务器.
```
const mongodbConnect = require('mongodb');

const mongodbServerUrl = 'mongodb://localhost:27017';

const mongodbName = 'demo';

// 这个例子中的client是一个对象,这意味着你可以创建多个客户端对象连接多个mongo服务端程序
const client = new mongodbConnect.MongoClient(mongodbServerUrl);

client.connect(function (error) {

    if(error){
        return console.log(error);
    }

    const db = client.db(mongodbName);


    client.close();
});
```


# 使用

虽然`mongo`命令行中使用的是JavaScript语法但是和Node给出的接口还是有很大区别的.

首先我们回顾一下mongo中的基本三种操作:
- 使用数据库
- 使用数据库表
- 对于表使用命令

与其对应的对象与方法就是如下的几个:
- 数据库对象
- 数据表对象
- 数据表上的方法

1. 获取数据库对象:
```
mongodbConnect.connect(mongodbServerUrl,(error,client)=>{
    
    const mongodbName = 'demo';
    
    // 等同于 use demo
    const database = client.db(mongodbName);
    
}
```

2. 获取数据表对象

回想一下在mongodb中显示所有的数据表的命令是什么?
```
show collections
```

没错接口就是数据库对象上的`collection`方法.

获取数据表对象:
```
// 参数是使用的数据库名称,返回的就是数据库对象
const database = client.db(mongodbName);

// 等于 db.user
const userTable = database.collection('user');
```

3. 数据表上的方法

再次回想一下mongodb中针对于数据表的操作`find``insert``delete``update`等.

这些方法直接挂载到了数据表对象上零学习成本.

常见的操作都在数据表对象上:
```
const userTable = database.collection('user');

userTable.find();
userTable.insertOne();
userTable.insertMany();
userTable.update();
userTalbe.updateMany();
userTable.delete();
// 等同于 db.user.find(),db.user.insertOne(),....
```


### client.close()

额外需要注意的就是`client.close()`方法它一旦调用后这个连接将会被终止掉.

如果你操作做完后记得调用这个方法,否则进程是不会终止的.

## 常见的CRUD操作

## 插入一条数据

```
const mongodbConnect = require('mongodb');

const mongodbServerUrl = 'mongodb://localhost:27017';

const mongodbName = 'demo';

mongodbConnect.connect(mongodbServerUrl, (error, client) => {

    if (error) {
        return console.log(error);
    }

    console.log('Connected succssfully to server');

    const database = client.db(mongodbName);

    const table = database.collection('user');

    table.insertOne(
        {
            name: 'gameover',
            age: 22
        },
        function (error, result) {

            if (error) {
                return console.log(error);
            }

            console.log(result);

            client.close();
        }
    )

});
```

__等同于`db.user.insertOne({name: 'gameover',age: 22})`__

由于数据的操作是异步的,所以有一个回调钩子用于通知--这个操作是否有异常(第一个参数)和命令执行完成的状态(第二个参数).

**注意**:客户端使用完成后要关闭连接,需要调用`close`方法.

## 更新一条数据

```
const mongodbConnect = require('mongodb');

const mongodbServerUrl = 'mongodb://localhost:27017';

const mongodbName = 'demo';

const client = new mongodbConnect.MongoClient(mongodbServerUrl);

client.connect(function (error) {

    if (error) {
        return console.log(error);
    }

    const db = client.db(mongodbName);

    const table = db.collection('user');
    
    // 更新一条数据
    table.updateOne({name: 'gameover'}, {$set:{age:23}}, function (error,{result}) {

        if(error){
            return console.log(error);
        }

        console.log(result);

        client.close();
    })
    
});
```

__等同于`db.user.updateOne({name: 'gameover'}, {$set:{age:23}})`.__

## 删除一条数据

```
const mongodbConnect = require('mongodb');

const mongodbServerUrl = 'mongodb://localhost:27017';

const mongodbName = 'demo';

const client = new mongodbConnect.MongoClient(mongodbServerUrl);

client.connect(function (error) {

    if (error) {
        return console.log(error);
    }

    const db = client.db(mongodbName);

    const table = db.collection('user');

    table.deleteOne({age:23},function (error,result) {

        if(error){
            return console.log(error);
        }

        console.log(result.result);

    });

});
```

__等同于`db.user.deleteOne({age:23})`.__

## find操作

之所以将`find`最后提到是因为他的操作和shell中的表现完全不一致.

如果你学过java操作过JDBC你你应该了解JDBC获取数据的时候返回的是一个结果集对象,结果集是一个对象,使用结果集的方法来获取多个字段或者单个字段中的内容.

和JDBC相似的是,在mongo中使用`find`操作后返回的也是一个对象,他提供一系列的操作,允许你更加细化的获取数据.

### 基础操作

官方文档中有如下的描述:
> find方法返回一个游标对象用户可以利用它来操作数据.这个游标对象实现了node的stream接口,允许用户将数据以流的方式传递给其他的流.

获取表中的所有数据:
```
const mongodbConnect = require('mongodb');

const mongodbServerUrl = 'mongodb://localhost:27017';

const mongodbName = 'demo';

const client = new mongodbConnect.MongoClient(mongodbServerUrl);

client.connect(function (error) {

    if (error) {
        return console.log(error);
    }

    const db = client.db(mongodbName);

    const table = db.collection('user');

    table.find().toArray(function (error,docs) {

        if(error){
            return console.log(error);
        }

        console.log(docs);

        client.close();
    })

});
```
你可以看到和之前API不同的是,我们调用了`find`的`toArray`方法,原因很简单在没有调用`toArray`方法以前我们没有获取实际的数据.

find允许你在后面做更多的限定和过滤操作,例如`skip`和`limit`方法用于进一步缩小可能捕获到的结果集.

如果不这样做,我们直接使用`toArray`方法将数据库中所有的内容都读取出来的话,这些数据都将会保存在内存中,这可不是个好主意.

我的意思是你应该知道你在做什么,不要把没有意义的数据拿出来,在调用`toArray`前我们并没有获取任何数据,所以调用`find`的时候我们需要尽可能的缩小捕获的数据范围,以用于节省内存.

也就是说find的操作分为三个步骤:
1. 调用find方法
2. 执行过滤或者其他操作(可选)
3. 调用真正获取内容的方法

用于真正获取内容的方法有三个:
1. toArray 获取所有结果
2. next 获取一个结果
3. each 以迭代的方式获取数据

中间可用的操作如下:
```
collection.find({}).project({a:1})                             // Create a projection of field a
collection.find({}).skip(1).limit(10)                          // Skip 1 and limit 10
collection.find({}).batchSize(5)                               // Set batchSize on cursor to 5
collection.find({}).filter({a:1})                              // Set query on the cursor
collection.find({}).comment('add a comment')                   // Add a comment to the query, allowing to correlate queries
collection.find({}).addCursorFlag('tailable', true)            // Set cursor as tailable
collection.find({}).addCursorFlag('oplogReplay', true)         // Set cursor as oplogReplay
collection.find({}).addCursorFlag('noCursorTimeout', true)     // Set cursor as noCursorTimeout
collection.find({}).addCursorFlag('awaitData', true)           // Set cursor as awaitData
collection.find({}).addCursorFlag('exhaust', true)             // Set cursor as exhaust
collection.find({}).addCursorFlag('partial', true)             // Set cursor as partial
collection.find({}).addQueryModifier('$orderby', {a:1})        // Set $orderby {a:1}
collection.find({}).max(10)                                    // Set the cursor max
collection.find({}).maxTimeMS(1000)                            // Set the cursor maxTimeMS
collection.find({}).min(100)                                   // Set the cursor min
collection.find({}).returnKey(10)                              // Set the cursor returnKey
collection.find({}).setReadPreference(ReadPreference.PRIMARY)  // Set the cursor readPreference
collection.find({}).showRecordId(true)                         // Set the cursor showRecordId
collection.find({}).sort([['a', 1]])                           // Sets the sort order of the cursor query
collection.find({}).hint('a_1')                                // Set the cursor hint
```

next操作小样:
```
col.find({a:1}).next(function(err, doc) {
    // doc == 单个内容
    client.close();
});
```

each操作小样:
```
table.find().each(function (error, docs) {

    if(docs){

        console.log(docs);

    }else{
        client.close();
        return false;
    }

})
```

### 批量操作(bulkWrite)

该操作允许你将多个不同的没有规则的操作合并起来然后一次性的交由服务器操作.

该操作的基本要素就是获取`col`对象后然后调用他的`bulkWrite`方法:
```
client.connect(function (error) {

    if (error) {
        return console.log(error);
    }

    const db = client.db(mongodbName);

    const table = db.collection('user');

    table.bulkWrite([
        {
            insertOne: {
                name: "zxb",
                age: 20
            }
        },
        {
            updateOne: {
                filter: { // 条件此处为命名为filter
                    age: 20
                },
                update: {
                    $set: {
                        age: 21
                    }
                }
            },
            upsert:true // 匹配到内容就更新反之就插入
        },


    ],function (err,result) {

        console.log(result);

        client.close();
    });

});
```


### 原子性操作

所谓的原子性操作指的是,针对于同一个内容的修改的时候出现多个线程同时操作这个数据是不被允许的.换句话说在数据库中的一条数据在同一时刻只能有一个线程对其操作.

当然这样做会降低性能,但是在类似于财务管理的系统中付出这样的代价是很有必要的.

该操作的三要素就是`findOneAndUpdate`,`findOneAndDelete`,`findOneAndReplace`.
```
// 更新文档并且获取返回原生结果
table.findOneAndUpdate({name:'zxb'}, {$set: {age:20}}, {
    returnOriginal: false
    , sort: [['age',1]]
    , upsert: true
}, function(err, result) {
    console.log(result);

    // 删除获取结果
    table.findOneAndDelete({a:2}, function(err, result) {

        console.log(result);

        client.close();
    });
});
```

输出:
```
{ lastErrorObject: { n: 1, updatedExisting: true },
  value: { _id: 5bf21a78b4b6cf150c7f688e, name: 'zxb', age: 20 },
  ok: 1 }
{ lastErrorObject: { n: 0 }, value: null, ok: 1 }
```

## 集合操作


