# 数据结构

## 1, VNODE
在preact中，表示节点的数据类型是VNODE，在React中是element
``` js
// in src/create-element.js

const vnode = {
  type,
  props,
  text,
  key,
  ref,
  _children: null,
  _dom: null,
  _lastDomChild: null,
  _component: null
};
```

``` js
// element in react

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

可以看到，preact和react的节点描述都包含以下几个属性：
* type
* props
* key
* ref

## 2, Component
在preact中，没有像react那样区分ReactDOMComponent, ReactCompositeComponent等，只实现了一个Component类用来描述React.Component

## 3, Transaction
preact没有transaction
