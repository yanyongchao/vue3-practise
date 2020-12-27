<template>
  <div class="home">
    <x-drag :list="list" />
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, onMounted } from 'vue';
import XDrag from '@/components/x-drag/index.vue'

export default defineComponent({
  name: 'Home',
  components: {
    XDrag
  },
  setup () {
    const data = reactive({
      list: [1, 2, 3, 4, 5, 6, 7, 8]
    })

    return {
      ...toRefs(data)
    }
  }
});
</script>
