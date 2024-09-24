---
title: 从element的一次PR开始
date: 2023-12-25
updated: 2023-12-25
---

## 缘起

那天我正刷着B站，主页突然跳出来up程序员小山与Bug的视频，名字叫 el-table固定表头滚动时，表头不跟手抖动的问题，跟着学到了一点调试小技巧，和大家分享下。

1、Ctrl + Shift + p  可以召唤出Chrome的命令

可以在里面方便的禁用js脚本

2、github上的 #数字 可以直接在github上的PR那边直接进入

比如

https://github.com/ElemeFE/element/pull/21863

这里的21863 就是编号

同样在releases 的文档种能找到这行

- Fix Tabl-header shake bug ([#21863](https://github.com/ElemeFE/element/pull/21863) by [@bofeng](https://github.com/bofeng))

指的就是这个功能的PR号，非常的方便，看别人源码也不容易大海捞针

3、如何简单排查某行代码是否会影响其他功能，如果测试写的好的话直接跑测试，那如果测试不方便或者看不出来尼，可以看下增加那段代码时候的提示，是否是为了解决bug或者新增的功能或者单纯的是提高性能。



然后来看下这个同步是怎么做的

```typescript
syncPostion() {
        const { scrollLeft, scrollTop, offsetWidth, scrollWidth } = this.bodyWrapper;
        const { headerWrapper, footerWrapper, fixedBodyWrapper, rightFixedBodyWrapper } = this.$refs;
        if (headerWrapper) headerWrapper.scrollLeft = scrollLeft;
        if (footerWrapper) footerWrapper.scrollLeft = scrollLeft;
        if (fixedBodyWrapper) fixedBodyWrapper.scrollTop = scrollTop;
        if (rightFixedBodyWrapper) rightFixedBodyWrapper.scrollTop = scrollTop;
        const maxScrollLeftPosition = scrollWidth - offsetWidth - 1;
        if (scrollLeft >= maxScrollLeftPosition) {
          this.scrollPosition = 'right';
        } else if (scrollLeft === 0) {
          this.scrollPosition = 'left';
        } else {
          this.scrollPosition = 'middle';
        }
      },
```

其实很简单，获取当前的scrollLeft 赋值给另一个 scrollLeft 就完成了，让我们来写个简单的同步

```html
// index.html
<div class="headerWrapper wrapper">
  <table>
    <colgroup>
      <col width="180">
      <col width="180">
      <col width="200">
      <col width="200">
    </colgroup>
    <thead>
      <tr>
        <th>凉</th>
        <th>风</th>
        <th>有</th>
        <th>信</th>
      </tr>
    </thead>
  </table>

</div>
<div class="bodyWrapper wrapper">
  <table>
    <colgroup>
      <col width="180">
      <col width="180">
      <col width="200">
      <col width="200">
    </colgroup>
    <tbody>
      <tr>
        <td>凉风有信</td>
        <td>秋月无边</td>
        <td>亏我思娇情绪好比度日如年</td>
        <td>恨天各一方难与秋娟再相见</td>
      </tr>
    </tbody>
  </table>

</div>

```

css

```css
.wrapper{
  width:300px;
  overflow: auto;
}

.wrapper > table{
  width:800px;
  
}
.headerWrapper{
  overflow: hidden;
}
```

下面就是见证奇迹的时刻

```javascript
function ready() {

  const headerWrapper = document.getElementById("headerWrapper");
  const bodyWrapper = document.getElementById("bodyWrapper");

  bodyWrapper.addEventListener('scroll', function() {
    window.requestAnimationFrame(() => {
      let top = this.scrollTop
      let left = this.scrollLeft
      headerWrapper.scrollTo(left, top);
    })
  })
}

document.addEventListener("DOMContentLoaded", ready);

```

在线练习链接 https://jsfiddle.net/knightgao/ksbyLfjg/59/

这样就实现了简单的js联动

那更进一步，按照力扣的命名规则，刚刚那题叫联动的话，那下面这题叫联动2

要求是n个互相影响尼，比如给了一排的div，要求拖动其中一个带动所有的进行滚动

1对1 到了  1对 N 

原理没变，多了一些要求

- 避免触发遍历的时候改变自己
- 避免scroll 事件循环触发

关于循环触发这里补充下

循环触发例子链接 https://jsfiddle.net/knightgao/5L9mkz1t/19/

当test1滚动的时候会设置同步test2的值，如果这时候 test2 scroll 事件 中设置 test1的值，就会导致循环触发，造成不必要的性能浪费

```html
<div class="headerWrapper wrapper">
  <table>
    <colgroup>
      <col width="400">
      <col width="400">
      <col width="400">
      <col width="400">
    </colgroup>
    <thead>
      <tr>
        <th>冲动</th>
        <th>的</th>
        <th>惩</th>
        <th>罚</th>
      </tr>
    </thead>
  </table>

</div>
<div class="bodyWrapper wrapper">
  <table>
    <colgroup>
      <col width="400">
      <col width="400">
      <col width="400">
      <col width="400">
    </colgroup>
    <tbody>
      <tr>
        <td>那夜我喝醉了拉着你的手</td>
        <td>胡乱地说话</td>
        <td>只顾着自己心中压抑的想法</td>
        <td>狂乱地表达</td>
      </tr>
    </tbody>
  </table>
</div>

<div class="bodyWrapper wrapper">
  <table>
    <colgroup>
      <col width="400">
      <col width="400">
      <col width="400">
      <col width="400">
    </colgroup>
    <tbody>
      <tr>
        <td>我迷醉的眼睛已看不清你表情</td>
        <td>忘记了你当时会有怎样的反应</td>
        <td>我拉着你的手放在我手心</td>
        <td>我错误地感觉到你也没有生气</td>
      </tr>
    </tbody>
  </table>
</div>

```

```css
.wrapper{
  width:600px;
  overflow: auto;
}

.wrapper > table{
  width:2000px;
  
}
.headerWrapper{
  overflow: hidden;
}
```



```javascript
function ready() {

  const nodes = document.getElementsByClassName("wrapper");

  function initScroller(nodes) {
    let max = nodes.length
    if (!max || max === 1) return
    let limit = 0; 
    nodes.forEach((ele, index) => {
      ele.addEventListener('scroll', function() {
        window.requestAnimationFrame(() => {
          if (!limit) { 
            limit = max - 1;
            let top = this.scrollTop
            let left = this.scrollLeft
            for (node of nodes) {
              if (node === this) continue;
              node.scrollTo(left, top);
            }
          } else
            --limit;
        })
      }, {
        passive: true
      });
    });
  }
  initScroller([...nodes]);
}

document.addEventListener("DOMContentLoaded", ready);
```