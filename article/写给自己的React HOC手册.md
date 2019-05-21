# 前言

HOC(高阶组件)是React中的一种组织代码的手段,而不是一个API.  

这种设计模式可以**复用在React组件中的代码与逻辑**,因为一般来讲React组件只能复用视图层也就是HTML部分.

高级组件实际上是函数,这类函数接收React组件处理传入的组件,然后返回一个新的组件.  
**注意**:前提是建立在不修改原有组件的基础上.

文字描述太模糊,借助于官方文档稍稍修改,我们可以更加轻松的理解高阶组件.

# 具体的实施

流程如下:
1. 找出组件中复用的逻辑
2. 创建适用于上方逻辑的函数
3. 利用这个函数来创建一个组件
4. enjoy it

## 找出组件中复用的逻辑

在实际开发中, 这种逻辑的组件非常常见:
1. 组件创建
  2. 向服务器拉取数据
3. 利用数据渲染组件
4. 监听数据的变化
5. 数据变化或者触发修改的事件
6. 利用变化后的数据再次渲染
7. 组件销毁移除监听的数据源

首先我们来创建一个生产假数据的对象来模拟数据源:
```javascript
const fakeDataGenerator = ()=>({
  timer: undefined,
  getData(){
    return ['hello', 'world'];
  },
  addChangeListener(handleChangeFun){ // 监听数据产生钩子

    if(this.timer){
      return;
    }

    this.timer = setInterval(()=> {
      handleChangeFun();
    },2000)
  },
  removeChangeListener(){ // 停止数据监听
    clearInterval(this.timer);
  }
});
```
然后来编写我们的组件A:
```javascript
const FakeDataForA = fakeDataGenerator();

class A extends React.Component {

  constructor(props) {// 1 组件创建
    super(props);

    this.state = {
      someData: fakeData.getData() // 1.1 向服务器拉取数据
    }

  }

  handleFakeDataChange = ()=>{ 
    this.setState({
      someData:fakeData.getData() // 4. 数据变化或者触发修改的事件
    });
  }

  componentDidMount(){
    // 3. 监听数据的变化
    // 4. 数据变化或者触发修改的事件
    fakeData.addChangeListener(this.handleFakeDataChange); 
  }

  componentWillUnmount(){
    fakeData.removeChangeListener(); // 6. 组件销毁移除监听的数据源
  }

  render() {
    return (
      {/*
        2. 利用数据渲染组件
        5. 利用变化后的数据再次渲染
      */}
      this.state.someData.map(name => (<span key={name}>{name}</span>)) 
    )
  }
}

ReactDOM.render(<A />, document.getElementById('root'));
```

然后我们再来创建一个组件B这个虽然渲染方式不同,但是数据获取的逻辑是一致的.  
在一般的开发过程中实际上也是遵循这个请求模式的,然后创建一个组件B:
```javascript
const FakeDataForB = fakeDataGenerator();

class B extends React.Component {

  constructor(props) {// 1 组件创建
    super(props);

    this.state = {
      someData: fakeData.getData() // 1.1 向服务器拉取数据
    }

  }

  handleFakeDataChange = ()=>{ 
    this.setState({
      someData:fakeData.getData() // 4. 数据变化或者触发修改的事件
    });
  }

  componentDidMount(){
    // 3. 监听数据的变化
    // 4. 数据变化或者触发修改的事件
    fakeData.addChangeListener(this.handleFakeDataChange); 
  }

  componentWillUnmount(){
    fakeData.removeChangeListener(); // 6. 组件销毁移除监听的数据源
  }

  render() {
    return (
      {/*
        2. 利用数据渲染组件
        5. 利用变化后的数据再次渲染
      */}
      this.state.someData.map(name => (<div key={name}>{name}</div>)) 
    )
  }
}

ReactDOM.render(<B />, document.getElementById('root'));
```
这里我把redner中原来渲染的`span`标签改为了`div`标签,虽然这是一个小小的变化但是请你脑补这是两个渲染结果完全不同的组件好了.  

这时候问题以及十分明显了**组件A和B明显有大量的重复逻辑但是借助于React组件却无法将这公用的逻辑来抽离**.

