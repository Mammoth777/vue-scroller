<template>
  <div class="con">
    <!-- <h1>title here</h1> -->
    <scroller
      :on-infinite="infinite"
      :on-refresh="refresh"
      :isFirstLoad = false
      ref="scroller"
    >
      <ul class="list">
        <li v-for="(item, index) in list" :key="index">{{item}}</li>
      </ul>
    </scroller>
  </div>
</template>

<script>
import scroller from '../components/scroller'
export default {
  name: "demo",
  components: {
    scroller
  },
  data() {
    return {
      list: [],
      index: 0,
      count: 0,
      hasMore: true
    };
  },
  methods: {
    refresh(done) {
      console.log("refresh");
      setTimeout(() => {
        done()
      }, 1000);
    },
    infinite(done) {
      this.index++;
      console.log("infinite", this.index);
      setTimeout(() => {
        for(let i = 0; i < 30; i++){
          this.list.push(this.count++);
        }
        if(this.count >= 100){
          done(false);
        }else{
          done()
        }
      }, 1000);
    }
  },
  created () {
    
  },
  mounted () {
    this.$refs.scroller.triggerInfinite(); // 第一次进入触发上拉加载
  }
};
</script>

<style lang="less" scoped>
.con {
  height: 100vh;
  overflow: hidden;
  // padding-bottom: 44px;
  // box-sizing: border-box;
  h1 {
    height: 44px;
    line-height: 44px;
    font-size: 20px;
    background-color: #ccc;
    overflow: hidden;
    text-align: center;
  }
  ul{
    padding: 0;
    margin: 0;
    li{
      background-color: #f2f2f2;
      list-style: none;
      padding: 10px;
      text-align: center;
    }
    li:nth-child(odd){
      background-color: lightgray;
    }
  }
}
</style>

