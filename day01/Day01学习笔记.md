# Day01学习笔记

## 为什么要学习react

设计很优秀，性能强大

大公司开源框架，稳定性保证

社区强大（全球）

`ReactNative`开发混合APP

## react的虚拟DOM和Diff算法

### 虚拟DOM

虚拟DOM的本质就是JS声明的UI元素

真实DOM是浏览器渲染的UI元素

而利用diff算法，把两者结合起来，更快速，更高效，更稳定的渲染

至于diff算法，以后再去了解

## JSX语法

1. 如果要使用 JSX 语法，必须先运行 `npm i babel-preset-react -D` 然后在 `.babelrc`中进行配置，才能支持 JSX 语法。
2. JSX 语法的本质还是以 `React.createElement`的形式来实现的
3. 如果要在 JSX 语法内部书写 JS 代码，那么所有的 JS 代码都必须写在 {} 内部
4. 当编译 JSX 代码的时候，遇到 < 那么会当做 html 标签解析，遇到 {} 就当 JS 代码去编译
5. 在 {} 的内部，可以书写任何符合规范的 JS 代码
6. 在 JSX 创建 DOM 的时候，所有的节点必须用唯一的根元素包裹
7. 注释也要用 {} 包起来

## 创建组件

### 无状态组件

无状态组件顾名思义，是不包含内部状态的，只能根据传入props值来展示，不涉及到state状态的操作，内部一切都是静态的。

无状态组件实际上是只有render方法的组件类，一般会使用函数形式来创建，并且是没有state状态

```javascript
function HelloComponent(props){
	return <div>Hello {props.name}</div>
}
```

无状态的组件的优点：

- 可读性更高，减少冗余代码
- 组件不会被实例化，整体渲染性能得到提升
- 组件没有this对象
- 组件没有生命周期
- 只能访问传入的props，减少bug

### ES6方法创建组件

`React.Component` 是以 ES6 的形式来创建 react 组件，是创建组件最好的方式

```javascript
class HelloComponent extends React.Component {
    // 构造器 
    constructor(props){
        // 必须的
        super(props);
        // state 状态
        this.state = {
            name: props.name || 'zs'
        }
        // 过时了，现在不需要了，用箭头函数的方式就好了
        // ES6 类中函数必须手动绑定，bind是绑定了返回一个函数
        // this.HelloWorld = this.HelloWorld.bind(this);
    }
    HelloWorld = (event) => {
        // 更新 name 值
        this.setState({
            name: event.target.value;
        })
    }
    render(){
        return (
        	<div>
            	<input onChange={this.HelloWorld} value={this.state.name} />
            </div>
        )
    }
}
```

## 组件通信

### 父组件向子组件通信

react 数据流是单向的，父组件向子组件通信也是最常见的，父组件通过 props 向子组件传递需要的信息

Child.jsx

```javascript
import React from 'react';
import PropTypes from 'prop-types';

exprot default function Child({name}){
    return <h1>Hello {name}</h1>
}
Child.propTypes = {
    name: PropTypes.string.isRequired,
};
```

Parent.jsx

```javascript
import React, { Component } from 'react';
import Child from './Child';

export default class Parent extends Component{
    render(){
        return (
        	<div>
            	<Child name='zs' />
            </div>
        )
    }
}
```

父组件通过在子组件添加name属性，把这个参数传递过去

子组件利用props来接收

### 子组件向父组件通信

List.jsx

```javascript
import React, {Component} from 'react';
import PropTypes from 'prop-types';
 
export default class List extends Component {
    static propTypes = {
        hideConponent： PropTypes.func.isRequired,
    }
	render(){
        return (
        	<div>
            	<p>李载赣神魔</p>
            	<button onClick={this.props.hideConponent}>隐藏List组件</button>
            </div>
        )
    }
}
```

App.jsx

```javascript
import React, {Component} from 'react';
import List from './components/List';

export default class App extends Component {
    constructor(...args){
        super(...args);
        this.state = {
            isShowList: false,
        };
    }
    showConponent = () => {
        this.setState({
            isShowList: true,
        });
    }
    hideConponent = () => {
        this.setState({
            isShowList: false,
        });
    }
    render(){
        return (
        	<div>
            	<button onClick={this.showConponent}>显示List组件</button>
				{
                    this.state.isShowList ? 
                        <list hideConponent={this.hideConponent} />
                    :
                    null
                }
            </div>
        )
    }
}
```

