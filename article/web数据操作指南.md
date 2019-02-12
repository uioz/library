# 简介

本篇文章主要探讨JavaScript中的数据操作.

JavaScript一直以来给人一种比较低能的感觉,例如无法读取系统上的文件,不能做一些底层的操作.

所以在页面上操作数据会交由服务器处理也就成了主流的做法.

但是很多人没有发现,实际上JavaScript以及在逐步增强这些功能,例如我们已经可以只在浏览器端进行读写文件了.

我个人认为这些功能不流行的原因,除了需求较小外,兼容性,以及不利于理解的API甚至没有过多介绍的文章都在客观上造成了某些影响.

# 涉及的内容

没有非必要的内容,对于文件操作来说只有必要的方式.

- Blob
- ArrayBuffer
- TypedArray
- DataView
- FileReader
- File
- URL

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
| Blob | 用于在JavaScript表示文件(本身不包含文件的内容) | 
| File | 用于表示文件对象(包含文件内容) |
| ArrayBuffer | 表示Buffer(仅仅提供一片内存空间) |
| TypedArray | 基于字节操作Buffer上的数据(拥有不同类型的储存方式) | 
| DataView | 基于字节操作Buffer上的数据 | 

上面描述的内容之间的关系很复杂,这里我们逐步来进行分析.

# 兼容性

我没有详细考证API的兼容性,不过从MDN提供的数据来看IE10以上的浏览器大部分都是兼容的.

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
  - 使用xxx写入
- 读取
  - 使用xxx读取

<!-- TODO 读写测试 -->

```javascript
```

## 字节序

简单来讲-使用DataView:
- 读取不需要考虑字节序的问题
- 写入需要考虑不同平台

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView#%E5%AD%97%E8%8A%82%E5%BA%8F

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

# Blob



