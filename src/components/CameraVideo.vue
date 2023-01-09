<template>
  <!--  原生摄像头-->
  <div>
    <div class="box">
      <video
        id="videoCamera"
        class="canvas"
        :width="videoWidth"
        :height="videoHeight"
        autoPlay
      ></video>
      <canvas
        id="canvasCamera"
        class="canvas"
        :width="videoWidth"
        :height="videoHeight"
      ></canvas>
    </div>
    <div>
      <el-button @click="photographBtn" icon="el-icon-camera" size="small"
        >拍照</el-button
      >
      <el-button
        v-if="os"
        @click="openCamera"
        icon="el-icon-video-camera"
        size="small"
        >打开摄像头</el-button
      >
      <el-button
        v-else
        @click="stopCamera"
        icon="el-icon-switch-button"
        size="small"
        >关闭摄像头</el-button
      >
      <el-button @click="resetBtn" icon="el-icon-refresh" size="small"
        >重置</el-button
      >
      <el-button @click="submitBtn" icon="el-icon-circle-close" size="small"
        >提交</el-button
      >
    </div>
  </div>
</template>

<script>
export default {
  name: "cameraVideo",
  data() {
    return {
      os: false, //控制摄像头开关
      thisVideo: null,
      thisContext: null,
      thisCancas: null,
      imgSrc: "",
      videoWidth: 500,
      videoHeight: 400,
    };
  },
  created() {
    this.openCamera();
  },
  methods: {
    submitBtn() {
      console.log(this.imgSrc);
    },
    // 调用摄像头权限
    openCamera() {
      //必须在model中render后才可获取到dom节点,直接获取无法获取到model中的dom节点
      this.$nextTick(() => {
        const _this = this;
        this.os = false; //切换成关闭摄像头
        this.thisCancas = document.getElementById("canvasCamera");
        this.thisContext = this.thisCancas.getContext("2d");
        this.thisVideo = document.getElementById("videoCamera");
        // 旧版本浏览器可能根本不支持mediaDevices，我们首先设置一个空对象
        if (navigator.mediaDevices === undefined) {
          navigator.mediaDevices = {};
        }
        // 一些浏览器实现了部分mediaDevices，我们不能只分配一个对象
        // 使用getUserMedia，因为它会覆盖现有的属性。
        // 这里，如果缺少getUserMedia属性，就添加它。
        if (navigator.mediaDevices.getUserMedia === undefined) {
          navigator.mediaDevices.getUserMedia = function (constraints) {
            // 首先获取现存的getUserMedia(如果存在)
            let getUserMedia =
              navigator.webkitGetUserMedia ||
              navigator.mozGetUserMedia ||
              navigator.getUserMedia;
            // 有些浏览器不支持，会返回错误信息
            // 保持接口一致
            if (!getUserMedia) {
              return Promise.reject(
                new Error("getUserMedia is not implemented in this browser")
              );
            }
            // 否则，使用Promise将调用包装到旧的navigator.getUserMedia
            return new Promise(function (resolve, reject) {
              getUserMedia.call(navigator, constraints, resolve, reject);
            });
          };
        }
        const constraints = {
          audio: false,
          video: {
            width: _this.videoWidth,
            height: _this.videoHeight,
            transform: "scaleX(-1)",
          },
        };
        navigator.mediaDevices
          .getUserMedia(constraints)
          .then(function (stream) {
            // 旧的浏览器可能没有srcObject
            if ("srcObject" in _this.thisVideo) {
              _this.thisVideo.srcObject = stream;
            } else {
              // 避免在新的浏览器中使用它，因为它正在被弃用。
              _this.thisVideo.src = window.URL.createObjectURL(stream);
            }
            _this.thisVideo.onloadedmetadata = function () {
              _this.thisVideo.play();
            };
          })
          .catch((err) => {
            console.log(err);
            this.$message({
              message: "没有开启摄像头权限或浏览器版本不兼容",
              type: "warning",
            });
          });
      });
    },
    //拍照按钮
    photographBtn() {
      // 点击，canvas画图
      this.thisContext.drawImage(
        this.thisVideo,
        0,
        0,
        this.videoWidth,
        this.videoHeight
      );
      // 获取图片base64链接 >>> 这里是base64格式的图片
      this.imgSrc = this.thisCancas.toDataURL("image/png");
    },
    //清空画布
    clearCanvas(id) {
      let c = document.getElementById(id);
      let cxt = c.getContext("2d");
      cxt.clearRect(0, 0, c.width, c.height);
    },
    //重置画布
    resetBtn() {
      this.imgSrc = "";
      this.clearCanvas("canvasCamera");
    },
    //关闭摄像头
    stopCamera() {
      if (this.thisVideo && this.thisVideo !== null) {
        this.thisVideo.srcObject.getTracks()[0].stop();
        this.os = true; //切换成打开摄像头
      }
    },
  },
};
</script>
<style scoped>
#canvasCamera{
  margin-left: 20px;
}
</style>