<template>
  <!-- <div id="map" class="map2d"></div> -->
  <olmap />
  <!-- <div>{{test}}</div> -->
  <tools ref="tools" />
  <leftTree ref="leftTreeVue" />
</template>

<script lang="ts">
import olmap from '@/components/olmap/olmap.vue'
import { computed, defineComponent, onBeforeMount, onBeforeUnmount, onMounted, reactive, toRefs } from 'vue'
import leftTree from './leftTree.vue'
import tools from './tools.vue'

import Global from '@/components/Global.vue'
export default defineComponent({
  components: {
    olmap,
    leftTree,
    tools
  },
  setup() {
    // let map = Global.olmap

    return {
      test: '123'
    }
  }
})
// export default {
//   name: "Map",
//   components:{
//     olmap
//   }
// };
</script>

<style scoped>
.map2d {
  width: 100%;
  height: 600px;
  border: 1px solid #eee;
}
</style>
