# react hooks入门

本项目基于`create-react-app`脚手架搭建，包含 hooks 的各种用法（useState，useEffect，useContext，useReducer，useRef，useMemo 和自定义 hooks）。

## 1.useState

> useState 是 react 自带的一个 hook 函数，它的作用是用来声明状态变量。

声明方式：

```
const [ count , setCount ] = useState(0);
```

案例代码：

```
import React, { useState, useEffect } from "react";

function ExampleUseState() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}

export default ExampleUseState;
```

## 1.2.useEffect

> 使用 useEffect 代替常用的生命周期函数

声明方式：

```
 useEffect(() => {
    // 副作用代码
    return () => {
      // 清除副作用
    };
  }, [依赖值]);
```

案例代码：

```
function ExampleUseEffect() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    console.log(`useEffect=>You clicked ${count} times`);
    return () => {
      console.log("useEffect=>离开当前页面");
    };
  }, [count]);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

## 1.3.useContext

> useContext 用于多组件间共享数据

声明方式：

```
const xxContext = createContext(); // 创建Context

const {value} = useContext(xxContext); // 获取Context
```

案例代码：

```
import React, { useState, createContext, useContext } from "react";

const CountContext = createContext(); // 创建Context

function Counter() {
  const count = useContext(CountContext); //一句话就可以得到count
  return <h2>{count}</h2>;
}

function ExampleUseContext() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button
        onClick={() => {
          setCount(count + 1);
        }}>
        click me
      </button>
      <CountContext.Provider value={count}>
        <Counter />
      </CountContext.Provider>
    </div>
  );
}
export default ExampleUseContext;
```

## 1.4.useReducer

> useReducer 用于实现类似 Redux 的功能

声明方式：

```
const [state, dispatch] = useReducer(reducer函数, state初始值);
```

案例代码：

```
import React, { useReducer } from "react";

function ExampleUseReducer() {
  const [count, dispatch] = useReducer((state, action) => {
    switch (action) {
      case "add":
        return state + 1;
      case "sub":
        return state - 1;
      default:
        return state;
    }
  }, 0);
  return (
    <div>
      <h2>现在的个数是{count}</h2>
      <button onClick={() => dispatch("add")}>Increment</button>
      <button onClick={() => dispatch("sub")}>Decrement</button>
    </div>
  );
}

export default ExampleUseReducer;
```

## 1.5.使用 useReducer 和 useContext 代替 Redux

> 在父组件使用 useReducer 生成 state 和 dispatch，使用 useContext 生成 Context，将 state 和 dispatch 传递给子组件，子组件使用 useContext 获取 state 和 dispatch。

案例代码：

**ExampleUseReducerRedux.js**

```
import React from "react";
import { Color } from "./Color";
import ShowArea from "./ShowArea";
import Buttons from "./Buttons";

function ExampleReducerRedux() {
  return (
    <div>
      <Color>
        <ShowArea />
        <Buttons />
      </Color>
    </div>
  );
}

export default ExampleReducerRedux;
```

**Color.js**

```
import React, { createContext, useReducer } from "react";

export const ColorContext = createContext({});

export const UPDATE_COLOR = "UPDATE_COLOR";

const reducer = (state, action) => {
  switch (action.type) {
    case UPDATE_COLOR:
      return action.color;
    default:
      return state;
  }
};

export const Color = (props) => {
  const [color, dispatch] = useReducer(reducer, "blue"); // 生成reducer
  return (
    <ColorContext.Provider value={{ color, dispatch }}>{props.children}</ColorContext.Provider>
  );
};
```

**Buttons.js**

```
import React, { useContext } from "react";
import { ColorContext, UPDATE_COLOR } from "./Color";

function Buttons() {
  const { dispatch } = useContext(ColorContext);
  return (
    <div>
      <button
        onClick={() => {
          dispatch({ type: UPDATE_COLOR, color: "red" });
        }}>
        红色
      </button>
      <button
        onClick={() => {
          dispatch({ type: UPDATE_COLOR, color: "yellow" });
        }}>
        黄色
      </button>
    </div>
  );
}

export default Buttons;
```

**ShowArea.js**

```
import React, { useContext } from "react";
import { ColorContext } from "./Color";

function ShowArea() {
  const { color } = useContext(ColorContext);
  return <div style={{ color }}>字体颜色为blue</div>;
}

export default ShowArea;
```

## 1.6.useMemo

> 使用 useMemo 优化 React hooks 性能

声明方式：

```
const 新值 = useMemo(fun函数, [依赖值]);
```

案例：

**ExampleUseMemo.js**

```
import React, { useState } from "react";
import ChildComponent from "./ChildComponent";

function ExampleUseMemo() {
  const [name, setName] = useState("小明");
  const [sex, setSex] = useState("男");
  return (
    <React.Fragment>
      <button
        onClick={() => {
          setName(new Date().getTime());
        }}>
        小红
      </button>
      <button
        onClick={() => {
          setSex(new Date().getTime() + "男");
        }}>
        女
      </button>
      <ChildComponent name={name}>{sex}</ChildComponent>
    </React.Fragment>
  );
}

export default ExampleUseMemo;
```

**ChildComponent.js**

```
import React, { useMemo } from "react";

function ChildComponent({ name, children }) {
  function changeName(name) {
    console.log("获得了name");
    return name + "信息";
  }
  const actionName = useMemo(() => changeName(name), [name]);
  return (
    <React.Fragment>
      <div>{actionName}</div>
      <div>{children}</div>
    </React.Fragment>
  );
}

export default ChildComponent;
```

## 1.7.useRef

> useRef 用来获取 Dom 元素

声明方式：

```
const xxx = useRef(null)
```

案例：

```
import React, { useRef } from "react";

function ExampleUseRef() {
  const inputElm = useRef(null);
  const onButtonClick = () => {
    inputElm.current.value = "Hello，Xiaoming";
  };
  return (
    <React.Fragment>
      <input ref={inputElm} type='text' />
      <button onClick={onButtonClick}>在input上展示文字</button>
    </React.Fragment>
  );
}

export default ExampleUseRef;
```

## 1.8.自定义 hooks

> 自定义 hooks 表示完成特定功能的函数，一定要使用 use 开头

声明方式：

```
function useHook() {
  return {
  }
}
```

案例：

**useWinSize**

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

**ExampleCustomHooks**

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

以上就是 hooks 的各种用法。
