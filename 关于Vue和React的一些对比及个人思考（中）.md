# 关于Vue和React的一些对比及个人思考（中）
Vue和React都是目前最流行、生态最好的前端框架之一。框架本身没有优劣之分，只有适用之别，选择符合自身业务场景、团队基础的技术才是我们最主要的目的。

博主1年前用的是Vue框架，近半年转技术栈到React框架，对于Vue和React都有一些基本的了解。接下来博主将与大家通过上、中、下三部一起走近Vue和React，共同探讨它们的差异。（比如你正在用vue，对react感兴趣也可以看下，反之亦然）

整体内容概览：

![整体内容概览](https://user-gold-cdn.xitu.io/2020/1/8/16f83120f3eb972a?w=650&h=846&f=png&s=41416)

中部将主要从**条件渲染**、**是否显示**、**列表渲染**、**计算属性**、**侦听器**、**`ref`**、**表单**、**插槽**八个方面对比vue和react，欢迎多多交流，共同探讨，感谢支持。

上部链接：[关于Vue和React的一些对比及个人思考（上）](https://juejin.im/post/5e153e096fb9a048297390c1)


## 9.条件渲染(v-if vs &&)
条件渲染用于根据条件判断是否渲染一块内容。
### vue
vue中用`v-if`指令条件性地渲染一块内容。只有当表达式返回真值时才被渲染，`v-if`还可以和`v-else-if`、`v-else`配合使用，用于不同条件下的渲染，类似js里的`if else`语法。

#### （1）基本用法
```
 <div v-if="showContent">This is content.</div>
```
data
```
  data() {
    return {
      showContent: false
    }
  }
```
当`showContent`为`false`时，不会渲染`DOM`节点，会留下一个注释标志。

![v-if为false](https://user-gold-cdn.xitu.io/2020/1/23/16fd0c68b9d7b409?w=306&h=70&f=png&s=1972)

`showContent`为`true`时，才会渲染`DOM`节点。

![v-if为true](https://user-gold-cdn.xitu.io/2020/1/23/16fd0c6da128e90d?w=296&h=66&f=png&s=2630)

#### （2）v-else 二重判断
`v-if`和`v-else`配合使用时，`v-else`必须要和`v-if`相邻，否则会报错。
```
  <div>
    <div v-if="showContent">This is content.</div>
    <div v-else>Hide content.</div>
  </div>
```
#### （3）v-else-if 多重判断
当有多重判断条件时，可以使用`v-else-if`，类似`v-if`的`else-if块`，`v-else` 元素必须紧跟在带`v-if`或者`v-else-if`的元素的后面，否则它将不会被识别。
```
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```
#### （4）template 使用 v-if
另外，当想切换多个元素时，在`<template>`上使用`v-if`可以针对元素进行分组。
```
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```
### react
react使用与运算符`&&`、三目运算符（`?:`）、`if else`来实现条件渲染的效果。
#### （1）与运算符
与运算符`&&`，实现效果类似`v-if`，左边变量为真值时，渲染右边的元素。
```
 return (
      <div>
        {showItem && <div>This is content.</div>}
      </div>
    );
```
#### （2）三目运算符（`?:`）
使用三目运算符（`?:`），实现类似`v-if v-else`效果，`showItem`变量为`true`时显示第一个元素，变量为`false`时显示第二个元素。
```
 return (
      <div>
        {
          showItem ?
            (<div>This is true content.</div>) : (<div>This is false content.</div>)
        }
      </div>
    );
```
#### （3）多重判断
当处理多重判断时，可以使用函数加`if else`多重判断或者`switch case`的形式来实现，实现类似`v-if v-else-if v-else`的效果。

Ⅰ.`if-else`多重判断
```
render() {
    const { type } = this.state;
    const toggeleShow = (type) => {
      if (type === 'A') {
        return <div>A</div>;
      } else if (type === 'B') {
        return <div>B</div>;
      } else if (type === 'C') {
        return <div>C</div>;
      } else {
        return null;
      }
    };

    return (
      <div>
        {toggeleShow(type)}
      </div>
    );
  }
```
Ⅱ.`switch case`多重判断
```
render () {
    const { type } = this.state;
    const toggeleShow = (type) => {
      switch (type) {
        case 'A':
          return <div>A</div>;
        case 'B':
          return <div>B</div>;
        case 'C':
          return <div>C</div>;
        default:
          return null;
      }
    };

    return (
      <div>
        {toggeleShow(type)}
      </div>
    );
  }
```
## 10.是否显示(v-show vs style+class)
另一个用于展示条件的元素的选项是`v-show`，react中可以通过`style`或者切换`class`的形式实现是否显示。
### vue
`v-show`渲染的元素会被渲染并保留在DOM中。`v-show`只是简单地切换元素的`css`属性`display`。
```
 <div v-show="showContent">This is content.</div>
```

`showContent`为`false`时，`style`的`display`属性值为`none`。

![v-show为false](https://user-gold-cdn.xitu.io/2020/1/23/16fd0c7b240dc994?w=430&h=29&f=png&s=2108)

`showContent`为`true`时，`style`的`display`属性值为`block`（元素默认`display`属性值）。

![v-show为true](https://user-gold-cdn.xitu.io/2020/1/23/16fd0c7e68239c4b?w=735&h=171&f=png&s=9676)

> 注意，v-show 不支持 `<template>` 元素，也不支持 v-else。

`v-if`与`v-show`对比总结：
- 1）`v-if`是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。
- 2）`v-if`也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。
- 3）相比之下，`v-show`就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。
- 4）一般来说，`v-if`有更高的切换开销，而`v-show`有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用`v-show`较好；如果在运行时条件很少改变，则使用`v-if`较好。

### react
react中通过修改`style`或者`class`来实现`v-show`类似效果。

通过修改`style`属性的`display`属性来实现切换是否显示。
```
<div style={{ display: showItem ? 'block' : 'none' }}>
    This is content.
</div>
```
通过变量判断修改`class`来实现切换效果，本质上也是切换元素的`display`属性来实现切换效果。
> 在react中动态修改元素样式时（比如切换`tab`、按钮选中状态），适用于使用`class`来实现。
```
 const itemClass = showItem ? 'show-item-class' : 'hide-item-class';
    return (
      <div className={itemClass}>
        This is content.
      </div >
    );
```
`class`样式：
```
.show-item-class {
  display: block;
}
.hide-item-class {
  display: none;
}
```
## 11.列表渲染(v-for vs map)
vue中使用`v-for`来渲染列表，react中使用`map`来渲染列表。不管是`v-for`还是`map`来渲染列表都需要添加`key`值（`key`在兄弟节点之间必须唯一），方便快速比较出新旧虚拟`DOM`树间的差异。
### vue
vue中可以使用`v-for`来渲染数组、对象、`<template>`、组件。

#### （1）渲染数组
渲染数组时，使用`(item, index) in items`形式的特殊语法，其中`items`是源数据数组，`item`则是被迭代的数组元素的别名，`index`表示当前元素的索引。
```
  <div>
    <div v-for="(item, index) in items" :key="item.message + index">
      {{item.message}}
    </div>
  </div>
```
data
```
 data() {
    return {
      items: [
        {
          message: 'Hello'
        },
        {
          message: 'Welcome'
        }
      ]
    }
  }
```
#### （2）渲染对象
`v-for`也可以用来遍历一个对象的属性，使用`(value, key, index) in obj`的形式，其中`key`表示对象的`key`值，`value`表示对象的`value`值，`index`表示当前索引。
> 在遍历对象时，采用`Object.keys()`的结果遍历，但是不能保证它的结果在不同的 JavaScript 引擎下都一致。
```
<div v-for="(value, key, index) in obj" :key="key + index">
  {{index}}.{{key}}: {{value}}
</div>
```
data
```
  data() {
    return {
      obj: {
        name: 'xiaoming',
        age: 18,
        sex: 'male',
        height: 175
      }
    }
  }
```
#### （3）渲染多个元素
在`<template>`上使用`v-for`来渲染一段包含多个元素的内容。
> 在`<template>`上使用`v-for`来渲染元素段时，不允许绑定key值。因为`template`上并不会生成实际`Dom`节点。可以给底下的子元素绑定`key`值。
```
<div>
    <template v-for="(item, index) in items">
      <div :key="item.message">{{ item.message }}</div>
      <div :key="item.message + index" class="divider"></div>
    </template>
  </div>
```
data
```
 data() {
    return {
     items: [
       {
         message: 'hello'
       },
       {
         message: 'world'
       },
       {
         message: 'welcome'
       }
     ]
    }
  },
```
生成`DOM`时，并不会生成实际`DOM`节点。

![template使用v-for](https://user-gold-cdn.xitu.io/2020/1/23/16fd0c847a668592?w=264&h=137&f=png&s=5778)

#### （4）渲染自定义组件列表
在自定义组件上，可以使用`v-for`渲染自定义组件列表，通过`props`将数据传递给组件。
> 在组件上使用`v-for`时，`key`是必须的。

`v-for`渲染自定义组件列表，将`item`通过`props`传递给组件。
```
 <my-component 
    v-for="(item, index) in items" 
    :key="item.message + index" 
    :item="item">
  </my-component>
```
`my-component`组件使用`props`接收父组件传来的数据。
```
<template>
  <div>{{item.message}}</div>
</template>

<script>
export default {
  props: ['item'],
  data() {
    return { }
  }
}
</script>
```

### react
react中使用`map()`方法来实现列表渲染。
#### （1）渲染数组
遍历数组中的每个元素，得到一组`jsx`元素列表。数组中的每一个元素需要添加唯一的`key`值。
```
  render() {
    const items = [
      {
        message: 'hello'
      },
      {
        message: 'world'
      },
      {
        message: 'welcome'
      }
    ];
    const listItems = items.map((item, index) => <div key={item.message + index}>{item.message}</div>);
    return (
      <div>
        {listItems}
      </div>
    );
  }
```
#### （2）渲染对象
对于对象，可以采用方法通过`Object.keys()`或者`Object.entries()`来遍历对象。
```
  render() {
    const obj = {
      name: 'xiaoming',
      age: 18,
      sex: 'male',
      height: 175
    };
    const renderObj = (obj) => {
      const keys = Object.keys(obj);
      return keys.map((item, index) => <div key={index}>{obj[item]}</div>);
    };
    return (
      <div>
        {renderObj(obj)}
      </div>
    );
  }
```
#### （3）渲染自定义组件列表
渲染自定义组件列表与`vue`中类似，需要给组件添加`key`值标识。
```
  render() {
    const items = [
      {
        message: 'hello'
      },
      {
        message: 'world'
      },
      {
        message: 'welcome'
      }
    ];
    const listItems = items.map((item, index) => 
          <ListItem message={item.message} key={item.message + index} />);
    return (
      <div>
        {listItems}
      </div>
    );
  }
```
## 12.计算属性(computed vs useMemo+useCallback)
计算属性表示根据组件的数据（包含组件自身的数据和接收父组件的`props`）需要二次计算并“保存”的数据，使用计算属性的好处是避免每次重复计算的开销（比如遍历一个巨大的数组并做大量的计算）。
### vue
vue中用`computed`来表示计算属性，可以定义多个计算属性，计算属性可以互相调用。计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值。vue中可以直接使用`this.xxx`直接获取到计算属性。
#### （1）基本用法
下面声明计算属性`reversedMessage`依赖于`message`，这就意味着只要 `message`还没有发生改变，多次访问`reversedMessage`计算属性会立即返回之前的计算结果。
```
<div>
    message: <input type="text" v-model="message" />
    <div>{{reversedMessage}}</div>
</div>
```
script
```
  data() {
    return {
      message:''
    }
  },
  computed: {
    reversedMessage() {
      return this.message.split('').reverse().join('')
    }
  }
```
#### （2）计算属性的setter
技术属性默认只有`getter`，也可以使用`setter`，当修改计算属性的时候，会触发`setter`回调。

script
```
data() {
    return {
      message:'',
      info: ''
    }
  },
  computed: {
    reversedMessage: {
      get() { // get回调
        return this.message.split('').reverse().join('')
      },
      set(newValue) { // set回调
        this.info = newValue
      }
    }
  },
  methods: {
    changeMessage(event) {
      // 修改reversedMessage计算属性
      this.reversedMessage = event.target.value;
    }
  }
```
### react
react hooks使用`useMemo`表示`memoized`的值，使用`useCallback`表示`memoized`的回调函数，实现与vue中`computed`类似的功能。
> 适用场景：子组件使用了`PureComponent`或者`React.memo`，那么你可以考虑使用`useMemo`和`useCallback`封装提供给他们的`props`，这样就能够充分利用这些组件的浅比较能力。
#### （1）useMemo
`useMemo`返回一个`memoized`的值。`useMemo`会依赖某些依赖值，只有在某个依赖项改变时才会重新计算`memoized`值。如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。`useMemo`可以作为性能优化的手段。
> 传入`useMemo`的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于`useEffect`的适用范畴，而不是`useMemo`。
```
function NewComponent(props) {
  const { num } = props;
  const [size, setSize] = useState(0);
  // max是useMemo返回的一个memoized的值
  const max = useMemo(() => Math.max(num, size), [num, size]);
  return (<div>
    <input
      type="number"
      value={size}
      onChange={(e) => setSize(e.target.value)} />
    <div>Max {max}</div>
  </div>);
}
```
#### （2）useCallback
`useCallback`把内联回调函数及依赖项数组作为参数传入`useCallback`，它将返回该回调函数的`memoized`版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如`shouldComponentUpdate`的子组件时，它将非常有用。
```
function NewComponent(props) {
  const [message, setMessage] = useState('hello world.');
  const handleChange = useCallback((value) => {
    setMessage(value);
  }, []);
  return (<div>
    <input
      type="number"
      value={message}
      onChange={(e) => handleChange(e.target.value)} />
    <div>{message}</div>
  </div>);
}
```
## 13.侦听器(watch vs getDerivedStateFromProps + componentDidUpdate)
侦听器是指通过监听`props`或者组件数据（`data`或`state`）的变化来执行一些异步或者数据操作。
### vue
vue中主要通过`watch`监听`props`、`data`、`computed`（计算属性）的变化，执行异步或开销较大的操作。

下面`ProjectManage`组件通过`watch`监听`projectId prop`的变化获取对应的项目信息。
```
export default {
  name: "ProjectManage",
  props: ["projectId"],
  data() {
    return {
      projectInfo: null
    };
  },
  watch: {
    projectId(newVaue, oldValue) {
      if (newVaue !== oldValue) {
        this.getProject(newValue);
      }
    }
  },
  methods: {
    getProject(projectId) {
      projectApi
        .getProject(projectId)
        .then(res => {
          this.projectInfo = res.data;
        })
        .catch(err => {
          this.$message({
            type: "error",
            message: err.message
          });
        });
    }
  }
};
```
### react
react中通过`static getDerivedStateFromProps()`和`componentDidUpdate()`实现监听器的功能。
#### （1）`static getDerivedStateFromProps()`
`getDerivedStateFromProps`会在调用`render`方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新`state`，如果返回`null`则不更新任何内容。

关于`getDerivedStateFromProps`有2点说明：
- 1）不管是`props`变化、执行`setState`或者`forceUpdate`操作都会在每次渲染前触发此方法。
- 2）当`state`的值在任何时候都取决于`props`的时候适用该方法。
```
class NewComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      info: ''
    }
  }
  static getDerivedStateFromProps(nextProps, prevState) {
    // state中的info根据props中的info保持同步
    if (nextProps.info !== prevState.info) {
      return {
        info: nextProps.info
      }
    }
    return null;
  }
  render() {
    const { info } = this.state;
    return <div>{info}</div>
  }
}
```
#### （2）`componentDidUpdate()` 
`componentDidUpdate()`方法在组件更新后被调用。首次渲染不会执行此方法。当组件更新后，可以在此处操作`DOM`、执行`setState`或者执行异步请求操作。
```
componentDidUpdate(prevProps, prevState, snapshot)
```
关于`componentDidUpdate`有4点说明:
- 1）`componentDidUpdate()`的第三个参数`snapshot`参数来源于`getSnapshotBeforeUpdate()`生命周期的返回值。若没有实现`getSnapshotBeforeUpdate()`，此参数值为`undefined`。
- 2）可以在`componentDidUpdate()`中直接调用`setState()`，但是它必需被包裹在一个条件语句里，否则会导致死循环。
- 3）可以在`componentDidUpdate()`对更新前后的`props`进行比较，执行异步操作。
- 4）如果`shouldComponentUpdate()`返回值为`false`，则不会调用`componentDidUpdate()`。

下面`NewComponent`组件在`componentDidUpdate()`里判断
```
class NewComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      projectInfo: null
    }
  }
  getProject = (projectId) => {
    projectApi
      .getProject(projectId)
      .then(res => {
        this.projectInfo = res.data;
      })
      .catch(err => {
        message.error(err.message);
      });
  }
  componentDidUpdate(prevProps) {
    if (this.props.projectId !== prevProps.projectId) {
      this.getProject(this.props.projectId);
    }
  }
  render() {
    const { projectInfo } = this.state;
    return <React.Fragment>
      <div>{projectInfo.name}</div>
      <div>{projectInfo.detail}</div>
      </React.Fragment>
  }
}
```
## 14.ref 
`ref`用来给元素或子组件注册引用信息，允许我们访问子组件或者子节点。

`ref`常用于：
- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
### vue
通过给组件或者子元素设置`ref`这个`attribute`为子组件或者子元素赋予一个ID引用。
> `$refs`只会在组件渲染完成之后生效，并且它们不是响应式的。这仅作为一个用于直接操作子组件的“逃生舱”——你应该避免在模板或计算属性中访问`$refs`。
#### （1）子元素引用ref
子元素上引用`ref`
```
 <div>
    <input type="text" v-model="message" ref="inputMessage"  />
  </div>
```
加载完毕后使输入框自动获取焦点
```
mounted() {
  this.$refs.inputMessage.focus();
}
```
#### （2）子组件引用ref
子组件引用`ref`常用于父组件使用子组件的方法。
> 常用表单验证就是采用这种方式验证的。
```
<template>
  <div>
    <el-form ref="createForm" label-width="80px" :model="form" :rules="rules">
      <el-form-item label="名称" prop="name">
        <el-input v-model="form.name"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="form.email"></el-input>
      </el-form-item>
    </el-form>
    <el-button @click="handleSubmit">提交</el-button>
  </div>
</template>

<script>
export default {
  name: 'CreateForm',
  data() {
    return {
      form: {
        name: '',
        email: ''
      },
      rules: {
        name: [{required: true, message: '名称不能为空', trigger: 'blur'}],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change']}
        ] 
      }
    }
  },
  methods: {
    handleSubmit() {
      this.$refs.createForm.validate((valid) => {
        console.log(valid);
      })
    }
  }
}
</script>
```
### react
react中不像vue中直接给`ref`传字符串类型值，`class`组件通过`React.createRef`绑定`ref`属性（React v16.0版本之后），函数组件通过`useRef`绑定`ref`属性，还可以使用`React.forwardRef`用于转发`ref`属性到子组件中。

#### （1）class组件绑定ref
通过`React.createRef`在构造函数中生成`ref`，在绑定到`input`元素上，加载完成后自动聚焦。
```
class NewComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      message: 'hello world'
    };
    this.inputRef = React.createRef();
  }
  componentDidMount() {
    this.inputRef.current.focus();
  }
  render() {
    const { message } = this.state;
    return (<div>
      <input
        type="number"
        ref={this.inputRef}
      />
      <div>{message}</div>
    </div>);
  }
}
```
#### （2）函数组件绑定ref
函数组件可以使用`useRef`绑定`ref`属性。`useRef`返回一个可变的`ref`对象，其 `.current`属性被初始化为传入的参数（`initialValue`）。返回的`ref` 对象在组件的整个生命周期内保持不变。
```
function NewComponent() {
  const [message, setMessage] = useState('hello world.');
  const inputRef = useRef(null);
  useEffect(() => {
    inputRef.current.focus();
  }, []);
  return (<div>
    <input type="number" ref={inputRef} />
    <div>{message}</div>
  </div>);
}
```
#### （3）React.forwardRef 转发 ref 到子组件
`React.forwardRef`会创建一个React组件，这个组件能够将其接受的`ref`属性转发到其组件树下的另一个组件中。

这种技术并不常见，但在以下两种场景中特别有用：
- 转发`refs`到`DOM`组件
- 在高阶组件中转发`refs`

父组件直接传递`ref`属性给子组件`NewComponent`。
```
function Parent() {
  const inputRef = useRef(null);
  useEffect(() => {
    inputRef.current.focus();
  }, []);
  return <div>
     <NewComponent ref={inputRef}><h2>This is refs.</h2></NewComponent>
  </div>;
}
```
子组件使用`React.forwardRef` 接受渲染函数作为参数，父组件加载完成后聚焦到`input`输入框。
```
const NewComponent = React.forwardRef((props, ref) => (<div>
  <input type="number" ref={ref} />
  <div>{props.children}</div>
</div>));
```
## 15.表单(v-model vs value)
对于表单，vue中使用`v-model`在表单组件上实现双向数据绑定，react中通过在表单组件上绑定`value`属性以受控组件的形式管理表单数据。
### vue
`v-model`指令在表单`<input>`、`<textarea>`及`<select>`元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。`v-model`本质上不过是语法糖。

`v-model`在内部为不同的输入元素使用不同的属性并抛出不同的事件：
- `text`和`textarea`元素使用`value`属性和`input`事件；
- `checkbox`和`radio`使用`checked`属性和`change`事件；
- `select`字段将`value`作为`prop`并将`change`作为事件。

#### （1）基本用法
##### Ⅰ.文本
`input`输入框上绑定`v-model`属性绑定`msg`，当修改`input`输入值时，`msg`会自动同步为用户输入值。
```
 <div>
  <input v-model="msg" />
</div>
```
`v-model`写法等价于`:value`和`@input`的结合，`:value`绑定输入值，`@input`表示接收输入事件修改`msg`的值为输入的值，从而实现双向绑定。
```
<div>
  <input :value="msg" @input="e => (msg = e.target.value)" />
</div>
```
##### Ⅱ.复选框
单个复选框绑定到布尔值
```
<div>
  <input type="checkbox" v-model="checked" />
  <label for="checkbox">{{ checked }}</label>
</div>
```
多个复选框绑定到数组
```
<div>
    <input type="checkbox" id="apple" value="apple" v-model="selectedFruits" />
    <label for="jack">apple</label>
    <input
      type="checkbox"
      id="banana"
      value="banana"
      v-model="selectedFruits"
    />
    <label for="john">banana</label>
    <input type="checkbox" id="mango" value="mango" v-model="selectedFruits" />
    <label for="mango">mango</label>
    <div>Selected fruits: {{ selectedFruits }}</div>
  </div>
```
data
```
 data() {
    return {
      selectedFruits: []
    };
  }
```
##### Ⅲ.选择框
1）选择框单选时,绑定字符串
```
<div>
  <select v-model="selected">
    <option
      v-for="option in options"
      :value="option.value"
      :key="option.value"
    >
      {{ option.text }}
    </option>
  </select>
  <div>Selected: {{ selected }}</div>
</div>
```
data
```
data() {
  return {
    selected: "A",
    options: [
      { text: "One", value: "A" },
      { text: "Two", value: "B" },
      { text: "Three", value: "C" }
    ]
  };
}
```
2）选择框多选时，绑定的是数组
```
<div>
  <select v-model="selected" multiple>
    <option
      v-for="option in options"
      :value="option.value"
      :key="option.value"
    >
      {{ option.text }}
    </option>
  </select>
  <div>Selected: {{ selected }}</div>
</div>
```
data
```
 data() {
  return {
    selected: ["A"], //多选时绑定的是数组
    options: [
      { text: "One", value: "A" },
      { text: "Two", value: "B" },
      { text: "Three", value: "C" }
    ]
  };
}
```
####（2）修饰符
vue对于`v-model`扩展了`.lazy`、`.number`、`.trim`修饰符增强功能。
##### Ⅰ.lazy
在默认情况下，`v-model`在每次`input`事件触发后将输入框的值与数据进行同步。添加了`.lazy`修饰符，转变为`change`事件进行同步。
```
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```
##### Ⅱ.number
`.number`修饰符可以自动将用户的输入转换为数值类型，尤其是处理数字类型表单项时尤其有用。
```
 <input type="number" v-model.number="num" />
```
##### Ⅲ.trim
`.trim`修饰符可以自动过滤用户输入的首尾空白字符。
```
 <input v-model.trim="msg" />
```
#### （3）自定义组件使用`v-model`
一个组件上的`v-model`默认会利用名为`value`的`prop`和名为 `input`的事件。

`InputMessage`绑定`v-model`值为`msg`
```
  <InputMessage v-model="msg" />
```
`InputMessage`组件通过`value props`接收值，`emit input`事件给父组件，修改父组件中`msg`的值。
```
<template>
  <input v-model="message" @input="$emit('input', $event.target.value)" />
</template>

<script>
export default {
  name: "InputMessage",
  props: ["value"],
  data() {
    return {
      message: this.value
    };
  }
};
```
#### （4）不能使用v-model管理的表单组件（file等）
对于像`<input type="file" />`这种类型的表单组件，不能使用`v-model`管理组件数据，可以通过`refs`管理表单组件的数据，这点和react中的非受控组件一致。
```
<template>
  <div>
    <input type="file" ref="file" @change="fileChange" />
  </div>
</template>

<script>
export default {
  name: "InputFile",
  data() {
    return {};
  },
  methods: {
    fileChange() {
      const file = this.$refs.file.files[0];
      console.log(file);
    }
  }
};
</script>
```
### react
react中，表单元素（`<input> <select> <checkbox>`）通常维护自己的`state`，将`state`赋值给`value`属性，并根据用户输入通过`setState()`更新`state`，以这种方式控制的表单元素称为“受控组件”。
#### （1）受控组件
受控组件中，`state`作为组件“唯一的数据源”，组件还控制着用户操作过程中表单发生的操作。
```
class CreateForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: ''
    }
  }

  nameChange = (event) => { // 接收事件作为参数
   this.setState({
     name: event.target.value
   });
  }

  render() {
    const { name } = this.state; 
    return (
    <div>
      <input value={name} onChange={this.nameChange} />
      <div> name: {name} </div>
    </div>)
  }
}
```
#### （2）非受控组件
在react中，对于不能使用`state`方式管理的表单组件称为非受控组件，非受控组件的值不能通过代码控制，表单数据交由`DOM`节点来处理。对于非受控组件，可以使用`ref`从`DOM`节点中获取表单数据。

`<input type="file" />`始终是一个非受控组件，通过创建`ref`的形式获取文件数据。
```
class CreateForm extends React.Component {
  constructor(props) {
    super(props);
    this.fileRef = React.createRef(null);
  }

  fileChange = (event) => {
    event.preventDefault();
    const file = this.fileRef.current.files[0];
    console.log(file)
  }

  render() {
    return (
    <div>
      <input type="file" ref={this.fileRef} onChange={this.fileChange} />
    </div>)
  }
}
```
## 16.插槽(slot vs Render Props+this.props.children)
vue和react中都实现了“插槽”(内容分发)功能，vue中主要通过`slot`实现插槽功能，react中通过`this.props.children`和`Render props`实现类似vue中的插槽功能。
### vue
vue中通过`<slot>`实现插槽功能，包含默认插槽、具名插槽、作用域插槽。
#### （1）默认插槽
默认插槽使用`<slot></slot>`在组件内预留分发内容的“占位”，在组件起始标签和结束标签可以包含任何代码，例如html或者其他组件。

关于默认插槽，有2点说明：
> （1）如果不使用插槽，插入到组件中的内容不会渲染。
> <br>（2）插槽可以设置后备内容，在不插入任何内容时显示。

使用默认插槽的组件：
```
<template>
  <div>
    <h2>Slot：</h2>
     <slot>Default content.</slot> // 使用slot预留插槽占位，slot中的内容作为后备内容
  </div>
</template>

<script>
export default {
  name: "SlotComponent",
  data() {
    return {};
  }
};
</script>
```
父组件使用该组件，在组件起始标签和结束标签添加插槽内容。
```
<slot-component>
  <h2>This is slot component.</h2>
</slot-component>
```
最终插槽内容会被插入到组件`<slot></slot>`占位的位置。
```
  <div>
    <h2>Slot：</h2>
    <h2>This is slot component.</h2>
  </div>
```
当`<slot-component>`没有添加插槽内容时，会渲染默认插槽内容。
```
<div>
    <h2>Slot：</h2>
    Default content.
  </div>
```
#### （2）具名插槽
默认插槽只能插入一个插槽，当插入多个插槽时需要使用具名插槽。`slot`元素有一个默认的`attribute name`，用来定义具名插槽。
> 默认插槽的`name`是`default`。

在向具名插槽提供内容的时候，我们可以在一个 `<template>`元素上使用`v-slot`指令，并以`v-slot `的参数的形式提供其名称，将内容插入到对应的插槽下。
> `v-slot:`也可以简写为`#`，例如`v-slot:footer`可以被重写为`#footer`。

插槽组件`slot-component`有`header footer`两个具名插槽和一个默认插槽。
```
<template>
  <div>
    <header>
      <slot name="header">
        Header content.
      </slot>
    </header>
    <main>
      <slot>
        Main content.
      </slot>
    </main>
    <footer>
      <slot name="footer">
        Footer content.
      </slot>
    </footer>
  </div>
</template>
```
向插槽中分别插入内容：
```
<slot-component>
  <template v-slot:header>
    <div>This is header content.</div>
  </template>
  <template>
    <div>This is main content.</div>
  </template>
  <template #footer>
    <div>This is footer content.</div>
  </template>
</slot-component>
```
最终html会被渲染为：
```
<div>
  <header>
    <div>This is header content.</div>
  </header>
  <main>
    <div>This is main content.</div>
  </main>
  <footer>
    <div>This is footer content.</div>
  </footer>
</div>
```
#### （3）作用域插槽
有时候我们需要在父组件中显示插槽组件的数据内容，这时候作用域插槽就派上用场了。

作用域插槽需要在`<slot>`元素上绑定`attribute`，这被称为插槽`prop`。在父级作用域中，可以使用带值的`v-slot`来定义我们提供的插槽`prop`的名字。

插槽组件
```
<template>
  <div>
    <header>
      <slot name="header">
        Header content.
      </slot>
    </header>
    <main>
      <slot :person="person">
        {{ person.name }}
      </slot>
    </main>
  </div>
</template>

<script>
export default {
  name: "SlotComponent",
  data() {
    return {
      person: {
        name: "xiaoming",
        age: 14
      }
    };
  },
  methods: {}
};
</script>
```
父组件作用域将包含所有插槽`prop`的对象命名为`slotProps`，也可以使用任意你喜欢的名字。
```
 <slot-component>
  <template slot="header">
    <div>This is header content.</div>
  </template>
  // 使用带值的`v-slot`来定义插槽`prop`的名字
  <template v-slot:default="slotProps"> 
    <div>{{ slotProps.person.name }}</div>
    <div>{{ slotProps.person.age }}</div>
  </template>
</slot-component>
```
最终html将被渲染为：
```
<div>
  <header>
    <div>This is header content.</div>
  </header>
  <main>
    <div>xiaoming</div>
    <div>14</div>
  </main>
</div>
```
### react
react中通过`this.props.children`和`Render props`实现类似vue中的插槽功能。
#### （1）this.props.children
每个组件都可以通过`this.props.children`获取包含组件开始标签和结束标签之间的内容，这个与vue中的默认插槽类似。
> 在`class`组件中使用`this.props.children`，在`function`组件中使用`props.children`。
`class`组件使用`this.props.children`获取子元素内容。
```
class NewComponent extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <div>{this.props.children}</div>
  }
}
```
`function`组件使用`props.children`获取子元素内容。
```
function NewComponent(props) {
  return <div>>{props.children}</div>
}
```
父组件使用`NewComponent`组件
```
 <NewComponent>
   <h2>This is new component header.</h2>
   <div>
     This is new component content.
   </div>
 </NewComponent>
```
最终html将被渲染为：
```
<div>
  <h2>This is new component header.</h2>
  <div>This is new component content.</div>
</div>
```
#### （2）Render props
`render prop`是指一种在React组件之间使用一个值为函数的`prop`共享代码的技术。`render prop`是一个用于告知组件需要渲染什么内容的函数`prop`。

比如我们常用的`react-router-dom`中的`Route`的`component prop`就采用了典型的`render prop`的用法。
```
<Router history={browserHistory}>
    <Route path="/" component={Index}> // component props接收具体的组件
      <IndexRoute component={HomePage} />
      <Route path="/users" component={Users} />
    </Route>
  </Router>
```
通过多个`render prop`即可实现类似vue中具名插槽的功能。

`NewComponent`定义了`header`、`main`、`footer prop`。
```
class NewComponent extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    const { header, main, footer, children } = this.props;
    return (<div>
              <header>
                {header || (<div>Header content.</div>)}
              </header>
              <main>
                {main || (<div>Main content.</div>)}
              </main>
              {children}
              <footer>
                {footer || (<div>Footer content.</div>)}
              </footer>
            </div>);
  }
}
```
父组件向子组件传递`render prop`。
```
 <NewComponent 
    header={<div>This is header content.</div>}
    content={<div>This is main content.</div>}
    footer={<div>This is footer content.</div>}>
    <div>
      This is new component children.
    </div>
</NewComponent>
```
最终html将被渲染为
```
<div>
  <header>
    <div>This is header content.</div>
  </header>
  <main>
    <div>This is main content.</div>
  </main>
  <div>This is new component children.</div>
  <footer>
    <div>This is footer content.</div>
  </footer>
</div>
```
## 结语
以上就是博主关于react和vue的一些对比以及个人思考的中部，觉得有收获的可以关注一波，点赞一波，码字不易，万分感谢，后续下部会尽快持续更新，感谢关注。
