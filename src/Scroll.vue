<template>
    <div v-for="row,index in headerData" :key="item">
      <div class="headerLine" style=""  v-for="column,key,i in row"  >
        <div class="header" :style="'position: fixed; top: 60px; left:' + i * 300 +'px;'">{{ column }}</div>
      </div>
    </div>
  <div class="wrapper" ref="wrapper" @scroll="onScroll">
    <div class="background" :style="{height:`${total_height}px`}"></div>


    <div  class="table" ref="container">

      <div v-for="row,index in runList" :key="item">
        <div  class="line" v-for="column,key,i in row.data"  >
          <div @dblclick="itemClick" :style="'top:' + index * (row.rospan * 50) +'px; z-index:' + (runList.length - index) +';left:' + i * 300 +'px;'" :class="[getClass(index), 'item']">{{ column === '' ? 'NULL':column }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import areaCodeData from './area_code.json'

export default {
  props: {
    cache_screens: { // 缓冲的屏幕数量
      type: Number,
      default: 1
    }
  },
  data () {
    return {
      city_data: null,
      headerData: [{id: '表id', code: '区划代码',name: '名称',level: '级别1-5,省市县镇村',pcode: '父级区划代码',create_time: '创建时间',created_by: '创建人',update_time: '更新时间',updated_time: '更新人'}],
      list: [], // 源数据
      runList: [], // 运行时的列表
      total_height: 0, // 列表总高度
      maxNum: 0,// 一屏幕容纳的最大数量
      distance: 0 // 存储滚动的距离
    }
  }, 
  created() {
    this.getApiData()

  },
  mounted () {
      this.genData()
      this.init();
      this.getRunData();
  },
  methods: {
    getApiData() {
    //   axios.get('http://www.licona.club:8000/gperp/basic-info/area-code/list/?page=1&pageSize=10000')
    //   .then(resp=>{
    //     console.log(1111)
    //     this.city_data = resp.data.result.results
    //     console.log(this.city_data.length)
    //     this.genData()
    //         this.init();
    // this.getRunData();
 
    //   })
    //   .catch(error=>{
    //       console.log(error);
    //       alert('网络错误，不能访问');
    //   })
    console.log(areaCodeData)
      this.city_data = areaCodeData
      console.log(this.city_data)

    },
    getClass (column) {
      switch (column) {
        case 0: return "top_line";
        default:
          return "";
      }
    },
    getPosition(row) {
      return 'top:' + row.top  + 'px;bottom:100px; left: 200px;'
    },
    init () {
      console.log(this.$refs)
      const containerHeight = parseInt(getComputedStyle(this.$refs.wrapper).height);
      //一屏的最大数量
      this.maxNum = Math.ceil(containerHeight / this.min_height);
      console.log(`maxNum:${this.maxNum}`);
    },
    onScroll (e) {
      if (this.ticking) {
        return;
      }
      this.ticking = true;
      requestAnimationFrame(() => {
        this.ticking = false;
      })
      const distance = e.target.scrollTop;
      this.distance = distance;
      this.getRunData(distance);
    },
    //二分法计算起始索引
    getStartIndex (scrollTop) {
      let start = 0, end = this.list.length - 1;
      while (start < end) {
        const mid = Math.floor((start + end) / 2);
        console.log(this.list[mid])
        const { top, height } = this.list[mid];
        console.log(top)
        console.log(height)
        if (scrollTop >= top && scrollTop < top + height) {
          start = mid;
          break;
        } else if (scrollTop >= top + height) {
          start = mid + 1;
        } else if (scrollTop < top) {
          end = mid - 1;
        }
      }
      return start;
    },
    getRunData (distance = null) {
      console.log(this.$refs.container)
      //滚动的总距离
      const scrollTop = distance ? distance : this.$refs.container.scrollTop;

      //在哪个范围内不执行滚动
      if (this.scroll_scale) {
        if (scrollTop > this.scroll_scale[0] && scrollTop < this.scroll_scale[1]) {
          return;
        }
      }

      //起始索引
      console.log(scrollTop)
      let start_index = this.getStartIndex(scrollTop);
      console.log(start_index)
      start_index = start_index < 0 ? 0 : start_index;

      //上屏索引
      let upper_start_index = start_index - this.maxNum * this.cache_screens;
      upper_start_index = upper_start_index < 0 ? 0 : upper_start_index;

      // 调整offset
      this.$refs.container.style.transform = `translate3d(0,${this.list[upper_start_index].top}px,0)`;

      //中间屏幕的元素
      const mid_list = this.list.slice(start_index, start_index + this.maxNum);

      // 上屏
      const upper_list = this.list.slice(upper_start_index, start_index);

      // 下屏元素
      let down_start_index = start_index + this.maxNum;

      down_start_index = down_start_index > this.list.length - 1 ? this.list.length : down_start_index;

      this.scroll_scale = [this.list[Math.floor(upper_start_index + this.maxNum / 2)].top, this.list[Math.ceil(start_index + this.maxNum / 2)].top];

      const down_list = this.list.slice(down_start_index, down_start_index + this.maxNum * this.cache_screens);

      this.runList = [...upper_list, ...mid_list, ...down_list];

    },
    //生成数据
    genData () {
      function getHeight () {
        return 50;
      }
      let total_height = 0;
      console.log(this.city_data)
      const list = this.city_data.map((data, index) => {
        const height = getHeight(data.type);
        const ob = {
          index,
          height,
          top: total_height,
          data,
          rospan: 1
        }
        // if(index === 0) {
        //   console.log(data)
        //   ob["rospan"] = 2
        // }

        total_height += height;
        return ob;
      })
      this.total_height = total_height; //  列表总高度
      this.list = list;
      this.min_height = 50; // 最小高度是50
    },
    itemClick(event) {
      this.replace(event.target)
    },
    replace(divElement) {
      var input = document.createElement("input");
      input.className = "edit-input"; 
      input.id = "edit-input"; 
      input.value = divElement.innerHTML
      input.style = "height: 49px; width: 100%"
      divElement.innerHTML = ''
      divElement.appendChild(input);
      input.focus()
      input.onblur = function() {
        divElement.innerHTML = input.value;
      }
    } 
  }
}
</script>
<style lang="scss" scoped>
.wrapper {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 110px;
  overflow-y: scroll;
  .background {
    position: absolute;
    top: 0px;
    left: 0;
    right: 0;
    z-index: -1;
  }
  .list {
    position: absolute;
    top: 0px;
    left: 0;
    right: 0;
  }
}
.headerLine {
  border-top: 1px solid gray;
  border-right:1px solid gray;
  position: fixed;
  text-align: center;
  box-sizing: border-box;
  font-size: 12;
  background-color: gray;
  right: 50px;
  .header {
    position: fixed;
    // border-bottom:1px solid gray;
    border-top:1px solid gray;
    border-right:1px solid gray;
    width: 300px;
    line-height: 49px;
    height: 49px;
    background-color: gray;
    color: white;
    z-index: 99999 !important;
  }
}

.top_line {
    border-top: 1px solid gray;
}
.bottom_line {
    border-top: 1px solid gray;
}
.bottom_right {
    border-top: 1px solid gray;
}
.line {
  border-top: 1px solid gray;
  border-right:1px solid gray;
  position: absolute;
  text-align: center;
  box-sizing: border-box;
  font-size: 12;
  .item {
    position: absolute;
    // border-bottom:1px solid gray;
    border-bottom:1px solid gray;
    border-right:1px solid gray;
    width: 300px;
    line-height: 49px;
    
    &.lt {
      margin-left: 10px;
    }
    &.gt {
      margin-right: 10px;
    }
  }
  .item:hover {
    background-color: aqua;
  }
}

.edit-input {
  height: 49px;
  width: 100%;
}
</style>