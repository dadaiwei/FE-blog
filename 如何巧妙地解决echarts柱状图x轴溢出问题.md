# 如何巧妙地解决echarts柱状图x轴溢出问题

## 问题来源
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;博主最近在使用echarts绘制柱状图的时候，就遇到了x轴左右两边溢出的问题。相信很多小伙伴也遇到这种问题，而且最最关键的是博主找遍了网上，**没找到解决方案！！！**
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;废话不多说，直接上图，明显看出柱状图x轴左右两边都溢出了。
<figure class="half">
<image width="400" src="https://user-gold-cdn.xitu.io/2019/6/1/16b11284885f932a?w=716&h=392&f=png&s=10961">
<br>
What Happened？
<br>
<image src="https://user-gold-cdn.xitu.io/2019/6/1/16b110a3126ce4a7?w=300&h=220&f=png&s=110845">
</figure>

## 代码分析
瞅一眼代码：

```
// 生成echarts实例
const mainChart = echarts.init(document.getElementById('main')); 
const series = [
        {
            data: [[0, 120], [1, 200], [2, 150], [3, 80], [4, 70], [5, 110], [6, 130]],
            type: 'bar',
            barWidth: 20
        },
        {
            data: [[0, 80], [1, 180], [2, 120], [3, 130], [4, 100], [5, 180], [6, 200]],
            type: 'bar',
            barWidth: 20
        },
        {
            data: [[0, 80], [1, 180], [2, 120], [3, 130], [4, 100], [5, 180], [6, 200]],
            type: 'bar',
            barWidth: 20
        }];
const option = {
    xAxis: {
       type: 'value', // 数值轴
       boundaryGap: true // 坐标轴两边是否留白
    },
    yAxis: {
        type: 'value' // 数值轴
    },
    series // 系列列表
};
mainChart.setOption(option); // 生成配置项
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;配置写的貌似没有问题，`xAxis`的`boundaryGap`属性值为`true`，柱状图显示应该没问题啊。去echarts github上搜索下，看到了一个类似的issue，貌似是echarts的一个Bug。
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;于是，博主只能自己去研究echarts的配置项，偶然发现一个很有意思的属性`onZero`。
<br>
<image src="https://user-gold-cdn.xitu.io/2019/6/1/16b118fe8f512acc?w=783&h=134&f=png&s=14010">
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分析：目前来看y轴是位于x轴的0刻度上，由于x轴内容显示不下，才会导致溢出超出范围。
<br>
## 思考验证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;换个思路：只要我们设置x轴的`min`比`series`中`data`最小值再小一个`interval`，x轴的`max`比`series`中`data`最大值再大一个`interval`，然后再设置`yAxis`的`onZero`值为`false`，应该就可以让内容显示在范围内了。
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;按照上面的思路，我们来进行验证：
```
const mainChart = echarts.init(document.getElementById('main'));
const series = [
        {
            data: [[0, 120], [1, 200], [2, 150], [3, 80], [4, 70], [5, 110], [6, 130]],
            type: 'bar', // 柱状图
            barWidth: 15 // 柱条宽度设为15，避免挤太近不好看
        },
        {
            data: [[0, 80], [1, 180], [2, 120], [3, 130], [4, 100], [5, 180], [6, 200]],
            type: 'bar',
            barWidth: 15
        },
        {
            data: [[0, 80], [1, 180], [2, 120], [3, 130], [4, 100], [5, 180], [6, 200]],
            type: 'bar',
            barWidth: 15
        }];
let xAxisMin = 0; // x轴最小值
let xAxisMax = 0; // x轴最大值
let xAxisMinInterval = 0; // x轴最小处的间隔
let xAxisMaxInterval = 0; // x轴最大处的间隔
let xValues = [];
// 获取所有x轴data值
for (let s of series) {
  for (let d of s.data) {
    if (!xValues.includes(d[0])) {
        xValues.push(d[0]);
    }
  }
}
xValues.sort((a ,b) => a - b); // 按大小对x轴data排序
xAxisMinInterval = xValues[1] - xValues[0]; // 最小处的间隔
xAxisMaxInterval = xValues[xValues.length - 1] - xValues[xValues.length - 2]; // 最大处的间隔
xAxisMin = xValues[0] - xAxisMinInterval; // x轴最小值
xAxisMax = xValues[xValues.length - 1] + xAxisMaxInterval; // x轴最大值
const option = {
    xAxis: {
        type: 'value', // 数值轴
        min: xAxisMin,  // x轴最小值
        max: xAxisMax, // x轴最大值
        boundaryGap: true // x轴两边是否留白
    },
    yAxis: {
        type: 'value',
        axisLine: {
            onZero: false // y轴是否在x轴0刻度上
        }
    },
    series
};
mainChart.setOption(option);
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;看下重新渲染出的图：
<figure class="half">
<image src="https://user-gold-cdn.xitu.io/2019/6/1/16b11aa778d4ad53?w=677&h=405&f=png&s=11776">
</figure>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们的问题完美解决，柱状图完整地显示在可视区域内。
<figure class="half">
<image src="https://user-gold-cdn.xitu.io/2019/6/1/16b11eb1ecde21eb?w=302&h=263&f=png&s=93335">
</figure>

## 另一个案例
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再看另一个柱状图x轴溢出问题，明显发现x轴左边溢出了。
<br>
<image src="https://user-gold-cdn.xitu.io/2019/6/1/16b11c45fe84cf66?w=700&h=390&f=png&s=10189">
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;看代码，区别在于
`xAxis type`值为`category`（类目轴）。
```
const mainChart = echarts.init(document.getElementById('main'));
const option = {
    xAxis: {
        type: 'category', // 类目轴
        data: [0, 1, 2, 3, 4, 5, 6],
        boundaryGap: false // x轴两边不留白
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [120, 200, 150, 80, 70, 110, 130],
        type: 'bar',
        barWidth: 30
    }]
};
mainChart.setOption(option);
```
其实，这里我们错误地设置了`boundaryGap`值，修改`boundaryGap`值为true即可。
```
xAxis: {
        type: 'category', // 类目轴
        data: [0, 1, 2, 3, 4, 5, 6],
        boundaryGap: true // x轴两边是否留白
    }
```
再看下渲染出的柱状图，OK，没有问题了。
<br>
<image src="https://user-gold-cdn.xitu.io/2019/6/1/16b11c920fb78567?w=667&h=389&f=png&s=10938">
<br>
## 结论
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.当echarts柱状图中`x`轴值为`category`（类目轴）时，`boundaryGap`值需要设为`true`，即可解决x轴左边溢出问题。
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.当echarts柱状图中`x`轴值为`value`（数值轴）时，通过计出`x`轴`data`的`min`和`max`值，然后设置`xAxis`的`min`和`max`属性，最后设置`yAxis`的`onZero`属性为`false`，即可解决x轴两边溢出问题。
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.当我们遇到一些奇奇怪怪的问题时，多思考，尝试换个思路，就能找到行得通的方案。
<br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最后，祝大家儿童节快乐，喜欢的话给文章点个赞吧，非常感谢，有疑问均可在文章下方留言。
<figure>
<image width="500" src="https://user-gold-cdn.xitu.io/2019/6/1/16b11d64f913fd5d?w=1024&h=614&f=png&s=753890">
</figure>