首先在父组件中，给子组件传递一个改变的方法，然后在子组件点击事件调用这个方法，就可以传递到父组件去，并执行父组件的方法

### 跨组件通信

层层组件传递props

例如A组件和B组件之间要进行通信,先找到A和B公共的父组件,A先向C组件通信,C组件通过props和B组件通信,此时C组件起的就是中间件的作用

使用context

context是一个全局变量,像是一个大容器,在任何地方都可以访问到,我们可以把要通信的信息放在context上,然后在其他组件中可以随意取到;
但是React官方不建议使用大量context,尽管他可以减少逐层传递,但是当组件结构复杂的时候,我们并不知道context是从哪里传过来的;而且context是一个全局变量,全局变量正是导致应用走向混乱的罪魁祸首.

### 没有嵌套关系的组件通信

使用自定义事件机制

在componentDidMount事件中,如果组件挂载完成,再订阅事件;在组件卸载的时候,在componentWillUnmount事件中取消事件的订阅;
以常用的发布/订阅模式举例,借用Node.js Events模块的浏览器版实现

```javascript
// 首先需要项目中安装events 包：
npm install events --save

// 在src下新建一个util目录里面建一个events.js
import { EventEmitter } from 'events';

export default new EventEmitter();

// list1.jsx
import emitter from '../util/events';
componentDidMount() {
    // 组件装载完成以后声明一个自定义事件
    this.eventEmitter = emitter.addListener('changeMessage', (message) => {
        this.setState({
                message,
        });
     });
}
componentWillUnmount() {
	emitter.removeListener(this.eventEmitter);
}

// List2.jsx
import emitter from '../util/events';
handleClick = (message) => {
    emitter.emit('changeMessage', message);
};
```

## 生命周期

react 的生命周期从广义上分为三个阶段：挂载  渲染  更新

因此可以把 react 的生命周期分为两类：挂载卸载过程和更新过程 

### 挂载卸载过程

#### constructor()

constructor() 中完成了了 react 数据的初始化，它接受两个参数：props和context，当内部想使用这个两个参数是，需使用 super() 传入这两个参数

当使用了 constructor() 就必须写 super() 否则 this 指向错误

#### componentWillMount()

componentWillMount() 代表组件已经经历了 constructor() 初始化数据，但是还未渲染  DOM。一般很少使用，更多是在服务端渲染的时候去使用。

#### componentDidMount()

组件第一次渲染已经完成了，此时 dom 节点已生成，可以在这里调用 ajax 请求

#### componentWillUnmount()

在此处完成组件的卸载和数据的销毁

clear 组件中所有的 setTimeout  setInterval

移除组件所有的监听 removeEventListener

总而言之，异步操作和监听方法甚至是某些数据存储都要在这里进行处理

### 更新过程

#### componentWillReceiveProps(nextProps)

在接收父组件改变后的 props 需要重新渲染组件的时候用的比较多

接收一个参数 nextProps

通过对比 nextProps 和 this.props，将 nextProps 的 state 为当前组件的 state，重新渲染

```javascript
componentWillReceiveProps(nextProps){
    nextProps.openNotice !== this.props.openNotice && this.setState({
        openNotice: nextProps.openNotice
    }, () => {
        console.log(this.state.openNotice: nextProps)
        //将state更新为nextProps,在setState的第二个参数（回调）可以打印出新的state
    })
}
```

#### shouldComponentUpdate(nextProps, nextState)

主要用于性能优化（部分更新）

唯一用于控制组件重新渲染的生命周期，由于在 react 中，setState 以后，state 发生变化，组件会进入重新渲染的流程，这里 return false 可以阻止组件的更新

因为 react 父组件的重新渲染会导致其所有子组件的重新渲染，这个时候其实我们是不需要所有的子组件都跟着重新渲染的，因此需要在子组件的该生命周期做判断

