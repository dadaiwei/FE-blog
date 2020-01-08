# 基于react-draggable实现边界拖拽功能

## 1.前言
博主最近在开发过程中，遇到这样一个小小的需求：
<br>
左边面板部分支持拖拽功能，可以让用户可以看到完整的目录。
![](https://user-gold-cdn.xitu.io/2019/6/26/16b93f483661abdc?w=992&h=569&f=png&s=17883)
>Tips：vscode编辑器资源管理器部分也支持拖拽功能，与边界拖拽功能类似。
## 2.思考
实现拖拽可以用以下两种方案：
<br>
（1）借助于html5中新增拖放相关的api
<br>
大致思路：
<br>设置元素可拖放（`draggable`）=> `ondragstart`设定拖放的数据类型和值 => `ondragover`设置在何处放置数据 => `ondrop`将元素拖放到目的地
<br>其中关于鼠标移动距离的计算可以借用`event.clientX`获取。
<br>
（2）借助于插件来实现
<br>博主去github上一顿找，找到一个还不错的插件react-draggable，[npm链接](https://user-gold-cdn.xitu.io/2019/6/26/16b9407467a71e57)。
<br>
>Tips：`react-draggable`的拖拽通过`CSS`中的`transform: translate(x, y)`来实现目标元素的移动。
## 3.react-draggable使用
（1）安装`react-draggable`
```
$ npm install react-draggable --save
```
（2）引用
```
import Draggable from 'react-draggable'; 
```
（3）常用props
```
allowAnyClick: boolean // 默认false，设为true非左键可实现点击拖拽
axis: string // 'x'：x轴方向拖拽、'y'：y轴方向拖拽、'none'：禁止拖拽
bounds: { left: number, top: number, right: number, bottom: number } | string 
    // 限定移动的边界，接受值：
    //（1）'parent':在移动元素的offsetParent范围内
    //（2）一个选择器，在指定的Dom节点内
    //（3）{ left: number, top: number, right: number, bottom: number }对象，限定每个方向可以移动的距离
cancel：制定给一个选择器组织drag初始化，例如'.body'
defaultClassName：string // 拖拽ui类名，默认'react-draggable'
drfaultClassNameDragging：string // 正在拖拽ui类名，默认'eact-draggable-dragging'
defaultClassNameDragged：string //拖拽后的类名，默认'react-draggable-dragged'
defaultPosition：{ x: number, y: number } // 起始x和y的位置
disabled：boolean // true禁止拖拽任何元素
grid：[number, number] // 正在拖拽的网格范围
handle：string // 初始拖拽的的选择器'.handle'
offsetParent：HTMLElement // 拖拽的offsetParent
onMouseDown: (e: MouseEvent) => void // 鼠标按下的回调
onStart: DraggableEventHandler // 开始拖拽的回调
onDrag：DraggableEventHandler // 拖拽时的回调
onStop：DraggableEventHandler // 拖拽结束的回调
position: {x: number, y: number} // 控制元素的位置
positionOffset: {x: number | string, y: number | string} // 相对于起始位置的偏移
scale：number // 定义拖拽元素的缩放
```
（4）Example
```
class App extends React.Component {
  eventLogger = (e: MouseEvent, data: Object) => {
    console.log('Event: ', e);
    console.log('Data: ', data);
  };
  render() {
    return (
      <Draggable
        axis="x"
        handle=".handle"
        defaultPosition={{x: 0, y: 0}}
        position={null}
        grid={[25, 25]}
        scale={1}
        onStart={this.handleStart}
        onDrag={this.handleDrag}
        onStop={this.handleStop}>
        <div>
          <div className="handle">Drag from here</div>
          <div>This readme is really dragging on...</div>
        </div>
      </Draggable>
    );
  }
}
```
## 4.实现
### 4.1思路
（1）左右box最好采用`flex`布局，左边box初始`width`设为`Npx`，右边box width为`calc(100% - Npx)`。
<br>
（2）在左边box内部设置一个拖拽的边界（drag-box），宽度为`5px`，`background-color`设为`transparent`，`cursor`设为`col-resiz`，如下图绿色部分所示：
![](https://user-gold-cdn.xitu.io/2019/7/1/16badaa512cdf067?w=438&h=279&f=png&s=12570)
（3）采用react-draggable包裹drag-box 沿x轴方向移动，设定可拖拽的范围，移动时设置`background-color`为可见色，同时更新左右box的`width`即可，拖拽结束drag-box `background-color`重置为`transparent`。
### 4.2代码实现
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码中引用了`styled-components`，这是一款是针对React写的一套`css in js`框架，简单来讲就是在`js`中写`css`，可以轻易实现组件`css`模块化，类似于`vue`组件中给`style`添加了`scoped`属性。
想要学习`styled-components`的小伙伴可以查看[链接](`styled-components`)。
```
import React, { Component } from 'react';
import Draggable from 'react-draggable';
import styled from 'styled-components';

// 容器
const Container = styled.div`
  display: flex;
  justify-content: flex-start;
`;

// 左边内容部分
const LeftContent = styled.div`
  position: relative;
  width: ${props => props.width}px;
  height: 100vh;
  padding: 20px;
  background-color: #E6E6FA;
  overflow: hidden;
  flex-grow:1;
`;

// 拖拽部分
const DraggableBox = styled.div`
  position: absolute;
  left: ${props => props.left}px;
  top: 0;
  width: 5px;
  height: 100vh;
  background-color: ${props => props.background};
  cursor: col-resize;
  z-index: 1000;
`;

// 右边内容部分
const RightContent = styled.div`
  width: calc(100% - ${props => props.leftBoxWidth}px);
  height: 100vh;
  padding: 20px;
  background-color: #FFF;
  flex-grow:1;
  z-index: 100;
`;

const Li = styled.li`
  white-space: nowrap;
`;

class DraggableExp extends Component {
  state = {
    initialLeftBoxWidth: 150, // 左边区块初始宽度
    leftBoxWidth: 150, // 左边区块初始宽度
    leftBoxMinWidth: 100, // 左边区块最小宽度
    leftBoxMaxWidth: 300, // 左边区块最大宽度
    dragBoxBackground: 'transparent' // 拖拽盒子的背景色
  }

  // 拖动时设置拖动box背景色，同时更新左右box的宽度
  onDrag = (ev, ui) => {
    const { initialLeftBoxWidth } = this.state;
    const newLeftBoxWidth = ui.x + initialLeftBoxWidth;
    this.setState({
      leftBoxWidth: newLeftBoxWidth,
      dragBoxBackground: '#FFB6C1'
    });
  };

  // 拖拽结束，重置drag-box的背景色
  onDragStop = () => {
    this.setState({
      dragBoxBackground: 'transparent'
    });
  };

  render() {
    const { initialLeftBoxWidth, 
            leftBoxWidth, 
            leftBoxMinWidth, 
            leftBoxMaxWidth, 
            dragBoxBackground } = this.state;
    return (
      <Container>
        <LeftContent width={leftBoxWidth}>
          <h3 style={{paddingLeft: 20}}>目录</h3>
          <ul>
            <Li>目录1</Li>
            <Li>目录2</Li>
            <Li>目录3</Li>
            <Li>这是个非常长非常长非常长的目录</Li>
          </ul>
          <Draggable 
            axis="x"
            defaultPosition={{ x: 0, y: 0 }}
            bounds={{ left: leftBoxMinWidth - initialLeftBoxWidth, right: leftBoxMaxWidth - initialLeftBoxWidth }}
            onDrag={this.onDrag}
            onStop={this.onDragStop}>
            <DraggableBox
              left={initialLeftBoxWidth - 5} 
              background={dragBoxBackground} />
          </Draggable>
        </LeftContent>
        <RightContent leftBoxWidth={leftBoxWidth}>
          <h3>这里是内容块</h3>
        </RightContent>
      </Container>
    )
  }
}

export default DraggableExp;
```
### 4.3实现效果
起始状态：
![](https://user-gold-cdn.xitu.io/2019/7/1/16bad8f1d2174938?w=474&h=314&f=png&s=9768)
拖拽过程中：
![](https://user-gold-cdn.xitu.io/2019/7/1/16bad8f755161d4a?w=536&h=312&f=png&s=10581)
拖拽结束：
![](https://user-gold-cdn.xitu.io/2019/7/1/16bada8be8f15278?w=522&h=209&f=png&s=9746)

以上就是博主借用react-draggable实现边界拖拽的小功能，感兴趣的小伙伴可以自己动手尝试一下。
<br>
（觉得不错的小伙伴可以给文章点个赞，码字不易，灰常感谢^_^）