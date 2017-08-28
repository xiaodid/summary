# MAIN DATA STRUCTURES
在react中，使用了几种基础的数据结构，在分析代码的时候，会经常看到这些数据结构的身影

```
     JSX                         ReactElement                ReactComponent
  <QueryTable />  ----> type: function QueryTable ---->  ReactCompositeComponent
  <div>           ----> type: 'div'               ---->  ReactDOMComponent
```

## 1, ReactElement
`ReactElement`，用来在react中描述组件的层级结构。是React中最基础的数据结构，react著名的dom diff算法，就是在比较`ReactElement`。render方法返回的就是一个`ReactElement`。

``` javascript
var element = {
  // This tag allow us to uniquely identify this as a React Element
  $$typeof: REACT_ELEMENT_TYPE,

  // Built-in properties that belong on the element
  type: type,
  key: key,
  ref: ref,
  props: props,

  // Record the component responsible for creating this element.
  _owner: owner,

  // __self and __source
  __self: self,
  __source: source
};
```

当我们在react中使用jsx语法描述一个组件结构时：
``` jsx
<div style={{ margin: '0 auto' }} >
  <h2>Counter: {counter}</h2>
  <button className='btn btn-primary' onClick={increment}>
    Increment
  </button>
  {' '}
  <button className='btn btn-secondary' onClick={doubleAsync}>
    Double (Async)
  </button>
</div>
```

会被转化成如下的ReactElement
``` javascript
// ReactElement.createElement = function (type, config, children)
React.createElement(
  // type
  "div",
  // config / props
  {
    style: {
      margin: "0 auto"
    }
  },
  // children
  React.createElement(
    "h2",
    null,
    "Counter: ",
    n),
  React.createElement(
    "button",
    {
      className: "btn btn-primary",
      onClick: t
    },
    "Increment"),
  " ",
  React.createElement(
    "button",
    {
      className: "btn btn-secondary",
      onClick: u
    },
    "Double (Async)")
  )
```

## 2, ReactComponent
ReactComponent，是react中最重要的数据结构，定义了一个Component的所有操作和属性，包括props，state，所有生命周期方法。
根据组件的不同类型，React定义了几种不同的Component：
* `ReactDOMComponent`
* `ReactCompositeComponent`
* `ReactDOMEmptyComponent`
* 等等

React会根据ReactElement的type属性，生成相应的ReactComponent，具体的代码在src/renderers/shared/stack/reconciler/instantiateReactComponent中。

> ReactReconciler

> 在代码中，我们会经常看到调用ReactReconciler.mountComponent, ReactReconciler.updateComponent。这个ReactReconciler，类似一个java里的接口，里面定义了一些方法，实现这个接口的class需要实现其中的方法。以实现类似java中的多态特性。
在ReactReconciler中，定义了mountComponent, unmountComponent, receiveComponent, performUpdateIfNecessary等方法，ReactCompositeComponent, ReactDOMComponent中，都实现了相应的方法，可以认为ReactCompositeComponent, ReactDOMComponent实现了ReactReconciler这个接口

这里所说的ReactComponent，并不是/src/isomorphic/modern/class/ReactComponent，这个class是我们在写代码时写的```class QueryTable extends React.Component```的那个Component。

## 3, DOMElement
这个就是原生的DOM，react在mount和update的时候，会生产相应的dom节点。React会把组件相应的DOMElement缓存在组件的Component实例中，以便以后方便使用。

## 4, DOMLayTree
每一个组件，都会有一个对应的`DOMLazyTree` Node结构：
``` javascript
{
  node: // DOM node
  children: [] // 子节点的dom node
  html:   // innerhtml
  text:   // text
}
```

`DOMLayTree`的作用，是把child连接到parent上，并处理不同浏览器（主要是IE、Edge和其他浏览器）在`createElement`上的性能问题。

## 5, Transaction
Transaction，并不是一个数据结构，他其实是一个模式（pattern)。React大量使用了Transaction。

## 6, Virtual DOM
React出世的时候，以Virtual DOM为卖点，Virtual DOM快。但后来测试证明，Virtual DOM并不像想象中那么快。现在在React的网站上，facebook不在宣传Virtual DOM了。但是，Virtual DOM是React的一个非常重要的概念。
有趣的是，React中并没有一个class或文件叫Virtual DOM。因为Virtual DOM是一个概念，一种方法。有人认为Virtual DOM就是ReactElement。也有人认为是ReactComponent。Facebook也没有官方说法。
