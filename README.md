# 基于 React 技术栈的 滴答清单 ⏰

## 所用技术栈

- react
- react-router
- redux

## 开发流程

### 1. 设计路由

- `/` 根目录 对应所有的清单条目
- `/detail/:id` 详细的清单条目 对应某条详细内容

### 2. 确定数据结构

```
{
  id: 编号,
  summarize: 一句话的概括,
  createTime: 创建时间,
  priorityLevel: 优先级（0,1,2,-1分别是高中低无）,
  endTime: 到期时间,
  status: 当前状态（未完成 0 / 已完成 1）,
  type: 任务类型（描述类 0 /子任务类 1）,
  percent: 任务完成百分比,
  details: {
    subTask: [
      {
        id: 编号,
        summarize: 描述,
        status: 状态,
      }
    ],
    description: 描述,
    comments: [
      {
        id: 编号,
        summarize: 内容,
        createTime: 创建时间,
      }
    ]
  },
}
```

## 小记

### 1. React 编写 日历组件📅

![]()

滴答清单中的日历组件主要有两个面板：

- 选择年月的面板（下面称为年月模板）
- 选择详细某一天的面板（下面称为详细模板）

整个组件的主要业务逻辑是这样的：

- 默认没有选中的日期，则按照当前的年月来渲染详细模板，其中高亮当天，如果存在选中的日期，则按照选中的年月来渲染详细模板，高亮当天的同时还需要选中对应的日期
- 详细模板共显示6行数据，当月数据高亮显示，详细模板可以切换月份
- 年月模板可以切换年份&月份，其中详细模板对应的月份被选中

由此可以确定，日历组件的根组件与外界的组件之间沟通最基础的需要：

- 已选中的年月日（可选的）
- 日历组件中选中某一天的回调函数（必须的）

由此，我们已经确定了日历组件最基本的 **props**

*当然你可以说还需要控制显隐、组件根节点的样式、月份修改的回调以及种种你所需要的 props 这里只是描述最基本的内容*

根据职责单一化的原则，最好是将根组件按照面板划分成两个子组件，避免组件过于复杂

在这里没有将日历组件划分成为两个子组件，而是将年月模板作为独立的组件，而详细模板直接编写在根组件中

至此，根组件除了详细模板交互之外，还需要实现控制两个面板的显隐

- <Calendar />
- <YearMonthCalendar />

详细模板

详细模板主要根据当前的年月按照周的形式来渲染日历

![详细模板的概念图](http://7xi77s.com1.z0.glb.clouddn.com/blog/images/WX20170713-112616.png)

每个详细模板都是 7 * 6 的网格，横轴按照周日到周六来排列，当前月不可能占满 42 个格子，其他部分需要上下月的内容来补足

除了渲染日历之外，还需要高亮当天 & 选中对应的日期

因为借助 React，我们所需要完成的部分就是使用已有的数据来生成驱动 React 渲染的数据结构

已有的数据：

- 当天的日期（环境变量，Date.now()）
- 选中的日期的（根组件的 props，是可选的）

React 所需要的数据结构是这样的：

```
[
  {
    year: 年
    month: 月
    day: 日
    type: 是不是当前月，还是补足的前一月/后一月
    singleTodayChosen: 是不是当天，需要高亮 
    chosen: 是不是选中的日期，需要选中
  } * 42
]
```

现在我们要做的就是要获得这个 42 个字段的数组

这个 42 个字段的数组，是由当前月的日期，前后补上上个月和下个月的日期

前面我们提到，日历详细模板是 7 * 6 的网格，我们肯定希望当前月在这个网格中居中显示，每个月的天数是 28 - 31 天之间，那么每个月占据的行数一般是 4 - 6 行

如果当前月正好占据了 4 行，那么就是上下各自空一行，当月从第二行开始渲染

如果当前月占据了 5 行，那么就需要判断当前月的第一天是不是超过了周三，来决定从第一行还是第二行来进行渲染

如果占据了 6 行，那么就是从开始占据到了结束，当月从第一行就开始渲染

那么，如何判断当前月占据了多少行呢？

![分析占据行的问题](http://7xi77s.com1.z0.glb.clouddn.com/blog/images/WX20170713-113028.png)

从上面的图可以很明显的看出来

`当月占据的行数 = 向上取整((当月第一天对应的周几位置 + 当月的天数) / 7)`

至此，我们有了 **当月占据的行数**，**当月开始渲染的那一行**

根据上面的信息，很容易就推算出前一个月、下一个月分别补足的数量

![流程图](http://7xi77s.com1.z0.glb.clouddn.com/blog/images/WX20170713-140113.png)

综上，我们根据当前的年、月，就可以渲染出这个 6 * 7 的数组

### 2. 公共 Reducer 的组织

### 3. 样式按需加载（代码按需加载/分离）

参考： http://serverless-stack.com/chapters/code-splitting-in-create-react-app.html

## Todos

1. CSSTransitionGroup 中存在的问题？子元素的变化卸载也会影响整体的动画
