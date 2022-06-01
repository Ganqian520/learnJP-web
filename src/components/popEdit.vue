<script setup>
import {ref,defineEmits} from 'vue'
let input = ref('')
const emit = defineEmits(['closePop','addCollection'])
</script>

<template>
<div class="container">
  <div class="mask" @click="()=>emit('closePop')"></div>
  <div class="et">
    <input class="input" type="text" v-model="input">
    <div class="button-div">
      <div class="button" @click="()=>emit('closePop')">取消</div>
      <div class="button" @click="()=>emit('addCollection',input)">确定</div>
    </div>
  </div>
</div>
</template>

<style scoped lang="scss">
.container{
  width: 100vw;
  height: 100vh;
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  .mask{
    width: 100vw;
    height: 100vh;
    background-color: rgba(0,0,0,0.7);
    position: absolute;
    z-index: -999;
  }
  .et{
    width: 70vw;
    height: 20vh;
    background-color: #fff;
    border-radius: 2vh;
    margin: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    .input{
      width: 60vw;
      flex: 1;
    }
    .button-div{
      width: 100%;
      flex: 1;
      display: flex;
      justify-content: space-around;
      align-items: center;
      .button{
        width: 5vh;
        height: 5vh;
        line-height: 5vh;
        text-align: center;
        border-radius: 1vh;
        background-color: rgb(52, 131, 151);
      }
    }
  }
}
</style>