#### componentWillUpdate(nextProps,nextState)

shouldComonentUpdate 返回 true 后，组件进入重新渲染的流程。

进入 componentWillUpdate，这里同样可以拿到 nextProps 和 nextState

#### componentDidUpdate(prevProps, prevState)

组件更新完毕后，react 只会在第一次初始化成功会进入 componentDidMount，之后每次重新渲染后都会进入这个生命周期，这里拿到的 prevProps 和 prevState，是更新前的 props 和 state

#### render()

render 函数插入 jsx 生成的 dom 结构，react会生成一份虚拟 dom 树，在每一次组件更新的时候，在此 react 会通过 diff 算法比较更新前后的新旧 dom 树，找到差异的 dom 节点，并重新渲染

### React新增的生命周期

#### getDerivedStateFromProps(nextProps, prevState)

代替 componentWillReceiveProps() 

老版本中的 componentWillReceiveProps() 方法判断两个前后 props 是否相同，如果不同再将新的 props 更新到相应的 state 上去，这样做一来会破坏 state 数据的单一数据源，导致组件状态变得不可预测，另一方面也会增加组件的重绘次数。

```javascript
// before
componentWillReceiveProps(nextProps) {
  if (nextProps.isLogin !== this.props.isLogin) {
    this.setState({ 
      isLogin: nextProps.isLogin,   
    });
  }
  if (nextProps.isLogin) {
    this.handleClose();
  }
}

// after
static getDerivedStateFromProps(nextProps, prevState) {
  if (nextProps.isLogin !== prevState.isLogin) {
    return {
      isLogin: nextProps.isLogin,
    };
  }
  return null;
}

componentDidUpdate(prevProps, prevState) {
  if (!prevState.isLogin && this.props.isLogin) {
    this.handleClose();
  }
}
```

这两者最大的不同就是:
 在 componentWillReceiveProps 中，我们一般会做以下两件事，一是根据 props 来更新 state，二是触发一些回调，如动画或页面跳转等。

1. 在老版本的 React 中，这两件事我们都需要在 componentWillReceiveProps 中去做。
2. 而在新版本中，官方将更新 state 与触发回调重新分配到了 getDerivedStateFromProps 与 componentDidUpdate 中，使得组件整体的更新逻辑更为清晰。而且在 getDerivedStateFromProps 中还禁止了组件去访问 this.props，强制让开发者去比较 nextProps 与 prevState 中的值，以确保当开发者用到 getDerivedStateFromProps 这个生命周期函数时，就是在根据当前的 props 来更新组件的 state，而不是去做其他一些让组件自身状态变得更加不可预测的事情。

#### getSnapshotBeforeUpdate(prevProps, prevState)

代替componentWillUpdate。
 常见的 componentWillUpdate 的用例是在组件更新前，读取当前某个 DOM 元素的状态，并在 componentDidUpdate 中进行相应的处理。
 这两者的区别在于：

1. 在 React 开启异步渲染模式后，在 render 阶段读取到的 DOM 元素状态并不总是和 commit 阶段相同，这就导致在
    componentDidUpdate 中使用 componentWillUpdate 中读取到的 DOM 元素状态是不安全的，因为这时的值很有可能已经失效了。
2. getSnapshotBeforeUpdate 会在最终的 render 之前被调用，也就是说在 getSnapshotBeforeUpdate 中读取到的 DOM 元素状态是可以保证与 componentDidUpdate 中一致的。
    此生命周期返回的任何值都将作为参数传递给componentDidUpdate（）。

#### componentDidCatch(error,info)

React 16 将提供一个内置函数 `componentDidCatch`，如果 `render()` 函数抛出错误，则会触发该函数。

```javascript
componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error 	  	reporting service
    logErrorToMyService(error, info);
}
```

#### 补充说明

1.React16新的生命周期弃用了componentWillMount、componentWillReceivePorps，componentWillUpdate

2.新增了getDerivedStateFromProps、getSnapshotBeforeUpdate来代替弃用的三个钩子函数（componentWillMount、componentWillReceivePorps，componentWillUpdate）

