<template lang="pug">
div#app
  div.bullet-wrap
    div.bullet-item(v-for="item in showingBullets" :style="{ transform:item.translate,animation:item.animate,color:'red' }" @animationend='removeBullet(item)' :key="item.id" :data-line="item.line") {{item.name}}
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
      showingBullets: [
        {
          id: getUUID(),
          name: "一场说走就走的旅行",
          isWished: false,
          line: 1,
          translate: "translate(0vw)",
          animate: "rightToleft 1s linear forwards"
        },
        {
          id: getUUID(),
          name: "一场说走就走的旅行",
          isWished: false,
          line: 1,
          translate: "translate(70vw)",
          animate: "rightToleft 5s linear forwards"
        },
        {
          id: getUUID(),
          name: "结束单身汪，一百年",
          isWished: false,
          line: 2,
          translate: "translate(-10vw)",
          animate: "rightToleft 1s linear forwards"
        },
        {
          id: getUUID(),
          name: "人生若只如初见，害",
          isWished: false,
          line: 2,
          translate: "translate(50vw)",
          animate: "rightToleft 4s linear forwards"
        },
        {
          id: getUUID(),
          name: "明年暴瘦10斤",
          isWished: false,
          line: 3,
          translate: "translate(20vw)",
          animate: "rightToleft 3s linear forwards"
        },
        {
          id: getUUID(),
          name: "多陪伴父母",
          isWished: false,
          line: 4,
          translate: "translate(40vw)",
          animate: "rightToleft 5s linear forwards"
        },
        {
          id: getUUID(),
          name: "多陪伴父母",
          isWished: false,
          line: 5,
          translate: "translate(-20vw)",
          animate: "rightToleft 0.5s linear forwards"
        },
        {
          id: getUUID(),
          name: "赚到1个亿,买别墅",
          isWished: false,
          line: 5,
          translate: "translate(60vw)",
          animate: "rightToleft 3s linear forwards"
        }
      ],
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
      this.currentLine = (this.currentLine % this.lines) + 2;
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
    removeBullet(bullet) {
      const index = this.showingBullets.findIndex(
        item => item.id === bullet.id
      );
      this.showingBullets.splice(index, 1);
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
  transform: translate(100vw);
  animation: rightToleft 7s linear forwards;
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
    /* transform: translate(100vw); */
  }
  100% {
    transform: translate(-100%);
  }
}
</style>
