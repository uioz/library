# 标题 mongoDb 查询

mongodb中使用db.collection.find()来进行数据的查询,默认情况下:

```
db.collection.find() == db.collection.find({}) == 全部内容
```

而内部的所有内容都是条件:

```
db.collection.find({字段:条件 [,and 字段:条件 ...] })
```

## 指定需要返回的键

find和findOne的第二个参数可以用于指定返回的键,但是需要注意的是objectId总是会被返回.

```javascript
db.tester.find({},{"priority":1})
```

但是可以使用这种方式将ID给去掉:

```javascript
db.tester.find({},{priority:1,_id:0})
```

# 查询条件

## 比较操作符

| 操作符 | 含义 |
| ------ | ---- |
| $lt    | <    |
| $lte   | <=   |
| $gt    | >    |
| $gte   | >=   |

例子:

```javascript
db.users.find({age:{$gte:18,$lte:30}})
```

### 不等操作符

$ne可以用于所有类型的数据,他表示查询不相等:

```javascript
db.users.find({username:{$ne:'joe'}})
```

## OR查询

两种方式:

- $in 一个键的多个值
- $or 多个键查询任意给定值

**提示**:与$in相反的操作符是$nin.

例子$in:

```javascript
db.tester.find({priority:{$in:[1]}})
```

例子$nin:

```javascript
db.tester.find({priority:{$nin:[1]}})
```

和$in以及$nin针对字段操作不同的是$or针对于条件进行操作,例如我们想查找优先级不为1或者状态为Runing的文档:

```javascript
db.tester.find({$or:[{priority:{$nin:[1]}},{status:'RUNNING'}]})
```

## Not查询

和$or一样$not查询是针对于条件操作,此外它有又被称为元条件句,可以用在其他条件之上.

这里直接照搬Mongodb权威指南中的介绍:

> 就拿取模运算符$mod来说$mod会将查询的值除以第一个给定值,若余数等于第二个给定的值则匹配成功:
>
> ```javascript
> db.users.find({num:{$mod:[5,1]}})
> ```
>
> 上面的查询会发那会的值为1,6,11,16等用户.但要是想返回为2,3,4,5,7...的用户就要使用$not了.
>
> ```javascript
> db.users.find({num:{$not:{$mod:[5,1]}}})
> ```

## 条件语义

在不同环境下使用操作符有不用的效果,但是在比对过update中的运算符和查询语句中的条件语句后:

- `db.tester.find({age:{$lt:30}})` -- 在内部
- `db.tester.update({},{$inc:{age:1}})` -- 在外层

总结:

- 在文档内部放置的是条件语句
- 在文档外部放置的是运算符

例外:

有一些元操作符(meta-operator)也位于外层文档中例如:

- $and
- $or
- $nor

```javascript
db.user.find({$and:[{x:{$lt:1},{x:4}}]})
```

**注意**:上面的条件似乎是自相矛盾的但是实际上并不是,例如x的值为[0,4]就可以通过验证,但是由于x是数组所以这里使用下方的操作效率会更高一些:

```javascript
db.users.find({x:{$lt:1,$in:[4]}})
```

# 特定类型的查询

## null

null的表现非常奇怪,首先他可以被查询到:

```javascript
db.tester.find({x:null})
```

这会匹配x键为null的文档,但是也会匹配没有该键的所有记录例如:

```javascript
db.tester.find({y:null})
```

如果此时没有y字段的存在就会将所有没有该字段的文档全部匹配上.

## 正则表达式

## 查询数组

数组的查询也很有特色,由于mongodb中的文档是不规则的,所以导致数组查询被当作了不规则查询,例如:

```javascript
db.tester.find({age:[1,2,3]})
```

实际上这条语句会查询如下的内容:

```javascript
{age:{$in:[1,2,3]}}
```

但是它内部是否使用了$in查询表达式这就很难说了.

### $all运算符

为了解决上述匹配数组子集的问题我们可以使用$all运算符.

使用如下的测试数据:

```javascript
db.tester.insert([{
	age:[1]
},{
	age:[1,2]
},{
	age:[1,2,3]
}]);
```

执行如下查询:

```javascript
db.tester.find({age:{$all:[1,2]}})
```

输出:

```json
{ "_id" : ObjectId("5c31af55b80ed932bafb763b"), "age" : [ 1, 2 ] }
{ "_id" : ObjectId("5c31af78b80ed932bafb763c"), "age" : [ 1, 2, 3 ] }
```

### 精确匹配

给字段提供一个数组可以进行精确匹配,要求数组中的内容和顺序完全一致,有如下的数据:

```javascript
db.tester.insert([{
	age:[1,2,3]
},{
	age:[3,2,1]
},{
	age:[1,2]
}]);

```

执行如下的查询:

```javascript
db.tester.find({age:[1,2,3]})
```

输出:

```json
{ "_id" : ObjectId("5c31b149b80ed932bafb763d"), "age" : [ 1, 2, 3 ] }
```

### 数组下标匹配

下标语法就是在原有的字段位置上使用`key.index`的方式:

```javascript
db.tester.find({'age.0':1});
```

这条语句告诉Mongodb查询age字段且下标0的值为1的文档.

### $size运算符

$size运算符可以查询数组长度:

```javascript
db.tester.find({age:{$size:3}})
```

但是却不可以和其他的运算符联用,这意味着你无法执行常见的数组大小判断等处理.

但是我们可以有一种变通的手段就是给文档添加一个键,在针对与数组的push和pop中向这个键增加减少值来代表数组的长度.

### $slice运算符

本节要操作的内容在MongoDb中被称之为Projection.

所谓的Projection简单的理解就是find的第二个选项.

使用第二个选项我们可以进行对于查找到的内容进行过滤.

例子:

```javascript
db.tseter.find({},{age:1})
```

**解析**:只显示age字段.

**注意**:但是_id字段依然会进行输出.

例子:

```javascript
db.tester.find({},{age:1,_id:0})
```

**解析**:显示age字段但是不显示_id字段.

$slice用于返回数组的子集,或者叫做显示数组中的指定范围,类似于python中的range表现也是几乎一致.

使用以下的数据:

```json
{ "_id" : 1, "name" : "dave123", favorites: [ "chocolate", "cake", "butter", "apples" ] }
{ "_id" : 2, "name" : "li", favorites: [ "apples", "pudding", "pie" ] }
{ "_id" : 3, "name" : "ahn", favorites: [ "pears", "pecans", "chocolate", "cherries" ] }
{ "_id" : 4, "name" : "ty", favorites: [ "ice cream" ] }
```

执行以下命令:

```javascript
db.tester.find({},{ favorites:{$slice:1} })
```

将返回如下结果:

```json
{ "_id" : 1, "name" : "dave123", "favorites" : [ "chocolate" ] }
{ "_id" : 2, "name" : "li", "favorites" : [ "apples" ] }
{ "_id" : 3, "name" : "ahn", "favorites" : [ "pears" ] }
{ "_id" : 4, "name" : "ty", "favorites" : [ "ice cream" ] }
```

可以看到输出的所有内容中指定的数组长度都是一.

不过需要注意的是:

```javascript
 db.tester.find({},{ favorites:{$slice:[3,5]} })
```

数组长度不够的也会返回空数组:

```json
{ "_id" : 1, "name" : "dave123", "favorites" : [ "apples" ] }
{ "_id" : 2, "name" : "li", "favorites" : [ ] }
{ "_id" : 3, "name" : "ahn", "favorites" : [ "cherries" ] }
{ "_id" : 4, "name" : "ty", "favorites" : [ ] }
```

不仅仅是这样甚至没有该字段的数据也会返回.

### 使用$运算符

**注意**:$运算符在不同情况下有不同的使用方式,这个操作是在`Projection`的语境下执行的,此外还有`update`下执行的$运算符.

我们有如下的数据集:

```json
{ "_id" : 1, "name" : "dave123", "favorites" : [ "chocolate", "cake", "butter", "apples" ] }
{ "_id" : 2, "name" : "li", "favorites" : [ "apples", "pudding", "pie" ] }
{ "_id" : 3, "name" : "ahn", "favorites" : [ "pears", "pecans", "chocolate", "cherries" ] }
{ "_id" : 4, "name" : "ty", "favorites" : [ "ice cream" ] }
{ "_id" : ObjectId("5c330afb272daa5bebae1df1"), "age" : 12 }
```

执行如下的命令:

```javascript
db.tester.find({favorites:{$in:['apples']}})
```

他会将数组中含有apples的文档获取到然后进行输出.

```json
{ "_id" : 1, "name" : "dave123", "favorites" : [ "chocolate", "cake", "butter", "apples" ] }
{ "_id" : 2, "name" : "li", "favorites" : [ "apples", "pudding", "pie" ] }
```

执行如下命令:

```javascript
 db.tester.find({favorites:{$in:['apples']}},{'favorites.$':1})
```

这会把匹配到的所有内容进行过滤:

1. 只保留favorites字段
2. 只保留查询字段中匹配到的第一个内容

```json
{ "_id" : 1, "favorites" : [ "apples" ] }
{ "_id" : 2, "favorites" : [ "apples" ] }
```

### 数组和范围查询

上一章和本章可以参考这个:

> https://blog.csdn.net/zhujun_xiaoxin/article/details/78996120

Mongodb中的数组行为一直很怪异.

我们有如下的数据:

```json
{x:5}
{x:15}
{x:25}
{x:[5,25]}
```

执行如下命令:

```javascript
db.test.find({x:{$gt:10,$lt:20}})
```

输出:

```json
{x:15}
{x:[5:25]}
```

在结果中含有数组的字段X也被包含了进来,同时身为标量的数据也被添加了进来.

而且我们想要的是数组中的大于10且小于20的元素而不是这个一个且的关系,但是在上述命令的执行中这个条件是或的关系.

但是Mongodb却将数组中的每一个元素都进行了匹配规则如下:

- 5 小于 20
- 25 大于 10

__使用$elemMatch运算符__

$elemMatch运算符可以针对数组字段进行查询查询条件的逐个匹配且只会匹配数组字段.

```javascript
db.tester.find({x:{$elemMatch:{$gt:10,$lt:20}}})
```

输出:

```

```

所以这么一做将不会有任何输出.

除此以外我们还可以使用数组索引来完成带有过滤需要的查询,这些内容在后面的索引章节中会提到.

## 查询内嵌文档

主要有两种方式:

1. 查询整个文档
2. 只针对键/值查询

有如下数据:

```json
{
    "name":{
    	"first":"Joe",
    	"last":"Schmoe"
	}
}
```

要查询姓名为Joe Schmoe 的人可以这样.

```javascript
db.people.find({"name":{"first":"Joe","last":"Schmoe"}})
```

需要注意的是这里的:

```json
{
	"name":{"first":"Joe","last":"Schmoe"}
}
```

是内嵌文档也就是说必需要完全匹配才可以,字段顺序和内容必需完全一致可想而知这样的查询限制太大.

我们可以使用如下的命令:

```javascript
db.people.find({"name.first":"Joe","name.last":"Schmoe"})
```

**注意**:由于在MongoDB中可以使用`.`表示进行文档的意思,所以在文档的键上面是不可以使用`.`的,这种情况经常发生在使用URL当做键的时候,通常可以在插入URL的时候对`.`做一个转换.

__复杂情况:__

考虑如下文档:

```javascript
	db.tester.insert({
		content: "",
		comments: [{
				author: "joe",
				score: 3,
				comment: "nice post"
			},
			{
				author: "mary',
				score: 6,
				comment: "terrible post"
			}
		]
	})
```

执行如下的命令:

```javascript
 db.tester.find({"comments.author":"joe","comments.score":6})
```

很显然我们想要comments字段中的符合以上规则的内容,但是它却会返回整个文档:

```json
{ "_id" : ObjectId("5c395847834dbdf20c4aab96"), "content" : "", "comments" : [ { "author" : "joe", "score" : 3, "comment" : "nice post" }, { "author" : "mary", "score" : 6, "comment" : "terrible post" } ] }
```

这不是我们想要的内容,这个时候就需要使用$elemMatch出场了.

执行如下的命令:

```javascript
db.tester.find({comments:{$elemMatch:{"author":"mary","score":{$gte:3}}}})
```

即使这么做也还是会返回文档,但是匹配规则变了它是匹配的数组中的每一个元素,而不是将测试所有的元素是否匹配规则.

想要获取文档片段可以使用如下的命令:

```javascript
 db.tester.find({comments:{$elemMatch:{"author":"mary","score":{$gte:3}}}},{"comments.$":1})
```

返回结果:

```javascript
{ "_id" : ObjectId("5c395847834dbdf20c4aab96"), "comments" : [ { "author" : "mary", "score" : 6, "comment" : "terrible post" } ] }
```

# $where查询

当键值查询能力不足的时候可以使用$where查询,它是一个查询子句它可以执行javascript这意味着它几乎可以做任何事情,在使用它之前你一定不要让客户端来直接操控它,或者另寻它法.

示例,比较文档中的值是否相等:

```javascript
	db.tester.insert({apple:1,banana:6,peach:3})
	db.tester.insert({apple:8,spinach:4,watermelon:4})
```

执行如下命令:

```javascript
db.tester.find({$where:function(){
		for(let currentKey in this){
			for(let otherKey in this){
				if(currentKey !== otherKey && this[currentKey] === this[otherKey]){
					return true;
				}
			}
		}
		return false;
	}})
```

如果函数返回true,文档就做为结果集的一部分返回,如果为false则不返回.

**注意**:不是非常必要时,不要使用$where效率太低,可以通过先执行常规查询在执行$where语句,如果可能的话使用$where前先使用索引进行过滤.

**注意**:使用$where不是唯一的手段,我们还可以使用聚合工具.

# 游标

游标概念是一个非常基础的概念几乎所有的数据库的驱动中都有它的身影.

幸运的是在Mongo Shell中是可以执行JavaScript的所以我们不需要研究具体语言下的驱动.

游标的使用分为如下三步:

1. 启用find方法后赋值给一个变量
2. 调用该变量的hasNext方法,询问是否有内容
3. 调用该变量的next方法获取返回值

```javascript
(function() {
	
	var cursor = db.tester.find();
	
	while(cursor.hasNext()){
		
		const content = cursor.next();
		
		printjson(content);
		
	}
	
})();
```

和其他语言类似的是在执行find的方法的时候并没有真正与服务器进行通信,所以在这个时候你可以配合其他操作符过滤以及给数据添加条件:

```javascript
(function() {
	
	var cursor = db.tester.find().sort({x:1}).limit(10).skip(10);
	
	cursor.forEach((value)=>{
		
		printjson(value);
		
	});
		
})();
```

一旦调用hasNext方法查询将会发送到服务器,而服务器也不会直接将所有的数据返回,根据如下规则:

1. 查询到的数据小于100条
2. 查询到的数据小于4MB

此时的数据将会被缓存,提供next方法返回.一旦数据消耗完毕则会再次进行请求.

## skip&limit&sort

这三个操作是游标操作中最常使用的三个操作.

- skip 跳过N条文档
- limit 指定输出文档上限
- sort 对文档进行排序

例子:

```javascript
db.c.find({desc:"mp3"}).limit(50).skip(50).sort({price:-1})
```

上面这个例子中查询的指定条件的文档,首先价格降序排列,并且从第50条记录开始输出,输出50条.

**注意**:如果略过太多的内容会有性能问题.

### 比较顺序

MongoDB处理不同的数据是有一定顺序的.

有时候一个键的值可能是多种类型的,针对这种情况,优先级从小到大,顺序如下:

