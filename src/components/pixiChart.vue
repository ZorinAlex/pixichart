<template>
    <div class="chart">
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
            miniChartMaxPints: 100,
            chartHeight: 400,
            minChartHeight: 100,
            infoText: null,
            points: [],
            currentPoints: [],
            colors: null,
            labelStyles: null,
            infoBoxColorFollow: true,
            isTimeMoved: false
        }),
        mounted() {
            this.colors = {
                background: '#0C0C0C',
                line: '#DBDBDB',
                point: '#7EDDA6',
                horizontalLines: '#262626',
                texts: '#a8a8a8',
                infoBoxBackground: '#219653',
                infoBoxText: '#cccccc',
                infoBoxValues: '#FFFFFF',
                infoPoint: '#219653',
                miniChartPickerLines: '#BAF6D3',
                miniChartSideBackground: '#121212',
                miniChartCenterBackground: '#219653',
                timePickerColor: '#219653',
            };
            this.labelStyles = {
                fontFamily: 'Roboto',
                horizontalLabels: {
                    fontSize: 11,
                    fontWeight: 'normal',
                },
                verticalLabels: {
                    fontSize: 11,
                    fontWeight: 'normal',
                },
                infoBoxValues: {
                    fontSize: 13,
                    fontWeight: 'normal',
                },
                infoBoxText: {
                    fontSize: 11,
                    fontWeight: 'normal',
                }
            };
            for (let property in this.colors) {
                this.colors[property] = PIXI.utils.string2hex(this.colors[property])
            }
            //for devtools
            window.PIXI = PIXI;
            //
            this.app = new PIXI.Application({ antialias: true, transparent: false, backgroundColor: this.colors.background, resolution: 1.25});
            window.addEventListener('resize', this.onDeviceScreenResize);
            this.$el.appendChild(this.app.view);
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
            this.addFonts();
            this.addInfoGraphics();
            this.addTestData();
        },
        methods:{
            addTestData(){
                //let points = [];
                // for(let i = 0; i < 50; i++){
                //     points.push({x: i*5, y: Math.tan(Math.random())*Math.random()*150, color: this.colors.point, shape: 'circle', size:4, info: "BUY ME A NUGGET"});
                // }
                // this.addPointsArray(points);
                let xc = 0;
                for(let i = 0; i < 30; i++){

                    setTimeout(()=>{
                        let points = [];
                        for(let i = 0; i < 50; i++){
                            xc = xc+50;

                            points.push({x: xc, y: Math.tan(Math.random())*Math.random()*150, color: this.colors.point, shape: 'circle', size:4, info: "BUY ME A NUGGET"});
                        }
                        this.addPointsArray(points);
                    }, 1000 * i)
                }
            },
            addFonts(){
                PIXI.BitmapFont.from("HLFont", {
                    fill: this.colors.texts,
                    fontSize: this.labelStyles.horizontalLabels.fontSize*2,
                    fontWeight: this.labelStyles.horizontalLabels.fontWeight,
                });
                PIXI.BitmapFont.from("VLFont", {
                    fill: this.colors.texts,
                    fontSize: this.labelStyles.horizontalLabels.fontSize*2,
                    fontWeight: this.labelStyles.horizontalLabels.fontWeight,
                });
                PIXI.BitmapFont.from("IBValues", {
                    fill: this.colors.infoBoxValues,
                    fontSize: this.labelStyles.infoBoxValues.fontSize*2,
                    fontWeight: this.labelStyles.infoBoxValues.fontWeight,
                }, {chars: [['a', 'z'], ['0', '9'], ['A', 'Z'], '.:- ']});
                PIXI.BitmapFont.from("IBText", {
                    fill: this.colors.infoBoxText,
                    fontSize: this.labelStyles.infoBoxValues.fontSize*2,
                    fontWeight: this.labelStyles.infoBoxValues.fontWeight,
                }, {chars: [['a', 'z'], ['0', '9'], ['A', 'Z'], '.:- ']});
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
            addPoint(x, y, color, shape, size, info){
                if(!_.isNumber(color)) color = PIXI.utils.string2hex(color);
                this.clearContainers();
                this.points.push({x,y,color,shape,size, info});
                this.drawMiniLineChart();
                this.addTimePickers();
                this.calcSelectedRegion();
                this.drawLineChart();
                this.drawHorizontalAxes();
                this.addValueLabels();
                this.addTimeLabels();
            },
            addPointsArray(pointsArray){
                if(!_.isNumber(pointsArray[0].color)) pointsArray =
                    _.map(pointsArray, (point)=> {
                        point.color = PIXI.utils.string2hex(point.color);
                        return point
                });
                this.clearContainers();
                this.points.push(...pointsArray);
                this.drawMiniLineChart();
                this.addTimePickers();
                this.moveTimePickers();
                // miniChartMaxPints
                this.calcSelectedRegion();
                this.drawLineChart();
                this.drawHorizontalAxes();
                this.addValueLabels();
                this.addTimeLabels();
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
                        this.linePointsGraphics.drawRect(x, y, size, size);
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
                        fontName: "HLFont",
                        fontSize: this.labelStyles.horizontalLabels.fontSize
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
                        fontName: "VLFont",
                        fontSize: this.labelStyles.verticalLabels.fontSize
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
                    this.miniLinePickerGraphics.beginFill(this.colors.timePickerColor);
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
            moveTimePickers(){
                if(!this.isTimeMoved && this.points.length> this.miniChartMaxPints){
                    const X_coord = this.points[this.points.length-this.miniChartMaxPints].x;
                    const X_right = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.right;

                        const {Xmin, Xmax} = this.minMaxXData();
                        const fullWidth = this.app.renderer.width/this.app.renderer.resolution - this.miniChartPadding.left - this.miniChartPadding.right;
                        const Xfactor = fullWidth/(Xmax - Xmin);
                        const X_position = X_coord*Xfactor+this.miniChartPadding.left - this.miniChartHorizontalPickerWidth;
                    if((X_right - X_position )> 100){
                        this.leftSideBg.width = this.app.renderer.width/this.app.renderer.resolution + this.miniChartHorizontalPickerWidth/2;
                        this.leftSide.x = X_position;
                        this.centerSide.x = this.leftSide.x + this.miniChartHorizontalPickerWidth/2;
                        this.centerSide.width = this.rightSide.x - this.leftSide.x + this.miniChartHorizontalPickerWidth;
                        this.leftSideBg.x = this.leftSide.x + this.miniChartHorizontalPickerWidth/2;
                    }
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
                this.isTimeMoved = true;
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
                this.lineGraphics.interactive = true;
                this.lineGraphics.hitArea = new PIXI.Rectangle(0, 0, this.viewWidth(), this.viewHeight());
                this.lineGraphics.name = "lineGraphics";
                this.lineGraphics.on('mousemove', this.onMoveInfographics);
                this.infoText = new PIXI.Container();
                this.infoText.alpha = 0;
                let textStyle = {
                    fontName: "IBValues",
                    fontSize: this.labelStyles.infoBoxValues.fontSize,
                };

                const label = new PIXI.BitmapText("", textStyle);
                label.name = 'infoLineText_1';
                label.x = 5;
                label.y = 5;

                let textStyle2 = {
                    fontName: "IBText",
                    fontSize: this.labelStyles.infoBoxText.fontSize,
                    maxWidth: 150
                };

                const labelInfo = new PIXI.BitmapText("", textStyle2);
                labelInfo.name = 'infoLineText_2';
                labelInfo.x = 5;
                labelInfo.y = 35;
                this.createInfoTextResizableBackground(80,40,8);
                this.infoText.addChild(label);
                this.infoText.addChild(labelInfo);

                let color = this.colors.infoPoint;
                if(this.infoBoxColorFollow) color = 0xFFFFFF;
                const infoPoint = new PIXI.Graphics();
                infoPoint.lineStyle(2, color, 1);
                infoPoint.beginFill(0,0);
                infoPoint.drawCircle(0, 0, 6);
                infoPoint.endFill();
                infoPoint.lineStyle(0);
                infoPoint.beginFill(color, 1);
                infoPoint.drawCircle(0, 0, 3);
                infoPoint.endFill();


                let infoPointTexture = this.app.renderer.generateTexture(infoPoint);
                this.infoPointSprite = new PIXI.Sprite(infoPointTexture);
                this.infoPointSprite.anchor.set(0.5);
                this.infoPointSprite.alpha = 0;

                this.infoGraphics.addChild(this.infoPointSprite);
                this.infoGraphics.addChild(this.infoText);
            },
            createInfoTextResizableBackground(width, height, radius){
                let color = this.colors.infoBoxBackground;
                if(this.infoBoxColorFollow) color = 0xFFFFFF;
                const corner = new PIXI.Graphics();
                corner.beginFill(color);
                corner.drawCircle(radius, radius, radius);
                corner.drawRect(radius, 0, radius, radius);
                corner.drawRect(0,radius, 2*radius, radius);
                corner.endFill();
                const center = new PIXI.Graphics();
                center.beginFill(color);
                center.drawRect(0,0, width-4*radius, height);
                center.endFill();
                const side = new PIXI.Graphics();
                side.beginFill(color);
                side.drawRect(0,0, 2*radius, height-4*radius);
                side.endFill();
                const cornerTexture = this.app.renderer.generateTexture(corner);
                const cornerLTSprite = new PIXI.Sprite(cornerTexture);
                cornerLTSprite.x=0;
                cornerLTSprite.y=0;
                cornerLTSprite.name='cornerLT';
                const cornerLBSprite = new PIXI.Sprite(cornerTexture);
                cornerLBSprite.anchor.set(0.5);
                cornerLBSprite.x=radius;
                cornerLBSprite.y=height-radius;
                cornerLBSprite.angle = -90;
                cornerLBSprite.name='cornerLB';
                const cornerRTSprite = new PIXI.Sprite(cornerTexture);
                cornerRTSprite.anchor.set(0.5);
                cornerRTSprite.x=width-radius;
                cornerRTSprite.y=radius;
                cornerRTSprite.angle = 90;
                cornerRTSprite.name='cornerRT';
                const cornerRBSprite = new PIXI.Sprite(cornerTexture);
                cornerRBSprite.anchor.set(0.5);
                cornerRBSprite.x=width-radius;
                cornerRBSprite.y=height-radius;
                cornerRBSprite.angle = 180;
                cornerRBSprite.name='cornerRB';
                const centerTexture = this.app.renderer.generateTexture(center);
                const centerSprite = new PIXI.Sprite(centerTexture);
                centerSprite.x=radius*2;
                centerSprite.y=0;
                centerSprite.name='center';
                const sideTexture = this.app.renderer.generateTexture(side);
                const sideLSprite = new PIXI.Sprite(sideTexture);
                sideLSprite.x=0;
                sideLSprite.y=2*radius;
                sideLSprite.name='sideL';
                const sideRSprite = new PIXI.Sprite(sideTexture);
                sideRSprite.x=width-2*radius;
                sideRSprite.y=2*radius;
                sideRSprite.name='sideR';
                this.infoText.addChild(cornerLTSprite);
                this.infoText.addChild(cornerLBSprite);
                this.infoText.addChild(cornerRTSprite);
                this.infoText.addChild(cornerRBSprite);
                this.infoText.addChild(centerSprite);
                this.infoText.addChild(sideLSprite);
                this.infoText.addChild(sideRSprite);
            },
            resizeInfoBackground(width, height, tint){
                const cornerLT = this.infoText.getChildByName('cornerLT');
                const cornerLB = this.infoText.getChildByName('cornerLB');
                const cornerRT = this.infoText.getChildByName('cornerRT');
                const cornerRB = this.infoText.getChildByName('cornerRB');
                const center = this.infoText.getChildByName('center');
                const sideL = this.infoText.getChildByName('sideL');
                const sideR = this.infoText.getChildByName('sideR');
                const radius = cornerLB.height/2;
                cornerLB.y = height-radius;
                cornerRT.x = width-radius;
                cornerRB.x = width-radius;
                cornerRB.y = height-radius;
                center.width = width - 4*radius;
                center.height = height;
                sideL.height = height - 4*radius;
                sideR.x = width-2*radius;
                sideR.height = height - 4*radius;
                if(!_.isNil(tint)){
                    cornerLT.tint = tint;
                    cornerLB.tint = tint;
                    cornerRT.tint = tint;
                    cornerRB.tint = tint;
                    center.tint = tint;
                    sideL.tint = tint;
                    sideR.tint = tint;
                }
            },
            onMoveInfographics(e){
                if(!_.isNil(e.target) && e.target.name === this.lineGraphics.name && !_.isNil(this.globalsParams)){
                    let x_coord = e.data.global.x;
                    const {Xmin, Ymin,Xfactor, Yfactor} = this.globalsParams;
                    const leftIndex = _.findIndex(this.currentPoints, point =>{
                        return (point.x - Xmin)*Xfactor>=x_coord - this.mainChartPadding.left
                    });
                    let color;
                    if(this.infoBoxColorFollow) color = this.lightenDarkenColor(this.currentPoints[leftIndex].color, -100);
                    const line = this.infoText.getChildByName('infoLineText_1');
                    line.text = `${this.currentPoints[leftIndex].x} \n${(this.currentPoints[leftIndex].y).toFixed(2)}`;
                    const lineInfo = this.infoText.getChildByName('infoLineText_2');
                    if(!_.isNil(this.currentPoints[leftIndex].info)){
                        lineInfo.visible = true;
                        lineInfo.text = this.currentPoints[leftIndex].info;
                        this.resizeInfoBackground(lineInfo.textWidth+10, 40 + lineInfo.textHeight, color);
                    }else{
                        this.resizeInfoBackground(80, 40, color);
                        lineInfo.visible = false
                    }
                    const X = (this.currentPoints[leftIndex].x- Xmin)*Xfactor;
                    const Y = this.viewHeight() - (this.currentPoints[leftIndex].y-Ymin)*Yfactor;
                    if(X > this.app.renderer.width/this.app.renderer.resolution - this.mainChartPadding.right - lineInfo.textWidth-20){
                        const X_text = X - lineInfo.textWidth-30;
                        this.infoText.x = X_text + 10;
                    }else {
                        this.infoText.x = X + 10;
                    }
                    this.infoText.y = Y - 20;
                    this.infoPointSprite.x = X;
                    this.infoPointSprite.y = Y;
                    this.infoPointSprite.alpha = 1;
                    this.infoPointSprite.tint = this.lightenDarkenColor(this.currentPoints[leftIndex].color, -100);
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
                this.centerSide.x = this.leftSide.x + this.miniChartHorizontalPickerWidth/2;
                this.centerSide.width = this.rightSide.x - this.leftSide.x;
                this.leftSideBg.x = this.leftSide.x + this.miniChartHorizontalPickerWidth/2;
                this.rightSideBg.x = this.rightSide.x + this.miniChartHorizontalPickerWidth/2;
            },
            lightenDarkenColor(colorCode, amount) {
                let r = (colorCode >> 16) + amount;
                if (r > 255) {
                    r = 255;
                } else if (r < 0) {
                    r = 0;
                }
                let b = ((colorCode >> 8) & 0x00FF) + amount;
                if (b > 255) {
                    b = 255;
                } else if (b < 0) {
                    b = 0;
                }
                let g = (colorCode & 0x0000FF) + amount;
                if (g > 255) {
                    g = 255;
                } else if (g < 0) {
                    g = 0;
                }
                return '0x' +(g | (b << 8) | (r << 16)).toString(16);
            }
        },
    }
</script>

<style>
    canvas{
        max-width: 100%
    }
</style>