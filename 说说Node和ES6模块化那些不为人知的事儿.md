# 说说 Node 和 ES6 模块化那些不为人知的事儿
## 前言
模块化已经成为前端开发中的必然趋势，模块化带来的好处如下：
- 1.避免变量污染，命名冲突
- 2.提高代码复用率
- 3.提高维护性
- 4.依赖关系的管理

文章主要就node的commonjs规范和es6规范进行了浅层次的研究，探索node和es6模块化一些不为人知的小秘密。
## 1.node模块化
node模块化遵循的是commonjs规范，CommonJs定义的模块分为: 模块标识(module)、模块导出(exports) 、模块引用(require)。

在node中，一个文件即一个模块，使用exports和require来进行处理。

导出形式有2种，module.exports和exports导出。
```
// 第一种
module.exports = { // 需要导出的对象
 ....  
}
// 第二种
exports.a = 'hello world';
```
导入形式：
```
const index = require('./index');
```

### 1.1.module是什么？

那么，问题来了，这个module到底是什么东西呢？

直接在文件中打印moule
```
// index.js
console.log(module)
````
然后执行index.js
```
node index.js
```
![](https://user-gold-cdn.xitu.io/2019/9/18/16d44302167604a1?w=726&h=346&f=png&s=94041)
<br>
module为该模块运行时生成的模块标识对象，标识该模块的一些信息。
<br>
module参数说明：
- id为当前文件
- exports为当前node文件模块儿导出的值
- parent为父级调用的module对象，如果为null则该文件没有被调用
- filename为当前文件名
- loaded是否被加载
- children 引入模块数组，数组项格式同module
- paths为node模块儿 node_modules 模块儿查找路径，一直查到根目录

在index.js中引入utils.js模块，打印module对象，可以看到children中包含了utils模块的信息。
```
const utils = require("./utils");
```
![](https://user-gold-cdn.xitu.io/2019/9/18/16d44375f11aad84?w=802&h=533&f=png&s=146177)
> module表示一个模块运行时该模块标识信息对象。

### 1.2.module.exports === exports？
exports表示该模块运行时生成的导出对象。

module对象也有一个exports属性，两者有什么关系呢？
```
console.log(exports);
console.log(module.exports);
console.log(exports === module.exports);
```

![](https://user-gold-cdn.xitu.io/2019/9/18/16d44400296dbee8?w=555&h=99&f=png&s=16442)

我们惊奇的发现module.exports与exports指向的都是空对象，且两者指向**同一个对象**！！！

现在我们分别改写module.exports和exports导出对象，然后在main.js中引入index.js，查看导入的对象。

index.js
```
module.exports = {
  name: "hello world"
};

exports.obj = {
  name: "welcome"
};
```
main.js
```
const obj = require("./index");

console.log(obj);
```

![](https://user-gold-cdn.xitu.io/2019/9/18/16d444888bfb0d24?w=543&h=55&f=png&s=18380)

我们惊奇的发现，导入的是module.exports导出的对象！！！

由于我们重新定义了modulex.export引用值，导入的是module.exports引用的对象，因此我们可以做出如下结论：

>一个模块真正导出的是module.exports的值，exports只是module.exports的一个引用。
### 1.3.模块缓存与导出引用数据类型
问题：
<br>
1.多次加载一个模块，是否多次执行源模块呢？
<br>
2.导出引用数据类型时，修改导出对象的引用属性值，那么源模块中属性值是否发生改变呢？

我们在main.js中修改导入对象name属性，然后在index.js中异步打印导出对象。

index.js
```
const obj = {
  name: "hello world"
};

setTimeout(() => {
  console.log("index");
  console.log(obj);
}, 500);

module.exports = obj;
```
main.js
```
const obj1 = require("./index");
const obj2 = require("./index");

obj1.name = "welcome";

console.log("obj1:");
console.log(obj1);
console.log("obj2:");
console.log(obj2);
```

![](https://user-gold-cdn.xitu.io/2019/9/18/16d446241099ceb8?w=543&h=173&f=png&s=31876)

可以看到，即使我们2次导入源模块，但是源模块只加载了一次，我们修改导出模块值，源模块的值也发生了变化。

结论：
- 1.导入模块时，会缓存导入模块，后续再导入该模块，从缓存中取导入模块。
- 2.当导出引用类型数据时，导出对象与导入对象指向的是同一个对象。

### 1.4.最佳实践
针对module.exports与exports，最佳用法如下：

1.导出对象时用module.exports
```
function add(x, y){
    return x+y;
}
function multiply(x, y){
    return x*y;
}
module.exports = {
    add,
    multiply
}
```
2.导出变量或方法时用exports
```
exports.name = 'hello world';
exports.func = function() {
    console.log('hello world');
}
```
> exports与module.exports最好不要同时使用，exports不要重新赋值到新对象，否则引用无效。

### 1.5.node模块化总结
针对node commonjs规范模块化，总结如下：
- 1.module对象为模块运行时生成的标识对象，提供模块信息；
- 2.exports为模块导出引用数据类型时，modulex.exports与exports指向的是同一对象，require导入的是module.exports导出的对象；
- 3.同一模块导入后存在模块缓存，后续多次导入从缓存中加载；
- 4.源模块的引用与导入该模块的引用是同一对象；
- 5.最好不要同时使用module.exports与exports，导出对象使用module.exports，导出变量使用exports。
## 2.es6模块化
接下来我们来看下es6模块化规范，es6的模块化通过export与import来处理。

常见的导出形式：
```
export default obj;