1. 最小值
2. null
3. 数字
4. 字符串
5. 对象/文档
6. 数组
7. 二进制数据
8. 对象ID
9. 布尔型
10. 日期型
11. 时间戳
12. 正则表达式
13. 最大值

### 避免使用skip略过大量的结果

__使用上次的结果来处理下次的内容__

```javascript
(function() {
	
	let i = 100;

	while(i--){
		db.tester.insert({
			index:i
		})
	}
		
})();
```

先查询前20条记录:

```javascript
(function() {
	

	const cursor = db.tester.find().sort({x:1}).limit(10);

	cursor.forEach((value,index)=>{

		printjson(value);

	});
		
})();
```

然后利用之前获取的数据,进行模糊查询后排序,然后再查询:

```javascript
(function() {
	

	let cursor = db.tester.find().sort({x:1}).limit(10);


	let lastDoc;

	while(cursor.hasNext()){
		lastDoc = cursor.next();
	}

	cursor = db.tester.find({index:{$lt:lastDoc.index}}).sort({x:1}).limit(10);

	cursor.forEach((value)=>{
		printjson(value);
	});
		
})();

```

__随机查询文档__

从文档中获取一个随机内容算是个常见问题.最笨的同样也是很慢的做法就是先计算文档总数,然后选择一个从0到文档数量之间的随机数.

这个方法明显是使用空间换时间的做法,同时也是利用依赖内部优化的典型实现.

在插入的时候我们就进的提供一个随机值.

```javascript
db.people.isnert({name:'joe',random:Math:random()})
db.people.isnert({name:'john',random:Math:random()})
db.people.isnert({name:'jim',random:Math:random()})
```

然后在查询的时候只需要再次提供一个随机数就可以了:

```javascript
const random = Math.random();
const result = db.foo.findOne({random:{$gt:random}});
```

当然你可能会遇到我们生成的随机数比任何内容都大的时候,此时它应该无法返回任何结果,只需要返方向继续查询:

```javascript
if(result === null){
    result = db.foo.findOne({random:{$lt:random}})
}
```

## 高级查询选项

查询分为两种:

1. plain query 简单查询
2. wrapped query 封装查询

简单查询如下:

```javascript
var cursor = db.foo.find({foo:"bar"})
```

而封装查询会在原有的查询基础上进行更加细微的操作.

**注意**:不同驱动以及SHELL的效果不同在mongo shell中直接支持这些操作的链式调用.

常见运算符一览:

| 名称         | 类型     | 描述                     |
| ------------ | -------- | ------------------------ |
| $maxscan     | integer  | 指定查询数量上限         |
| $min         | document | 查询的开始条件(前置查询) |
| $max         | document | 查询的结束条件(后置查询) |
| $showDiskLoc | boolean  | 显示该结果在磁盘上的位置 |

## 获取一致结果

**警告**:该操作已过时.

数据处理的通常的做法就是先把数据从MongoDB中取出来,然后做一些处理再把取出来的数据存回去.

```javascript
(function() {
	

	const cursor = db.tester.find();

	while(cursor.hasNext()){

		var doc = process(cursor.next());

		db.tester.save(doc);

	}
		
})();
```

问题就在于看似虚拟数据的处理实际上是和具体的物理设备有关联的.

MongoDB在实际储存中是按照起始文档的大小一块块的顺序在硬盘中排列的,没有一个数据是重复的.

当一个数据变大的时候这个块空间存放不下数据的时候MongoDB会将它移动到末尾的位置中.

而游标是从前向后扫描的一旦触发上面的情况,原本包含有100文档的内容就有可能被扫描100+N次.

为了避免这种情况的发生,我们可以使用snapshot(快照)功能,它会按照_id来进行扫描而不是磁盘的顺序.

这样做会降低性能所以在使用的时候你得考虑是否真的需要使用它.

```javascript
db.tester.find().snapshot()
```

新的解决办法:

针对不会变化的键添加唯一索引在查询的时候使用hint强制使用该键进行查询.

```javascript
db.users.find().hint( { age: 1 } )
```

