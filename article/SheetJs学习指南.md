# 简介

SheetJs是前端操作Excel的最佳选择,而`js-xlsx`是它的社区版本.

`js-xlsx`将注意力集中到了数据转换和导出上,所以它支持相当多种类的数据解析和导出.不仅仅局限于支持xlsx格式.

[支持的导入格式](https://github.com/SheetJS/js-xlsx#guessing-file-type)

[支持的导出格式](https://github.com/SheetJS/js-xlsx#supported-output-formats)

它可以:

- 解析符合格式的数据
- 导出符合格式的数据
- 利用中间层来操作数据

可以运行在:

- 浏览器端
- Node端

## 浏览器端特色

- 纯浏览器端解析数据
- 纯浏览器端导出数据

## Node端特色

- 读写文件
- 流式读写

本篇文章力求精简,主要讨论一下`js-xlsx`的工作流程和基本概念以及使用方式.

# 概念

`js-xlsx`提供了一个中间层用于操作数据,他将不同类型的文件抽象成同一个js对象,从而规避了操作不同种类数据数据之间的复杂性.

并且围绕着这个对象提供了一系列的抽象功能,本小节主要讨论这些数据对象与Excel数据之间的关系.

而浏览器端和Node端的区别仅仅在于怎样导入文件和导出文件上而已,对于数据的操作,双方的接口是一致的.

## 引入

`js-xlsx`的引入非常简单,浏览器端引入可以是最基本`script`标签的形式.

```html
<script lang="javascript" src="dist/xlsx.full.min.js"></script>
```

在node端,使用npm安装如下模块:

```bash
npm install xlsx --save
```

在Node中如下引入:

```javascript
const xlsx = require('xlsx');
```

[详细文档地址](https://github.com/SheetJS/js-xlsx#installation)

## 对应关系

在这个表格中我列举了Excel与`js-xlsx`之间的关系:

| Excel名词                 | js-xlsx中的抽象类型 |
| ------------------------- | ------------------- |
| 工作簿                    | workBook            |
| 工作表                    | Sheets              |
| Excel引用样式(单元格地址) | cellAddress         |
| 单元格                    | cell                |

有了这个基本的对应关系我们就可以轻松的理解后续的操作,例如在我们使用Excel的过程中,获取一个数据的流程如下:

1. 打开工作簿
2. 打开一个工作表
3. 选中一片区域或者一个单元格
4. 针对数据进行操作
5. 保存(另存为)

那么在`js-xlsx`中获取一个单元格内容的[操作如下](https://github.com/SheetJS/js-xlsx#working-with-the-workbook):

```javascript
// 先不要关心我们的workbook对象是从哪里来的
var first_sheet_name = workbook.SheetNames[0]; // 获取工作簿中的工资表名字
var address_of_cell = 'A1'; // 提供一个引用样式(单元格下标)

var worksheet = workbook.Sheets[first_sheet_name]; // 获取对应的工作表对象

var desired_cell = worksheet[address_of_cell]; // 获取对应的单元格对象

var desired_value = (desired_cell ? desired_cell.v : undefined);// 获取对应单元格中的数据
```

## 数据格式

__图片:工作簿的数据结构__

![1548853228009](F:\library\article\assets\1548853228009.png)

一旦我们的Excel文件被解析那么这个Excel表中的所有内容都会被解析上面的这个对象.而且这整个过程是同步完成的.

所以我们可以使用键的方式来直接获取数据,在上面的例子中我们就利用键一层层的向下获取数据.

上图中常用的键一共有两个:

- `SheetNames`以字符串数组的形式保存了所有的工作表的名称
- Sheets下的内容都是工作表,而键名就是`SheetNames`中的

而Excel的数据单位由小到大有如下排序如下:

- 单元格
- 工作表
- 工作簿

### 单元格格式

在Excel中单元格有多种格式,而`js-xlsx`会将其解析为对应的JavaScript的格式.

**常见格式**如下:

| 键   | 描述                                 |
| ---- | ------------------------------------ |
| v    | 源数据(未经处理的数据)               |
| w    | 格式化后的文本(如果能够被格式化)     |
| t    | 单元格类型(具体类型请看下方的表格)   |
| r    | 解码后的富文本(如果可以被解码)       |
| h    | 渲染成HTML格式的富文本(如果可以的话) |
| c    | 单元格注释                           |
| z    | 格式化成字符串的数值(如果需要的话)   |

>  完整格式[链接](https://github.com/SheetJS/js-xlsx#cell-object)

**t表示的类型如下**:

| 类型(字符串) | 类型    | 描述                      |
| ------------ | ------- | ------------------------- |
| b            | Boolean | Boolean:布尔类型          |
| e            | Error   | Number:错误代码           |
| n            | Number  | Number:数值               |
| d            | Date    | Date:数据被解析为Date     |
| s            | Text    | String:数据被解析为String |
| z            | Stub    | Excel中保留的空白单元格   |

[详细文档地址-包含错误代码以及完整的列表](https://github.com/SheetJS/js-xlsx#data-types).

__解析后单元格数据格式:__

![1548774339329](F:\library\article\assets\1548774339329.png)

这个数据在Excel中保存在A1的位置上,文本类型,单元格内容为`xm`.

### 单元格地址

`js-xlsx`使用有两种方式来描述操作中的单元格区域.

一种是[单元格地址对象](https://github.com/SheetJS/js-xlsx#general-structures)(Cell address object)另外一种是地址范围(Cell range).

地址对象格式如下:

```javascript
const start = { c: 0, r: 0 };
const end = { c: 1, r: 1 };
```

上方地址对象对应的范围如下:

```javascript
const range = 'A1:B2';
```

我们不难发现两者之间对应的关系:

- 地址对象描述的是一个起始坐标(从0开始)到结束坐标之间的范围.

- 地址范围就是Excel中的引用样式.

**注意**:这两个概念会在工作表读写中使用到.

# API

`js-xlsx`提供的接口非常清晰主要分为两类:

- `xlsx`对象本身提供的功能
  - 解析数据
  - 导出数据
- `utils`工具类
  - 将数据添加到数据表对象上
  - 将二维数组以及符合格式的对象或者HTML转为工作表对象
  - 将工作簿转为另外一种数据格式
  - 行,列,范围之间的转码和解码
  - 工作簿操作
  - 单元格操作

## 读取数据并解析

这里提供一个简单的Node例子(Node10+):

```javascript
const xlxs = require('xlsx');
const {readFile} = require('fs').promises;

(async function (params) {
    
    // 获取数据
    const excelBuffer = await readFile('./books.xlsx');
    
    // 解析数据
    const result = xlxs.read(excelBuffer,{
        type:'buffer',
        cellHTML:false,
    });
    
    console.log('TCL: result', result);

})();
```

还可以使用`utils.book_new()`创建一个新的工作簿对象:

```javascript
const
    xlsx = require('xlsx'),
    { utils } = xlsx;

const workBook= utils.book_new(); // 创建一个工作簿
```

然后使用跟多的工具来操作工作簿对象:

```javascript
// 接着上面的例子

const ws_data = [
  [ "S", "h", "e", "e", "t", "J", "S" ],
  [  1 ,  2 ,  3 ,  4 ,  5 ]
];

const workSheet = XLSX.utils.aoa_to_sheet(ws_data);// 使用二维数组创建一个工作表对象

utils.book_append_sheet(workBook,workSheet,'工作表名称');// 向工作簿追加一个工作表

console.log(workBook);
```

[详细的解析文档](https://github.com/SheetJS/js-xlsx#parsing-workbooks)

[详细解析选项](https://github.com/SheetJS/js-xlsx#parsing-options)

## 数据填充

工作表是实际存放数据的地方,在大部分情况下我们的操作都是对于工作表对象的操作.

`js-xlsx`提供了多种方式来操作数据,这里提供最常见的几种操作:

- 利用现有的数据结构创建工作表
  - 二维数组
  - JSON
- 修改工作表数据
  - 二维数组
  - JSON

### 创建工作表

```javascript
const workSheet = utils.aoa_to_sheet([[1,2,3,new Date()],[1,2,,4]],{
    sheetStubs:false,
    cellStyles:false,
    cellDates:true // 解析为原生时间
});

console.log(workSheet);
```

二维数组的关系非常容易理解,数组中的每一个数组代表一行.

__图片:二维数组结果__

![1548857097733](F:\library\article\assets\1548857097733.png)

```javascript
const workSheet = 
utils.json_to_sheet([
{ '列1': 1, '列2': 2, '列3': 3 },
{ '列1': 4, '列2': 5, '列3': 6 }
],{
    header:['列1','列2','列3'],
    skipHeader:true// 跳过上面的标题行
})

console.log(workSheet);
```

__图片:JSON效果__

![1548857328315](F:\library\article\assets\1548857328315.png)

[详细文档地址](https://github.com/SheetJS/js-xlsx#utility-functions)

### 修改数据表数据

```javascript
const workSheet = 
utils.json_to_sheet([
{ '列1': 1, '列2': 2, '列3': 3 },
{ '列1': 4, '列2': 5, '列3': 6 }
],{
    header:['列1','列2','列3'],
    skipHeader:true// 跳过上面的标题行
})

utils.sheet_add_aoa(workSheet,[
    [7,8,9],
    ['A','B','C']
],{
    origin:'A1' // 从A1开始增加内容
});


console.log(workSheet);
```

__图片:二维数组结果__

![1548857673280](F:\library\article\assets\1548857673280.png)

```javascript
const workSheet = 
utils.json_to_sheet([
{ '列1': 1, '列2': 2, '列3': 3 },
{ '列1': 4, '列2': 5, '列3': 6 }
],{
    header:['列1','列2','列3'],
    skipHeader:true// 跳过上面的标题行
})

utils.sheet_add_json(workSheet,[
    { '列1': 7, '列2': 8, '列3': 9 },
    { '列1': 'A', '列2': 'B', '列3': 'C' }
],{
    origin:'A1',// 从A1开始增加内容
    header: ['列1', '列2', '列3'],
    skipHeader: true// 跳过上面的标题行
});


console.log(workSheet);
```

__图片:JSON效果__
![1548857865584](F:\library\article\assets\1548857865584.png)

[详细文档地址](https://github.com/SheetJS/js-xlsx#utility-functions)

## 数据导出

数据导出分为两个部分:

- 利用工具类将工作簿对象转为其他数据结构
- 调用`write`或者`writeFile`方法

### 转换为其他的数据结构

这里就不提供详细的用例了,可以转换的格式如下:

![1548858135268](F:\library\article\assets\1548858135268.png)

[详细文档地址](https://github.com/SheetJS/js-xlsx#formulae-output)

### 输出文件

这里提供一个简单的Node例子(Node10+):

```javascript
const
    xlsx = require('xlsx'),
    { utils } = xlsx;

const {writeFile} =require('fs').promises;

const workBook= utils.book_new();
const workSheet = utils.aoa_to_sheet([[1,2,3]],{
    cellDates:true,
});

// 向工作簿中追加工作表
utils.book_append_sheet(workBook, workSheet,'helloWorld');

// 浏览器端和node共有的API,实际上node可以直接使用xlsx.writeFile来写入文件,但是浏览器没有该API
const result = xlsx.write(workBook, {
    bookType: 'xlsx', // 输出的文件类型
    type: 'buffer', // 输出的数据类型
    compression:true // 开启zip压缩
});

// 写入文件
writeFile('./hello.xlsx',result)
.catch((error)=>{
    console.log(error);
});

```

[write方法文档以及输出选项](https://github.com/SheetJS/js-xlsx#writing-options)

[支持的输出文件格式](https://github.com/SheetJS/js-xlsx#file-formats)

# 引用

> https://github.com/SheetJS/js-xlsx