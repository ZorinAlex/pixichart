<template>
  <div>
    <pixi-chart/>
  </div>
</template>

<script>
  import pixiChart from "./components/pixiChart";
  export default {
    name: "App",
    components: {
      pixiChart
    },
    data: ()=>({

    }),
    methods: {

    },
    computed:{
    },
    mounted() {

    }
  };
</script>
<style lang="scss" scoped>
  .strategy-description{
    margin: 15px 0;
    .strategy-description__item{
      div{
        width: 250px;
      }
    }
  }
</style>