<template>
  <div class="x-drag">
    <div class="item-wrapper"
      :style="{ height: `${dataList.length * itemHeight}px` }"
      :ref="setItemWrapRef"
      >
      <div v-for="(item, index) in dataList"
        :key="index"
        :ref="setItemRef"
        :class="['item', `${index === cur ? 'zIndex' : ''}`, `${itemTransition ? 'itemTransition' : ''}`]"
        :style="`transform: translateY(${index === cur ? tranY : item.tranY}px);`"
        :data-key="item.key"
        @touchstart="touchstart(index, $event)"
        @touchmove="touchmove"
        @touchend="touchend">
        <div class="info">{{item.data}}</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watchEffect, reactive, onMounted, toRefs, computed } from 'vue';

interface IDragData {
  tranY: number,
  data: any,
  key: number
}

export default defineComponent({
  name: 'HelloWorld',
  props: {
    list: {
      type: Array,
      default: []
    }
  },
  setup(props, context) {
    const data = reactive({ 
      dataList: props.list.map((item, index) => ({ tranY: 0, data: item, key: index })),
      itemHeight: 0,
      itemTransition: false,
      dragging: false,
      cur: -1,
      tranY: 0,
      startY: 0
    })

    let itemRefs: Array<HTMLElement> = []
    let itemWrapRef: HTMLElement
    let itemWrapRect: any
    let originKey: number = -1
    let tranY: number = 0

    const setItemRef = (el: HTMLElement) => {
      itemRefs.push(el)
    }

    const setItemWrapRef = (el: HTMLElement) => {
      itemWrapRef = el
      itemWrapRect = el.getBoundingClientRect()
    }

    const getPosition = () => {
      data.dataList.forEach(item => {
        item.tranY = item.key * data.itemHeight
      })
    }

    const touchstart = (index: number, e: TouchEvent) => {
      e.preventDefault()
      const { pageY } = e.changedTouches[0]
      data.cur = index
      data.startY = pageY
      data.tranY = data.startY - itemWrapRect.top - data.itemHeight / 2
      tranY = data.tranY
      data.itemTransition = false
      data.dragging = true
    }

    const touchmove = (e: TouchEvent) => {
      if (!data.dragging) return
      const { pageY } = e.changedTouches[0]
      const currentKey = Number((e as any).currentTarget.dataset.key)
      data.tranY = pageY - data.startY + tranY
      let endKey = Math.round(data.tranY / data.itemHeight)
      endKey = endKey < 0 ? 0 : endKey
      endKey = endKey > data.dataList.length - 1 ? data.dataList.length - 1 : endKey
      if (originKey === currentKey || currentKey === endKey) return
      originKey = currentKey
      insert(currentKey, endKey)
    }

    const touchend = (e: TouchEvent) => {
      if (!data.dragging) return
      data.startY = 0
      data.tranY = 0
      data.itemTransition = true
      data.cur = -1
      data.dragging = false
      originKey = -1
      tranY = 0
    }

    const insert = (origin: number, end: number) => {
      if (origin < end) {
        data.dataList.forEach((item => {
          if (item.key > origin && item.key <= end) {
            item.key = item.key -1
          } else if (item.key === origin) {
            item.key = end
          }
        }))
        getPosition()
      } else if (origin > end) {
        data.dataList.forEach((item => {
          if (item.key >= end && item.key < origin) {
            item.key = item.key + 1
          } else if (item.key === origin) {
            item.key = end
          }
        }))
        getPosition()
      }
    }

    watchEffect(() => {
      data.dataList = props.list.map((item, index) => ({ tranY: 0, data: item, key: index }))
    })

    onMounted(() => {
      const itemHeight: number = itemRefs[0].offsetHeight
      data.itemHeight = itemHeight
      getPosition()
    })

    return {
      ...toRefs(data),
      setItemRef,
      setItemWrapRef,
      touchstart,
      touchmove,
      touchend
    }
  }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
  @import "./index.less";
</style>
