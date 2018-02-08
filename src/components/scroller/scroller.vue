<template>
  <div 
    class="scroller"
    ref="scroller"
    @scroll="outterscroll"
    @touchstart="tchstart"
    @touchmove="tchmove"
    @touchend="tchend"
  >
    <div class="top">
      <slot name="top">
        <span class="spinner"></span>
      </slot>
    </div>
    <div class="content"
      :style="{transform: `translateY(${pulldownlen || pulluplen}px)`}"
    >
      <slot class="inner"></slot>
    </div>
    <div class="bottom" :style="{'bottom': `${bottomPosition}px`}" v-show="pullState===2">
      <slot name="bottom">
        <span class="spinner"></span>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "scroller",
  data() {
    return {
      topPercent: 0, // 页面卷动比例
      startPos: 0, // 触摸开始位置
      limitPulldown: 50, // 下拉阈值
      limitPullup: 50, // 上拉阈值
      pulldownlen: 0, // 下拉距离
      pulluplen: 0, // 上拉距离
      pullState: 0 // 拉动状态, 0: 无 , 1:下拉, 2:上拉
    };
  },
  props: {
    onInfinite: {
      type: Function,
      required: false,
      default: function(done){
        setTimeout(() => {
          done();
        }, 500);
      },
    }, // 上拉
    onRefresh: {
      type: Function,
      required: false,
      default: function(done){
        this.onInfinite(done);
      },
    }, // 下拉
    isFirstLoad: {
      // 是否第一次加载
      type: Boolean,
      default: false,
    },
    bottomPosition: {
      // 底部动画
      type: Number,
      default: 0,
    },
  },
  computed: {
    reachBottom() {
      // 触底
      return this.topPercent === 1;
    },
    reachTop() {
      // 触顶
      return this.topPercent === 0;
    },
    loading() {
      return this.pullState ? true : false; // 0则未加载, 非0则正在加载中
    }
  },
  methods: {
    // 滚动条滚动
    outterscroll() {
      const scroller = this.$refs.scroller; // 当前组件根元素
      let sTop = scroller.scrollTop; // 滚动距离
      let sHeight = scroller.scrollHeight; // 可滚动高度
      let offsetHeight = scroller.offsetHeight; // 总高度
      /* console.log({
        sTop,
        sHeight,
        offsetHeight
      }); */
      this.topPercent = sTop / (sHeight - offsetHeight); // 移动百分比;
    },
    tchstart(e) {
      // 在非 loading 状态下, 重置状态
      if (!this.loading) {
        this.pulldownlen = 0;
        this.pulluplen = 0;
        this.startPos = e.touches[0].clientY;
      } else {
        null;
      }
    },
    tchmove(e) {
      let len = e.touches[0].clientY - this.startPos;
      if (this.reachTop && len > 0) {
        // 下拉
        this.pulluplen = 0;
        this.pulldownlen = len / 7;
      } else if (this.reachBottom && len < 0) {
        // 上拉
        this.pulldownlen = 0;
        this.pulluplen = len / 7;
      }
    },
    tchend() {
      // 触顶, 且超过 限制
      if (this.reachTop && this.pulldownlen > this.limitPulldown) {
        this.pulldownlen = this.limitPulldown;
        this.pullState = 1;
      } else {
        this.pulldownlen = 0;
      }
      // 触底, 且超过 限制 , 且有可加载的数据
      if (this.reachBottom && this.pulluplen < -this.limitPullup && this.hasMore) {
        this.pulluplen = -this.limitPullup;
        this.pullState = 2;
      } else {
        this.pulluplen = 0;
      }
    },
    
    done(hasMore) {
      console.log(hasMore);
      if(hasMore === false){
        this.hasMore = false;
      }else{
        this.hasMore = true;
      }
      //完成加载后, 重置各种状态
      this.pulldownlen = 0;
      this.pulluplen = 0;
      this.pullState = 0;
    },
    // 手动触发 上拉加载
    triggerInfinite() {
      this.onInfinite && this.onInfinite(this.done);
    }
  },
  watch: {
    loading() {
      switch (this.pullState) {
        case 0: // 无动作
          break;
        case 1: // 下拉
          this.onRefresh && this.onRefresh(this.done);
          break;
        case 2: // 上拉
          this.onInfinite && this.onInfinite(this.done);
          break;
        default:
          break;
      }
    }
  },
  created() {
    this.isFirstLoad && this.onInfinite(this.done); // 第一次进入是否自动调用上拉加载
  }
};
</script>

<style lang="less" scoped>
.scroller {
  width: 100%;
  height: 100%;
  max-width: 100vw;
  max-height: 100vh;
  overflow: hidden;
  overflow-y: scroll;
  position: relative;
  .content {
    transition: all 0.2s ease-out;
    background-color: transparent;
  }
  .top {
    position: absolute;
    width: 100%;
    top: 0;
  }
  .bottom {
    position: fixed;
    width: 100vw;
    z-index: -1;
  }
  span.spinner {
    display: block;
    height: 28px;
    width: 28px;
    margin: 11px auto;
    background: url("data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/PjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+PHN2ZyB0PSIxNTE3NDc3MTY3NzA2IiBjbGFzcz0iaWNvbiIgc3R5bGU9IiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHAtaWQ9IjI1NjciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB3aWR0aD0iMjAwIiBoZWlnaHQ9IjIwMCI+PGRlZnM+PHN0eWxlIHR5cGU9InRleHQvY3NzIj48L3N0eWxlPjwvZGVmcz48cGF0aCBkPSJNNDEuNjM5IDU5NC43OThsMjkwLjM2NSAxOTMuOTQ5LTUyLjg5OSA4Mi42NTRjNjUuMTQ5IDQyLjQzNCAxNDEuOTI1IDY3LjM2MyAyMjQuNDUyIDY3LjM2MyAxODAuNDQxIDAgMzM0LjE4Ni0xMTcuMzk1IDM5NS4yNzMtMjgyLjY4OCAyMS4zNzYtNTcuODg1IDU0Ljk1NS03MS45ODIgODQuMDI1LTYzLjY4My0zOC4yNzcgMjI1LjY5NS0yMzQuMjAxIDM5Ny43MjUtNDcwLjc5IDM5Ny43MjUtMjM1Ljc2MSAwLTQzMS4xNzUtMTcwLjgwNC00NzAuNDIzLTM5NS4zMTl6IG05NDIuMjUtMTU2LjI0NUw2OTEuNTgxIDIyNi40MjlsNDYuMzg1LTc3LjY1MkM2NzMuNjMgMTA4LjUwOSA1OTguNDMgODQuOTAzIDUxNy43MiA4NC45MDNjLTE4NC45NDggMC0zNDEuOTkgMTIxLjc5MS00MDAuNzY4IDI5MS44NDYtMTAuNDk3IDE4LjY4NS00MS4wMTcgMzMuODAxLTcwLjY3NiAyOS4wMDZDOTQuNjY3IDE5Mi45NzggMjg0LjYzNCAzNC4wMSA1MTIuMDY1IDM0LjAxYzIzOC45NjMgMCA0MzYuMzg0IDE3NS41MDMgNDcxLjgyNiA0MDQuNTQzeiIgcC1pZD0iMjU2OCI+PC9wYXRoPjwvc3ZnPg==");
    background-size: cover;
    animation: round 1s linear infinite;
  }
}

// animation
@keyframes round {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>

