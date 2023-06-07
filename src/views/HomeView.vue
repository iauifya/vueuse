<template>
  <!-- useLastChanged 紀錄最新一次更改資料的時間 -->
  <div>
    <input v-model="input" type="text" placeholder="Type anything...">
    <div>Last changed: <span class="text-primary">{{ timeago }}</span></div>
  </div>
  <p>我是分隔線</p>
  <hr>

  <!-- useRefHistory 跟蹤響應式資料的變化 -->
  <p>
    <button @click="undo"> Undo </button>
    <button @click="redo"> Redo </button>
  <ul>
    <li v-for="entry in history" :key="entry.timestamp">
      {{ entry }}
    </li>
  </ul>
  </p>
  <textarea v-model="text" />
  <p>我是分隔線</p>
  <hr>

  <!-- onClickOutside 關閉 modal -->
  <button @click="open = true">Open Popup</button>
  <div class="popup" v-if="open">
    <div class="popup-content" ref="popup">
      <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Itaque iste dolorem illo voluptates quaerat voluptas
        aliquid neque eum odit. Eveniet sequi voluptates quos, assumenda porro saepe! Explicabo delectus exercitationem
        deleniti!</p>
    </div>
  </div>
  <!-- ref="popup 用來訪問popup所有屬性與方法" -->
  <p>我是分隔線</p>
  <hr>

  <!-- 使用intersectionobserver 跟蹤元素的可見性-->
  <div ref="root" class="root">
    <p class="notice">
      Scroll me down!
    </p>
    <div ref="target" class="target">
      <p>Hello world!</p>
    </div>
  </div>
  <div class="text-center">
    Element
    <div :value="targetIsVisible" true="inside" false="outside" class="font-bold">
      {{ targetIsVisible }}
    </div>
    the viewport
  </div>
  <p>我是分隔線</p>
  <hr>

  <!-- 使用useTransition做個數位載入動畫 -->
  <h2>
    <p>Join over</p>
    <p>{{ Math.round(output) }}+ </p>
    <p>Develops</p>
  </h2>
  <!-- 應用到顏色變色動畫上 -->
  <h2 :style="{ color: color }">COLOR CHANGING</h2>
</template>

<script setup>
import { timestamp, useLastChanged, useTimeAgo, useRefHistory, onClickOutside, useIntersectionObserver,useTransition,TransitionPresets } from '@vueuse/core'
import { ref,computed } from 'vue'
const input = ref('')
const ms = useLastChanged(input, { initialValue: timestamp() - 1000 * 60 * 5 })
const timeago = useTimeAgo(ms)

const text = ref('')
const { history, undo, redo } = useRefHistory(text, {
  deep: true,
  capacity: 10,
})

const open = ref(false)
const popup = ref()
onClickOutside(popup, () => {   //onClickOutside(target,handler,options={})這個函式表示當點擊到popup以外的範圍時，會關掉popup
  open.value = false
})

const root = ref(null)
const target = ref(null)
const targetIsVisible = ref(false)
useIntersectionObserver(target, ([{ isIntersecting }]) => {
  targetIsVisible.value = isIntersecting
},
  { root }
)
// const { stop } = useIntersectionObserver(target,([{isIntersecting}])=>{
//   targetIsVisible.value = isIntersecting
//   if(isIntersecting){
//     stop()  //呼叫這個函數來停止觀察交叉點，一旦targetIsVisible被設定為true，observer就會停止，即使我們捲動離開目標元素，我們的值也會保持為true
//   }
// })
const count = ref(0)
const output = useTransition(count,{
  duration: 3000,
  transition: TransitionPresets.easeOutExpo  
})
count.value = 5000

const source = ref([0,50,0])
const outputTwo = useTransition(source,{ //把欲改變的目標套進該函式中
  duration: 3000,
  transition: TransitionPresets.easeOutExpo  
})
const color = computed(()=>{
  const [r,g,b] = outputTwo.value
  //return `rgb($(r),$(g),$(b))`
  return `rgb(${r},${g},${b})`
})
source.value = [255,255,255]
</script>

<style scoped>
button {
  border: none;
  outline: none;
  margin-right: 10px;
  background-color: #2ecc71;
  color: white;
  padding: 5px 10px;
  ;
}

.popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.6);
}

.popup-content {
  min-width: 300px;
  padding: 20px;
  width: 30%;
  background: #fff;
}

/* .title{
    opacity: 0;
  } */
.root {
  border: 2px dashed #ccc;
  height: 200px;
  margin: 0 2rem 1rem;
  overflow-y: scroll;
}

.notice {
  text-align: center;
  padding: 2em 0;
  margin-bottom: 300px;
  font-style: italic;
  font-size: 1.2rem;
  opacity: 0.8;
}

.target {
  border: 2px dashed var(--vp-c-brand);
  padding: 10px;
  max-height: 150px;
  margin: 0 2rem 400px;
}
</style>