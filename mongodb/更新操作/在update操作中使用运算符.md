# $inc

`$inc`运算符用于修改文档,用于增加或者减少某一个字段的值.

示例原有的文档内容:

```json
{
  _id: 1,
  sku: "abc123",
  quantity: 10,
  metrics: {
    orders: 2,
    ratings: 3.5
  }
}
```

使用`$inc`运算符:

```javascript
db.products.update(
   { sku: "abc123" },
   { $inc: { quantity: -2, "metrics.orders": 1 } }
)
```

结果:

```json
{
   "_id" : 1,
   "sku" : "abc123",
   "quantity" : 8,
   "metrics" : {
      "orders" : 3,
      "ratings" : 3.5
   }
}
```

# $set

`$set`运算符用于替换指定字段中的值.

# $unset

`$unset`用于删除指定的字段.

# $push

`$push`运算符用于向数组字段添加内容,没有该字段则创建反之则添加.

具体的语法定义如下:

```
{ $push: { <field1>: <value1>, ... } }
```

示例原有集合中的文档内容(脚本插入):

```javascript
const result = db.tester.insert({
    _id:1,
    name:'ASCll',
    age:20
});
```

然后新增一个comment字段:

```javascript
const result = db.tester.update({name:'ASCll'},{
	$push:{
		comments:{
			name:'joe',
			email:'joe@example.com',
			content:'nice post.'
		}
	}
});
```

输出:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"comments": [{
		"name": "joe",
		"email": "joe@example.com",
		"content": "nice post."
	}]
}
```

如果我们在此基础上继续执行一次,数组中会增加一个新的内容:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"comments": [{
		"name": "joe",
		"email": "joe@example.com",
		"content": "nice post."
	}, {
		"name": "joe",
		"email": "joe@example.com",
		"content": "nice post."
	}]
}
```

## $push运算符配合修改器

可以使用的修改器:

- $each 向一个字段中添加多个内容
- $slice 限制数组元素个数必须配置$each使用
- $sort 针对数组进行排序必须配合$each使用
- $position 指定数组追加的位置

__使用$each__

回到之前没有comments字段的数据上:

```json
{ "_id" : 1, "name" : "ASCll", "age" : 20 }
```

执行以下命令:

```javascript
	const result = db.tester.update({name:'ASCll'},{
		$push:{
			scores:{
				$each:[80,90,100]
			}
		}
	});
```

最后文档变为如下结果:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"scores": [80, 90, 100]
}
```

再次执行更新然后输出:

```javascript
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"scores": [80, 90, 100, 80, 90, 100]
}
```

__使用$slice__

借用上面的例子我们来测试一下`$slice`运算符.

现在我们的`scores`字段内容如下:`[80, 90, 100, 80, 90, 100]`.

执行如下命令,添加3个元素然后只保留1个:

```javascript
	const result = db.tester.update({name:'ASCll'},{
		$push:{
			scores:{
				$each:[80,90,100],
				$slice:1
			}
		}
	});
```

文档结果如下:

```javascript
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"scores": [80]
}
```

这个测试告诉我们:

1. $slice用于限制数组元素个数
2. $slice限制的是整个字段中的内容长度不是本次插入的元素的长度

在此执行如下的命令:

```javascript
	const result = db.tester.update({name:'ASCll'},{
		$push:{
			scores:{
				$each:[80,90,100,110,120],
				$slice:-3
			}
		}
	});
```

根据之前的规则scores执行完成后元素内容应如下[80,80,90,100,110,120].

我们限制了-3所以从后往前数3个应该是[100,110,120].

执行后输出如下:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"scores": [100, 110, 120]
}
```

和预测中的一致.

__使用$sore__

$sore有两种排序模式,是根据具体使用情况而决定的:

1. $push的是文档,此时$sort需要指定字段
2. $push的是非文档,此时$sort不需要指定字段

