<template>
  <div id="app">
    <div id="container">
      <h1>终边相同角</h1>
      <canvas id="canvas">您的浏览器不支持canvas</canvas>
      <div id="control">
        <p><label>蓝色线条<input type="range" min="-1080" max="1080" step="1" v-model.number="blueLineAngle">角度{{blueLineAngle}}°</label></p>
        <p><label>红色线条<input type="range" min="-1080" max="1080" step="1" v-model.number="redLineAngle">角度{{redLineAngle}}°</label></p>
      </div>
      <div id="reset">
        <svg class="icon" style="width: 1em; height: 1em;vertical-align: middle;fill: currentColor;overflow: hidden;" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="1430"><path d="M713.536 255.232l-58.624 72.192L960 355.712 868.8 64l-72.512 89.344A458.88 458.88 0 0 0 523.52 64C269.76 64 64 269.184 64 522.24c0 253.12 205.76 458.24 459.52 458.24a459.648 459.648 0 0 0 429.44-294.72 65.408 65.408 0 0 0-37.824-84.48 65.728 65.728 0 0 0-84.8 37.76 328.32 328.32 0 0 1-306.752 210.56c-181.312 0-328.32-146.56-328.32-327.36 0-180.736 147.008-327.296 328.32-327.296 69.376 0 135.232 21.504 189.952 60.288" fill="#666666" p-id="1431"></path></svg>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      blueLineAngle : 30,
      redLineAngle : 60,
      context : null,
      activeControl : '',
    }
  },
  mounted () {
    this.context = document.getElementById('canvas').getContext('2d');
    this.initCanvas();
    this.bindEvents();
    this.drawCanvas();
  },
  methods : {
    initCanvas : function () {
      this.context.canvas.width = document.body.clientWidth;
      this.context.canvas.height = document.body.clientHeight;
    },
    bindEvents : function () {
      this.context.canvas.addEventListener('mousedown', this.mouseDown);
      this.context.canvas.addEventListener("mousemove", this.mouseMove);
      this.context.canvas.addEventListener("mouseup", this.mouseUp);
      this.context.canvas.addEventListener("mouseleave", this.mouseLeave);
      this.context.canvas.addEventListener("touchstart", this.mouseDown);
      this.context.canvas.addEventListener("touchmove", this.mouseMove);
      this.context.canvas.addEventListener("touchend", this.mouseUp);
      window.addEventListener("resize", this.resize);
      document.getElementById('reset').addEventListener('click', this.reset);
    },
    reset : function () {
      this.blueLineAngle = 30;
      this.redLineAngle = 60;
      this.activeControl = '';
    },
    resize : function () {
      this.initCanvas();
      this.drawCanvas();
    },
    mouseDown : function (event) {
      //鼠标坐标
      var x =(event.type === 'touchstart' ? event.touches[0].clientX : event.clientX) - this.context.canvas.getBoundingClientRect().left;
      var y = (event.type === 'touchstart' ? event.touches[0].clientY : event.clientY) - this.context.canvas.getBoundingClientRect().top;
      //画两个透明的圆，并检测鼠标是否在圆上
      this.context.save();
      this.context.beginPath();
      this.context.arc(Math.cos((360 - this.blueLineAngle/180) * Math.PI)*100 + this.context.canvas.width/2, Math.sin((360 - this.blueLineAngle/180) * Math.PI)*100 + this.context.canvas.height/2, 20, 0, 2*Math.PI);
      this.context.restore();
      if(this.context.isPointInPath(x, y))
      {
        this.activeControl = 'blue';
        return;
      }
      this.context.save();
      this.context.beginPath();
      this.context.arc(Math.cos((360 - this.redLineAngle/180) * Math.PI)*150 + this.context.canvas.width/2, Math.sin((360 - this.redLineAngle/180) * Math.PI)*150 + this.context.canvas.height/2, 20, 0, 2*Math.PI);
      this.context.restore();
      if(this.context.isPointInPath(x, y))
      {
        this.activeControl = 'red';
        return;
      }
      this.activeControl = '';
    },
    mouseMove : function (event) {
      event.preventDefault();
      if(!this.activeControl) return;
      //计算角度
      var x = (event.type === 'touchmove' ? event.changedTouches[0].clientX : event.clientX) - this.context.canvas.getBoundingClientRect().left - this.context.canvas.width/2;
      var y = (event.type === 'touchmove' ? event.changedTouches[0].clientY : event.clientY) - this.context.canvas.getBoundingClientRect().top - this.context.canvas.height/2;
      var angle = x === 0 ? 90 : Math.ceil((Math.atan(Math.abs(y)/Math.abs(x)))*180/Math.PI);
      //根据象限计算新角度
      if(x >= 0)
      {
        angle = y <= 0 ? angle : 360 - angle;
      }
      else
      {
        angle = y < 0 ? 180 - angle : 180 + angle;
      }
      var oldCircle, circle,oldQuadrant,newQuadrant;
      switch (this.activeControl) {
        case 'red':
          //吸附效果
          if(Math.abs((this.blueLineAngle + 10 * 360)%360 - angle) < 5) angle = (this.blueLineAngle + 10 * 360)%360;
          //根据象限切换和原圈数，计算新圈数
          oldCircle = circle = Math.floor((this.redLineAngle + 10 * 360)/360) - 10;
          oldQuadrant = Math.floor((this.redLineAngle + 10 * 360)/90)%4;
          newQuadrant = Math.floor(angle/90)%4;
          if(oldQuadrant === 0 && newQuadrant === 3) circle--;
          else if(oldQuadrant === 3 && newQuadrant === 0) circle++;
          this.redLineAngle = Math.max(-1080, Math.min(1080, angle + circle * 360));
          break;
        case 'blue':
          if(Math.abs((this.redLineAngle + 10 * 360)%360 - angle) < 5) angle = (this.redLineAngle + 10 * 360)%360;
          circle = Math.floor((this.blueLineAngle + 10 * 360)/360) - 10;
          oldQuadrant = Math.floor((this.blueLineAngle + 10 * 360)/90)%4;
          newQuadrant = Math.floor(angle/90)%4;
          if(oldQuadrant === 0 && newQuadrant === 3) circle--;
          else if(oldQuadrant === 3 && newQuadrant === 0) circle++;
          this.blueLineAngle = Math.max(-1080, Math.min(1080, angle + circle * 360));
          break;
      }
    },
    mouseUp : function () {
      this.activeControl = '';
    },
    mouseLeave : function () {
      this.activeControl = '';
    },
    drawCanvas : function () {
      this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height);
      this.drawCoordinateSystem();
      this.drawLine();
    },
    drawCoordinateSystem : function () {
      //确定轴长度 = 画布高度和宽度较小值的1/2
      var axisLong = Math.round(Math.min(this.context.canvas.width, this.context.canvas.height)/1.1);
      //确定刻度长度 = 轴长度/10 取整
      var axisAtom = Math.floor((axisLong - 20)/20);
      /**
       * 画x轴
       */
      //line
      this.context.save();
      this.context.beginPath();
      this.context.moveTo((this.context.canvas.width-axisLong)/2, this.context.canvas.height/2);
      this.context.lineTo((this.context.canvas.width+axisLong)/2, this.context.canvas.height/2);
      this.context.lineWidth = 2;
      this.context.strokeStyle = '#000';
      this.context.stroke();
      this.context.restore();
      //arrow
      this.context.save();
      this.context.beginPath();
      this.context.moveTo((this.context.canvas.width+axisLong)/2, this.context.canvas.height/2 - 5);
      this.context.lineTo((this.context.canvas.width+axisLong)/2 + 10, this.context.canvas.height/2);
      this.context.lineTo((this.context.canvas.width+axisLong)/2, this.context.canvas.height/2 + 5);
      this.context.closePath();
      this.context.fillStyle = '#000';
      this.context.fill();
      this.context.restore();
      //number
      this.context.save();
      this.context.strokeStyle = '#000';
      this.context.lineWidth = 2;
      this.context.font = '12px serif';
      this.context.textAlign = 'center';
      this.context.fillStyle = 'rgba(0,0,0,.5)';
      for (var i = -10; i <= 10; i++)
      {
        if(i === 0) continue;
        this.context.beginPath();
        this.context.moveTo(this.context.canvas.width/2 + i * axisAtom, this.context.canvas.height/2);
        this.context.lineTo(this.context.canvas.width/2 + i * axisAtom, this.context.canvas.height/2-5);
        this.context.stroke();
        this.context.fillText(i.toString(), this.context.canvas.width/2 + i * axisAtom, this.context.canvas.height/2 + 12);
      }
      this.context.restore();
      /**
       * 画y轴
       */
      //line
      this.context.save();
      this.context.moveTo(this.context.canvas.width/2, (this.context.canvas.height+axisLong)/2);
      this.context.lineTo(this.context.canvas.width/2, (this.context.canvas.height-axisLong)/2);
      this.context.lineWidth = 2;
      this.context.strokeStyle = '#000';
      this.context.stroke();
      this.context.restore();
      //arrow
      this.context.save();
      this.context.beginPath();
      this.context.moveTo(this.context.canvas.width/2 - 5, (this.context.canvas.height-axisLong)/2);
      this.context.lineTo(this.context.canvas.width/2, (this.context.canvas.height-axisLong)/2 - 10);
      this.context.lineTo(this.context.canvas.width/2 + 5, (this.context.canvas.height-axisLong)/2);
      this.context.closePath();
      this.context.fillStyle = '#000';
      this.context.fill();
      this.context.restore();
      //number
      this.context.save();
      this.context.lineWidth = 2;
      this.context.font = '12px serif';
      this.context.textAlign = 'center';
      this.context.fillStyle = 'rgba(0,0,0,.5)';
      for (i = -10; i <= 10; i++)
      {
        if(i === 0) continue;
        this.context.beginPath();
        this.context.moveTo(this.context.canvas.width/2, this.context.canvas.height/2 + i * axisAtom);
        this.context.lineTo(this.context.canvas.width/2 + 5, this.context.canvas.height/2 + i * axisAtom);
        this.context.stroke();
        this.context.fillText(i.toString(), this.context.canvas.width/2 - 12, this.context.canvas.height/2 + i * axisAtom + 6);
      }
      this.context.restore();
      /**
       * 画圆点
       */
      this.context.save();
      this.context.font = 'italic 16px serif';
      this.context.textAlign = 'center';
      this.context.fillStyle = 'blue';
      this.context.fillText('Ο', this.context.canvas.width/2 - 10, this.context.canvas.height/2 + 14);
      this.context.restore();
    },
    drawLine : function () {
      var i, plus, start = 40;
      //line
      this.context.save();
      this.context.beginPath();
      this.context.moveTo(this.context.canvas.width/2, this.context.canvas.height/2);
      this.context.lineTo(Math.cos((360 - this.blueLineAngle/180) * Math.PI)*100 + this.context.canvas.width/2, Math.sin((360 - this.blueLineAngle/180) * Math.PI)*100 + this.context.canvas.height/2);
      this.context.lineWidth = 2;
      this.context.strokeStyle = 'blue';
      this.context.stroke();
      this.context.restore();
      //control point
      this.context.save();
      this.context.beginPath();
      this.context.shadowBlur = 5;
      this.context.shadowColor = 'blue';
      this.context.arc(Math.cos((360 - this.blueLineAngle/180) * Math.PI)*100 + this.context.canvas.width/2, Math.sin((360 - this.blueLineAngle/180) * Math.PI)*100 + this.context.canvas.height/2, 6, 0, 2*Math.PI);
      this.context.lineWidth = 2;
      this.context.strokeStyle = 'blue';
      this.context.stroke();
      this.context.fillStyle = '#eee';
      this.context.fill();
      this.context.restore();
      //弧线
      this.context.save();
      this.context.beginPath();
      plus = this.blueLineAngle > 0 ? 1 : -1;
      this.context.moveTo(start + this.context.canvas.width/2, this.context.canvas.height/2);
      for(i = 0; i <= Math.abs(this.blueLineAngle); i=i+5)
      {
        this.context.lineTo(Math.cos((360 - i/180) * Math.PI)*(start+i*0.025 * plus) + this.context.canvas.width/2, this.context.canvas.height/2 + Math.sin((360 - i/180) * Math.PI)*(start+i*0.025 * plus) * plus)
      }
      this.context.strokeStyle = 'blue';
      this.context.stroke();
      this.context.restore();
      //arrow
      this.context.save();
      this.context.beginPath();

      this.context.restore();

      //line
      this.context.save();
      this.context.beginPath();
      this.context.moveTo(this.context.canvas.width/2, this.context.canvas.height/2);
      this.context.lineTo(Math.cos((360 - this.redLineAngle/180) * Math.PI)*150 + this.context.canvas.width/2, Math.sin((360 - this.redLineAngle/180) * Math.PI)*150 + this.context.canvas.height/2);
      this.context.lineWidth = 2;
      this.context.strokeStyle = 'red';
      this.context.stroke();
      this.context.restore();
      //control point
      this.context.save();
      this.context.beginPath();
      this.context.shadowBlur = 5;
      this.context.shadowColor = 'red';
      this.context.arc(Math.cos((360 - this.redLineAngle/180) * Math.PI)*150 + this.context.canvas.width/2, Math.sin((360 - this.redLineAngle/180) * Math.PI)*150 + this.context.canvas.height/2, 6, 0, 2*Math.PI);
      this.context.lineWidth = 2;
      this.context.strokeStyle = 'red';
      this.context.stroke();
      this.context.fillStyle = '#eee';
      this.context.fill();
      this.context.restore();
      //弧线
      this.context.save();
      this.context.beginPath();
      start += 3;
      plus = this.redLineAngle > 0 ? 1 : -1;
      this.context.moveTo(start + this.context.canvas.width/2, this.context.canvas.height/2);
      for(i = 0; i <= Math.abs(this.redLineAngle); i=i+5)
      {
        this.context.lineTo(Math.cos((360 - i/180) * Math.PI)*(start+i*0.025*plus) + this.context.canvas.width/2, this.context.canvas.height/2 + Math.sin((360 - i/180) * Math.PI)*(start+i*0.025*plus) * plus)
      }
      this.context.lineWidth = 2;
      this.context.strokeStyle = 'red';
      this.context.stroke();
      this.context.restore();
    }
  },
  watch : {
    blueLineAngle : function () {
      this.drawCanvas();
    },
    redLineAngle : function () {
      this.drawCanvas();
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
  #container.pointer{
    cursor: pointer;
  }
  #container h1{
    position: absolute;
    left: 10px;
    top: 10px;
    margin: 0;
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
  #canvas {
  }
  #control{
    position: absolute;
    bottom : 10px;
    left: 10px;
    box-shadow: 0 0 5px rgba(0,0,0,.3);
    border-radius: 6px;
    padding: 0 10px;
    background-color: #f7f7f7;
  }
  #control p{
  }
  #control p label{

  }
  #control p input{
  }
</style>
