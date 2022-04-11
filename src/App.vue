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
      getColor(action) {
        switch (action) {
          case "SELL":
            return { color: "#ff0b0c", shape: "circle" };
          case "TS_STEP":
            return { color: "#7e0b0c", shape: "circle" };
          case "BUY":
            return { color: "#00ed12", shape: "rect" };
          case "TB_STEP":
            return { color: "#287e20", shape: "rect" };
          case "SL_STEP":
            return { color: "#00028b", shape: "triangle" };
          case "SL":
            return { color: "#001ff8", shape: "triangle" };
          case "NONE":
            return { color: "#9c9c9c", shape: "star" };
        }
      },
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