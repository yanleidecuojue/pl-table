<template>
  <div>
    <button @click="generate">生成数据</button>
    <!-- <ul>
      <li v-for="(val, index) of list" :key="index">
        <span style="color: red">{{ val }}</span>
      </li>
    </ul> -->
    <div class="container">
      <ul :style="listStyle">
        <li v-for="(val, index) of renderList" :key="index" :style="itemStyle">
          {{ val }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, ref, computed, unref } from 'vue';
  // 可视区元素数量
  const viewLength = 6
  // 缓冲区元素数量
  const bufferLength = 4
  // 元素高度
  const itemHeight = 100


  export default defineComponent({
    name: 'CustomerModal',
    components: {},
    emits: ['success', 'register'],
    setup() {

      let renderList: any = ref(['1'])
      let list: any = ref()
      let listStyle: any = {
        height: '100px',
        padding: '0px'
      }
      let itemStyle: any = {
        transform: `translateY(0px)`
      }
      function generate () {
        const arr = []
        for (let i = 0; i < 10000; ++i) {
          arr.push(i.toLocaleString())
        }
        list.value = arr
        // 扩充列表高度
        listStyle.height = `${list.length * itemHeight}px`
        // 排布可见区和缓冲区数据
        renderList.value = list.value.slice(0, viewLength + bufferLength)
        console.log(renderList)
      }
      // function handleScroll (e: any) {
      //   console.log(111)
      //   console.log(e.target.scrollTop)
      //   // 当前滚动高度
      //   const scrollTop = e.target.scrollTop

      //   // 根据当前滚动高度计算元素偏移量
      //   const itemOffset = Math.floor(scrollTop / itemHeight)

      //   // 根据元素偏移量计算元素的高度偏移量
      //   // 例如：滚动高度为240，元素高度为100，则元素高度偏移量为 Math.floor(240/100)*100
      //   const offsetY = itemOffset * itemHeight
      //   itemStyle.transform = `translateY(${offsetY}px)`

      //   // 重新排布可见区和缓冲区数据
      //   renderList.value = list.value.slice(itemOffset, itemOffset + viewLength + bufferLength)
      // }
      return { list, renderList, listStyle, itemStyle, generate };
    }
  })
</script>

<style scoped lang="less">
.container {
  height: 600px;
  overflow-y: scroll;
  ul {
    li {
      height: 100px;
      text-align: center;
      color: red;
      border-bottom: solid #eee 1px;
    }
  }
}
</style>