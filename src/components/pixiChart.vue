<template>
    <div class="chart">
        <canvas id="chart-canvas"></canvas>
    </div>
</template>

<script>
    import * as PIXI from 'pixi.js'
    import * as _ from 'lodash';
    export default {
        name: "pixiChart",
        data: ()=>({
            app: null,
            chartContainer: null,
            valuesLabelsContainer: null,
            timeLabelsContainer: null,
            lineGraphics: null,
            infoGraphics: null,
            miniLineGraphics: null,
            miniLineChartContainer: null,
            miniLineChartPickerContainer: null,
            miniLinePickerGraphics: null,
            selectionEnabled: false,
            selectedTarget: null,
            centerSide: null,
            leftSide: null,
            rightSide: null,
            leftSideBg: null,
            rightSideBg: null,
            leftPoint: null,
            rightPoint: null,
            dragStartPosition: null,
            dragSpriteStartX: 0,
            maxHorizontalLabels: 20,
            horizontalLines: 5,
            mainChartPadding: {top: 20, bottom: 50, left: 25, right: 25},
            miniChartPadding: {top: 5, bottom: 5, left: 25, right: 25},
            miniChartHorizontalPickerWidth: 10,
            miniChartMinimumSelection: 10,
            chartHeight: 400,
            minChartHeight: 100,
            infoText: null,
            points: [],
            currentPoints: [],
            colors: null
        }),
        mounted() {
            this.colors = {
                background: '#0C0C0C',
                line: '#DBDBDB',
                point: '#7EDDA6',
                horizontalLines: '#262626',
                texts: '#a8a8a8',
                infoBoxBackground: '#219653',
                infoBoxText: '#FFFFFF',
                infoPoint: '#219653',
                miniChartPickerLines: '#BAF6D3',
                miniChartSideBackground: '#121212',
                miniChartCenterBackground: '#219653'
            };
            for (let property in this.colors) {
                this.colors[property] = PIXI.utils.string2hex(this.colors[property])
            }
            //for devtools
            window.PIXI = PIXI;
            //
            this.app = new PIXI.Application({ antialias: true, transparent: false, backgroundColor: this.colors.background, resolution: 1.25, view:window.document.getElementById('chart-canvas')});
            window.addEventListener('resize', this.onDeviceScreenResize);
            this.app.renderer.view.style.display = "block";
            this.app.renderer.autoResize = true;
            this.app.renderer.resize(this.$el.offsetWidth, this.chartHeight);
            this.timeLabelsContainer = new PIXI.Container();
            this.app.stage.addChild(this.timeLabelsContainer);
            this.timeLabelsContainer.y = this.viewHeight() + this.mainChartPadding.top;
            this.valuesLabelsContainer = new PIXI.Container();
            this.valuesLabelsContainer.y = this.mainChartPadding.top;
            this.app.stage.addChild(this.valuesLabelsContainer);
            this.chartContainer = new PIXI.Container();
            this.app.stage.addChild(this.chartContainer);
            this.horlineGraphics = new PIXI.Graphics();
            this.horlineGraphics.x = this.mainChartPadding.left;
            this.horlineGraphics.y = this.mainChartPadding.top;
            this.chartContainer.addChild(this.horlineGraphics);
            this.lineGraphics = new PIXI.Graphics();
            this.lineGraphics.x = this.mainChartPadding.left;
            this.lineGraphics.y = this.mainChartPadding.top;
            this.chartContainer.addChild(this.lineGraphics);
            this.linePointsGraphics = new PIXI.Graphics();
            this.linePointsGraphics.x = this.mainChartPadding.left;
            this.linePointsGraphics.y = this.mainChartPadding.top;
            this.chartContainer.addChild(this.linePointsGraphics);
            this.infoGraphics = new PIXI.Container();
            this.infoGraphics.x = this.mainChartPadding.left;
            this.infoGraphics.y = this.mainChartPadding.top;
            this.chartContainer.addChild(this.infoGraphics);
            //miniChart
            this.miniLineChartContainer = new PIXI.Container();
            this.app.stage.addChild(this.miniLineChartContainer);
            this.miniLineChartContainer.x = this.miniChartPadding.left;
            this.miniLineChartContainer.y = this.chartHeight - this.minChartHeight - this.miniChartPadding.top;
            this.miniLineGraphics = new PIXI.Graphics();
            this.miniLineChartPickerContainer= new PIXI.Container();
            this.miniLineChartPickerContainer.y = this.chartHeight - this.minChartHeight*1.5;
            this.app.stage.addChild(this.miniLineChartPickerContainer);
            this.miniLinePickerGraphics = new PIXI.Graphics();
            this.globalsParams = null;
            this.infoPointSprite = null;
            PIXI.BitmapFont.from("Roboto", {
                fill: this.colors.texts,
                fontSize: 44,
                fontWeight: 'normal',
            });
            this.addInfoGraphics();
            this.addTestData();
        },
        methods:{
            addTestData(){
                let points = [];
                for(let i = 0; i < 50; i++){
                    points.push({x: i*5, y: Math.tan(Math.random())*Math.random()*150, color: this.colors.point, shape: 'circle', size:4});
                }
                this.addPointsArray(points);
            },
            redrawLineChart(){
                this.lineGraphics.clear();
                this.linePointsGraphics.clear();
                this.drawLineChart();
                this.drawHorizontalAxes();
                this.addValueLabels();
                this.addTimeLabels();
            },
            clearContainers(){
                this.lineGraphics.clear();
                this.miniLineGraphics.clear();
                this.linePointsGraphics.clear();
            },
            addPoint(x, y, color, shape, size){
                this.clearContainers();
                this.points.push({x,y,color,shape,size});
                this.drawMiniLineChart();
                if(!this.selectionEnabled) this.addTimePickers();
                this.currentPoints.push({x,y,color,shape,size});
                this.drawLineChart();
                this.drawHorizontalAxes();
                this.addValueLabels();
                this.addTimeLabels();
            },
            addPointsArray(pointsArray){
                this.clearContainers();
                this.points.push(...pointsArray);
                this.drawMiniLineChart();
                this.addTimePickers();
                this.currentPoints.push(...pointsArray);
                this.drawLineChart();
                this.drawHorizontalAxes();
                this.addValueLabels();
                this.addTimeLabels();
                this.calcSelectedRegion();
            },
            drawLineChart(){
                this.lineGraphics.lineStyle(1, this.colors.line, 1);
                const {Ymin, Ymax} = this.minMaxYDataCurrent();
                const {Xmin, Xmax} = this.minMaxXDataCurrent();
                const Yfactor = this.viewHeight()/(Ymax - Ymin);
                const Xfactor = this.viewWidth()/(Xmax - Xmin);
                this.globalsParams = {Xmin, Ymin, Xmax, Ymax, Xfactor, Yfactor};
                _.forEach(this.currentPoints, (point, index)=>{
                    const x_coord = (point.x - Xmin)*Xfactor;
                    const y_coord = this.viewHeight() - (point.y-Ymin)*Yfactor;
                    if(index === 0){
                        this.lineGraphics.moveTo(x_coord,y_coord);
                    }else{
                        this.lineGraphics.lineTo(x_coord,y_coord);
                    }
                    this.drawPoint(x_coord, y_coord, point.color, point.shape, point.size)
                });
            },
            drawPoint(x, y, color, shape, size){
                this.linePointsGraphics.lineStyle(0);
                this.linePointsGraphics.beginFill(color, 1);
                switch (shape) {
                    case 'circle':
                        this.linePointsGraphics.drawCircle(x, y, size);
                        break;
                    case 'rectangle':
                        this.linePointsGraphics.drawRect(x, y, 3, size);
                        break;
                    case 'star':
                        this.linePointsGraphics.drawStar(x, y, 5, size);
                        break;
                }
                this.linePointsGraphics.endFill();
            },
            drawMiniLineChart(){
                this.miniLineGraphics.lineStyle(1, this.colors.line, 1);
                const {Ymin, Ymax} = this.minMaxYData();
                const {Xmin, Xmax} = this.minMaxXData();
                const miniChartHeight = this.minChartHeight - this.miniChartPadding.top - this.miniChartPadding.bottom;
                const Yfactor = miniChartHeight/(Ymax - Ymin);
                const fullWidth = this.app.renderer.width/this.app.renderer.resolution-this.miniChartPadding.left - this.miniChartPadding.right;
                const Xfactor = fullWidth/(Xmax - Xmin);
                _.forEach(this.points, (point, index)=>{
                    const x_coord = point.x*Xfactor;
                    const y_coord = miniChartHeight - (point.y-Ymin)*Yfactor;
                    if(index === 0) {
                        this.miniLineGraphics.moveTo(x_coord,y_coord)
                    }else{
                        this.miniLineGraphics.lineTo(x_coord,y_coord);
                    }
                });
                this.miniLineChartContainer.addChild(this.miniLineGraphics)
            },
            drawHorizontalAxes(){
                const horizontalLineStep = this.viewHeight()/this.horizontalLines;
                this.horlineGraphics.lineStyle(1, this.colors.horizontalLines, 1);
                _.forEach(_.range(0, this.horizontalLines+1), line =>{
                    this.horlineGraphics.moveTo(0,this.viewHeight() -  horizontalLineStep * line);
                    this.horlineGraphics.lineTo(this.viewWidth(),this.viewHeight() - horizontalLineStep * line);
                })
            },
            addValueLabels(){
                const horizontalLineStep = this.viewHeight()/this.horizontalLines;
                const {Ymin, Ymax} = this.minMaxYDataCurrent();
                const labelValueStep = (Ymax - Ymin)/this.horizontalLines;
                if(this.valuesLabelsContainer.children.length === 0){
                    let textStyle = {
                        fontName: "Roboto",
                        fontSize: 11
                    };
                    _.forEach(_.range(0, this.horizontalLines+1), line =>{
                        const y_coord = this.viewHeight() - horizontalLineStep * line;
                        const label = new PIXI.BitmapText((Math.round(Ymax-labelValueStep*line)).toString(), textStyle);
                        label.x = 2;
                        label.y = this.viewHeight() -y_coord-label.height/2;
                        this.valuesLabelsContainer.addChild(label);
                    })
                }else{
                    _.forEach(_.range(0, this.horizontalLines+1), line =>{
                        this.valuesLabelsContainer.children[line].text = Math.round(Ymax-labelValueStep*line).toString()
                    })
                }
            },
            addTimeLabels(){
                if(this.timeLabelsContainer.children.length === 0){
                    let textStyle = {
                        fontName: "Roboto",
                        fontSize: 11
                    };
                    _.forEach(_.range(0, this.maxHorizontalLabels+1), () =>{
                        const label = new PIXI.BitmapText("", textStyle);
                        label.x = -100;
                        label.y = 5;
                        label.anchor.set(1, 1);
                        label.rotation = -Math.PI /2;
                        this.timeLabelsContainer.addChild(label);
                    })
                }
                    const {Xmin, Xmax} = this.minMaxXDataCurrent();
                    const Xfactor = this.viewWidth()/(Xmax - Xmin);
                    const spaceFactor = Math.ceil(this.currentPoints.length / this.maxHorizontalLabels);
                    _.forEach(_.range(0, this.maxHorizontalLabels+1), index =>{
                        if(this.currentPoints.length>index*spaceFactor){
                            this.timeLabelsContainer.children[index].text = (this.currentPoints[index*spaceFactor].x).toFixed(0);
                            this.timeLabelsContainer.children[index].x = (this.currentPoints[index*spaceFactor].x - Xmin)*Xfactor + this.mainChartPadding.left+ 7;
                        }else{
                            this.timeLabelsContainer.children[index].x = -100;
                        }
                    })

            },
            addTimePickers(){
                if(_.isNil(this.miniLineChartPickerContainer.getChildByName('left'))){
                    const miniChartHeight = this.minChartHeight - this.miniChartPadding.top - this.miniChartPadding.bottom;

                    this.miniLinePickerGraphics.lineStyle(0);
                    this.miniLinePickerGraphics.beginFill(this.colors.infoBoxBackground);
                    this.miniLinePickerGraphics.drawRoundedRect(0, 10, this.miniChartHorizontalPickerWidth, miniChartHeight-20, this.miniChartHorizontalPickerWidth/3);
                    this.miniLinePickerGraphics.endFill();
                    this.miniLinePickerGraphics.lineStyle(2, this.colors.infoBoxBackground, 1);
                    this.miniLinePickerGraphics.moveTo(this.miniChartHorizontalPickerWidth/2,0);
                    this.miniLinePickerGraphics.lineTo(this.miniChartHorizontalPickerWidth/2,miniChartHeight);
                    this.miniLinePickerGraphics.lineStyle(1, this.colors.miniChartPickerLines, 1);
                    this.miniLinePickerGraphics.moveTo(this.miniChartHorizontalPickerWidth/2-1,miniChartHeight/2-10);
                    this.miniLinePickerGraphics.lineTo(this.miniChartHorizontalPickerWidth/2-1,miniChartHeight/2+10);
                    this.miniLinePickerGraphics.moveTo(this.miniChartHorizontalPickerWidth/2+1,miniChartHeight/2-10);
                    this.miniLinePickerGraphics.lineTo(this.miniChartHorizontalPickerWidth/2+1,miniChartHeight/2+10);

                    let texture = this.app.renderer.generateTexture(this.miniLinePickerGraphics);
                    let leftSide = new PIXI.Sprite(texture);
                    let rightSide = new PIXI.Sprite(texture);
                    leftSide.x = this.miniChartPadding.left - this.miniChartHorizontalPickerWidth;
                    leftSide.y = miniChartHeight/2;
                    leftSide.name = 'left';
                    rightSide.x = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.right;
                    rightSide.y = miniChartHeight/2;
                    rightSide.name = 'right';
                    leftSide.interactive = true;
                    leftSide.buttonMode = true;
                    leftSide.on('pointerdown', this.onDragStart);
                    leftSide.on('pointerup', this.onDragEnd);
                    leftSide.on('pointerupoutside', this.onDragEnd);
                    rightSide.interactive = true;
                    rightSide.buttonMode = true;
                    rightSide.on('pointerdown', this.onDragStart);
                    rightSide.on('pointerup', this.onDragEnd);
                    rightSide.on('pointerupoutside', this.onDragEnd);
                    rightSide.cursor = 'col-resize';
                    leftSide.cursor = 'col-resize';
                    this.leftSide = leftSide;
                    this.rightSide = rightSide;


                    this.miniLinePickerGraphics.clear();
                    this.miniLinePickerGraphics.beginFill(this.colors.miniChartSideBackground);
                    this.miniLinePickerGraphics.drawRect(0, 0, this.miniChartHorizontalPickerWidth, miniChartHeight);
                    this.miniLinePickerGraphics.endFill();
                    texture = this.app.renderer.generateTexture(this.miniLinePickerGraphics);
                    let leftSideBg = new PIXI.Sprite(texture);
                    let rightSideBg = new PIXI.Sprite(texture);
                    leftSideBg.y = miniChartHeight/2;
                    rightSideBg.y = miniChartHeight/2;
                    leftSideBg.anchor.set(1, 0);
                    rightSideBg.anchor.set(0, 0);
                    leftSideBg.alpha = 0.7;
                    rightSideBg.alpha = 0.7;
                    leftSideBg.position.x = this.miniChartPadding.left - this.miniChartHorizontalPickerWidth/2;
                    leftSideBg.width = this.app.renderer.width/this.app.renderer.resolution;
                    rightSideBg.position.x = rightSide.position.x + this.miniChartHorizontalPickerWidth/2;
                    rightSideBg.width = this.app.renderer.width/this.app.renderer.resolution;
                    this.leftSideBg = leftSideBg;
                    this.rightSideBg = rightSideBg;

                    this.miniLinePickerGraphics.clear();
                    this.miniLinePickerGraphics.beginFill(this.colors.miniChartCenterBackground);
                    this.miniLinePickerGraphics.drawRect(0, 0, 10, miniChartHeight);
                    this.miniLinePickerGraphics.endFill();
                    texture = this.app.renderer.generateTexture(this.miniLinePickerGraphics);
                    this.centerSide = new PIXI.Sprite(texture);
                    this.centerSide.alpha = 0.1;
                    this.centerSide.interactive = true;
                    this.centerSide.buttonMode = true;
                    this.centerSide.name = 'center';
                    this.centerSide.on('pointerdown', this.onDragStart);
                    this.centerSide.on('pointerup', this.onDragEnd);
                    this.centerSide.on('pointerupoutside', this.onDragEnd);
                    this.centerSide.y = miniChartHeight/2;
                    this.centerSide.x = this.miniChartPadding.left - this.miniChartHorizontalPickerWidth+ this.miniChartHorizontalPickerWidth/2;
                    this.centerSide.width = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.right - this.miniChartPadding.left+ this.miniChartHorizontalPickerWidth;
                    this.centerSide.cursor = 'move';
                    this.miniLineChartPickerContainer.addChild(this.centerSide);
                    this.miniLineChartPickerContainer.addChild(leftSideBg);
                    this.miniLineChartPickerContainer.addChild(rightSideBg);
                    this.miniLineChartPickerContainer.addChild(leftSide);
                    this.miniLineChartPickerContainer.addChild(rightSide);
                }
            },
            onDragStart(e) {
                //e.target.alpha = 0.5;
                this.dragStartPosition = {x: e.data.global.x, y: e.data.global.y};
                this.dragSpriteStartX = e.target.position.x;
                this.selectedTarget = e.target;
                this.app.stage.interactive = true;
                this.app.stage.on('pointermove', this.onDragMove);
            },
            onDragEnd() {
                //this.selectedTarget.alpha = 1;
                this.app.stage.interactive = false;
            },
            onDragMove(e) {
                let x_coord = e.data.global.x;
                const leftSideStartPosition = this.miniChartPadding.left - this.miniChartHorizontalPickerWidth;
                const rightSideStartPosition = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.right;
                switch (this.selectedTarget.name) {
                    case 'left':
                        if(x_coord < leftSideStartPosition){
                            x_coord = leftSideStartPosition
                        }else if(x_coord > this.rightSide.x - this.miniChartHorizontalPickerWidth - this.miniChartMinimumSelection){
                            x_coord = this.rightSide.x - this.miniChartHorizontalPickerWidth - this.miniChartMinimumSelection
                        }
                        this.leftSideBg.position.x = x_coord+ this.miniChartHorizontalPickerWidth/2;
                        this.selectedTarget.position.x = x_coord;
                        this.centerSide.x = x_coord + this.miniChartHorizontalPickerWidth/2;
                        this.centerSide.width = this.rightSide.x - x_coord ;
                        break;
                    case 'right':
                        if(x_coord < this.leftSide.x + this.miniChartHorizontalPickerWidth + this.miniChartMinimumSelection){
                            x_coord = this.leftSide.x + this.miniChartHorizontalPickerWidth + this.miniChartMinimumSelection
                        }else if(x_coord > rightSideStartPosition){
                            x_coord = rightSideStartPosition
                        }
                        this.rightSideBg.position.x = x_coord + this.miniChartHorizontalPickerWidth/2;
                        this.selectedTarget.position.x = x_coord;
                        this.centerSide.width = x_coord - this.leftSide.x;
                        break;
                    case 'center':
                        this.centerSide.x = this.dragSpriteStartX + x_coord - this.dragStartPosition.x;
                        if(this.centerSide.x<leftSideStartPosition+this.miniChartHorizontalPickerWidth){
                            this.centerSide.x=leftSideStartPosition+this.miniChartHorizontalPickerWidth
                        }else if(this.centerSide.x + this.centerSide.width> rightSideStartPosition){
                            this.centerSide.x = rightSideStartPosition - this.centerSide.width
                        }
                        this.leftSide.x = this.centerSide.x - this.miniChartHorizontalPickerWidth/2;
                        this.rightSide.x = this.centerSide.x + this.centerSide.width- this.miniChartHorizontalPickerWidth/2;
                        this.leftSideBg.x = this.leftSide.x + this.miniChartHorizontalPickerWidth/2;
                        this.rightSideBg.x = this.rightSide.x + this.miniChartHorizontalPickerWidth/2;

                        break;
                }

                this.calcSelectedRegion();
                this.redrawLineChart();
            },
            calcSelectedRegion(){
                const {Xmin, Xmax} = this.minMaxXData();
                const fullWidth = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.left - this.miniChartPadding.right;
                const Xfactor = fullWidth/(Xmax - Xmin);
                const leftX = this.leftSide.position.x+this.miniChartHorizontalPickerWidth;
                const rightX = this.rightSide.position.x;
                const leftIndex = _.findIndex(this.points, point =>{
                    return point.x*Xfactor + this.miniChartPadding.left>=leftX
                });
                let rightIndex  = _.findIndex(this.points, point =>{
                    return point.x*Xfactor  + this.miniChartPadding.left>=rightX
                });
                let leftY;
                if(leftIndex === 0){
                    leftY = this.points[0].y
                }else{
                    const lambda = ((leftX - this.miniChartPadding.left)/Xfactor - this.points[leftIndex - 1].x)/(this.points[leftIndex].x - (leftX-this.miniChartPadding.left)/Xfactor);
                    leftY = Math.round((this.points[leftIndex - 1].y + lambda*this.points[leftIndex].y)/(1+lambda));
                }
                let rightY;
                if(rightIndex === this.points.length){
                    rightY = this.points[this.points.length].y
                }else{
                    const lambda = ((rightX - this.miniChartPadding.left)/Xfactor - this.points[rightIndex - 1].x)/(this.points[rightIndex].x - (rightX-this.miniChartPadding.left)/Xfactor);
                    rightY = Math.round((this.points[rightIndex - 1].y + lambda*this.points[rightIndex].y)/(1+lambda));
                    if(_.isNaN(rightY)) rightY = this.points[rightIndex].y
                }
                this.currentPoints = this.points.slice(leftIndex, rightIndex);
                const leftPoint = {x: Math.round((leftX - this.miniChartPadding.left)/Xfactor), y: leftY};
                const rightPoint = {x: Math.round((rightX - this.miniChartPadding.left)/Xfactor), y: rightY};
                this.currentPoints = [{x: leftPoint.x, y: leftPoint.y, color: 'white', shape: 'none'},...this.currentPoints, {x: rightPoint.x, y: rightPoint.y, color: 'white', shape: 'none'}]
            },
            viewHeight(){
                return this.chartHeight - this.mainChartPadding.top - this.mainChartPadding.bottom - this.minChartHeight;
            },
            viewWidth(){
                return this.app.renderer.width/this.app.renderer.resolution - this.mainChartPadding.left - this.mainChartPadding.right;
            },
            minMaxYData(){
                const Ymin = _.minBy(this.points, point=> point.y).y;
                const Ymax = _.maxBy(this.points, point=> point.y).y;
                return {Ymin, Ymax}
            },
            minMaxXData(){
                const Xmin = _.minBy(this.points, point=> point.x).x;
                const Xmax = _.maxBy(this.points, point=> point.x).x;
                return {Xmin, Xmax}
            },
            minMaxYDataCurrent(){
                const Ymin = _.minBy(this.currentPoints, point=> point.y).y;
                const Ymax = _.maxBy(this.currentPoints, point=> point.y).y;
                return {Ymin, Ymax}
            },
            minMaxXDataCurrent(){
                const Xmin = _.minBy(this.currentPoints, point=> point.x).x;
                const Xmax = _.maxBy(this.currentPoints, point=> point.x).x;
                return {Xmin, Xmax}
            },
            addInfoGraphics(){
                const lineGraphics = new PIXI.Graphics;
                lineGraphics.lineStyle(2, 0xFFFFFF, 1);
                lineGraphics.moveTo(0, 0);
                lineGraphics.lineTo(0, this.viewHeight());
                this.lineGraphics.interactive = true;
                this.lineGraphics.hitArea = new PIXI.Rectangle(0, 0, this.viewWidth(), this.viewHeight());
                this.lineGraphics.name = "lineGraphics";
                this.lineGraphics.on('mousemove', this.onMoveInfographics);
                this.infoText = new PIXI.Container();
                this.infoText.alpha = 0;
                PIXI.BitmapFont.from("RobotoBig", {
                    fill: "#ffffff",
                    fontSize: 13,
                    fontWeight: 'normal',
                }, {chars: [['a', 'z'], ['0', '9'], ['A', 'Z'], '.:']});
                let textStyle = {
                    fontName: "RobotoBig",
                    fontSize: 13
                };
                const label = new PIXI.BitmapText("", textStyle);
                label.name = 'infoLineText_1';
                label.x = 5;
                label.y = 5;

                const infoTextBg = new PIXI.Graphics();
                infoTextBg.beginFill(this.colors.infoBoxBackground);
                infoTextBg.alpha = 1;
                infoTextBg.drawRoundedRect(0, 0, 80, 40, 5);
                infoTextBg.endFill();
                let infoTextureBg = this.app.renderer.generateTexture(infoTextBg);
                let infoTextureBgSprite = new PIXI.Sprite(infoTextureBg);
                this.infoText.addChild(infoTextureBgSprite);
                this.infoText.addChild(label);

                const infoPoint = new PIXI.Graphics();
                infoPoint.lineStyle(2, this.colors.infoPoint, 1);
                infoPoint.beginFill(0,0);
                infoPoint.drawCircle(0, 0, 6);
                infoPoint.endFill();
                // infoPoint.lineStyle(0);
                // infoPoint.beginFill(this.colors.infoPoint, 1);
                // infoPoint.drawCircle(0, 0, 3);
                // infoPoint.endFill();


                let infoPointTexture = this.app.renderer.generateTexture(infoPoint);
                this.infoPointSprite = new PIXI.Sprite(infoPointTexture);
                this.infoPointSprite.anchor.set(0.5);
                this.infoPointSprite.alpha = 0;

                this.infoGraphics.addChild(this.infoPointSprite);
                this.infoGraphics.addChild(this.infoText);
            },
            onMoveInfographics(e){
                if(!_.isNil(e.target) && e.target.name === this.lineGraphics.name && !_.isNil(this.globalsParams)){
                    let x_coord = e.data.global.x;
                    const {Xmin, Ymin,Xfactor, Yfactor} = this.globalsParams;
                    const leftIndex = _.findIndex(this.currentPoints, point =>{
                        return (point.x - Xmin)*Xfactor>=x_coord - this.mainChartPadding.left
                    });
                    const line = this.infoText.getChildByName('infoLineText_1');
                    line.text = `X: ${this.currentPoints[leftIndex].x} \nY: ${(this.currentPoints[leftIndex].y).toFixed(2)}`;
                    const X = (this.currentPoints[leftIndex].x- Xmin)*Xfactor;
                    const Y = this.viewHeight() - (this.currentPoints[leftIndex].y-Ymin)*Yfactor;

                    this.infoText.x = X + 10;
                    this.infoText.y = Y - 20;
                    this.infoPointSprite.x = X;
                    this.infoPointSprite.y = Y;
                    this.infoPointSprite.alpha = 1;
                    this.infoPointSprite.tint = this.currentPoints[leftIndex].color;
                    this.infoText.alpha = 1;
                }else{
                    this.infoPointSprite.alpha = 0;
                    this.infoText.alpha = 0;
                }
            },
            onDeviceScreenResize(){
                this.app.renderer.resize(this.$el.offsetWidth, this.chartHeight);
                this.lineGraphics.hitArea = new PIXI.Rectangle(0, 0, this.viewWidth(), this.viewHeight());
                this.clearContainers();
                this.drawMiniLineChart();
                this.drawLineChart();
                this.drawHorizontalAxes();
                this.addValueLabels();
                this.addTimeLabels();
                this.replaceTimePickers()
            },
            replaceTimePickers(){
                const {Xmin, Xmax} = this.minMaxXData();
                const fullWidth = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.left - this.miniChartPadding.right;
                const Xfactor = fullWidth/(Xmax - Xmin);
                const xLeftCoord = this.currentPoints[0].x;
                const xRightCoord = this.currentPoints[this.currentPoints.length -1].x;

                this.leftSideBg.width = this.app.renderer.width/this.app.renderer.resolution;
                this.rightSideBg.width = this.app.renderer.width/this.app.renderer.resolution;
                this.leftSide.x = xLeftCoord*Xfactor+this.miniChartPadding.left - this.miniChartHorizontalPickerWidth;
                this.rightSide.x = xRightCoord*Xfactor +this.miniChartPadding.left ;
                this.centerSide.x = this.leftSide.x + this.miniChartHorizontalPickerWidth;
                this.centerSide.width = this.rightSide.x - this.leftSide.x - this.miniChartHorizontalPickerWidth;
                this.leftSideBg.x = this.leftSide.x;
                this.rightSideBg.x = this.rightSide.x + this.miniChartHorizontalPickerWidth;
            }
        },
    }
</script>

<style scoped>
    .chart canvas{
        max-width: 100%
    }
</style>