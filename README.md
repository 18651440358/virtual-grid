# zc-virtual-grid 虚列网格列表



## 所有属性

| 属性    | 类型   | 解释             | 默认值 | 是否必须 |
| ------- | ------ | ---------------- | ------ | -------- |
| data    | Array  | 数据来源         | -      | 是       |
| size    | Number | 页面同时显示个数 | 20     |          |
| columns | Number | 显示列数         | 3      |          |
| gap     | Number | 条目间隙         | 30     |          |



## 所有事件

| 事件名     | 解释                             | 参数 | 返回值 |
| ---------- | -------------------------------- | ---- | ------ |
| loadhandle | 网格列表滚动触底触发加载钩子函数 | -    | -      |



## 使用案例

```vue
<template>
  <div class="page-continer">
    <zc-virtual-grid
      :data="testList"
      :size="20"
      :gap="20"
      :columns="3"
      v-slot="{ item }"
      @loadhandle="loadHandle"
      ref="virtual"
    >
      <div
        class="item-continer"
        :style="{ backgroundColor: item.backgroundColor }"
      >
        {{ item.code }}
      </div>
    </zc-virtual-grid>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import ZcVirtualGrid from "./components/virtual-grid/virtual-grid.vue";
import { tokenFun } from "./utils/token.js";
export default {
  name: "App",
  components: { ZcVirtualGrid },
  setup() {
    // 测试数据
    const testList = ref([]);

    // 随机颜色
    const colors = ref([
      "#4DC591",
      "#FF6500",
      "#FFBE3F",
      "#6F4FF9",
      "#05A3E6",
      "#E05B5B",
      "#203B54",
      "#343026",
    ]);

    onMounted(() => {
      for (let i = 0; i < 100; i++) {
        testList.value.push({
          code: tokenFun(),
          backgroundColor:
            colors.value[Math.floor(Math.random() * colors.value.length)],
        });
      }
    });

    // 获取虚拟网格实例
    const virtual = ref(null);

    // 列表触底加载事件
    function loadHandle() {
      console.log("loading....");
      // 模拟加载数据
      setTimeout(() => {
        for (let i = 0; i < 100; i++) {
          testList.value.push({
            code: tokenFun(),
            backgroundColor:
              colors.value[Math.floor(Math.random() * colors.value.length)],
          });
        }
        // 加载完毕之后需要手动设置 canLoadNewContent = true 才能在下一次触发加载事件
        virtual.value.canLoadNewContent = true;
        console.log("load complete!");
      }, 3000);
    }

    return {
      testList,
      colors,
      loadHandle,
      virtual,
    };
  },
};
</script>

<style>
body {
  margin: 20px;
  font-family: Avenir;
}
.page-continer {
  height: calc(100vh - 40px);
  display: flex;
}
.item-continer {
  height: 300px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: #fff;
}
</style>

```