3.React16并没有删除这三个钩子函数，但是不能和新增的两个钩子函数（getDerivedStateFromProps、getSnapshotBeforeUpdate）混用。注意：React17将会删除componentWillMount、componentWillReceivePorps，componentWillUpdate

4.新增了对错误处理的钩子函数（componentDidCatch）

### 事件处理

和DOM元素操作有点类似，但是又不一样

事件类型采用小驼峰命名法，因此是 onClick 而不是 onclick，其他的事件也一样

直接将函数的生命当成事件句柄传递

阻止默认行为必须使用 **preventDefault** 

react中，默认的`onClick`事件传播方式为冒泡，如果希望用事件捕获的方式来触发事件 可以使用`onClickCapture`来绑定事件，也就是在事件类型后面加一个后缀`Capture`

在合成事件系统中，所有的事件都是绑定在`document`元素上，即，虽然我们在某个`react`元素上绑定了事件，但是，最后事件都委托给`document`统一触发。

`e.stopPropagation()`阻止了事件冒泡

在`SyntheticEvent`中，我们依然可以获取到事件发生时的`event`对象：

`react`鼓励我们使用合成事件，但是，在某些需求中，还是需要通过原生事件来进行处理，这时，就涉及到合成事件和原生事件的混合使用

### Refs & DOM

在 react 中，可以使用 refs 来访问 dom，或者在 render 中创建 react 对象

使用 refs 的主要用途是：

1. 做一些动画的交互
2. 媒体控件的播放
3. 获取焦点，获取文本等

使用refs的方式有两种，一种是使用`React.createRef() API`，另一种是使用 `回调形式的refs`。

#### 使用`React.createRef() API`

```javascript
import React, { Component } from 'react'
export default class App extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // 直接使用原生 API 使 text 输入框获得焦点
    // 注意：我们通过 "current" 来访问 DOM 节点
    this.textInput.current.focus();
    console.log(this.textInput.current);//这边输出input的dom对象
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到
    // 构造器里创建的 `textInput` 上
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />

        <input
          type="button"
          value="点击我获取焦点"
          onClick={this.focusTextInput}
        />
      </div>
    );
   }
 }
```

#### 回调 Refs

React 将在组件挂载时，会调用 ref 回调函数并传入 DOM 元素，当卸载时调用它并传入 null。在 componentDidMount 或 componentDidUpdate 触发前，React 会保证 refs 一定是最新的。

```javascript
import React, { Component } from 'react'
export default class App extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = null;
    this.setTextInputRef = element => {
      this.textInput = element;
    };
    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    // 使用 `ref` 的回调函数将 text 输入框 DOM 节点的引用存储到 React
    // 实例上（比如 this.textInput）
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="点击我获取焦点"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

后面来讲下，父子组件怎么样传递refs，你可以在组件间传递回调形式的 refs，代码如下

```javascript
import React, { Component } from 'react'
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class App extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.inputElement = null;
    this.setTextInputRef = element => {
      this.inputElement = element;
    };
    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      console.log(this.inputElement)
      if (this.inputElement) this.inputElement.focus();
    };
  }
  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }
  render() {
    return (
      <div> 
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
      <input
          type="button"
          value="点击我获取焦点"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

> 在上面的例子中，App 把它的 refs 回调函数当作 inputRef props 传递给了 CustomTextInput，而且 CustomTextInput 把相同的函数作为特殊的 ref 属性传递给了 <input>。结果是，在 App 中的 this.inputElement 会被设置为与 CustomTextInput 中的 input 元素相对应的 DOM 节点。

> 使用 `React.forwardRef` 来获取传递 ref，React 传递 ref 给 fowardRef 内函数 (props, ref) => ...，作为其第二个参数，然后转发到它渲染的 DOM

```javascript
import React, { Component } from 'react'
const CustomTextInput = React.forwardRef((props, ref) => (
  <div>
      <input ref={ref} />
   </div>
));

class App extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.inputElement = React.createRef();
    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      console.log(this.inputElement)
      this.inputElement.current.focus();
    };
  }

  render() {
    return (
      <div> 
      <CustomTextInput
        ref={this.inputElement}
      />
      <input
          type="button"
          value="点击我获取焦点"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

