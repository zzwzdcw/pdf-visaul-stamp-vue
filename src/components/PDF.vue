<template>
  <div>
    <h2>调用后端盖打</h2>
    <canvas
      v-bind:width="width"
      v-bind:height="height"
      class="vertical"
      ref="pdfcanvas"
      @click="stampTest"
    ></canvas>
    <br />
    <button @click="back()">上一页</button>
    <button @click="next()">下一页</button>
    <button @click="clear()">清 空</button>
    <button @click="change()">切 换</button>
    <button @click="stamp()">盖 章</button>
    <button @click="sign()">签 字</button>

  </div>
</template>

<script>
export default {
  name: "PdfView",
  data() {
    return {
      nowPage: 1,
      PageTotal: 1,
      pdfList: [{}],
      pdfListBack: [{}],
      isVertical: false,
      width: 842,
      height: 595,
      point: {
        x: 0,
        y: 0,
      },
      backEndPoint: {
        pointVoList: [{}],
        vertical: false,
      },
    };
  },
  methods: {
    //请求后端盖章的代码
    back() {
      if (this.nowPage > 1) {
        this.getCanvasBase64();
        this.nowPage = this.nowPage - 1;
        this.fullCanvas();
      } else {
        alert("已经是第一页");
      }
    },
    next() {
      if (this.nowPage < this.pdfList.length) {
        this.getCanvasBase64();
        this.nowPage = this.nowPage + 1;
        this.fullCanvas();
      } else {
        alert("已经是最后一页");
      }
    },
    fullCanvas() {
      //填充图片
      const cnv = this.$refs.pdfcanvas;
      const cxt = cnv.getContext("2d");
      const image = new Image();
      console.log(this.pdfList);
      image.src = this.pdfList[this.nowPage - 1];
      image.onload = () => {
        cxt.drawImage(image, 0, 0);
      };
    },
    getCanvasBase64() {
      const cnv = this.$refs.pdfcanvas;
      this.pdfList[this.nowPage - 1] = cnv.toDataURL("image/png");
    },
    clear() {
      this.backEndPoint.pointVoList = [];
      this.pdfList = JSON.parse(sessionStorage.getItem("pdfList"));
      this.fullCanvas();
    },
    signTest(mouse){

      //圆的半径，根据印章的大小来配置
      let r = 50;
      //减3是因为设置了border为3px
      let x = Number(mouse.offsetX) - 3;
      let y = Number(mouse.offsetY) - 3;
      //在itext中，坐标需要进行转换
      this.point.x = x - r;
      this.point.y = y + r;
      //获取canvas实例
      const cnv = this.$refs.pdfcanvas;
      const cxt = cnv.getContext("2d");
      //画圆模拟盖章
      cxt.beginPath();
      cxt.arc(x, y, r, 0, (360 * Math.PI) / 180);
      cxt.closePath();
      cxt.stroke();
      //因w3c坐标系和itext坐标系不同，因此需要再次转换
      this.point.y = this.height - this.point.y;
      console.log("圆的左下坐标为：" + this.point.x + "," + this.point.y);
      this.backEndPoint.pointVoList.push({
        x: this.point.x,
        y: this.point.y,
        page: this.nowPage,
      });
    },
    sign(){
      this.backEndPoint.vertical = this.isVertical;
      fetch("http://localhost:9000/sign", {
        method: "post",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(this.backEndPoint),
      })
        .then((response) => response.json())
        .then((data) => {
          //填充图片
          this.pdfList = data.data;
          this.fullCanvas();
        });
    },
    stamp() {
      this.backEndPoint.vertical = this.isVertical;
      fetch("http://localhost:9000/stamp", {
        method: "post",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(this.backEndPoint),
      })
        .then((response) => response.json())
        .then((data) => {
          //填充图片
          this.pdfList = data.data;
          this.fullCanvas();
        });
    },
    //模拟签名
    stampTest(mouse) {
      //减3是因为设置了border为3px
      let x = Number(mouse.offsetX) - 3;
      let y = Number(mouse.offsetY) - 3;
      //在itext中，坐标需要进行转换
      let x1 = x - 50;
      let x2 = x + 50;
      let y1 = y - 15;
      let y2 = y + 15;

      this.point.x = x - 50;
      this.point.y = y + 15;
      //获取canvas实例
      const cnv = this.$refs.pdfcanvas;
      const cxt = cnv.getContext("2d");
      //画长方形模拟签字
      cxt.beginPath();
      cxt.moveTo(x1,y1);
      cxt.lineTo(x2,y1);
      cxt.lineTo(x2,y2);
      cxt.lineTo(x1,y2);
      cxt.lineTo(x1,y1);
      cxt.stroke();
      //因w3c坐标系和itext坐标系不同，因此需要再次转换
      this.point.y = this.height - this.point.y;
      console.log("长方形的左下坐标为：" + this.point.x + "," + this.point.y);
      this.backEndPoint.pointVoList.push({
        x: this.point.x,
        y: this.point.y,
        page: this.nowPage,
      });
    },
    //获取pdf
    getPDf() {
      this.backEndPoint.pointVoList = [];
      let url = "http://localhost:9000/";
      if (this.isVertical) {
        url = url + "vertical";
      } else {
        url = url + "thwartwise";
      }
      fetch(url)
        .then((response) => response.json())
        .then((data) => {
          this.pdfList = data.data;
          sessionStorage.setItem("pdfList", JSON.stringify(this.pdfList));
          this.fullCanvas();
        });
    },
    //切换长宽
    change() {
      if (this.isVertical) {
        this.width = 842;
        this.height = 595;
        this.isVertical = false;
        this.getPDf();
      } else {
        this.width = 595;
        this.height = 842;
        this.isVertical = true;
        this.getPDf();
      }
    },
  },
  mounted() {
    this.getPDf();
  },
};
</script>

<style scoped>
.vertical {
  border: 3px solid #ccc;
}
</style>
