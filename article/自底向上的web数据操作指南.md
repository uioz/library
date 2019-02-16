# 简介

本篇文章主要探讨JavaScript中的数据操作.

JavaScript一直以来给人一种比较低能的感觉,例如无法读取系统上的文件,不能做一些底层的操作.

所以在页面上操作数据会交由服务器处理也就成了主流的做法.

但是很多人没有发现,实际上JavaScript以及在逐步增强这些功能,现在我们就已经可以放心的在web端进行文件操作了.

# 起因

N个月前我去新浪面试实习,我提到了原来我做过一个页面配合上传Excel可以完成一些功能.

我的这番话勾起了面试官在实际编码中遇到了一些问题,就是如何不通过服务器来操作数据,我和她讨论了一番,最后不了了之了(当然也没过).

N个月后实习被坑:joy:,成了无业游民闲来无事正好也好奇这个问题然后就研究了一下.

# 涉及的内容

没有非必要的内容,对于文件操作来说以下API都是必须了解的,本文也会渐进式的讨论这些内容.

- Blob
- ArrayBuffer
- TypedArray
- DataView
- FileReader
- File
- URL

# 兼容性

我没有详细考证API的兼容性,不过从MDN提供的数据来看IE10以上的浏览器大部分都是兼容的.

# 总览

一般来说操作一个文件都要经历如下的步骤:
- 知道文件的地址(存放的位置)
- 读取
- 保存到Buffer中,重复上步骤直至结束
- 进行数据编辑
- 知道要写入的地址
- 获取要写入的数据,从Buffer中获取还是所有数据
- 写入
- 写入完成

API名称以及对应的职责:
| 名称 | 职责 |
| ---- | ---- |
| URL  | 制造文件地址 |
| FileReader | 读取文件的接口 |
| Blob | 用于在JavaScript表示文件 |
| File | 用于表示文件对象 |
| ArrayBuffer | 表示Buffer(仅仅提供一片内存空间) |
| TypedArray | 基于数组操作Buffer上的数据(操作的最小单位是数组元素) |
| DataView | 基于字节操作Buffer上的数据 |

上面描述的内容之间的关系很复杂,这里我们逐步来进行分析.

# ArrayBuffer

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

ArrayBuffer对象用于表示一段缓冲区域(可以理解为一段可控的内存区域),它仅仅表示这片被开辟的区域但是不提供操作方式.

```javascript
const arraybuffer = new ArrayBuffer(8) // 创建一个长度为8字节大小的Buffer
```
默认ArrayBuffer中每一个字节都被填充了0.

利用这个对象我们可以完成如下的操作:
- 获取
  - 该Buffer的大小(字节)
  - 该Buffer的副本(范围)
- 修改
  - 该Buffer的大小
- 判断
  - 给定的数据是否是操作视图(实例方法)
- 异常
  - 当创建的Buffer长度超过`Number.MAX_SAFE_INTEGER`的大小会产生错误

```javascript

const arraybuffer = new ArrayBuffer(8);

console.log(arraybuffer.byteLength); // 获取长度
console.log(arraybuffer.slice(4,8)); // 获取副本
// 截止到2019年2月12日 20:11:05没有浏览器实现该功能
console.log(arraybuffer.transfer(arraybuffer,16));// 修改原有Buffer
console.log(ArrayBuffer.isView({})) // false 是否是视图

```

# DataView
> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView

DataView用于操作ArrayBuffer中的数据,这也是它构造函数中接受一个ArrayBuffer的原因:
```javascript
const arraybuffer = new ArrayBuffer(8);
const dataview = new DataView(arraybuffer); // 默认的视图大小就是buffer的大小
const offset = new DataView(arraybuffer, 0, arraybuffer.byteLength); // 默认的偏移量以及长度
```
利用这个对象我们可以完成如下的操作:
- 获取
  - 被该视图引入的Buffer(只读)
  - 该视图从Buffer中读取的自己长度(只读)
  - 该视图从Buffer中读取的偏移量(只读)
- 异常
  - 如果由偏移（byteOffset）和字节长度（byteLength）计算得到的结束位置超出了 buffer 的长度.
- 写入
  - 使用xxx类型写入(见下方)
- 读取
  - 使用xxx类型读取

可以使用的类型:

| 类型名称 | 对应的方法            |
| -------- | --------------------- |
| Int8     | getInt8,setInt8       |
| Uint8    | getUint8,setUint8     |
| Int16    | getInt16,setInt16     |
| Uint16   | getUint16,setUint16   |
| Int32    | getInt32,setInt32     |
| Uint32   | getUint32,setUint32   |
| Float32  | getFloat32,setFloat32 |
| Float64  | getFloat64,setFloat64 |

简单实例:

```javascript
        const arraybuffer = new ArrayBuffer(1); // 一个字节
        const dataview = new DataView(arraybuffer); // 默认的视图大小就是buffer的大小
        
        dataview.setInt8(0,127) // 从0开始写入一个int8(8位无符号整形,一个字节)
        dataview.getInt8(0) // 从偏移0开始读取一个int8(8位无符号整形,一个字节)
        console.log(dataview.getInt8(0));
        dataview.setInt16(0,65535); // 错误超出了ArrayBuffer的空间 int16占两个字节
```

## 字节序

简单来讲-使用DataView:
- 在读写时不用考虑平台字节序问题。

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView#%E5%AD%97%E8%8A%82%E5%BA%8F
>
> https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F

可以利用这个函数来进行判断:
```javascript
var littleEndian = (function() {
  var buffer = new ArrayBuffer(2);
  new DataView(buffer).setInt16(0, 256, true /* 设置值时使用小端字节序 */);
  // Int16Array 使用系统字节序，由此可以判断系统是否是小端字节序
  return new Int16Array(buffer)[0] === 256;
})();
console.log(littleEndian); // true or false
```

# TypedArray

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray

在上面一节中我们使用get和set的方式基于数据类型来读写内存(ArrayBuffer)中的数据.

而所谓的TypedArray就是使用类似于操作数组的方式来操作我们的Buffer可以理解为数组中的每一个元素都是不同类型的数据,这样一来我们可以使用数组上的很多方法,相较于干巴巴的使用get和set更加灵活一些,少掉点头发.

名字叫做TypedArray的这个对象或者全局构造函数并不存在于JavaScript中.因为类型数组并不只有一个,但是TypedArray代指的这些内容拥有统一的构造函数,统一的属性统一的方法,不同的只是他们的名字以及所对应的数据类型.

```
TypedArray()指的是以下的其中之一： 

Int8Array(); 
Uint8Array(); 
Uint8ClampedArray();
Int16Array(); 
Uint16Array();
Int32Array(); 
Uint32Array(); 
Float32Array(); 
Float64Array();
```

看到这里我们立马联想到了之前DataView上不同的Get和Set,概念是一样的,不同于ArrayBuffer的是,这里的最小数据单位是数组中的元素,不同类型元素所占用的空间是不同的,但是我们不需要考虑在字节层面上进行控制.

接下来我们利用Int8Array来进行讨论:

- 构造函数
  - 传入一个数值来表示类型数组中元素的数量
  - 传入任意一个类型数组在保留其原有的长度上进行数据类型转换
- 方法(静态)
  - Int8Array.from()通过可迭代对象创建一个类型数组
  - Int8Array.of()通过可变参数创建一个类型数组

例子:

```javascript
// 32无符号能表示的最大的数值 占4个字节
const int32 = new Int32Array(1); // 使用length
int32[0] = 4294967295;

// 8位无符号能表示最大的内容是127 占1个字节
const int8 = new Int8Array(int32); // 使用另外一个类型数组
console.log(int8[0]) // -1 32位转8位要确保,32位的值在8位的范围内否则无法保证精度

const from = Int8Array.from([0,127]);
console.log(from.length === 2) // true

const of = Int8Array.of(0,127);
console.log(of.length === 2)// true
```

- 属性(静态)
  - [`TypedArray.BYTES_PER_ELEMENT`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/BYTES_PER_ELEMENT)
  - *TypedArray*.length
  - [`TypedArray.name`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/name)
  - [`get TypedArray[@@species\]`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/@@species)
  - [`TypedArray.prototype`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/prototype)
