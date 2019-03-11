<template>
  <div id="app">
    <div id="container">
      <h1>平行四边形面积割补法</h1>
      <canvas id="canvas">您的浏览器不支持canvas</canvas>
      <div id="reset">
        <svg viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg"><path d="M713.536 255.232l-58.624 72.192L960 355.712 868.8 64l-72.512 89.344A458.88 458.88 0 0 0 523.52 64C269.76 64 64 269.184 64 522.24c0 253.12 205.76 458.24 459.52 458.24a459.648 459.648 0 0 0 429.44-294.72 65.408 65.408 0 0 0-37.824-84.48 65.728 65.728 0 0 0-84.8 37.76 328.32 328.32 0 0 1-306.752 210.56c-181.312 0-328.32-146.56-328.32-327.36 0-180.736 147.008-327.296 328.32-327.296 69.376 0 135.232 21.504 189.952 60.288" fill="#666666"></path></svg>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      parallelogram: [
        [10, -15, 20],
        [5, -5, 15],
        [-10, -15, 5],
        [10, -10, 15],
      ],
      toolSize: 50,
      zoom: 10,
      activeKnife: false,
      activeTool: 0,
      isMouseClick: false,
      isMouseDown: false,
      isMouseMove: false,
      mouseX: 0,
      mouseY: 0,
      tvParallelograms: [],
      tvCutLineStartX: 0,
      tvCutLineStartY: 0,
      tvCutLineEndX: 0,
      tvCutLineEndY: 0,
      tvColors : ['rgba(255, 0, 0, .5)', 'rgba(0, 255, 0, .5)', 'rgba(0, 0, 255, .5)'],
      holdPolygon : undefined,
      holdPoint: undefined
    }
  },
  mounted () {
    this.context = document.getElementById('canvas').getContext('2d');
    this.initCanvas();
    this.bindEvents();
    this.drawCanvas();
    this.changeTool(1);
  },
  methods : {
    reset : function () {
      this.activeKnife = false;
      this.mouseX = this.mouseY = 0;
      this.changeTool(this.activeTool);
    },
    initCanvas : function () {
      this.context.canvas.width = document.body.clientWidth;
      this.context.canvas.height = document.body.clientHeight;
      //设定放大比例
      this.zoom = Math.floor(Math.min(this.context.canvas.width, this.context.canvas.height)/80)*2;
    },
    bindEvents : function () {
      this.context.canvas.addEventListener('mousedown', this.mouseDown);
      this.context.canvas.addEventListener("mousemove", this.mouseMove);
      this.context.canvas.addEventListener("mouseup", this.mouseUp);
      this.context.canvas.addEventListener("mouseleave", this.mouseLeave);
      this.context.canvas.addEventListener("touchstart", this.mouseDown);
      this.context.canvas.addEventListener("touchmove", this.mouseMove);
      this.context.canvas.addEventListener("touchend", this.mouseUp);
      this.context.canvas.addEventListener("touchcancel", this.mouseLeave);
      this.context.canvas.addEventListener("click", this.mouseClick);
      window.addEventListener("resize", this.resize);
      document.getElementById('reset').addEventListener('click', this.reset);
    },
    resize : function () {
      this.initCanvas();
      this.drawCanvas();
    },
    mouseClick : function () {
      console.log(event.type);
      this.isMouseClick = true;
    },
    mouseDown : function (event) {
      console.log(event.type);
      this.isMouseMove = this.isMouseDown = true;
      let mouseXY = this.getMouseXY(event);
      this.mouseX = mouseXY.x;
      this.mouseY = mouseXY.y;
      this.holdPoint = mouseXY;
      if(this.activeKnife)
      {
        this.tvCutLineEndX = this.tvCutLineEndY = 0;
        this.tvCutLineStartX = mouseXY.x;
        this.tvCutLineStartY = mouseXY.y;
      }
    },
    mouseMove : function (event) {
      console.log(event.type);
      event.preventDefault();
      let mouseXY = this.getMouseXY(event);
      this.mouseX = mouseXY.x;
      this.mouseY = mouseXY.y;
      if(!this.isMouseMove) return;
      this.isMouseDown = false;
      if(this.activeKnife)
      {
        this.tvCutLineEndX = mouseXY.x;
        this.tvCutLineEndY = mouseXY.y;
      }
      if(this.holdPolygon && this.holdPoint)
      {
        this.movePolygon(mouseXY);
        this.holdPoint = mouseXY;
      }
    },
    mouseUp : function () {
      console.log(event.type);
      this.isMouseMove = this.isMouseDown = false;
      this.holdPolygon = this.holdPoint = undefined;
    },
    mouseLeave : function () {
      console.log(event.type);
      this.isMouseMove = this.isMouseDown = false;
      this.holdPolygon = this.holdPoint = undefined;
    },
    getMouseXY : function (event) {
      return {
        x : (event.type.substr(0, 5) === 'touch' ? event.changedTouches[0].clientX : event.clientX) - this.context.canvas.getBoundingClientRect().left,
        y : (event.type.substr(0, 5) === 'touch' ? event.changedTouches[0].clientY : event.clientY) - this.context.canvas.getBoundingClientRect().top
      };
    },
    drawCanvas : function () {
      this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height);
      this.context.canvas.style.cursor = this.activeKnife ? 'none' : 'pointer';
      this.drawBackground();
      this.drawTools();
      this.cutParallelogram();
      this.drawTv();
      if(this.activeKnife && this.mouseX && this.mouseY) this.drawKnife(this.mouseX - this.toolSize/10, this.mouseY - this.toolSize*0.4);
      this.isMouseClick = false;
    },
    drawBackground : function () {
      //画背景
      this.context.save();
      this.context.fillStyle = '#424242';
      this.context.fillRect(0, 0, this.context.canvas.width, this.context.canvas.height);
      //画栅格
      this.context.strokeStyle = 'rgba(255,255,255,.1)';
      for (let i = 0; i < this.context.canvas.width; i += 25) {
        this.context.beginPath();
        this.context.moveTo(i, 0);
        this.context.lineTo(i, this.context.canvas.height);
        this.context.stroke();
      }
      for (let i = 0; i < this.context.canvas.height; i += 25) {
        this.context.beginPath();
        this.context.moveTo(0, i);
        this.context.lineTo(this.context.canvas.width, i);
        this.context.stroke();
      }
      this.context.restore();
    },
    drawTools: function () {
      //确定位置
      let startX = this.context.canvas.width - this.toolSize * 2;
      let startY = this.context.canvas.height / 2 - this.toolSize * (this.parallelogram.length + 1) / 2;
      //点击小刀
      this.context.beginPath();
      this.context.arc(startX + this.toolSize / 2, startY + this.toolSize / 2, this.toolSize / 2, 0, 2 * Math.PI);
      //不允许多次切割
      if(this.isMouseClick && this.tvParallelograms.length <= 1 && this.context.isPointInPath(this.mouseX, this.mouseY))
      {
        this.activeKnife = !this.activeKnife;
        this.mouseX = this.mouseY = 0;
      }
      //画小刀
      this.drawKnife(startX, startY);
      //点击四边形
      for(let i = 1; i <= this.parallelogram.length; i++){
        //点击
        this.context.beginPath();
        this.context.arc(startX + this.toolSize / 2, startY + i * this.toolSize + this.toolSize / 2, this.toolSize / 2, 0, 2 * Math.PI);
        if(this.isMouseClick && this.context.isPointInPath(this.mouseX, this.mouseY))
        {
          this.changeTool(i);
          this.activeKnife = false;
          this.mouseX = this.mouseY = 0;
        }
      }
      //画平行四边形
      let i = 1;
      for(let j of this.parallelogram){
        let points = this.transferParallelogram2Points(startX + this.toolSize/2, startY + i * this.toolSize + this.toolSize/2, j[0], j[1], j[2]);
        this.drawParallelogram(points, this.activeTool === i++ ? 'orange' : '');
      }
    },
    drawKnife: function (startX, startY) {
      let rem = this.toolSize/50;
      //刀头
      this.context.save();
      this.context.beginPath();
      this.context.moveTo(startX + 25*rem, startY + 20*rem);
      this.context.lineTo(startX + 5*rem, startY + 20*rem);
      this.context.arcTo(startX + 5*rem, startY + 30*rem, startX + 20*rem, startY + 30*rem, 10*rem);
      this.context.lineTo(startX + 25*rem, startY + 30*rem);
      this.context.closePath();
      this.context.fillStyle = this.activeKnife ? '#fff' : 'rgba(255,255,255,.3)';
      this.context.fill();
      this.context.restore();
      //刀身
      this.context.save();
      this.context.beginPath();
      this.context.moveTo(startX + 25*rem, startY + 18*rem);
      this.context.lineTo(startX + 45*rem, startY + 18*rem);
      this.context.lineTo(startX + 45*rem, startY + 32*rem);
      this.context.lineTo(startX + 25*rem, startY + 32*rem);
      this.context.lineTo(startX + 25*rem, startY + 18*rem);
      this.context.arc(startX + 42*rem, startY + 21*rem, 2*rem, 0, 2*Math.PI, true);
      this.context.closePath();
      this.context.fillStyle = this.activeKnife ? '#ff4d44' : 'rgba(255,255,255,.3)';
      this.context.fill();
      this.context.restore();
    },
    changeTool : function (tool) {
      this.activeTool = tool;
      if(!this.activeTool)
      {
        this.tvParallelograms = [];
        return;
      }
      let parallelogramSample = this.parallelogram[this.activeTool-1];
      let parallelogramPoints = this.transferParallelogram2Points(Math.round(this.context.canvas.width/2 - this.toolSize), Math.round(this.context.canvas.height/2), parallelogramSample[0] * this.zoom, parallelogramSample[1] * this.zoom, parallelogramSample[2] * this.zoom);
      this.tvParallelograms = [parallelogramPoints];
    },
    drawParallelogram: function (parallelogramPoints, color/*optional*/, border/*optional*/) {
      this.drawPolygon(parallelogramPoints, color, border);
    },
    transferParallelogram2Points: function (centerX, centerY, cornerX, cornerY, parallelogramW) {
      let startX = centerX - (cornerX+parallelogramW)/2;
      let startY = centerY - cornerY/2;
      return [
        {x: startX, y: startY},
        {x: startX + cornerX, y: startY + cornerY},
        {x: startX + cornerX + parallelogramW, y: startY + cornerY},
        {x: startX + parallelogramW, y: startY}
      ];
    },
    drawPolygon: function (points, color/*optional*/, border/*optional*/) {
      this.context.save();
      this.context.beginPath();
      for (let i of points)
      {
        this.context.lineTo(i.x, i.y);
      }
      this.context.closePath();
      this.context.fillStyle = color || 'rgba(255,255,255,.3)';
      this.context.fill();
      this.context.strokeStyle = border || 'transparent';
      this.context.stroke();
      this.context.restore();
    },
    drawTv: function () {
      if(!this.tvParallelograms.length) return;
      //画四边形
      let colorI = 0;
      for (let i in this.tvParallelograms){
        if(!this.tvParallelograms.hasOwnProperty(i)) continue;
        let polygon =  this.tvParallelograms[i];
        this.drawParallelogram(polygon, this.tvColors[colorI++]);
        if(!this.activeKnife && this.isMouseDown && this.context.isPointInPath(this.mouseX, this.mouseY)) {
          this.holdPolygon = i;
        }
        if(colorI >= this.tvColors.length) colorI = 0;
      }
      //画切线
      if(!this.activeKnife || !this.tvCutLineStartX || !this.tvCutLineEndX) return;
      this.context.save();
      this.context.beginPath();
      this.context.moveTo(this.tvCutLineStartX, this.tvCutLineStartY);
      this.context.lineTo(this.tvCutLineEndX, this.tvCutLineEndY);
      this.context.lineWidth = 2;
      this.context.strokeStyle = 'orange';
      this.context.stroke();
      this.context.restore();
    },
    cutParallelogram : function () {
      if(this.isMouseMove) return;
      if(!this.activeKnife || !this.tvCutLineStartX || !this.tvCutLineEndX) return;
      let newTvParallelograms = [];
      for (let p of this.tvParallelograms){
        //查找凸多边形与切割线相交的点
        let crossPoints = this.findParallelogramCrossPoint(p);
        if(crossPoints.length !== 2) newTvParallelograms.push(p);
        else{
          //分割凸多边形
          let polygons = this.cutPolygon(p, crossPoints);
          for(let n of polygons) newTvParallelograms.push(n);
        }
      }
      //更新电视机上的多边形
      this.tvParallelograms = newTvParallelograms;
      //删除划线
      if(this.tvParallelograms.length > 1) this.activeKnife = false;
      this.mouseX = this.mouseY = this.tvCutLineEndX = this.tvCutLineEndY = this.tvCutLineStartX = this.tvCutLineStartY = 0;
    },
    findParallelogramCrossPoint: function(parallelogramPoints){
      let lines = [];
      let prePoint;
      for(let p of parallelogramPoints)
      {
        if(prePoint)
        {
          lines.push({start: prePoint, end: p});
        }
        prePoint = p;
      }
      //闭合
      lines.push({
        start: parallelogramPoints[parallelogramPoints.length - 1],
        end: parallelogramPoints[0]
      });
      let crossPoints = [];
      for (let line of lines)
      {
        let crossPoint = this.findCrossPoint(line.start.x, line.start.y, line.end.x, line.end.y, this.tvCutLineStartX, this.tvCutLineStartY, this.tvCutLineEndX, this.tvCutLineEndY);
        if(crossPoint)
        {
          crossPoints.push({line:line, point:crossPoint});
          if(crossPoints.length >= 2) break;
        }
      }
      return crossPoints;
    },
    findCrossPoint: function (x1, y1, x2, y2, x3, y3, x4, y4) {
      //排除平行
      if(x1 === x2 && x3 === x4) return false;
      if(y1 === y2 && y3 === y4) return false;
      let x = ((x2 - x1) * (x3 - x4) * (y3 - y1) - x3 * (x2 - x1) * (y3 - y4) + x1 * (y2 - y1) * (x3 - x4)) / ((y2 - y1) * (x3 - x4) - (x2 - x1) * (y3 - y4));
      let y = ((y2 - y1) * (y3 - y4) * (x3 - x1) - y3 * (y2 - y1) * (x3 - x4) + y1 * (x2 - x1) * (y3 - y4)) / ((x2 - x1) * (y3 - y4) - (y2 - y1) * (x3 - x4));
      //TODO 精度问题hack
      if(x1 === x2) x = Math.round(x);
      if(y1 === y2) y = Math.round(y);
      if(x >= Math.max(Math.min(x1, x2), Math.min(x3, x4)) && x <= Math.min(Math.max(x1, x2), Math.max(x3, x4)) && y >= Math.max(Math.min(y1, y2), Math.min(y3, y4)) && y <= Math.min(Math.max(y1, y2), Math.max(y3, y4))) return {x : x, y : y};
      return false;
    },
    cutPolygon: function (points, crossPoints) {
      let afterPolygons = [];
      let bakPolygon = [];
      let tempPolygon = [];
      points.push(points[0]);
      for(let i in points){
        if(!points.hasOwnProperty(i)) continue;
        let point  = points[i];
        tempPolygon.push(point);
        //下一条线是否被切割
        if(points.hasOwnProperty(i-0+1))
        {
          let nextPoint = points[i-0+1];
          for (let j in crossPoints){
            if(!crossPoints.hasOwnProperty(j)) continue;
            let crossPoint = crossPoints[j];
            if((crossPoint.line.start === point && crossPoint.line.end === nextPoint) || crossPoint.line.end === point && crossPoint.line.start === nextPoint){
              tempPolygon.push(crossPoint.point);
              if(bakPolygon.length === 0)
              {
                bakPolygon = tempPolygon;
                tempPolygon = [];
              }
              else
              {
                tempPolygon.push(bakPolygon[bakPolygon.length-1]);
                afterPolygons.push(tempPolygon);
                bakPolygon.push(crossPoint.point);
                tempPolygon = bakPolygon;
                bakPolygon = [];
              }
              crossPoints.splice(j, 1);
              break;
            }
          }
        }
      }
      tempPolygon.splice(-1);
      afterPolygons.push(tempPolygon);
      return afterPolygons;
    },
    movePolygon: function (movePoint) {
      let moveX = movePoint.x - this.holdPoint.x;
      let moveY = movePoint.y - this.holdPoint.y;
      let newPoints = [];
      for (let i in this.tvParallelograms[this.holdPolygon])
      {
        if(!this.tvParallelograms[this.holdPolygon].hasOwnProperty(i)) continue;
        newPoints.push({x: this.tvParallelograms[this.holdPolygon][i].x + moveX, y : this.tvParallelograms[this.holdPolygon][i].y + moveY});
      }
      this.tvParallelograms[this.holdPolygon] = newPoints;
    }
  },
  watch : {
    drawCanvasVars : function () {
      this.drawCanvas();
    }
  },
  computed : {
    drawCanvasVars : function () {
      return [this.mouseX, this.mouseY, this.tvParallelograms, this.isMouseMove, this.isMouseClick]
    }
  }
}
</script>

<style lang="css">
  html, body, #app, #container{
    height: 100%;
  }
  body{
    padding: 0;
    margin: 0;
    font-family: -apple-system, "PingFang SC", "Helvetica Neue", STHeiti, "Microsoft Yahei", Tahoma, Simsun, sans-serif;
  }
  #container{
    position: relative;
  }
  #container h1{
    position: absolute;
    left: 10px;
    top: 10px;
    margin: 0;
    color: #fff;
  }
  #canvas{
    display: block;
  }
  #reset {
    position: absolute;
    border-radius: 50%;
    box-shadow: 0 0 5px rgba(0,0,0,.3);
    right: 10px;
    top: 10px;
    background: #fff;
    width: 30px;
    height: 30px;
    text-align: center;
    line-height: 1;
    padding-top: 5px;
    box-sizing: border-box;
    cursor: pointer;
  }
  #reset:hover{
    background-color: #f2f2f2;
  }
  #reset svg{
    width: 1em;
    height: 1em;
    vertical-align: middle;
    fill: currentColor;
    overflow: hidden;
  }
  #canvas {
  }
</style>
