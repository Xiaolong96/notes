<template>
  <div class="map-container">
    <header>
      <span class="header-info">
        {{ date }}
      </span>
      <span class="header-info">
        {{ time }}
      </span>
    </header>
    <div class="canvas-wrap">
      <canvas id="myCanvas">
        浏览器不支持Canvas,请升级或改用其它浏览器！
      </canvas>
    </div>
    <footer>
      <!-- <div class="footer-info">
        已清扫面积：
        <span>{{ totalCleaningArea }}㎡</span>
      </div> -->
      <div class="footer-info">
        已清扫时间：
        <span>{{ totalCleaningTime }}分钟</span>
      </div>
    </footer>
  </div>
</template>
<script>
import deepClone from '../../utils/index';

export default {
  name: 'Map',
  props: {
    totalCleaningArea: {
      type: Number,
      default: 0,
    },
    totalCleaningTime: {
      type: Number,
      default: 0,
    },
    points: {
      type: Array,
      default: () => [],
    },
    addPoints: {
      type: Array,
      default: () => [],
    },
    date: {
      type: String,
      default: '--月--日',
    },
    time: {
      type: String,
      default: '--:--',
    },
  },
  data() {
    return {
      myCanvas: null,
      ctx: null,
      count: 1,
      allPoints: [],
    };
  },
  computed: {
  },
  watch: {
    points: {
      handler(val) {
        if (val.length !== 0) {
          this.allPoints = val;
          this.ctx.lineWidth = 2;
          this.ctx.strokeStyle = 'rgba(255,255,255,0.9)';
          this.drawLine(this.getPoints(val), this.ctx);
        }
      },
      deep: true,
    },
    addPoints: {
      handler(val) {
        if (val.length === 0) {
          this.$emit('finishDrawing');
          return;
        }
        if (val.length === 1 && this.allPoints.length === 0) {
          this.allPoints = this.allPoints.concat(val);
          this.$emit('finishDrawing');
          return;
        }
        this.ctx.lineWidth = 2;
        this.ctx.strokeStyle = 'rgba(255,255,255,0.9)';
        this.count = 1;
        const points = this.calcWaypoints(this.getAddPoints(this.allPoints, val));
        this.drawLineWithAnimation(points, this.ctx);
      },
      deep: true,
    },
  },
  mounted() {
    if (this.points.length !== 0) {
      this.allPoints = this.points;
    }
    this.initCanvas();
  },
  methods: {
    initCanvas() {
      this.myCanvas = document.querySelector('#myCanvas');
      const ctx = this.myCanvas.getContext('2d');
      this.ctx = ctx;
      const rem = parseInt(document.querySelector('html').style.fontSize, 10);
      this.myCanvas.width = document.documentElement.clientWidth - 0.1 * rem;
      this.myCanvas.height = document.documentElement.clientHeight - 1.6 * rem;
      this.drawDot(ctx);
      if (this.points.length || this.addPoints.length) {
        console.log(this.points.length, 'this.points.length');
        console.log(this.addPoints, 'this.addPoints.length');
        this.drawTrack(ctx);
      }
    },
    drawDot(context) {
      const ctx = context;
      const rowSpacing = this.myCanvas.width / 25;
      const columnSpacing = this.myCanvas.height / 30;
      for (let i = 0; i < 26; i += 1) {
        for (let j = 0; j < 31; j += 1) {
          ctx.beginPath();
          ctx.fillStyle = 'rgba(255,255,255,0.9)';
          ctx.arc(rowSpacing * i, columnSpacing * j, 0.5, 0, 2 * Math.PI);
          ctx.closePath();
          ctx.fill();
        }
      }
    },
    drawTrack(context) {
      const ctx = context;
      let points = [];
      ctx.lineWidth = 2;
      ctx.strokeStyle = 'rgba(255,255,255,0.9)';
      if (this.points.length !== 0) {
        this.drawLine(this.getPoints(this.points), ctx);
        // if (this.addPoints.length !== 0) {
        //   points = this.calcWaypoints(this.getAddPoints(this.points, this.addPoints));
        // }
      } else {
        points = this.calcWaypoints(this.getAddPoints(this.allPoints, this.addPoints));
      }
      this.drawLineWithAnimation(points, ctx);
    },
    getPoints(points) {
      const res = [];
      for (let i = 0, l = points.length; i < l; i += 1) {
        const item = {};
        item.x = points[i].x + 13;
        item.y = points[i].y + 15;
        res.push(item);
      }
      console.log(res);
      return res;
    },
    getAddPoints(allPoints, addPoints) {
      const addPointsCp = deepClone(addPoints);
      const len = allPoints.length;
      if (len !== 0) {
        addPointsCp.unshift(allPoints[len - 1]);
      }
      const res = [];
      for (let i = 0, l = addPointsCp.length; i < l; i += 1) {
        const item = {};
        item.x = addPointsCp[i].x + 13;
        item.y = addPointsCp[i].y + 15;
        res.push(item);
      }
      this.allPoints = this.allPoints.concat(allPoints).concat(addPoints);
      return res;
    },
    calcWaypoints(points) {
      const waypoints = [];
      const rowSpacing = this.myCanvas.width / 25;
      const columnSpacing = this.myCanvas.height / 30;
      for (let i = 1, len = points.length; i < len; i += 1) {
        const pt0 = points[i - 1];
        const pt1 = points[i];
        const dx = pt1.x - pt0.x;
        const dy = pt1.y - pt0.y;
        for (let j = 0; j < 100; j += 1) {
          const x = (pt0.x + dx * j / 100) * rowSpacing;
          const y = (pt0.y + dy * j / 100) * columnSpacing;
          waypoints.push({
            x,
            y,
          });
        }
      }
      return waypoints;
    },
    drawLine(points, ctx) {
      if (points.length === 0) {
        return;
      }
      const rowSpacing = this.myCanvas.width / 25;
      const columnSpacing = this.myCanvas.height / 30;
      ctx.beginPath();
      ctx.moveTo(points[0].x * rowSpacing, points[0].y * columnSpacing);
      for (let i = 1, len = points.length; i < len; i += 1) {
        ctx.lineTo(points[i].x * rowSpacing, points[i].y * columnSpacing);
      }
      ctx.stroke();
      this.$emit('finishDrawing');
    },
    drawLineWithAnimation(points, ctx) {
      if (this.count < points.length - 1) {
        requestAnimationFrame(() => {
          this.drawLineWithAnimation(points, ctx);
        });
      } else {
        window.setTimeout(() => {
          this.$emit('finishDrawing');
        }, 0);
      }
      ctx.beginPath();
      ctx.moveTo(points[this.count - 1].x, points[this.count - 1].y);
      ctx.lineTo(points[this.count].x, points[this.count].y);
      ctx.stroke();
      this.count += 1;
    },
    clearCanvas(ctx) {
      ctx.clearRect(0, 0, this.myCanvas.width, this.myCanvas.height);
    },
  },
};
</script>

<style lang="scss" scoped>
 .map-container{
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
  height: auto;
  header {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: .9rem;
    width: 100%;
    background-image: linear-gradient(0deg, #4EBBF5 0%, #3767EC 100%);
    .header-info {
      color: #fff;
      font-size: .15rem;
      &:first-child{
        margin-bottom: .05rem;
      }
    }
  }
  footer {
    position: absolute;
    bottom: 0;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: .6rem;
    width: 100%;
    background-image: linear-gradient(0deg, #3767EC 0%, #4EBBF5 100%);
    .footer-info {
      color: #fff;
      font-size: .15rem;
      &:first-child{
        margin-right: .1rem;
      }
    }
  }
  .canvas-wrap {
    position: absolute;
    top: .9rem;
    bottom: .6rem;
    width: 97.3%;
    padding: .05rem;
    background: #387bf6;
    // #myCanvas {
    //   // background: #fff;
    // }
  }
}
</style>
