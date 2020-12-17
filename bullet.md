---
title: 实现简单的弹幕
tags: js
categories: js
---


最近做年终活动，许愿的时候希望可以以弹幕的形式出现。

## 显示一条弹幕

先写段静态的文字。

超简单：
![bullet1](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/bullet1.png)

但是一行通常不止一个弹幕，肯定不能`block`，所以这里的`bullet-item`需要处理下，这里我用定位的方式（主要为了之后的弹道，定位更方便）。

```vue
<!-- App.vue -->
<template lang="pug">
div#app
  div.bullet-wrap
    div.bullet-item 减肥成功！
</template>

<style>
body {
  margin: 0;
}
.bullet-wrap {
  height: 375px;
  background-color: #eee;
  position: relative;
}
.bullet-item {
  position: absolute;
}
</style>

```

### 让文字在左边屏幕外

让文字在**左边屏幕外**，表面看起来消失的样子

```css
.bullet-item {
  transform: translateX(-100%);
}
```

### 让文字在右边屏幕外

让文字在**右边屏幕外**，表面看起来消失的样子

```css
.bullet-item {
  transform: translateX(100vw);
}
```

### 让文字从右边屏幕外到左边屏幕外

让文字从右边屏幕外到左边屏幕外，表面看起来从左边诞生，从右边离去

```css
.bullet-item {
  position: absolute;
  animation: rightToLeft 7s linear both;
}
@keyframes rightToLeft {
  0% {
    transform: translate(100vw);
  }
  100% {
    transform: translate(-100%);
  }
}
```

![bullet2](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/bullet2.gif)

## 多行显示

上面一条弹幕的感觉已经有了。  
接下来，看看多行弹幕。  

弹幕的的行，说好听点是弹道，一般将弹幕区域分成好几个弹道，就像跑道一样，每个运动员有自己的跑道。

### 两个弹道

两个弹道的逻辑，其实是在于，弹道靠`top`控制位置，不同的弹道`top`不一样

```pug
  div.bullet-wrap
    div.bullet-item(data-line="1") 减肥成功！
    div.bullet-item(data-line="2") 没有bug!
```

```css
.bullet-item[data-line="1"] {
  top: 0;
}
.bullet-item[data-line="2"] {
  top: 75px;
}
```

### 多个弹道，错峰运行

多个弹道，其实就是将上面改下，这么先设置5个弹道。

但因为动画是同时开启的，所以看起来，怪怪的。

错峰的逻辑如下，以下也是本文的**核心**：

首先分成两个集合

- 等待集合：将要显示的弹幕集合
- 显示集合：显示在屏幕上的弹幕集合，这也是页面循环的集合，出现在这个集合里，弹幕就开始滚动

显示弹幕的逻辑

- 先确定弹道，接下来的弹幕就出现在这个弹道里
- 从等待集合里取出第一个
- 设置此弹幕的弹道
- 此弹幕放进显示集合里，此弹幕开始滚动

隔一段时间执行，上续操作，弹幕就源源不断。

![bullet3](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/bullet3.gif)

```vue
<template lang="pug">
div#app
  div.bullet-wrap
    div.bullet-item(v-for="item in showingBullets" :key="item.id" :data-line="item.line") {{item.name}}
</template>
<script>
const getUUID = () => Math.random() + Math.random();
export default {
  data() {
    return {
      // 将要显示的弹幕队列
      waitBullets: [
        { id: getUUID(), name: "一场说走就走的旅行", line: 0 },
        { id: getUUID(), name: "结束单身汪", line: 0 },
        { id: getUUID(), name: "明年暴瘦10斤", line: 0 },
        { id: getUUID(), name: "多陪伴父母", line: 0 },
        { id: getUUID(), name: "赚到1个亿,买别墅", line: 0 }
      ],
      showingBullets: [],
      lines: 5,
      currentLine: 1
    };
  },
  mounted() {
    this.showNextBullet();
    setInterval(this.showNextBullet, 700);
  },
  methods: {
    showNextBullet() {
      if (!this.waitBullets.length) {
        return;
      }
      // 先确定弹道，跟上一个弹道错开即可
      this.currentLine = (this.currentLine % this.lines) + 1;
      // 从等待集合里取出第一个
      const currentBullet = this.waitBullets.shift();
      // 设置弹幕的弹道
      currentBullet.line = this.currentLine;
      // 弹幕放进显示集合里，弹幕开始滚动
      this.showingBullets.push(currentBullet);
    }
  }
};
</script>

<style>
html,
body {
  min-height: 100%;
  overflow: hidden;
}
body {
  margin: 0;
}
.bullet-wrap {
  height: 375px;
  background-color: #eee;
  position: relative;
}
.bullet-item {
  position: absolute;
  animation: rightToleft 7s linear both;
}
.bullet-item[data-line="1"] {
  top: 0;
}
.bullet-item[data-line="2"] {
  top: 75px;
}
.bullet-item[data-line="3"] {
  top: 150px;
}
.bullet-item[data-line="4"] {
  top: 225px;
}
.bullet-item[data-line="5"] {
  top: 300px;
}
@keyframes rightToleft {
  0% {
    transform: translate(100vw);
  }
  100% {
    transform: translate(-100%);
  }
}
</style>

```