在一般的开发中没有这么完美重复的逻辑代码,例如在生命周期函数中B组件可能多了几个操作或者A组件数据源获取的地址不同.  
但是这里依然存在大量的可以被复用的逻辑.  

## 一个返回组件的函数

这种函数的第一个参数接收一个React组件,然后返回这个组件:
```javascript
function MyHoc(Wrap) {
  return class extends React.Component{
    render(){
      <Wrap ></Wrap>
    }
  }
}
```
就目前来说这个函数没有任何实际功能只是将原有的组件包装返回而已.  

但是如果我们将组件A和B传入到这个函数中,而使用返回的函数,我们可以得到了什么.  
我们获取了在原有的组件上的一层包装,利用这层包装我们可以把组件A和B的共同逻辑提取到这层包装上.  

我们来删除组件A和B有关数据获取以及修改的操作:
```javascript
class A extends React.Component {

  componentDidMount(){
    // 这里执行某些操作 假设和另外一个组件不同
  }

  componentWillUnmount(){
    // 这里执行某些操作 假设和另外一个组件不同
  }

  render() {
    return (
      this.state.data.map(name => (<span key={name}>{name}</span>))
    )
  }
}

class B extends React.Component {

  componentDidMount(){
    // 这里执行某些操作 假设和另外一个组件不同
  }

  componentWillUnmount(){
    // 这里执行某些操作 假设和另外一个组件不同
  }

  render() {
    return (
      this.state.data.map(name => (<div key={name}>{name}</div>))
    )
  }
}
```

然后将在这层包装上的获取到的外部数据使用props来传递到原有的组件中:
```javascript
function MyHoc(Wrap) {
  return class extends React.Component{

    constructor(props){

      super(props);

      this.state = {
        data:fakeData // 假设这样就获取到了数据, 先不考虑其他情况
      }

    }

    render(){
      return <Wrap data={this.state.data} {...this.props}></Wrap> {/* 通过 props 把获取到的数据传入 */}
    }
  }
}
```
在这里我们在 HOC 返回的组件中获取数据, 然后把数据传入到内部的组件中, 那么数据获取的这种功能就被单独的拿了出来. 
这样组件A和B只要关注自己的 `props.data` 就可以了完全不需要考虑数据获取和自身的状态修改.

但是我们注意到了组件A和B原有获取数据源不同,我们如何在包装函数中处理?

这点好解决,利用函数的参数差异来抹消掉返回的高阶组件的差异.

既然A组件和B组件的数据源不同那么这个函数就另外接收一个数据源作为参数好了.  
并且我们将之前通用的逻辑放到了这个内部的组件上:
```javascript
function MyHoc(Wrap,fakeData) { // 这次我们接收一个数据源
  return class extends React.Component{

    constructor(props){

      super(props);
      this.state = {
        data: fakeData.getData() // 模拟数据获取
      }
      
    }

    handleDataChange = ()=>{
      this.setState({
        data:fakeData.getData()
      });
    }

    componentDidMount() {
      fakeData.addChangeListener(this.handleDataChange);
    }

    componentWillUnmount(){
      fakeData.removeChangeListener();
    }

    render(){
      <Wrap data={this.state.data} {...this.props}></Wrap>
    }
  }
}
```

## 利用高阶组件来创建组件

经过上面的思考,实际上已经完成了99%的工作了,接下来就是完成剩下的1%,把它们组合起来.

伪代码:
```javascript
const
  FakeDataForA = FakeDataForAGenerator(),
  FakeDataForB = FakeDataForAGenerator(); // 两个不同的数据源

function(Wrap,fakdData){ // 一个 HOC 函数
  return class extends React.Components{};
}

class A {}; // 两个不同的组件
class B {}; // 两个不同的组件

const 
  AFromHoc = MyHoc(A,FakeDataForA),
  BFromHoc = MyHoc(B,FakeDataForB); // 分别把不同的数据源传入, 模拟者两个组件需要不同的数据源, 但是获取数据逻辑一致
```
这个时候你就可以渲染自己的高阶组件`AFromHoc`和`BFromHoc`了.  
这两个组件使用不同的数据源来获取数据,通用的部分已经被抽离.  

