<template>
  <div class="container">
    <custom-virtual-scroller :list="dataList" :item-height="50" v-slot="{ item }">
      <div class="rowItem">
        <div>{{ item.id }},</div>
        <div>{{ item.name }},</div>
        <div>{{ item.address }},</div>
        <div>{{ item.area }},</div>
        <button>檢視</button>
        <button>編輯</button>
        <button>刪除</button>
      </div>
    </custom-virtual-scroller>
  </div>
</template>

<script setup>
import { onBeforeMount, onMounted, ref, reactive, computed, watch } from "vue";
import CustomVirtualScroller from "@/components/CustomVirtualScroller.vue";

const dataList = ref([]);

const getDataList = (itemCount) => {
  const newDataList = [...Array(itemCount).keys()].map((v, i) => ({
    id: i + 1,
    name: `第${i + 1}項`,
    address: `板橋區${i + 1}號~~~~~`,
    area: `${i + 1}區`
  }));
  return newDataList;
};

onBeforeMount(() => {
  dataList.value = getDataList(100000);
})
</script>

<style lang="scss" scoped>
.container {
  height: 400px;
  width: 600px;
  
  .rowItem{
    display: flex;
    margin: 8px;
  }
}
</style>
