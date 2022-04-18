# Pixi Line Chart
## Simple Line Chart Component for Vue 3.0 <img src="https://vuejs.org/images/logo.png" width="25" height="25">

## Features
- Ability to add points reactive, push ponts to points array property, several or one by one
- Ability to handle big datasets with bottom horizontal navigation
- Vertical autoscale by min and max values on seleted region
- Full colors and sizes customization
- Draw data with real values horizontal scale by number or Date timestamp
- Add different poins colors and each point personal info text

## Installation

```sh
npm i pixi-line-chart
```

## Props

| Prop Name | Type | Default | Short Description |
| ------ | ------ | ------ | ------ |
| maxHorizontalLabels | Number | 20 | Number of horizontal labels |
| horizontalLines | Number | 5 | Number of horizontal axe lines |
| miniChartHorizontalPickerWidth | Number | 10 | picker width |
| miniChartMinimumSelection | Number | 10 | min selection region size |
| miniChartMaxPints | Number | 100 | auto scale points selection |
| chartHeight | Number | 600 | chart height |
| mainChartPadding | Object | {top: 20, bottom: 80, left: 25, right: 25} | padding for big chart |
| miniChartPadding | Object | {top: 5, bottom: 5, left: 25, right: 25} | padding for mini chart |
| isDateTime | Boolean | true | horizontal labels type |
| dateTimeFormat | Object | {locales: 'en-US', options: {dateStyle:'short', timeStyle:'short'}} | format for Date type |
| colors | Object | see table below | chart colors customize |
| labelStyles | Object | see table below | chart labels customize |
| infoBoxColorFollow | Boolean | true | color of infobox will depend on point color |
| pointsData | Array |  | Data points array |
                        
### Colors Objet property
| Prop Name | Default Color |
| ------ | ------ |
| background | ![#0C0C0C](https://via.placeholder.com/15/0C0C0C/000000?text=+) `#0C0C0C`|
| line | ![#DBDBDB](https://via.placeholder.com/15/DBDBDB/000000?text=+) `#DBDBDB` |
| point | ![#7EDDA6](https://via.placeholder.com/15/7EDDA6/000000?text=+) `#7EDDA6` |
| horizontalLines | ![#262626](https://via.placeholder.com/15/262626/000000?text=+) `#262626` |
| texts | ![#a8a8a8](https://via.placeholder.com/15/a8a8a8/000000?text=+) `#a8a8a8` |
| infoBoxBackground | ![#219653](https://via.placeholder.com/15/219653/000000?text=+) `#219653` |
| infoBoxText | ![#cccccc](https://via.placeholder.com/15/cccccc/000000?text=+) `#cccccc` |
| infoBoxValues | ![#FFFFFF](https://via.placeholder.com/15/FFFFFF/000000?text=+) `#FFFFFF` |
| infoPoint | ![#219653](https://via.placeholder.com/15/219653/000000?text=+) `#219653` |
| miniChartPickerLines | ![#BAF6D3](https://via.placeholder.com/15/BAF6D3/000000?text=+) `#BAF6D3` |
| miniChartSideBackground | ![#121212](https://via.placeholder.com/15/121212/000000?text=+) `#121212` |
| miniChartCenterBackground | ![#219653](https://via.placeholder.com/15/219653/000000?text=+) `#219653` |
| timePickerColor | ![#219653](https://via.placeholder.com/15/219653/000000?text=+) `#219653` |

### labelStyles Objet property
| Prop Name | Default |
| ------ | ------ |
| fontFamily | Roboto |
| horizontalLabels | Object |
| horizontalLabels.fontSize | 11 |
| horizontalLabels.fontWeight | normal |
| verticalLabels | Object |
| verticalLabels.fontSize | 11 |
| verticalLabels.fontWeight | normal |
| infoBoxValues | Object |
| infoBoxValues.fontSize | 13 |
| infoBoxValues.fontWeight | normal |
| infoBoxText | Object |
| infoBoxText.fontSize | 11 |
| infoBoxText.fontWeight | normal |

## Example

```javascript
<template>
  <div>
    <pixi-line-chart
            :pointsData="points"
            :is-date-time="false"
            :main-chart-padding="{top: 20, bottom: 60, left: 25, right: 25}"
    />
    <button @click="add">ADD POINT</button>
    <button @click="addArr">ADD ARRAY</button>
  </div>
</template>

<script>
  import pixiLineChart from "./components/pixiLineChart";
  export default {
    name: "App",
    components: {
      pixiLineChart
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
    }
  };
</script>
```

## License
MIT