# 函数约定

## HOC函数不要将多余的props传递给被包裹的组件

HOC函数需要像透明的一样,经过他的包装产生的新的组件和传入前没有什么区别.  
这样做的目的在于,我们不需要考虑经过HOC函数后的组件会产生什么变化而带来额外的心智负担.  
如果你的HOC函数对传入的组件进行了修改,那么套用这种HOC函数多次后返回的组件在使用的时候.  
你不得不考虑这个组件带来的一些非预期行为.  

所以请不要将原本组件不需要的props传入:
```javascript
render() {
  // 过滤掉非此 HOC 额外的 props，且不要进行透传
  const { extraProp, ...passThroughProps } = this.props;

  // 将 props 注入到被包装的组件中。
  // 通常为 state 的值或者实例方法。
  const injectedProp = someStateOrInstanceMethod;

  // 将 props 传递给被包装组件
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

## HOC是函数!利用函数来最大化组合性

因为HOC是一个返回组件的函数,只要是函数可以做的事情HOC同样可以做到.  
利用这一点,我们可以借用在使用React之前我们就已经学会的一些东西.

例如定义一个高阶函数用于返回一个高阶组件:
```javascript
function HighLevelHoc(content) {
  return function (Wrap, className) {
    return class extends React.Component {
      render() {
        return (
          <Wrap {...this.props} className={className} >{content}</Wrap>
        )
      }
    }
  }
}

class Test extends React.Component {
  render() {
    return (
      <p>{this.props.children || 'hello world'}</p>
    )
  }
}

const H1Test = HighLevelHoc('foobar')(Test, 1);


ReactDOM.render(<H1Test />, document.getElementById('root'));
```

或者干脆是一个不接收任何参数的函数:
```javascript
function DemoHoc(Wrap) { // 用于向 Wrap 传入一个固定的字符串
  return class extends React.Component{
    render(){
      return (
        <Wrap {...this.props}>{'hello world'}</Wrap>
      )
    }
  } 
}

function Demo(props) {
  return (
    <div>{props.children}</div>
  )
}

const App = DemoHoc(Demo);

ReactDOM.render(<App />, document.getElementById('root'));
```

# 注意

## 不要在 render 方法中使用 HOC 

我们都知道 React 会调用 `render` 方法来渲染组件, 当然 React 也会做一些额外的工作例如性能优化.  
在组件重新渲染的时候 React 会判断当前 render 返回的组件和未之前的组件是否相等 `===` 如果相等 React 会递归更新组件, 反之他会彻底的卸载之前的旧的版本来渲染当前的组件.  

HOC每次返回的内容都是一个新的内容:
```javascript
function Hoc(){
  return {}
}
console.log( Hoc()===Hoc() ) // false
```

如果在 `render` 方法中使用:
```javascript
render() {
  const DemoHoc = Hoc(MyComponent); // 每次调用 render 都会返回一个新的对象
  // 这将导致子树每次渲染都会进行卸载，和重新挂载的操作！
  return <DemoHoc />;
}
```

## 记得复制静态方法

React 的组件一般是继承 `React.Component` 的子类.  
不要忘记了一个类上除了实例方法外还有静态方法, 使用 HOC 我们对组件进行了一层包装会覆盖掉原来的静态方法:
```javascript
class Demo extends React.Component{
  render(){
    return (
      <div>{this.props.children}</div>
    )
  }
}

Demo.echo = function () {
  console.log('hello world');
}

Demo.echo();// 是可以调用的

// -------- 定一个类提供一个静态方法

function DemoHoc(Wrap) {
  return class extends React.Component{
    render(){
      return (
        <Wrap>{'hello world'}</Wrap>
      )
    }
  } 
}

const App = DemoHoc(Demo);

// ----- HOC包装这个类