此外$sort使用1来代表升序-1来代表降序.

再一次使用之前的文档中的内容`{ "_id" : 1, "name" : "ASCll", "age" : 20 }`.

执行如下命令来进行非文档类型的排序:

```javascript
	const result = db.tester.update({name:'ASCll'},{
		$push:{
			scores:{
				$each:[80,90,100,110,120],
				$sort:-1
			}
		}
	});
```

输出:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"scores": [120, 110, 100, 90, 80]
}
```

这个时候我们还配合使用$slice修改器来执行常见获取最大元素中的前N位的操作.

继续执行如下命令:

```javascript
	const result = db.tester.update({name:'ASCll'},{
		$push:{
			scores:{
				$each:[80,90,100,110,120],
				$sort:-1,
				$slice:3
			}
		}
	});

```

输出:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"scores": [120, 120, 110]
}
```

清空我们的集合后再次插入我们的基本文档:

```json
{ "_id" : 1, "name" : "ASCll", "age" : 20 }
```

然后执行如下的命令,注意这次我们的$sort指定了字段.

```javascript
	const result = db.tester.update({ _id: 1 },
	{
		$push: {
			quizzes: {
				$each: [ { id: 3, score: 8 }, { id: 4, score: 7 }, { id: 5, score: 6 } ],
				$sort: { score: 1 }
			}
		}
	}
	);
```

输出:

```json
{
	"_id": 1,
	"name": "ASCll",
	"age": 20,
	"quizzes": [{
		"id": 5,
		"score": 6
	}, {
		"id": 4,
		"score": 7
	}, {
		"id": 3,
		"score": 8
	}]
}
```

最后的结果以score字段升序排序.

## 将数组当作Set(集合)使用

我们使用`$ne`选择器来进行初次尝试,集合和数组的不同就在于元素不会进行重复.

$ne指的是not equal(不等于)他会选择所有值不相等的内容.

对于集合更新操作,如果相等就不更新内容反之则跟新内容.

清空我们之前使用的集合,然后输入以下指令:

```javascript
	db.tester.insert([{
		name:"ASCll"
	},{
		name:"zhao"
	}]);
```

假设这个集合的用途是保存某一篇文章的引用信息作者也包括在内,我们要选择非作者的所有人为其添加一个引用链接数组.

执行如下的命令:

```javascript
const result = db.tester.update({
    name:{
        $ne:"zhao"
    }
},{
    $push:{
        links:['xxxxxxxxxx']
    }
});
```

输出:

```json
{
	"_id": ObjectId("5c272b3b70e530cc63f5bcfb"),
	"name": "ASCll",
	"age": 20,
	"links": [
		["xxxxxxxxxx"]
	]
} {
	"_id": ObjectId("5c272b3b70e530cc63f5bcfc"),
	"name": "zhao",
	"age": 20
}
```

本例中名字为ASCll的用户被添加了一个数组字段.

使用`$ne`来进行操作局限太大,完全就不是我们所熟知的Set(集合),

### 使用$addToSet运算符

$addToSet运算符可以将多个元素插入到一个集合中且保证元素的唯一性.

不过坑的是这条语句有很多无聊的规则,分两种情况:

1. 向某个集合添加一个元素
2. 向某个集合添加一个数组

__添加一个元素__

清空我们之前的数据表然后插入如下的数据:

```javascript
	const result = db.tester.insert([{
		_id:1,
		item:'Demo',
		tags:['hello','world']
	}]);
```

然后使用$addToSet进行数据的更新我们执行两次数据更新来测试数据的唯一性:

```javascript
	const result = db.tester.update({_id:1},{
		$addToSet:{
			tags:"foo"
		}
	});
```

输出:

```json
{
	"_id": 1,
	"item": "Demo",
	"tags": ["hello", "world", "foo"]
}
```

可以看到foo没有被连续添加两次.

__添加数组__

接着上面的例子,我们来测试插入一个数组会有什么效果:

