<script setup>
import {ref,defineEmits} from 'vue'
const props = defineProps({
  from:String,
})
const emit = defineEmits(['edit','confirm'])
let isShow = ref(false)

</script>

<template>
<div class="anchor" @mouseenter="()=>isShow=true" @mouseleave="()=>isShow=false" :style="{opacity:isShow ? 1 :0}">
  <div class="button" @click="emit('edit')">编辑</div>
  <div class="button" @click="emit('confirm')">确定</div>
</div>
</template>

<style scoped lang="scss">
.anchor{
  width: 10vh;
  height: 5vh;
  text-align: center;
  line-height: 5vh;
  background-color: rgb(26, 1, 24);
  position: absolute;
  bottom: 0;
  right: 0;
  display: flex;
  .button{
    flex: 1;
    line-height: 5vh;
    text-align: center;
  }
}
</style>
