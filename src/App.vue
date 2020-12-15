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
        { id: getUUID(), name: "一场说走就走的旅行", isWished: false, line: 0 },
        { id: getUUID(), name: "结束单身汪", isWished: false, line: 0 },
        { id: getUUID(), name: "明年暴瘦10斤", isWished: false, line: 0 },
        { id: getUUID(), name: "多陪伴父母", isWished: false, line: 0 },
        { id: getUUID(), name: "赚到1个亿,买别墅", isWished: false, line: 0 }
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
      // 先确定弹道
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
