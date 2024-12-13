<script lang="ts" setup>
import {ref, watch, defineEmits, inject} from 'vue';
import type {TextAnimationOptions} from '@/types';
// region 接收来自父组件的数据
const props = defineProps<{
  message: string,
  ok: boolean
}>();
// 接收动画选项
const textAnimationOptions: TextAnimationOptions = inject('textAnimationOptions') as TextAnimationOptions
// endregion

// region 向父组件发数据
interface Emits {
  (event: 'close', ok: boolean): void
}

const emits = defineEmits<Emits>()
// endregion

// region 动画参数初始化以及动画开始与停止函数
// 偏移量
const y = ref(0);
// 动画持续时间
const duration = textAnimationOptions.duration * 1000;

// 记录动画的id
let textAnimationId: number | null = null;

// 开始动画
function textAnimation() {
  y.value -= textAnimationOptions.speed   // 移动距离
  textAnimationId = requestAnimationFrame(textAnimation)
}

// 停止动画
function stopAnimation() {
  if (textAnimationId !== null) {
    cancelAnimationFrame(textAnimationId);
    textAnimationId = null;
  }
}

// endregion

// region   监听ok的变化
watch(
    () => props.ok,
    (value) => {
      if (value) {
        // 开始动画
        if (textAnimationId === null) {
          textAnimation();
          setTimeout(() => {
            stopAnimation();
            setTimeout(() => {
              y.value = 0; // 恢复原样
              // 向父组件发送关闭事件
              emits('close', props.ok);
            }, 2000);
          }, duration);
        }
      } else {
        stopAnimation();
      }
    }
);
// endregion
</script>

<template>
  <div
      :class="{'message-box': true}"
      :style="{'transform': `translate(0, ${y}px)`}"
  >
    <p>{{ message }}</p>
  </div>
</template>

<style scoped lang="less">
@keyframes MsgBoxAnimation {
  0% {
    transform: translate(0, 0);
  }

  100% {
    transform: translate(0, -2000px);
  }
}

.start {
  animation: MsgBoxAnimation 10s 1s linear;
}

.message-box {
  //animation: MsgBoxAnimation 100s 0.5s linear;
  position: relative;
  //width: 10vw;
  max-width: 100px;
  padding: 5px;
  background-color: #fff;
  border: 1px solid transparent;
  border-radius: 5px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  //font-family: Arial, sans-serif;
  //color: #333;
  margin: 10px auto;
}

.message-box::after {
  content: '';
  position: absolute;
  bottom: 25%;
  left: -20px;
  border-width: 10px;
  border-style: solid;
  border-color: transparent #fff transparent transparent;
}

</style>