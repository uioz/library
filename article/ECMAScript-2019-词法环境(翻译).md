网上有很多与执行上下文的文章, 绝大多数还讨论的是 ES3(2002) 中的内容.

这篇文章是对 ECMAScript2019 规范一个章节部分内容的翻译, 主要内容是 "执行上下文与可执行代码".

> https://www.ecma-international.org/ecma-262/10.0/index.html#sec-executable-code-and-execution-contexts

我英语是真的不行😶, 这篇文章中估计存在一些错误, 不过这篇文章我应该会持续更新, 直到可以描述 JavaScript 的基本运行为止.

# 词法环境

词法环境是一种规范类型，它使用 ECMAScript 词法嵌套的结构去定义标识符与特定变量和函数的关联. 词法环境由环境记录和可能为 null 的外部词法环境引用. 通常一个词法环境和具体的 ECMAScript 代码的语法结构相关, 例如 函数声明, 代码块, try 语句的 catch 块和每次代码被求值时所创建的词法环境.

"环境记录"中记录标识符绑定, 这些标识符绑定在其相关联的词法环境作用域中创建. 它被称为词法环境的 "环境记录".

外部环境引用用于词法环境值的逻辑嵌套建模. 内部词法环境的外部引用引用着逻辑上围绕着内部词法环境的外部词法环境. 当然, 外部词法环境也会拥有它的外部词法环境. 一个词法环境作为外部环境时可以作为多个内部环境的外部环境. 例如, 如果一个函数声明内部拥有两个函数声明这些嵌套函数的词法环境的外部词法环境就是当前调用它们的函数的词法环境.

全局词法环境是一种没有外部环境的词法环境. 也就是说全局词法环境的外部环境的引用时 null. 全局词法环境的环境记录可能被标识符绑定以及一个关联的全局对象该对象属性上提供了某些全局环境的标识符绑定所预填充. 当 ECMAScript 代码一旦执行, 额外的属性可能会被添加到全局对象上而最初的属性也可能会被修改.

模块的环境是一个包含模块的顶级声明的绑定的词法环境. 它同时也包含那些通过 import 导入的具体绑定. 模块环境的的外部环境是全局环境.

函数环境是对应着 ECMAScript 函数对象(function object) 调用的词法环境. 函数环境可能会建立一个新的的 this 绑定. 函数环境同时捕获那些需要支持 super 方法调用的状态.

词法环境以及环境记录值是存粹的规范概念, 不需要 ECMAScript 实例有对应的具体实现. 一段 ECMAScript 程序是无法直接访问这些值的.

## 环境记录

在这个规范中有两种主要的环境记录值:

1. 声明式的环境记录
   1. 定义 ECMAScript 语法元素例如 函数声明, 变量声明, 以及 Catch 子句的效果, 这些元素直接将标识符绑定与 ECMAScript 语言值进行关联.
2. 对象式环境记录
   1. 定义 ECMAScript 元素例如 将标识符绑定与某个对象属性关联起来的 with 语句.

全局环境记录和函数环境记录是专门用于脚本全局声明和函数内顶级声明的特殊环境记录.

出于规范化的目的环境记录值是记录规范类型值且可以通过一个简单的面向对象的结构理解, 结构如下

- 环境记录
  - 声明式环境记录
    - 函数环境记录
    - 模块环境记录
  - 对象式环境记录
  - 全局环境记录

抽象类包含了定义在 [表14](https://www.ecma-international.org/ecma-262/10.0/index.html#table-15)  中的抽象规范方法. 这些抽象方法对于不同的字类都有不同的具体算法.

表14: 环境记录的抽象方法

| 方法                         | 含义                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| HasBinding(N)                | Determine if an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records) has a binding for the String value N. Return true if it does and false if it does not. |
| CreateMutableBinding(N, D)   | Create a new but uninitialized mutable binding in an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records). The String value N is the text of the bound name. If the Boolean argument D is true the binding may be subsequently deleted. |
| CreateImmutableBinding(N, S) | Create a new but uninitialized immutable binding in an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records). The String value N is the text of the bound name. If S is true then attempts to set it after it has been initialized will always throw an exception, regardless of the strict mode setting of operations that reference that binding. |
| InitializeBinding(N, V)      | Set the value of an already existing but uninitialized binding in an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records). The String value N is the text of the bound name. V is the value for the binding and is a value of any [ECMAScript language type](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-ecmascript-language-types). |
| SetMutableBinding(N, V, S)   | Set the value of an already existing mutable binding in an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records). The String value N is the text of the bound name. V is the value for the binding and may be a value of any [ECMAScript language type](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-ecmascript-language-types). S is a Boolean flag. If S is true and the binding cannot be set throw a TypeError exception. |
| GetBindingValue(N, S)        | Returns the value of an already existing binding from an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records). The String value N is the text of the bound name. S is used to identify references originating in [strict mode code](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-strict-mode-code) or that otherwise require strict mode reference semantics. If S is true and the binding does not exist throw a ReferenceError exception. If the binding exists but is uninitialized a ReferenceError is thrown, regardless of the value of S. |
| DeleteBinding(N)             | Delete a binding from an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records). The String value N is the text of the bound name. If a binding for N exists, remove the binding and return true. If the binding exists but cannot be removed return false. If the binding does not exist return true. |
| HasThisBinding()             | Determine if an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records) establishes a `this` binding. Return true if it does and false if it does not. |
| HasSuperBinding()            | Determine if an [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records) establishes a `super` method binding. Return true if it does and false if it does not. |
| WithBaseObject()             | If this [Environment Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-environment-records) is associated with a `with` statement, return the with object. Otherwise, return undefined. |