- 属性(实例)
  - [`TypedArray.prototype.buffer`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/buffer)
  - [`TypedArray.prototype.byteLength`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/byteLength)
  - [`TypedArray.prototype.byteOffset`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/byteOffset)
  - [`TypedArray.prototype.length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/length)
- 方法(实例)
  - 方法是在是太多了Array上的方法TypedArray基本都有,例举太多都是照搬MDN,给个贴上大家自行查阅吧:joy:.
  - [方法列表](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray#%E6%96%B9%E6%B3%95_2)

例子(类数组操作):

```javascript
const int8 = new Int8Array(2);
int8[0] = 0;
int8[1] = 127;

int8.forEach((value)=>console.log(value));

for (const elem of int8) {
	console.log(elem);
}
```

# Blob

> https://developer.mozilla.org/zh-CN/docs/Web/API/Blob
>
> Blob` 对象表示一个不可变、原始数据的类文件对象。Blob 表示的不一定是JavaScript原生格式的数据

这说明了什么意思,类似于ArrayBuffer一样,ArrayBuffer本身没有为了达到某种目的而提供具体的操作方法,他的存在就类似于一个占位符一样,Blob对象也是类似的概念,在JavaScript中我们使用Blob对象来表示一个文件,当这个文件需要进行操作的时候我们在利用其他途径对这个Blob对象进行操作.(个人理解)

Blob的API和ArrayBuffer非常相似,因为他们有着非常密切的联系,创建Blob对象有两种方式,对应着两种具体的需求:

- 直接调用构造函数传入JavaScript中的数据结构
- 使用File对象创建,用于表示文件

这里我们不讨论由File对象创建的情况,这部分留到下节中讨论.

- 构造函数
  - 你可以利用现有的JavaScript数据结构来创建一个Blob对象
  - 你可以选择这个Blob对象的MIME类型
  - 你可以控制这个Blob对象中的换行符在系统中表现的行为
  - [具体参考](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/Blob)
- 属性(实例)
  - size - Blob对象所包含的数据大小
  - type - Blob对象所描述的MIME类型
- 方法(实例)
  - slice()类似于ArrayBuffer.slice()从原有的Blob中分离出一部分组成新的Blob对象

例子:

```javascript
        const blob1 = new Blob([JSON.stringify({
                content: 'success'
            })], {
                type: 'application/json'
        });

        const blob2 = new Blob(['<a id="a"><b id="b">hey!</b></a>'],{
            type:'text/html'
        });
```

**注意**:Blob对象接受的第一个参数是一个数组.

Blob对象还可以根据其他数据结构进行创建:

- ArrayBuffer
- ArrayBufferView(TypedArray)
- Blob

> https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/Blob

乍一看Blob对象看似很鸡肋,不过在JavaScript中能装载数据还可以指定MIME类型,这种情况多半都是用于和外部进行交互.

回顾前面的内容,我们知道了如何创建一片内存中的区域,还知道了如何利用不同的工具来对这篇内存进行操作,最重要的一个用于描述文件Blob对象接受ArrayBuffer和TypedArray,那么还能玩出什么花样呢?

# File

> 文件（**File**）接口提供有关文件的信息，并允许网页中的 JavaScript 访问其内容。
>
> https://developer.mozilla.org/zh-CN/docs/Web/API/File

File对象用于描述文件,这个对象虽然可以利用构造函数自行创建,但是大多数情况下都是利用浏览器上的`<input>`元素或者拖拽API来获取的.

File对象继承Blob对象,所以继承了Blob对象上的原型方法和属性,和Blob纯粹表示文件不同,File更加接地气一点,他还拥有了我们操作系统上常见的一些特征:

- 属性(实例)
  - lastModified 最后修改时间
  - name 文件名称
  - size 文件大小
  - type MIME类型
  - [详细介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/File#Properties)
- 构造函数
  - [详细介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/File/File)

例子:

```javascript
        // 创建buffer
        const buffer = new Int8Array(2);
        console.log(buffer.byteLength); // 2
        buffer[0] = 0;
        buffer[1] = 127
        console.log(buffer[0]); // 127
        // 利用buffer创建一个file对象
        const file = new File([buffer],'text.txt',{
            type:'text/plain',
            lastModified:Date.now()
        });

        // file继承blob所以可以使用slice方法,返回一个blob对象
        const blob = file.slice(1,2,'text/plain');
        console.log(blob.size); //1
```

File对象目前看来依然扮演者'载体'的角色,不过在将他交由其他的API的时候才是他真正发挥威力的地方.

# FileReader

FileReader一看名字我就有一种想喊`JavaScript`(浏览器端)永不为奴的冲动.前面铺垫了那么多终于可以看到真正可以实际利用的内容了.

> `FileReader` 对象允许Web应用程序异步读取存储在用户计算机上的文件（或原始数据缓冲区）的内容，使用 [`File`](https://developer.mozilla.org/zh-CN/docs/Web/API/File) 或 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象指定要读取的文件或数据。
>
> https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader

FileReader和前面的所有提到内部不同的地方在于,这个API有事件,你可以使用`onXXX`和`addEventListener`进行监听.

基本工作流程:

1. 获取用户提供的文件对象(通过input或者拖拽)
   1. 或者自己创建File或者(Blob)对象
2. 新建一个FileReader()实例
3. 监听对应的方法来获取读取内容完成后的回调
4. 利用不同的方法读取文件内容
   1. 读取为`fileReader.ArrayBuffer()`
   2. 读取为DataURL`fileReader.readAsDataURL()`
   3. 读取为字符串`fileReader.readAsText()`

示例1读取计算机上的文件:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>blob</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>

<body>
    <!-- 建议选中一个文本 -->
    <label for="file">读取文件<input id="file" type="file" ></label>
    <script type="text/javascript">
        document.getElementById('file').addEventListener('change',(event)=>{

            const files = event.srcElement.files;

            if(files.length === 0){
                return console.log('没有选择任何内容');
            }

            const file = files[0];

            console.log(file instanceof File); // true
            console.log(file instanceof Blob); // true

            const reader = new FileReader();

            reader.addEventListener('abort',()=>console.log('读取中断时候触发'));
            reader.addEventListener('error',()=>console.log('读取错误时候触发'));
            reader.addEventListener('loadstart',()=>console.log('开始读取的时候触发'));
            reader.addEventListener('loadend',()=>console.log('读取结束触发'));
            reader.addEventListener('progress',()=>console.log('读取过程中触发'));

            // 当内容读取完成后再load事件触发
            reader.addEventListener('load',(event)=>{

                // 输出文本文件的内容
                console.log(event.target.result)

            });
            // 读取一个文本文件
            reader.readAsText(file);

        });
    </script>
</body>

</html>
```

如果一切顺利你就可以从计算机上读取一个文件并且以文本的形式展现在了控制台中.

而且不仅如此,利用:

```javascript
reader.readAsArrayBuffer(file)
```

我们可以读取任何类型的数据,然后再内存中进行修改,剩下的就差保存了.

## FileReaderSync

这个API是FileReader的同步版本,这意味着代码执行到读取的时候会等待文件的读取,所以这个API只能在workers里面使用,如果在主线程中调用它会阻塞用户界面的执行.

由于是同步读取,所以没有回调掉必要存在,也就不需要监听事件了.

> https://developer.mozilla.org/zh-CN/docs/Web/API/FileReaderSync

# URL

前面我们讨论完成了数据的读取,在FileReader中我们已经可以获取ArrayBuffer然后使用DateView和TypedArray就可以修改ArrayBuffer完成文件的修改,接下来我们旅行中的最后一程.

> https://developer.mozilla.org/zh-CN/docs/Web/API/URL

在JavaScript(浏览器端)中我们可以使用URL来创建一个URL对象:

```javascript
new URL('https://www.xxx.com?q=10')
```

他返回的对象包含如下的内容:

```
// 控制台
new URL('https://www.xxx.com?q=10')

URL
hash: ""
host: "www.xxx.com"
hostname: "www.xxx.com"
href: "https://www.xxx.com/?q=10"
origin: "https://www.xxx.com"
password: ""
pathname: "/"
port: ""
protocol: "https:"
search: "?q=10"
searchParams: URLSearchParams {  }
username: ""
```

可见该对象是一个工具对象用于帮助我们更加容易的处理URL.

例子([来自MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/URL#%E4%BE%8B%E5%AD%90)):

```javascript
var a = new URL("/", "https://developer.mozilla.org"); // Creates a URL pointing to 'https://developer.mozilla.org/'
var b = new URL("https://developer.mozilla.org");      // Creates a URL pointing to 'https://developer.mozilla.org'
var c = new URL('en-US/docs', b);                      // Creates a URL pointing to 'https://developer.mozilla.org/en-US/docs'
var d = new URL('/en-US/docs', b);                     // Creates a URL pointing to 'https://developer.mozilla.org/en-US/docs'
var f = new URL('/en-US/docs', d);                     // Creates a URL pointing to 'https://developer.mozilla.org/en-US/docs'
var g = new URL('/en-US/docs', "https://developer.mozilla.org/fr-FR/toto");
                                                       // Creates a URL pointing to 'https://developer.mozilla.org/en-US/docs'
var h = new URL('/en-US/docs', a);                     // Creates a URL pointing to 'https://developer.mozilla.org/en-US/docs'
var i = new URL('/en-US/docs', '');                    // Raises a SYNTAX ERROR exception as '/en-US/docs' is not valid
var j = new URL('/en-US/docs');                        // Raises a SYNTAX ERROR exception as 'about:blank/en-US/docs' is not valid
var k = new URL('http://www.example.com', 'https://developers.mozilla.com');
                                                       // Creates a URL pointing to 'https://www.example.com'
var l = new URL('http://www.example.com', b);          // Creates a URL pointing to 'https://www.example.com'
```

实际上这和Node中的URL对象十分相似:

```
// 终端
> Node
> new URL('https://www.xxx.com/?q=10')
URL {
  href: 'https://www.xxx.com/?q=10',
  origin: 'https://www.xxx.com',
  protocol: 'https:',
  username: '',
  password: '',
  host: 'www.xxx.com',
  hostname: 'www.xxx.com',
  port: '',
  pathname: '/',
  search: '?q=10',
  searchParams: URLSearchParams { 'q' => '10' },
  hash: '' }
```

它和我们讨论的文件下载有什么关系呢,在我们在浏览器中一切可以利用的资源都有唯一的标识符那就是URL.

而我们自定义或者读取的文件需要通过URL对象创建一个指向我们定义资源的链接.

那么URL对象上提供了两个静态方法:

- [`URL.createObjectURL()`](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL) 创建根据URL或者Blob创建一个URL
- [`URL.revokeObjectURL()`](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/revokeObjectURL) 销毁之前已经创建的URL实例

那么生成的这个URL,可以被用在任何使用URL的地方,在这个例子中我们读取一个图片,然后将它赋值给`img`标签的`src`属性,这会在你的浏览器中打开一张图片.

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>blob</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>

<body>
    <label for="file">读取文件<input id="file" accept="image/*" type="file" ></label>
    <img id="img" src="" alt="">
    <script type="text/javascript">
        document.getElementById('file').addEventListener('change',(event)=>{

            const files = event.srcElement.files;

            if(files.length === 0){
                return console.log('没有选择任何内容');
            }

            const file = files[0];

            document.getElementById('img').src = URL.createObjectURL(file);
            
        });
    </script>
</body>

</html>
```

我们的图片被如下格式的URL所描述:

```
blob:http://127.0.0.1:5500/b285f19f-a4e2-48e7-b8c8-5eae11751593
```

## 导出文件实践

利用浏览器在解析到MIME为`application/octet-stream`类型的内容是会自动提供下载的特性.

我们有如下对策:

1. 创建一个File对象修改他的type为`application/octet-stream`
2. 使用这个File利用`URL.createObjectURL()`创建一个URL
3. 重定向到这个URL,让浏览器自动弹出下载框

```javascript
const
    buffer = new ArrayBuffer(1024),
    array = new Int8Array(buffer);

array.fill(1);

const 
    blob = new Blob(array),
    file = new File([blob],'test.txt',{
        lastModified:Date.now(),
        type:'application/octet-stream'
    });

saveAs(file,'test.txt')

const url = window.URL.createObjectURL(file);

window.location.href = url;
```

上面这种方式简单粗:joy:,不过导出的文件你得修改文件名称.

我们只需要稍稍利用利用a标签就可以优雅的完成这项任务:

```javascript
const
    buffer = new ArrayBuffer(1024),
    array = new Int8Array(buffer);

array.fill(1);

const 
    blob = new Blob(array),
    file = new File([blob],'test.txt',{
    lastModified:Date.now(),
        type:'text/plain;charset=utf-8'
    });

const 
    url = window.URL.createObjectURL(file),
    a = document.createElement('a');

a.href = url;
a.download = file.name; // see https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a#%E5%B1%9E%E6%80%A7
a.click();
```

大功告成,利用HTML5的API我们终于可以愉快的在WEB上操作数据啦!

# MDN上几篇不错的指引

分别是:

- [在web应用程序中操作文件指南](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications)
- [JavaScript 类数组对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Typed_arrays)
- [Base64的编码与解码](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowBase64/Base64_encoding_and_decoding)

# 参考

> https://github.com/SheetJS/js-xlsx
>
> https://github.com/eligrey/FileSaver.js

