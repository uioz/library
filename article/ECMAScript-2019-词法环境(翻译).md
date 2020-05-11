# 词法环境

词法环境是一种规范类型它定义标识符与 ECMAScript 代码词法嵌套中具体变量与函数之间的关联.

词法环境由环境记录和一个可能为 null 的外部环境引用.

通常一个词法环境和具体的 ECMAScript 代码的语法结构相关, 例如 函数声明, 代码块, try 语句的 catch 块, 和每次代码被求值时所创建的词法环境.

一个环境记录记录了那些在对应的词法环境作用域所创建的标识符绑定. 它被称为词法环境的 "环境记录".

外部环境引用用于对词法环境中的值的逻辑嵌套进行建模. 内部词法环境的外部引用引用着逻辑上围绕着内部词法环境的外部词法环境. 当然, 外部词法环境也会拥有它的外部词法环境. 一个词法环境作为外部环境时可以作为多个内部环境的外部环境. 例如, 如果一个函数声明内部拥有两个嵌套的函数声明那么每一个嵌套函数的词法环境都将其外部词法环境作为当前对周围函数求值的词法环境. (太拗口了, 简单来说每一个嵌套函数的外部环境引用就是这个定义了两个函数的词法环境.)

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

出于规范化的目的环境记录值是记录规范类型中的值而且可以通过一个简单的面向对象方式的方式想象, 其结构如下

- 环境记录
  - 声明式环境记录
    - 函数环境记录
    - 模块环境记录
  - 对象式环境记录
  - 全局环境记录

抽象类包含了定义在 [表14](https://www.ecma-international.org/ecma-262/10.0/index.html#table-15)  中的抽象规范方法. 这些抽象方法对于不同的字类都有不同的具体算法.

Table 14: Abstract Methods of Environment Records

| Method                       | Purpose                                                      |
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
| 词法环境       | Lexical Environments                                         |
| 环境记录       | Environment Records                                          |
| 标识符绑定     | identifier bindings                                          |
| 声明式环境记录 | declarative Environment Records                              |
| 对象式环境记录 | object Environment Records                                   |
| 全局环境记录   | Global Environment Records                                   |
| 函数环境记录   | function Environment Records                                 |
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
| 代码执行状态                                                 | [执行上下文](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts)相关连代码执行, 暂停, 恢复执行, 所需要的全部状态. |
| 函数                                                         | 如果当前[执行上下文](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts)正在执行[函数对象](https://www.ecma-international.org/ecma-262/10.0/index.html#function-object)的代码, 那么组件的值就是这个[函数对象](https://www.ecma-international.org/ecma-262/10.0/index.html#function-object). 如果正在执行的上下文执行的是 [Script](https://www.ecma-international.org/ecma-262/10.0/index.html#prod-Script) 或者 [Module](https://www.ecma-international.org/ecma-262/10.0/index.html#prod-Module) 的代码, 这个是值就是 null. |
| [范围](https://www.ecma-international.org/ecma-262/10.0/index.html#realm) | 关联代码从中访问 ECMAScript 资源的[范围记录](https://www.ecma-international.org/ecma-262/10.0/index.html#realm-record). |
| 脚本还是模块                                                 | The [Module Record](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-abstract-module-records) or [Script Record](https://www.ecma-international.org/ecma-262/10.0/index.html#script-record) from which associated code originates. If there is no originating script or module, as is the case for the original [execution context](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-execution-contexts) created in [InitializeHostDefinedRealm](https://www.ecma-international.org/ecma-262/10.0/index.html#sec-initializehostdefinedrealm), the value is null. |

| 中文名称               | 英文名称                          |
| ---------------------- | --------------------------------- |
| 代理                   | agent                             |
| 代理的执行中执行上下文 | agent's running execution context |



















