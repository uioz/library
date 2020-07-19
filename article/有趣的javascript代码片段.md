# 片段1

```javascript
var a = {
  a:'hello world',
  b:function(){
    this.a = 'foo bar';
  }
}

var b = {
  a:'game over',
  b:a.b
}


b.b();
console.log(a.a);
console.log(b.a);

var result = new b.b();
console.log(result.a);
console.log(a.a);
console.log(b.a);

```

# 片段2

```javascript
function foo(){
  console.log(this.a);
}

function bar(){
  "use strict";
  console.log(this.a);
}

var a = 2;

foo();
foo.call(null); // print what
bar(); // print what
bar.call(); // print what

```

# 片段3

> 来自 你不知道的javascript卷一  软绑定

```javascript
if (!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this; // 捕获所有 curried 参数
        var curried = [].slice.call(arguments, 1);
        var bound = function() {
            return fn.apply((!this || this === (window || global)) ? obj: this, curried.concat.apply(curried, arguments));
        };
        bound.prototype = Object.create(fn.prototype);
        return bound;
    };
}
```

# 片段4

```javascript
var a = "hello"

console.log(a instanceof String)
console.log(typeof a === 'string')
console.log(new String(a) instanceof String)
console.log(typeof new String(a) === 'string')
```

# 片段5

```javascript
var proto = {};

Object.defineProperty(proto,'hello',{
	writeable:false,
	value:'world'
});

var obj = Object.create(proto);

console.log(obj.hello) // print what ?

obj.hello = 'foobar';

console.log(obj.hello === 'foobar') // print what
```

```javascript
var 
	proto = {},
	innerValue = 'foo';

Object.defineProperty(proto,'hello',{
	get(){ // es6 style
		return innerValue;
	},
	set(){ // es6 style
		innerValue = innerValue + 'bar';
	}
});

var obj = Object.create(proto);

console.log(obj.hello) // print what?

obj.hello = 'hello world';

console.log(Object.keys(obj).length) // print what ?

console.log('hello' in obj) // print what?

console.log(obj.hello) // print what?
```

```javascript
var objOne = {
	a:2
}

var objTwo = Object.create(objOne);

console.log(objOne.hasOwnProperty("a")); // print what?
console.log(objTwo.hasOwnProperty("a")); // print what?

objTwo.a++;

console.log(objOne.a); // print what?
console.log(objTwo.a); // print what?

console.log(objTwo.hasOwnProperty("a")); // print what?
```

# 模拟 call 的实现

```javascript
Function.prototype.call2 = function (context) {

  if(context == null){
    context = window;
  }

  var args = [];
  var argLengths = arguments.length;
  var index = 0;
  while (index < argLengths) {
    args.push('arguments[' + index + ']');
    index++;
  }

  context.__call2 = this;
  //                                       此处的 args.toString() 自动返回 "arg1,arg2,arg3" 的格式
  const result = eval('context.__call2(' + args + ')'); 
  delete context.__call2;
  return result;
};

const obj = {
  name: "test",
};

function foobar() {
  console.log(this);
}

foobar.call2(obj);

```

# 模拟 bind 的实现

```javascript
Function.prototype.bind2 = function(context) {

	var that = this;
	var args = Array.prototype.slice.call(arguments, 1);

	function handler() {

		return that.apply(
			this instanceof handler ? this : context,
			args.concat(
				Array.prototype.slice.call(arguments)
			)
		)

	}

	handler.prototype = Object.create(that.prototype);

	return handler

}

function test() {
	this.a = 10;
	console.log(this)
}

var obj = {
	name: 'obj'
}

test.bind2(obj)()
```

## 模拟 new 的实现

```javascript

```



`**Object.preventExtensions()**`

`**Object.seal()**`

**Object.isSealed()**

`**propertyIsEnumerable()**`

**Object.getOwnPropertyNames()**

isPrototypeOf(

hasOwnProperty(

147