export const name = "hello world";
// export { name1, name2, …, };
```
导入形式：
```
import obj, { name } from "./exp/exports";
```
### 2.1.export
在es6中，一个文件表示一个模块，模块通过export向外暴露接口，export常用于向外导出多个变量。

正确写法：
```
export var name = 1;
// 等价于
var name = 1;
export { name }

// 导出多个变量
var a = 1;
var b = 2;
var c = 3;
export { a, b, c }

// 导出对象
export const obj = {
    name: 'hello world'
};

// 导出函数
export function fun1() {
    console.log('hello world');
}

// 直接导出引入模块
export * from './index';
```
错误写法：
```
var name = 1;
export name;
```
同时
引入模块时，import导入变量名称需要与export导出的变量一一对应。
```
import { a, b, c } from './index';
```
### 2.2.export default
export default表示一个模块默认的对外接口，一个模块只能有一个export default。

export default是export的语法糖，表示导出一个default接口。
```
const obj = {
  name: "hello world"
};

export default obj;
// 等价于
export { obj as default }; // as表示导出别名为default的接口
```
引入默认导出
```
import obj from './index';
// 等价于
import { default as obj } from './index';
```
### 2.3.import * as xx
as表示别名，在import和export中可以这样用：

export
```
export { name as hello };
```
import
```
import { hello as name } from './index';
```
我们有时候也会看到import * 和 export * 形式。
import 
```
import * as obj './index';
```
看个例子，index.js导出obj和name变量，默认导出arr数组，在app.js中通过`import * as xxx`引入index.js。

index.js
```
const obj = {
  name: "hello world"
};

const name = "hello world";

const arr = [1, 2, 3];

export { obj, name };
export default arr;
```
app.js
```
import * as module from "./exp/exports";
console.log(module);
console.log(module.default);
console.log(module.name);
console.log(module.obj);
```
可以看出我们引入的是一个Module对象，描述该模块导出的信息。
<br>
其中default表示默认导出，对应数组arr，name和obj分别表示导出的name和obj变量。

![](https://user-gold-cdn.xitu.io/2019/9/19/16d477d3f46e2845?w=710&h=304&f=png&s=32696)

### 2.4.多次导入和导出引用数据类型
当多次导出对象变量，且修改导出变量对象时，会发生什么呢？

index.js
```
const obj = {
  name: "hello world"
};

console.log("加载了index.js");
setTimeout(() => {
  console.log("index:");
  console.log(obj);
}, 500);

export default obj;
```
app.js
```
import obj1 from "./exp/index";
import obj2 from "./exp/index";

obj1.name = "welcome";
console.log("obj1:");
console.log(obj1);
console.log("obj2:");
console.log(obj2);
```
查看结果

![](https://user-gold-cdn.xitu.io/2019/9/19/16d47919848d1df5?w=948&h=184&f=png&s=12357)

不难看出，index.js组件import了2次，只加载了1次，并且源对象obj发生了改变。

结论：
- 1.同一模块import多次，只会执行1次。
- 2.导出引用数据类型时，导出对象与导入对象指向同一个变量，修改导出变量对象，源对象也会发生改变。
### 2.5.最佳实践
1.当导出一个变量时，推荐使用export default。
<br>
例如：react中导出组件时，一般都使用export default。
```
// main.jsx

import React from 'react';

 class Main extends React.Component{
  render() {
    return (
      <div>Hello World.</div>
    )
  }
}

export default Main;
```
2.当导出多个变量时，推荐使用export。
```
// math.js

function add(a, b) {
  return a + b;
}

function reduce(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

export { add, reduce, multiply };
```
3.当需要导出多个变量且需要默认导出时，可以同时使用export和export default。
```
const obj = {
  name: "hello world"
};

const name = "hello world";

export { obj, name };
export default function getName() {
  return obj.name;
}
```
### 2.6.es6模块化总结
针对es6规范模块化，总结如下：
- 1.es6通过export和import实现导出导入功能；
- 2.es6 export支持导出多个变量，export default是export形式的语法糖，表示导出default接口；
- 3.`import * as xx from 'xx.js'`导入的是Module对象，包含default接口和其他变量接口；
- 4.多个模块导入多次，只会执行一次；
- 5.导出引用数据类型时，导出对象与导入对象指向同一个变量，修改导出变量对象，源对象也会发生改变。
- 6.导出单个变量建议使用export default，导出多个变量使用export。

## 结语
以上就是博主探索的关于node和es6模块化的一些小知识，觉得有收获的可以**关注一波**，**点赞一波**，码字不易，万分感谢。