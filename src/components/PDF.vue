<template>
  <div>
    <canvas
      v-bind:width="width"
      v-bind:height="height"
      class="vertical"
      ref="pdfcanvas"
      @click="stampTest"
    ></canvas>
    <br />
    <button @click="clear()">清 空</button>
    <button @click="change()">切 换</button>
    <button @click="stamp()">盖 章</button>
  </div>
</template>

<script>
export default {
  name: "PdfView",
  data() {
    return {
      isVertical: false,
      width: 842,
      height: 595,
      img: "",
      backEndPoint: {
        x: 0,
        y: 0,
        isVertical:false,
      },
    };
  },
  methods: {
    stamp() {
      this.backEndPoint.isVertical = this.isVertical;
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
          const cnv = this.$refs.pdfcanvas;
          const cxt = cnv.getContext("2d");
          const image = new Image();
          image.src = data.data;
          image.onload = () => {
            cxt.drawImage(image, 0, 0);
          };
        });
    },
    stampTest(mouse) {
      let r = 30;
      //减3是因为设置了border为3px
      let x = Number(mouse.clientX) - 3;
      let y = Number(mouse.clientY) - 3;
      //防止xy的值为0
      x = x < 0 ? 0 : x;
      y = y < 0 ? 0 : y;
      console.log("前端点击位置" + x + "," + y);
      //在itext中，坐标需要进行转换
      this.backEndPoint.x = x + r * Math.cos((135 * Math.PI) / 180);
      this.backEndPoint.y = y + r * Math.cos((135 * Math.PI) / 180);
      this.backEndPoint.x = this.backEndPoint.x < 0 ? 0 : this.backEndPoint.x;
      this.backEndPoint.y = this.backEndPoint.y < 0 ? 0 : this.backEndPoint.y;
      //因w3c坐标系和itext坐标系不同，因此需要再次转换
      this.backEndPoint.y = Math.abs(this.height - this.backEndPoint.y);
      console.log("圆的左上坐标为：" + this.backEndPoint.x + "," + this.backEndPoint.y);
      this.img = localStorage.getItem("img");
      //获取canvas实例
      const cnv = this.$refs.pdfcanvas;
      const cxt = cnv.getContext("2d");
      //画圆模拟盖章
      cxt.beginPath();
      cxt.arc(x, y, r, 0, (360 * Math.PI) / 180);
      cxt.closePath();
      cxt.stroke();
    },
    clear() {
      let url = "http://localhost:9000/";

      if (this.isVertical) {
        url = url + "vertical";
      } else {
        url = url + "thwartwise";
      }
      console.log(url);
      fetch(url)
        .then((response) => response.json())
        .then((data) => {
          console.log(data);
          //填充图片
          const cnv = this.$refs.pdfcanvas;
          const cxt = cnv.getContext("2d");
          const image = new Image();
          image.src = data.data;
          image.onload = () => {
            cxt.drawImage(image, 0, 0);
          };
        });
    },
    change() {
      if (this.isVertical) {
        this.width = 842;
        this.height = 595;
        this.isVertical = false;
        this.clear();
      } else {
        this.width = 595;
        this.height = 842;
        this.isVertical = true;
        this.clear();
      }
    },
  },
  mounted() {
    this.clear();
  },
};
</script>

<style scoped>
.vertical {
  border: 3px solid #ccc;
}
</style>
