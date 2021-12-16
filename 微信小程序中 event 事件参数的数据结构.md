## 微信小程序中 event 事件参数的数据结构

### 1. 概览

在前端中 event 变量是在出发某一事件，比如点击、改变等时由系统自动创建的一个变量，这里面会存着我们的事件的参数。

比如，在一次事件中，得到了如下的 event 参数。

前端代码：

```html
<vant-cell wx:for="{{future}}" wx:for-item="detail" title="{{detail.temperature}}" value="{{detail.weather}}" label="{{detail.date}}" data-title="{{detail.temperature}}" border="false" bind:click="getCard"></vant-cell>
```

在页面 js 中，click 事件函数 getCard 负责将 event 变量输出到 console。

获得的 event 如下：

```json
{type: "click", timeStamp: 622655, target: {…}, currentTarget: {…}, mark: {…}, …}
changedTouches: undefined
currentTarget:
dataset: {title: "-7/6℃"}
id: ""
__proto__: Object
detail: {x: 281.640625, y: 272.4375}
mark: {}
mut: false
target:
dataset: {title: "-7/6℃"}
id: ""
__proto__: Object
timeStamp: 622655
touches: undefined
type: "click"
```

