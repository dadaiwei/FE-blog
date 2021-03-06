# 关于Vue和React的一些对比及个人思考（下）
Vue和React都是目前最流行、生态最好的前端框架之一。框架本身没有优劣之分，只有适用之别，选择符合自身业务场景、团队基础的技术才是我们最主要的目的。

博主1年前用的是Vue框架，近半年转技术栈到React框架，对于Vue和React都有一些基本的了解。接下来博主将与大家通过上、中、下三部一起走近Vue和React，共同探讨它们的差异。（比如你正在用vue，对react感兴趣也可以看下，反之亦然）

整体内容概览：

![整体内容概览](https://user-gold-cdn.xitu.io/2020/1/8/16f83120f3eb972a?w=650&h=846&f=png&s=41416)

下部将主要从**逻辑复用**、**diff算法**、**路由**、**状态管理**四个方面对比vue和react，欢迎多多交流，共同探讨，感谢支持。

上部链接：[关于Vue和React的一些对比及个人思考（上）](https://juejin.im/post/5e153e096fb9a048297390c1)

中部链接：[关于Vue和React的一些对比及个人思考（中）](https://juejin.im/post/5e292746e51d451c8771d16e)

## 17.逻辑复用(mixin vs HOC + Render props + hooks)
### vue
vue中的逻辑复用通常是放到mixin中，具体定义如下：
> 混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。

下面一个例子实现一个取消请求的mixin，功能是当组件销毁的时候取消该组件中未完成的的请求。
```
// cancelTokenMixin.js

import axios from 'axios';

const CancelToken = axios.CancelToken;

const cancelTokenMixin = {
  data () {
    return {
      cancelToken: null, // cancelToken实例
      cancel: null // cancel方法
    };
  },

  created () {
    // 初始化生成cancelToken实例，注册cancel方法
    this.cancelToken = new CancelToken(c => {
      this.cancel = c;
    });
  },

  beforeDestroy () {
    // 组件销毁阶段调用cancel方法取消请求
    this.cancel(`${this.pageName}取消请求`);
  }
};

export default cancelTokenMixin;
```
全局注册该mixin：
```
import cancelTokenMixin from './cancelTokenMixin';

Vue.use(cancelTokenMixin);
```
组件内部使用该mixin:
```
import cancelTokenMixin from './cancelTokenMixin';

export default {
  name: 'xxx',
  mixins: [cancelTokenMixin]
}
```
### react
react中对于逻辑复用可以采用HOC(高阶组件)、render props、自定义hooks三种方式实现。
#### (1)HOC(高阶组件)
> 高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。

具体而言，高阶组件是参数为组件，返回值为新组件的函数。
```
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```
下面实现一个高阶组件，功能是当组件销毁的时候取消该组件未完成的请求。

先用公共方法实现cancalToken实例和cancel方法，实现如下：
```
import axios from 'axios';

export const generateCancelToken = () => {
  let cancel = null;
  const cancelToken = new axios.CancelToken(c => {
    cancel = c;
  });
  return {
    cancel,
    cancelToken
  };
};
```
高阶组件`withCancelRequest`初始化阶段生成`cancelToken`实例和`cancel`方法，将`cancelToken`实例作为`prop`传递给包装组件，在`componentWillUnmount`阶段取消请求，具体实现如下：
```
import React, { Component } from 'react';
import { generateCancelToken } from '../utils/index';

function withCancelRequest (WrappedComponent, pageName) {
  return class extends Component {
    constructor(props) {
      super(props);
      // 初始化生成cancelToken实例和cancel方法
      const { cancel, cancelToken } = generateCancelToken();
      this.cancelToken = cancelToken;
      this.cancel = cancel;
    }
    componentWillUnmount () {
      // 组件销毁阶段调用cancel方法取消请求
      let message = '取消页面请求';
      if (pageName) {
        message = `${pageName}取消请求`;
      }
      this.cancel(message);
    }
    render () {
      // 将cancelToken传给包裹组件
      return <WrappedComponent cancelToken={this.cancelToken} />;
    }
  };
}

export default withCancelRequest;
```
> Tips：HOC组件的componentWillUnmount会先于包装组件的componentWillUnmount触发，在这时取消请求即可。

HOC优点：
- 1.支持ES6
- 2.复用性强，支持多层嵌套
缺点：
- 1.由于多次嵌套高阶组件，很难确保每个高阶组件中的属性名是不相同的，属性容易被覆盖。
- 2.产生了很多无用的组件，加深了组件层级。

#### (2)Render props
Render props是指一种在React组件之间使用一个值为函数的prop共享代码的简单技术。Render props是一个用于告知组件需要渲染什么内容的函数prop。

创建Render props的方式：
- 1.接收一个外部传递进来的props属性
- 2.将内部的state作为参数传递给调用组件的props属性方法

下面的例子实现了`<Mouse>`组件封装了所有关于监听mousemove事件和存储鼠标(x, y)位置的行为，接收render属性，动态决定需要渲染什么。
```
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>移动鼠标!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```
Render props优点：
- 1.解决了HOC的缺点
- 2.支持ES6
- 3.不会产生无用的空组件加深层级
缺点：
- 1.嵌套过深也会形成回调地狱
- 2.无法在return 语句外访问数据
#### (3)hooks
React hooks是React 16.0版本之后推出的新特性，hooks可以在不编写class的情况下使用state及其他的React特性。

基本hooks：`useState`、`useEffect`、`useContext`
附加hooks：`useReducer`、`useMemo`、`useCallback`、`useRef`、`useLayoutEffect`

hooks可以在无需修改组件结构的情况下复用状态逻辑。
> 自定义hooks表示完成特定功能的函数，一定要使用use开头。

下面实现一个自定义hooks：
```
import { useState, useEffect, useCallback } from "react";

function useWinSize() {
  const [size, setSize] = useState({
    width: document.documentElement.clientWidth,
    height: document.documentElement.clientHeight
  });
  const onResize = useCallback(() => {
    setSize({
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight
    });
  }, []);
  useEffect(() => {
    window.addEventListener("resize", onResize);
    return () => {
      window.removeEventListener("resize", onResize);
    };
  }, []);
  return size;
}

export default useWinSize;
```
组件中引用该hooks：
```
import React from "react";
import useWinSize from "./useWinSize";

function ExampleCustomHooks() {
  const size = useWinSize();
  return (
    <div>
      页面Size：{size.width} * {size.height}
    </div>
  );
}

export default ExampleCustomHooks;
```
hooks优点：
- 1.hooks简单易懂，更容易复用代码
- 2.不需要再关注组件的生命周期
- 3.避免了组件多层嵌套的问题

缺点：
- 1.hooks不能用于循环、条件判断、函数嵌套中
- 2.状态不同步，异步回调状态引用还是之前的
## 18.diff算法
### vue
vue中diff比较存在两种不同的粒度，先是组件级别，再是元素级别。

![新旧节点比较](./关于Vue和React的一些对比及个人思考/新旧节点比较.png)

1.当组件中的data数据变化时，触发data数据的setter，会通过notify通知组件对应的Watcher，执行updateComponent等一连串操作，进行新旧节点比较。

2.对比新旧节点树，判断节点类型是否相同
<br>
（1）类型不同，创建新节点，替换旧节点
<br>
（2）类型相同，继续比较节点属性
<br>
（3）节点属性不同时，更新节点属性，再比较子节点
<br>
（4）新dom树没有子节点时，删除旧dom树的节点列表
<br>
（5）新dom树有子节点，旧dom树没有子节点时，新增子节点
<br>
（6）新旧dom树都有子节点，调用updateChildren比较子节点列表。

3.子节点更新策略

![子节点更新策略](./关于Vue和React的一些对比及个人思考/子节点比较.png)

1.旧节点的起始位置为oldStartIdx,截至位置为oldEndIdx,新节点的起始位置为newStartIdx,截至位置为newEndIdx。
<br>
2.新旧children的起始位置的元素两两对比，顺序是vnode, oldVnode; newEndVnode, oldEndVnode;newEndVnode, oldVnode;newStartIdx, oldEndIdx
<br>
3.vnode, oldVnode节点相同，执行一次patchVnode过程，也就是递归对比相应子节点，并替换节点的过程。
oldStartIdx，newStartIdx都像右移动一位。
<br>
4.newEndVnode, oldEndVnode节点相同，执行一次patchVnode过程，递归对比相应子节点，并替换节点。oldEndIdx， newEndIdx都像左移动一位。
<br>
5.newEndVnode, oldVnode节点相同，执行一次patchVnode过程，并将旧的oldVnode移动到尾部,oldStartIdx右移一味，newEndIdx左移一位。
<br>
6.newStartIdx, oldEndIdx节点相同，执行一次patchVnode过程，并将旧的oldEndVnode移动到头部,oldEndIdx左移一味，newStartIdx右移一位。
<br>
7.四种组合都不相同，则会搜索旧节点所有子节点，找到将这个旧节点和vnode执行patchVnode过程。
<br>
8.不断对比的过程使得oldStartIdx不断逼近oldEndIdx，newStartIdx不断逼近newEndIdx。当oldEndIdx <= oldStartIdx说明旧节点已经遍历完了，此时只要批量增加新节点即可。当newEndIdx <= newStartIdx说明旧节点还有剩下，此时只要批量删除旧节点即可。

### react
react中diff存在三种不同的粒度，先是组件树级别，再是组件级别，最后元素级别。

1.两棵树分层比较，并且只在同一层进行比较。比如旧树A的子节点B移动到C节点。

![react新旧节点比较](./关于Vue和React的一些对比及个人思考/react新旧节点比较.png)

具体操作如下：

![react tree diff](./关于Vue和React的一些对比及个人思考/react树层级操作.png)

先创建新的B节点，添加到C的子节点中，然后再删除A的子节点B完成。

2.基于组件层级比较也是参照原策略比较，React允许用户通过shouldComponentUpdate()来判断该组件是不是需要进行diff比较，但是如果调用了forceUpdate方法，强制进行diff比较。

![react组件diff](./关于Vue和React的一些对比及个人思考/react组件diff.png)

具体操作如下：

![react组件层级操作](./关于Vue和React的一些对比及个人思考/react组件层级操作.png)

当组件F变为I的时候，先判端F和I是不同类型的组件，先删除F及其子节点，再重新创建I及其子节点。

3.元素级别比较diff比较主要涉及插入、移动、删除3种节点操作。

插入：新的元素是全新的节点，将该元素插入新的集合中

移动：元素已经存在于集合中，只是位置发生改变。

删除：元素不在新集合中，需要删除该元素。

## 19.路由(vue-router vs react-router-dom)
### vue
vue中使用vue-router负责路由管理，vue-router支持hash与history模式。
### react
## 20.状态管理(vuex vs redux)