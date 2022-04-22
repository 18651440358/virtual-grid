<template>
  <!--  虚拟Grid列表  -->
  <div class="virtual-grid-container" ref="container" @scroll="scrollHandle">
    <!-- 网络列表容器 -->
    <div
      class="virtual-grid-list"
      ref="list"
      :style="[gridStyle, virtualStyle]"
    >
      <template v-for="item in visualData">
        <slot :item="item"></slot>
      </template>
      <!-- 加载触发视图 -->
      <div
        class="load-trigger-view"
        ref="loading"
        :style="{ gridColumnEnd: `span ${columns}` }"
      >
        加载更多...
      </div>
    </div>
    <!-- 占位高度 -->
    <div
      class="virtual-occupying-height"
      :style="{ height: `${totalHeight}px` }"
    ></div>
  </div>
</template>

<script>
import {
  ref,
  computed,
  onMounted,
  nextTick,
  watch,
  onBeforeUnmount,
} from "vue";
export default {
  emits: ["loadhandle"],
  props: {
    // 数据来源
    data: Array,
    // 页面同时显示个数
    size: {
      type: Number,
      default: 20,
    },
    // 显示列数
    columns: {
      type: Number,
      default: 3,
    },
    // 条目间隙
    gap: {
      type: Number,
      default: 30,
    },
  },
  setup(props, ctx) {
    // 获取虚拟网格容器
    const container = ref(null);
    // 获取滚动列表
    const list = ref(null);

    // 起始下标
    const startIndex = ref(0);
    // 结束下标
    const endIndex = ref(0);

    // 网格样式
    const gridStyle = computed(() => {
      return {
        gridTemplateColumns: `repeat(${props.columns}, 1fr)`,
        gridGap: `${props.gap}px`,
      };
    });

    // 顶部缓冲计算
    const aboveBuffer = computed(() => {
      return Math.min(startIndex.value, props.columns);
    });
    // 底部缓冲计算
    const belowBuffer = computed(() => {
      return Math.min(props.data.length - endIndex.value, props.columns);
    });

    // 可视区域计算
    const visualData = computed(() => {
      let start = startIndex.value - aboveBuffer.value;
      let end = endIndex.value + belowBuffer.value;
      return props.data.slice(start, Math.min(end, props.data.length));
    });

    // 初始化网格
    function init() {
      nextTick(() => {
        if (endIndex.value + props.size < props.data.length)
          endIndex.value = endIndex.value + props.size - props.columns;
        else endIndex.value = props.data.length;
      });
    }

    // 总高度
    const totalHeight = ref(0);
    // 位置记录
    const milepost = ref([]);

    // 初始化高度
    function initListTotalHeight() {
      let num = 0;
      nextTick(() => {
        if (props.data.length > 3 && endIndex.value >= 3)
          num = Math.max(
            list.value.children[0].getBoundingClientRect().height,
            list.value.children[1].getBoundingClientRect().height,
            list.value.children[2].getBoundingClientRect().height
          );
        totalHeight.value = num * Math.ceil(props.data.length / props.columns);
        // 初始化高度记录值
        milepost.value = [];
        let index = 0;
        for (let i = 0; i < props.data.length; i++) {
          if (i > 0 && i % props.columns === 0) index++;
          milepost.value.push({
            top: index * props.gap + index * num,
            bottom: index * props.gap + (index + 1) * num,
          });
        }
      });
    }

    // 使用二分查找寻找高度记录
    function getStartIndex() {
      let start = 0;
      let end = milepost.value.length - 1;
      let tempIndex = null;
      while (start <= end) {
        let midIndex = parseInt((start + end) / 2, 10);
        let midValue = milepost.value[midIndex].bottom;
        if (midValue === srcollTop.value) {
          return midIndex + 1;
        } else if (midValue < srcollTop.value) {
          start = midIndex + 1;
        } else if (midValue > srcollTop.value) {
          if (tempIndex === null || tempIndex > midIndex) {
            tempIndex = midIndex;
          }
          end = end - 1;
        }
      }
      return tempIndex;
    }

    // 虚列列表位移
    const virtualStyle = computed(() => {
      let startOffset = 0;
      if (startIndex.value >= props.columns) {
        let size =
          milepost.value[startIndex.value].top -
          (milepost.value[startIndex.value - aboveBuffer.value]
            ? milepost.value[startIndex.value - aboveBuffer.value].top
            : 0);
        startOffset = milepost.value[startIndex.value].top - size;
      } else {
        startOffset = 0;
      }
      return {
        transform: `translate3d(0,${startOffset}px,0)`,
      };
    });

    // 已滚动高度
    const srcollTop = ref(0);
    // 滚动处理函数
    function scrollHandle() {
      srcollTop.value = container.value.scrollTop;
      startIndex.value = getStartIndex();
      endIndex.value =
        startIndex.value + props.size - props.columns - props.columns;
    }

    // 获取加载视图组件
    const loading = ref(null);
    // 监听事件
    let io;

    // 是否可以加载新内容
    const canLoadNewContent = ref(true);

    onMounted(() => {
      // 初始化
      init();
      initListTotalHeight();
      // 监听加载组件
      io = new IntersectionObserver((e) => {
        if (canLoadNewContent.value && e[0].isIntersecting) {
          // 触发加载
          ctx.emit("loadhandle");
          canLoadNewContent.value = false;
        }
      });
      // 间隔100毫秒
      io.POLL_INTERVAL = 100;
      nextTick(() => {
        // 监听加载视图
        if (loading.value) io.observe(loading.value);
      });
    });

    // 当数据发现变化时重新计算
    watch(props.data, () => {
      initListTotalHeight();
    });

    onBeforeUnmount(() => {
      io.disconnect();
    });

    return {
      container,
      list,
      startIndex,
      endIndex,
      gridStyle,
      aboveBuffer,
      belowBuffer,
      visualData,
      init,
      totalHeight,
      milepost,
      initListTotalHeight,
      getStartIndex,
      virtualStyle,
      srcollTop,
      scrollHandle,
      loading,
      canLoadNewContent,
    };
  },
};
</script>

<style lang="less">
// 虚列列表容器
.virtual-grid-container {
  flex: 1;
  height: 100%;
  max-height: 100vh;
  position: relative;
  overflow-x: hidden;
  overflow-y: scroll;
  // scrollbar-width: none;
  // -ms-overflow-style: none;
}
// .virtual-grid-container::-webkit-scrollbar {
//   display: none;
// }
// 数据列表
.virtual-grid-list {
  left: 0;
  right: 0;
  display: grid;
  position: absolute;
  grid-area: 1 / 1 / auto / auto;
  place-self: center;
  grid-template: repeat();
}
// 加载组件
.load-trigger-view {
  display: flex;
  justify-content: center;
}
</style>