```javascript
	const result = db.tester.update({_id:1},{
		$addToSet:{
			tags:["foo","bar"]
		}
	});
```

输出:

```
{
	"_id": 1,
	"item": "Demo",
	"tags": ["hello", "world", "foo", ["foo", "bar"]]
}

```

喜闻乐见没有像预期的那样工作.

__$appToSet配合$each使用__

继续使用上面的例子(二维数组之前的那个版本).

输入如下的内容:

```javascript
	const result = db.tester.update({_id:1},{
		$addToSet:{
			tags:{
				$each:['foo','bar']
			}
		}
	});
```

怎么样是不是感觉复杂了不少:joy:但是完成的事情却是$push搭配$ne完成不了的.

输出:

```json
{
	"_id": 1,
	"item": "Demo",
	"tags": ["hello", "world", "foo", "bar"]
}
```

### 从数组中删除元素

有几种从数组中删除元素的方式.

- $pop 将数组视为一个栈弹出栈顶或者栈尾的元素
- $Pull 删除数组中所有符合条件的元素

__测试$pop运算符__

继续使用上面的例子中的文档,内容如下:

```json
{
    "_id": 1, 
    "item": "Demo", 
    "tags": [
        "hello", 
        "world", 
        "foo", 
        "bar"
    ]
}
```

执行如下的命令弹出栈顶和栈尾:

```javascript
	let result = db.tester.update({_id:1},{
		$pop:{
            tags:1
        }
	});
    
    result = db.tester.update({_id:1},{
        $pop:{
            tags:-1
        }
    });
```

输出:

```json
{
    "_id": 1, 
    "item": "Demo", 
    "tags": [
        "world", 
        "foo"
    ]
}
```

可以看到原来的栈顶和栈尾不存在了.

__测试$pull运算符__

$pull运算符比$pop要复杂一些,因为$Pull不仅可以操作数组还可以操作文档数组.

先来看看最简单的情况,操作一个数组,这次我们使用最新的数据,内容如下:

```json
{
   _id: 1,
   fruits: [ "apples", "pears", "oranges", "grapes", "bananas" ],
   vegetables: [ "carrots", "celery", "squash", "carrots" ]
}
```

我们将`vegetables`字段中所有的`carrots`进行删除:

```javascript
    const result = db.tester.update({},{
        $pull:{
            vegetables:'carrots'
        }
    });
```

输出:

```json
{
    "_id": 1, 
    "fruits": [
        "apples", 
        "pears", 
        "oranges", 
        "grapes", 
        "bananas"
    ], 
    "vegetables": [
        "celery", 
        "squash"
    ]
}
```

这次使用带有条件的$pull,将集合非`apples`的元素都删掉:

```javascript
    const result = db.tester.update({},{
        $pull:{
            fruits:{
                $ne:'apples'
            }
        }
    });
```

输出:

```json
{
    "_id": 1, 
    "fruits": [
        "apples"
    ], 
    "vegetables": [
        "celery", 
        "squash"
    ]
}
```

### $位置运算符

简而言之$用于表示update中查询匹配到的数组字段中的首个元素.

例如我们有如下的数据:

```javascript
db.tester.insert([
   { "_id" : 1, "grades" : [ 85, 80, 80 ] },
   { "_id" : 2, "grades" : [ 88, 90, 92 ] },
   { "_id" : 3, "grades" : [ 85, 100, 90 ] }
])
```

当你匹配到多个数组元素的时候可以使用$来代指第一个:

```javascript
   const result = db.tester.update({_id:1,grades:80},{
        $set:{
            "grades.$":82
        }
    });
```

输出:

```json
{ "_id" : 1, "grades" : [ 85, 82, 80 ] }
{ "_id" : 2, "grades" : [ 88, 90, 92 ] }
{ "_id" : 3, "grades" : [ 85, 100, 90 ] }
```