## 发送新弹幕

发送新弹幕，其实没啥，送到等待队列里即可。

![bullet4](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/bullet4.gif)

```js
clickSend() {
    if (!this.newBullet) { return; } 
    const newBullet = { id: getUUID(), name: this.newBullet, line: 0 };
    this.waitBullets.push(newBullet);
}
```

```pug
  div.input-wrap
    input(v-model.trim="newBullet" type='text' maxlength='12' placeholder='来说点什么')
    button.btn(@click="clickSend") 发送
```

## 优化

- 弹幕从屏幕上消失之后，需要从显示队列里移除
- 组件销毁前，清除定时器
- 想无限循环弹幕列表，需要在等待队列推出第一个的时候，顺手将这个再塞到等待队列里


```vue
<template lang="pug">
div#app
  div.bullet-wrap
    div.bullet-item(v-for="item in showingBullets" @animationend='removeBullet' :key="item.id" :data-line="item.line") {{item.name}}
  div.input-wrap
    input(v-model.trim="newBullet" type='text' maxlength='12' placeholder='来说点什么')
    button.btn(@click="clickSend") 发送
</template>
<script>
const getUUID = () => Math.random() + Math.random();
export default {
  data() {
    return {
      // 将要显示的弹幕队列
      waitBullets: [
        { id: getUUID(), name: "一场说走就走的旅行", isWished: false, line: 0 },
        { id: getUUID(), name: "结束单身汪", isWished: false, line: 0 },
        { id: getUUID(), name: "明年暴瘦10斤", isWished: false, line: 0 },
        { id: getUUID(), name: "多陪伴父母", isWished: false, line: 0 },
        { id: getUUID(), name: "赚到1个亿,买别墅", isWished: false, line: 0 }
      ],
      showingBullets: [],
      lines: 5,
      currentLine: 1,
      newBullet: "",
      isInfinite: true
    };
  },
  mounted() {
    this.showNextBullet();
    const timer = setInterval(this.showNextBullet, 700);
    // 组件销毁前，清除定时器
    this.$once("hook:beforeDestroy", () => {
      clearInterval(timer);
    });
  },
  methods: {
    showNextBullet() {
      if (!this.waitBullets.length) {
        return;
      }
      // 先确定弹道，跟上一个弹道错开即可
      this.currentLine = (this.currentLine % this.lines) + 1;
      // 从等待集合里取出第一个
      const currentBullet = this.waitBullets.shift();
      // 想要无限循环的话
      this.isInfinite &&
        this.waitBullets.push({
          id: getUUID(),
          name: currentBullet.name,
          isWished: false,
          line: 0
        });
      // 设置弹幕的弹道
      currentBullet.line = this.currentLine;
      // 弹幕放进显示集合里，弹幕开始滚动
      this.showingBullets.push(currentBullet);
    },
    clickSend() {
      if (!this.newBullet) {
        return;
      }
      const newBullet = {
        id: getUUID(),
        name: this.newBullet,
        isWished: false,
        line: 0
      };
      this.waitBullets.push(newBullet);
    },
    removeBullet() {
      this.showingBullets.shift();
      console.log(this.showingBullets);
    }
  }
};
</script>

<style>
html,
body {
  min-height: 100%;
  overflow: hidden;
}
body {
  margin: 0;
}
.bullet-wrap {
  height: 375px;
  background-color: #eee;
  position: relative;
}
.bullet-item {
  position: absolute;
  animation: rightToleft 7s linear both;
}
.bullet-item[data-line="1"] {
  top: 0;
}
.bullet-item[data-line="2"] {
  top: 75px;
}
.bullet-item[data-line="3"] {
  top: 150px;
}
.bullet-item[data-line="4"] {
  top: 225px;
}
.bullet-item[data-line="5"] {
  top: 300px;
}
@keyframes rightToleft {
  0% {
    transform: translate(100vw);
  }
  100% {
    transform: translate(-100%);
  }
}
</style>

```

## 引用

- [Vue 实现弹幕效果](https://segmentfault.com/a/1190000022549145)