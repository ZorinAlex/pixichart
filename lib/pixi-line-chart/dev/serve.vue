<script>
import { defineComponent } from 'vue';
import PixiLineChart from '@/pixi-line-chart.vue';

export default defineComponent({
  name: 'ServeDev',
  components: {
    PixiLineChart
  }
});
</script>

<template>
  <div id="app">
    <pixi-line-chart />
  </div>
</template>
