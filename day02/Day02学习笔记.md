# Day02学习笔记

## 条件渲染

可以根据条件判断来渲染相对应的组件，需要注意的点是，这一部分需要手动实现，没有 vue 中 v-if v-show 指令

可使用 JavaScript 运算符 if 或者条件运算符，三元表达式等 JavaScript 代码去实现条件渲染

### 元素变量

你可以使用变量来储存元素。 它可以帮助你有条件地渲染组件的一部分，而其他的渲染部分并不会因此而改变。

### 与运算符 &&

通过花括号包裹代码，你可以在 JSX 中嵌入任何表达式。这也包括 JavaScript 中的逻辑与 (&&) 运算符。它可以很方便地进行元素的条件渲染。

### 三元运算符

另一种内联条件渲染的方法是使用 JavaScript 中的三目运算符  isLogin ? true : false

### 阻止组件渲染

在极少数情况下，你可能希望能隐藏组件，即使它已经被其他组件渲染。若要完成此操作，你可以让 `render` 方法直接返回 `null`，而不进行任何渲染。

## 组合 vs 继承

this.props对象的属性与组件的属性一一对应，但是有一个例外，就是this.props.children属性，它代表组件的所有子节点

这个特殊的 children prop 来将他们的子组件传递到渲染结果中

这使得别的组件可以通过 JSX 嵌套，将任意组件作为子组件传递给它们。

````JavaScript
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}

function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
````

`` JSX 标签中的所有内容都会作为一个 `children` prop 传递给 `FancyBorder` 组件。因为 `FancyBorder` 将 `{props.children}` 渲染在一个 `` 中，被传递的这些子组件最终都会出现在输出结果中。

少数情况下，你可能需要在一个组件中预留出几个“洞”。这种情况下，我们可以不使用 `children`，而是自行约定：将所需内容传入 props，并使用相应的 prop。

```javascript
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

 `<Contacts />`和 `<Chat />`之类的 React 元素本质就是对象（object），所以你可以把它们当作 props，像其他数据一样传递。这种方法可能使你想起别的库中“槽”（slot）的概念，但在 React 中没有“槽”这一概念的限制，你可以将任何东西作为 props 进行传递。

## React 哲学

### 将设计好的 UI 划分为组件层级

根据功能以及样式划分为不同的组件

这一个整体是一个组件，输入框是一个组件，渲染列表是一个组件，列表标题是一个组件，列表内容也是一个组件，根据实际情况来调整。

### 用 React 创建一个静态版本

这也是前端开发的基本流程，先写出一个静态的版本，再去调试功能方面的问题

### 确定 UI state 的最小（且完整）表示

只保留应用所需的可变 state 的最小集合，其他数据均由它们计算产生。这个意思大概是，确定什么组件需要state，什么组件只需要props。

通过问自己以下三个问题，你可以逐个检查相应数据是否属于 state：

1. 该数据是否是由父组件通过 props 传递而来的？如果是，那它应该不是 state。
2. 该数据是否随时间的推移而保持不变？如果是，那它应该也不是 state。
3. 你能否根据其他 state 或 props 计算出该数据的值？如果是，那它也不是 state。

### 确定 state 放置的位置

我们已经确定了应用所需的 state 的最小集合。接下来，我们需要确定哪个组件能够改变这些 state，或者说*拥有*这些 state。

### 添加反向数据流

我们已经能把数据从上往下传递渲染了，现在做的是操作子组件，能够把数据提交到父组件上去，这就是完整的功能了。

## 搭建一下

