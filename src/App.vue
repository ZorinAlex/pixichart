<template>
  <div>
    <pixi-chart
            :pointsData="points"
            :is-date-time="false"
            :main-chart-padding="{top: 20, bottom: 40, left: 25, right: 25}"
    />
    <button @click="add">ADD POINT</button>
    <button @click="addArr">ADD ARRAY</button>
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
      points: [],
      x: 100
    }),
    methods: {
      add(){
        this.points.push({x: this.x, y: Math.tan(Math.random())*Math.random()*150, color: '#3e41e5', shape: 'circle', size: 4, info: "BUY ME A NUGGET"});
        this.x += 100;
      },
      addArr(){
        this.points.push(
                ...[
                  {x: this.x, y: Math.tan(Math.random())*Math.random()*150, color: '#e2030b', shape: 'circle', size: 4, info: "BUY ME A NUGGET"},
                  {x: this.x+100, y: Math.tan(Math.random())*Math.random()*150, color: '#e2030b', shape: 'circle', size: 4, info: "BUY ME A NUGGET"},
                  {x: this.x+150, y: Math.tan(Math.random())*Math.random()*150, color: '#e2030b', shape: 'circle', size: 4, info: "BUY ME A NUGGET"},
                  {x: this.x+200, y: Math.tan(Math.random())*Math.random()*150, color: '#e2030b', shape: 'circle', size: 4, info: "BUY ME A NUGGET"}
                ]
        );
        this.x += 300
      }
    },
    computed:{
    },
    mounted() {

    }
  };
</script>