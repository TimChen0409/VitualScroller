<template>
    <div class="virtual-scroller" @scroll="onScroll">
        <div class="phantom" :style="{ height: scrollBarHeight + 'px' }">
            <ul :style="{ 'margin-top': `${scrollTop}px` }">
                <li v-for="item in visibleList" :key="item.id"
                    :style="{ height: `${props.itemHeight}px`, 'line-height': `${props.itemHeight}px` }">
                    <slot :item="item"></slot>
                </li>
            </ul>
        </div>
    </div>
</template>
<script setup>
import { onMounted, ref, reactive, computed, watch } from 'vue';

const itemTopCache = [];

const visibleCount = ref(10);
const startIndex = ref(0);
const endIndex = ref(10);
const scrollTop = ref(0);
const scrollBarHeight = ref(0);
const bufferItemCount = ref(10);
const itemHeightCache = ref({});

const props = defineProps({
    list: {
        type: Array,
        default: [],
        required: true
    },
    itemHeight: {
        type: Number,
        default: 50
    },
})

const visibleList = computed(() => {
    return props.list.slice(startIndex.value, endIndex.value + bufferItemCount.value);
});

//缓存每個元素的scrollTop
const generateItemTopPositionCache = () => {
    const allHeight = props.list.reduce((pre, current, i) => {
        const heightSum = pre + props.itemHeight;
        itemHeightCache.value[i] = pre;
        itemTopCache[i] = i === 0 ? 0 : itemTopCache[i - 1] + props.itemHeight;
        return heightSum;
    }, 0)
    scrollBarHeight.value = allHeight;
};

//Binary Search減少搜尋startIndex的時間
const getStartIndex = (scrollTop) => {
    let [left, right] = [0, itemTopCache.length - 1];

    while (right >= left) {//如果結束位置比起始位置大就繼續找
        let middle = Math.floor((left + right) / 2);
        if (scrollTop === itemTopCache[middle]) {//比對到了直接回傳資料在陣列中的位置
            return middle
        } else if (scrollTop > itemTopCache[middle]) {//要找的數字比中間大，找右邊
            left = middle + 1;
        } else {//要找的數字比中間小，找左邊
            right = middle - 1;
        }
    }
    return left;//預設找不到時回傳第一個，避免startIndex=-1造成滾動時閃爍
};

//滾動事件(每次觸發時更新startIndex,endIndex和scrollTop值)
const onScroll = (e) => {
    const targetScrollTop = e.target.scrollTop;
    startIndex.value = getStartIndex(targetScrollTop);
    endIndex.value = startIndex.value + visibleCount.value;
    scrollTop.value = itemTopCache[startIndex.value] || 0;
}

onMounted(() => {
    console.log('child');
    generateItemTopPositionCache();
})
</script>
<style scoped lang="scss">
.virtual-scroller {
    height: 100%;
    overflow: auto;

    &::-webkit-scrollbar {
        width: 6px;
    }

    &::-webkit-scrollbar-thumb {
        background-color: rgba(18, 18, 18, 0.38);
        border-radius: 20px;
    }

    &::-webkit-scrollbar-thumb:hover {
        background-color: rgba(18, 18, 18, 0.58);
    }

    .phantom {
        overflow: hidden;
    }
}
</style>