后面的几个章节的内容就是分别详细介绍不同类型的环境记录对应的具体方法的算法, 内容太多翻译意义不大.

**术语对照**:

| 中文名称       | 英文名称                                                     |
| -------------- | ------------------------------------------------------------ |
| 词法环境       | Lexical Environment                                          |
| 环境记录       | Environment Record                                           |
| 标识符绑定     | identifier binding                                           |
| 声明式环境记录 | declarative Environment Record                               |
| 对象式环境记录 | object Environment Record                                    |
| 全局环境记录   | Global Environment Record                                    |
| 函数环境记录   | function Environment Record                                  |
| 记录规范类型   | the [Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-list-and-record-specification-type) specification type |

# 范围

在 ECMAScript 代码执行前必须关联到某个范围上. 从概念上将一个范围由下面的内容组成:

- 一组内置对象
- 全局环境
- 在全局环境作用域中加载完成的 ECMAScript 代码
- 以及其他关联的状态和资源

**注意**: 全局环境是一种没有外部环境引用的词法环境.

一个范围在规范中由范围记录表示, 而规范记录由下列  [Table 20](https://www.ecma-international.org/ecma-262/10.0/index.html#table-21) 字段组成:

| 字段名称         | 值                                                           | 含义                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [[Intrinsics]]   | [Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-list-and-record-specification-type) whose field names are intrinsic keys and whose values are objects | The intrinsic values used by code associated with this [realm](https://www.ecma-international.org/ecma-262/10.0/index.html#realm) |
| [[GlobalObject]] | Object                                                       | The [global object](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-global-object) for this [realm](https://www.ecma-international.org/ecma-262/10.0/index.html#realm) |
| [[GlobalEnv]]    | [Lexical Environment](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-lexical-environments) | The [global environment](https://www.ecma-international.org/ecma-262/10.0/index.html#global-environment) for this [realm](https://www.ecma-international.org/ecma-262/10.0/index.html#realm) |
| [[TemplateMap]]  | A [List](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-list-and-record-specification-type) of [Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-list-and-record-specification-type) { [[Site]]: [Parse Node](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-syntactic-grammar), [[Array]]: Object }. | Template objects are canonicalized separately for each [realm](https://www.ecma-international.org/ecma-262/10.0/index.html#realm) using its [Realm Record](https://www.ecma-international.org/ecma-262/10.0/index.html#realm-record)'s [[TemplateMap]]. Each [[Site]] value is a [Parse Node](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-syntactic-grammar) that is a [TemplateLiteral](https://www.ecma-international.org/ecma-262/10.0/index.html#prod-TemplateLiteral). The associated [[Array]] value is the corresponding template object that is passed to a tag function.NOTEOnce a [Parse Node](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-syntactic-grammar) becomes unreachable, the corresponding [[Array]] is also unreachable, and it would be unobservable if an implementation removed the pair from the [[TemplateMap]] list. |
| [[HostDefined]]  | Any, default value is undefined.                             | Field reserved for use by host environments that need to associate additional information with a [Realm Record](https://www.ecma-international.org/ecma-262/10.0/index.html#realm-record). |

本章节的其他部分就是将具体的算法, 东西太多就不翻译了.

**术语对照**:

| 中文名称 | 英文名称          |
| -------- | ----------------- |
| 范围     | realms            |
| 内置对象 | intrinsic objects |
| 范围记录 | realm record      |

# 执行上下文

执行上下文是一个特殊装置用于 ECMAScript 实例其运行时代码执行的追踪. 在任何时间, 每一个代理只能有一个执行上下文在执行代码. 这被称为代理的执行中执行上下文. 在本规范中所有 "执行中的执行上下文" 实际都是对 "围绕着代理的执行中的执行上下文" 的引用.

执行上下文栈用于追踪执行上下文. 执行中的执行上下文总是栈顶元素. 一个新的执行上下文总是当控制权从一个当前执行中的执行上下文相关的可执行代码转移到另外一个未关联执行上下文的可执行代码时被创建. 新创建的执行上下文被压到栈顶然后称为执行中的执行上下文.

一个执行上下文包含可执行程序相关的代码的所有状态而不管其是否有必要追踪. 每个执行上下文至少具有 [Table 21](https://www.ecma-international.org/ecma-262/10.0/index.html#table-22) 中列出的状态组件:

| 组件                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| code evaluation state(代码执行状态)                          | [执行上下文](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts)相关连代码执行, 暂停, 恢复执行, 所需要的全部状态. |
| Function(函数)                                               | 如果当前[执行上下文](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts)正在执行[函数对象](https://www.ecma-international.org/ecma-262/10.0/index.html#function-object)的代码, 那么组件的值就是这个[函数对象](https://www.ecma-international.org/ecma-262/10.0/index.html#function-object). 如果正在执行的上下文执行的是 [Script](https://www.ecma-international.org/ecma-262/10.0/index.html#prod-Script) 或者 [Module](https://www.ecma-international.org/ecma-262/10.0/index.html#prod-Module) 的代码, 这个是值就是 null. |
| Realm([范围](https://www.ecma-international.org/ecma-262/10.0/index.html#realm)) | 关联代码从中访问 ECMAScript 资源的[范围记录](https://www.ecma-international.org/ecma-262/10.0/index.html#realm-record). |
| ScriptOrModule(脚本或模块)                                   | 模块记录或者脚本记录与对应的源代码有关. 如果不存在起始脚本或者模块, 这种情况就像初始上下文通过 [InitializeHostDefinedRealm](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-initializehostdefinedrealm) 建立一样, 其值为 null. |

由执行中的执行上下文所执行的代码可以在本规范中定义的节点处暂缓执行. 一旦执行中的执行上下文暂停另外一个执行上下文就有可能称为运行中的执行上下文运行它的代码. 随后被暂停的执行上下文可能重新称为执行中的执行上下文在原来暂停处执行它剩余的代码. 运行中/执行上下文之间的状态转换以 后进/先出 的堆栈方式进行. 但是有些 ECMAScript 的特性要求运行中的执行上下文进行非 后进/先出 的过渡方式.

运行中的执行上下文的范围组件值又被称为当前范围记录. 运行中执行上下文的函数组件值也被称为活动函数对象.

ECMAScript 代码的执行上下文具有 [表22](https://www.ecma-international.org/ecma-262/10.0/index.html#table-23) 中提到的额外状态组件.

| 组件     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| 词法环境 | 识别这个执行上下文中的词法环境, 用于解析由代码产生的标识符引用. |
| 变量环境 | 识识别这个执行上下文中的词法环境, 中的环境记录持有的由变量声明所创建的绑定. |

执行上下文的词法环境和变量环境永远都是词法环境.

表示生成器对象求值的执行上下文具有表23中列出的其他状态组件。

Table 23: Additional State Components for Generator Execution Contexts

| 组件   | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| 生成器 | 此执行上下文正在执行的生[成器对象](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts)。 |

在大多数情况下只有运行中上下文(执行上下文栈最上面的那个)是直接被规范中的算法操纵的. 因此当术语 "词法环境" 和 "变量环境" 没有明确时, 所指代的是在运行中执行上下文对应的组件.

执行上下文存粹是规范机制, 不需要任何 ECMAScript 实例去创建与之对应的具体部件.

本章后面介绍的都是运行中执行上下文中的各个抽象算法, 内容太多这里就不翻译了.

**术语对照**:

| 中文名称               | 英文名称                                                     |
| ---------------------- | ------------------------------------------------------------ |
| 代理                   | agent                                                        |
| 代理的执行中执行上下文 | agent's running execution context                            |
| 初始上下文             | original [execution context](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts) |
| 当前范围记录           | current realm record                                         |
| 活动函数对象           | active function object                                       |
| 变量声明               | [VariableStatement](https://www.ecma-international.org/ecma-262/10.0/index.html#prod-VariableStatement) |
| 生成器对象             | GeneratorObject                                              |



















