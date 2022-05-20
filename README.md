# 框选checkbox的实现

## 题目要求

用 Web 技术实现以下效果【框选checkbox】

![](https://bbfe-camp.github.io/contest-2022/box-selection-checkbox.gif)

## 实现方法

**大概思路：**

1. 首先用 js 创建一定数量的 checkbox
2. 用 **canvas** 实现**按住左键拖动**出现**框选区域**，**松开则消失**
3. **监听**是否有 checkbox 出现在框选区域，一旦**被覆盖：**
   - **清除**所有勾选框**勾选状态**
   - 被**覆盖**的checkbox**被勾选**

### 首先创建 checkbox 勾选框

简单设置一个500 * 500 大盒子，大盒子中装入一个容器(用来绝对定位，覆盖在canvas上方)，容器中放入一个小盒子。然后通过代码往小盒子中放入对应数量的 checkbox，之后再将小盒子复制几份，将容器通过 flex 样式纵向排布并沿主轴和垂直轴居中，js代码如下：

```js
// 创建checkBox
function createCheckBox () {
  let checkbox = document.createElement('input')
  checkbox.setAttribute('type', 'checkbox')
  checkbox.setAttribute('style', 'zoom:150%;')

  let cbox = document.querySelector('.cbox')
  for (let i = 0; i < 6; i++) {
    let element = checkbox.cloneNode(true)
    cbox.append(element)
  }

  let container = document.querySelector('.container')
  for (let i = 0; i < 4; i++) {
    let element = cbox.cloneNode(true)
    container.appendChild(element)
  }
}
```

### Canavas 实现拖拽选中框效果

首先设置个与大盒子相等大小的 canvas 画布，将canvas放入大盒子中，然后设置大盒子的定位为 relative，之后再设置画布 canvas 定位为 position 使其浮动在大盒子之上，但浮动在 container 之下

```html
<style>
.box {
  width: 500px;
  height: 500px;
  border: 1px solid black;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}
#myCanvas {
  position: absolute;
}
</style>
<div class="box">
  <canvas id="myCanvas" width="500px" height="500px"></canvas>
  <div class="container" style="position: absolute;">
    <div class="cbox"></div>
  </div>
</div>
```

添加鼠标监听事件，监听鼠标按下、按下移动和松开事件。按下移动的时候首先清除画布，再绘制方形，松开之后再次清除画布，并设置拖拽结束标志状态

```js
// 监听鼠标按下拖拽事件
function onPressDrag () {
  myCanvas.onmousedown = function (e) {
    e = e || window.event
    let startX = e.clientX - box.offsetLeft
    let startY = e.clientY - box.offsetTop
    console.log(startY)
    document.onmousemove = function (e) {
      e = e || window.event
      let moveX = e.clientX - box.offsetLeft
      let moveY = e.clientY - box.offsetTop
      // 清除原有的图形
      ctx.clearRect(0, 0, myCanvas.width, myCanvas.height)
      // 重绘矩形选中区域
      ctx.strokeRect(startX, startY, moveX-startX, moveY-startY)
      // 检查选中区域是否存在 checkbox
      checkChose(startX, startY, moveX, moveY)
    }
    document.onmouseup = function (e) {
      e = e || window.event
      let endX = e.clientX - box.offsetLeft
      let endY = e.clientY - box.offsetTop
      document.onmousemove = null
      document.onmouseup = null
      ctx.clearRect(0, 0, myCanvas.width, myCanvas.height)
      // 结束拖拽标志
      dragFlag = true
    }
  }
}
```

### 监听判断 checkbox 是否在区域内

每次重绘矩形区域都要监听判断 chebox 是否在区域内，判断方法为：

1. 首先计算出拖拽区域上下左右边界
2. 之后判断每个 checkbox 的四个边界点中是否有存在在区域内的点，存在则勾选状态（一旦判断到有 checkbox 在区域内，先根据拖拽结束标志判断是否需要清除所有 checkbox 的勾选状态）

代码如下：

```js
// 监听所有 checkbox，检查它们是否在框选区域内
function checkChose (startX, startY, moveX, moveY) {
  // 获取边界
  let left = startX > moveX ? moveX : startX
  let right = startX > moveX ? startX : moveX
  let top = startY > moveY ? moveY : startY
  let bottom = startY > moveY ? startY : moveY

  let cboxs = document.querySelectorAll(".cbox")
  let container = document.querySelector(".container")
  
  // 判断选中区域
  // 由于 鼠标定位的位置与实际 checkbox 坐标有误差(没找到误差原因) 所以下方值部分是靠固定值计算
  let initvalue = 8
  for (let i = 0; i < cboxs.length; i++) {
    const childNodes = cboxs[i].childNodes;
    for (let j = 0; j < childNodes.length; j++) {
      const element = childNodes[j];
      // checkbox 四个点
      let positionX1 = initvalue + container.offsetLeft + 30 * j
      let positionY1 = initvalue + container.offsetTop + 30 * i
      let positionX2 = initvalue + container.offsetLeft + 30 * j + 13
      let positionY2 = initvalue + container.offsetTop + 30 * i + 13
      let points = [{x:positionX1, y:positionY1}, {x:positionX2, y:positionY2}, {x:positionX1, y:positionY2}, {x:positionX2, y:positionY1}]
      if (jugePoint(points, left, right, top, bottom)) {
        // 清除所有 checkbox 状态
        if (dragFlag) {
          clearCheckboxStatus()
          dragFlag = false
        }
        element.checked = true
      }
    }
  }
}
```

其中涉及的判断四点及清除状态的方法如下：

```js
// 清除所有 checkbox 状态 
function clearCheckboxStatus () {
  let checkboxs = document.querySelectorAll(".cbox>input[type]")
  for (let i = 0; i < checkboxs.length; i++) {
    checkboxs[i].checked = false
  }
}

// 判断框选按钮四个点是否有某个点在区域内
function jugePoint (points, left, right, top, bottom) {
  for (let i = 0; i < points.length; i++) {
    let position = points[i];
    if ( position.x >= left && position.x <= right && position.y >= top && position.y <= bottom) {
      return true
    } 
  }
  return false
}
```

## 展示

[![OLvXcT.gif](https://s1.ax1x.com/2022/05/20/OLvXcT.gif)](https://imgtu.com/i/OLvXcT)

