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

```
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

`**Object.preventExtensions()**`

`**Object.seal()**`

**Object.isSealed()**

`**propertyIsEnumerable()**`

**Object.getOwnPropertyNames()**

isPrototypeOf(

hasOwnProperty(

147