App.echo(); // error 这个静态方法不见了
```

**解决方式**

在 HOC 内部直接将原来组件的静态方法复制就可以了:
```javascript
function DemoHoc(Wrap) {

  const myClass = class extends React.Component{
    render(){
      return (
        <Wrap>{'hello world'}</Wrap>
      )
    }
  }

  myClass.echo = Wrap.echo;

  return myClass;
}
```
不过这样一来 HOC 中就需要知道被复制的静态方法名是什么, 结合之前提到的灵活使用 HOC 我们可以让 HOC 接收静态方法参数名称:
```javascript
function DemoHoc(Wrap,staticMethods=[]) { // 默认空数组

  const myClass = class extends React.Component{
    render(){
      return (
        <Wrap>{'hello world'}</Wrap>
      )
    }
  }

  for (const methodName of staticMethods) { // 循环复制
    myClass[methodName] = Wrap[methodName];
  }

  return myClass;
}

// -----
const App = DemoHoc(Demo,['echo']);
```

此外一般我们编写组件的时候都是一个文件对应一个组件, 这时候我们可以把静态方法导出.  
HOC 不拷贝静态方法, 而是需要这些静态方法的组件直接引入就好了:
> 来自官方文档
```javascript
// 使用这种方式代替...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...单独导出该方法...
export { someFunction };

// ...并在要使用的组件中，import 它们
import MyComponent, { someFunction } from './MyComponent.js';
```

## 透传 ref

`ref` 作为组件上的特殊属性, 无法像普通的 `props` 那样被向下传递.  

例如我们有一个组件, 我们想使用 ref 来引用这个组件并且试图调用它的 `echo` 方法:
```javascript
class Wraped extends React.Component{
  constructor(props){
    super(props);
    this.state = {
      message:''
    }
  }

  echo(){
    this.setState({
      message:'hello world'
    });
  }

  render(){
    return <div>{this.state.message}</div>
  }
}
```

我们使用一个 HOC 包裹它:
```javascript
function ExampleHoc(Wrap) {
  return class extends React.Component{
    render(){
      return <Wrap></Wrap>
    }
  }
}

const Example = ExampleHoc(Wraped);
// 得到了一个高阶组件
```

现在我们把这个组件放入到 APP 组件中进行渲染, 并且使用 `ref` 来引用这个返回的组件, 并且试图调用它的 `echo` 方法:
```javascript
const ref = React.createRef();

class App extends React.Component {

  handleEcho = () => {
    ref.current.echo();
  }

  render() {
    return (
      <div>
        <Example ref={ref}></Example>
        <button onClick={this.handleEcho}>echo</button> {/* 点击按钮相当于执行echo */}
      </div>
    )
  }
}
```
但是当你点击按钮试图触发子组件的事件的时候它不会起作用, 系统报错没有 `echo` 方法.  

实际上 `ref` 被绑定到了 HOC 返回的那个匿名类上, 想要绑定到内部的组件中我们可以进行 `ref` 透传.  
默认的情况下 `ref` 是无法被进行向下传递的因为 `ref` 是特殊的属性就和 `key` 一样不会被添加到 `props` 中, 因此 React 提供了一个 `API` 来实现透传 `ref` 的这种需求.  

这个 `API` 就是 `React.forwardRef`.  

这个方法接收一个函数返回一个组件, 在这个含中它可以读取到组件传入的 `ref` , 某种意义上 `React.forwardRef` 也相当于一个高阶组件:
```javascript
const ReturnedCompoent = React.forwardRef((props, ref) => {
  // 我们可以获取到在props中无法获取的 ref 属性了
  return // 返回这个需要使用 ref 属性的组件
});
```

我们把这个 `API` 用在之前的 HOC 中:
```javascript
function ExampleHoc(Wrap) {
  class Inner extends React.Component {
    render() {
      const { forwardedRef,...rest} = this.props;
      return <Wrap ref={forwardedRef} {...rest} ></Wrap> // 2. 我们接收到 props 中被改名的 ref 然后绑定到 ref 上
    }
  }
  return React.forwardRef((props,ref)=>{ // 1. 我们接收到 ref 然后给他改名成 forwardedRef 传入到props中
    return <Inner {...props} forwardedRef={ref} ></Inner>
  })
}
```

这个时候在调用 `echo` 就没有问题了:
```javascript
handleEcho = () => {
  ref.current.echo();
}
```