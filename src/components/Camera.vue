<template>
  <div class="container">
    <div class="main col-md-12 non-printable">
      <div class="camera-button mb-1">
        <button
          type="button"
          class="button is-rounded"
          :class="{ 'is-primary': !isCameraOpen, 'is-danger': isCameraOpen }"
          @click="toggleCamera"
        >
          <span v-if="!isCameraOpen">Open Camera</span>
          <span v-else>Close Camera</span>
        </button>
      </div>

      <div v-show="isCameraOpen && isLoading" class="camera-loading">
        <ul class="loader-circle">
          <li></li>
          <li></li>
          <li></li>
        </ul>
      </div>

      <div
        v-if="isCameraOpen"
        v-show="!isLoading"
        class="camera-box"
        :class="{ flash: isShotPhoto }"
      >
        <div class="camera-shutter" :class="{ flash: isShotPhoto }"></div>

        <video
          v-show="!isPhotoTaken"
          ref="camera"
          :width="1000"
          :height="750"
          autoplay
        ></video>

        <canvas
          v-show="isPhotoTaken"
          id="photoTaken"
          ref="canvas"
          :width="1000"
          :height="750"
        ></canvas>
      </div>

      <div v-if="isCameraOpen && !isLoading" class="camera-shoot">
        <button type="button" class="btn btn-sm" @click="takePhoto">
          <img src="@/assets/camera--v2.png" />
        </button>
      </div>

      <div v-if="isPhotoTaken && isCameraOpen" class="camera-save">
        <button
          id="savePhoto"
          save="my-photo.jpg"
          class=""
          role="button"
          @click="saveImage"
        >
          Save
        </button>
      </div>
    </div>

    <div class="sidebar col-md-12 printable">
      <h3 class="h3 non-printable pt-5">Gallery</h3>
      <button @click="downloadGallery" class="non-printable">Download</button>
      <div class="gallery">
        <img v-for="snap in snaps" :src="snap" :key="snap" class="img" />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Home",
  data: function() {
    return {
      isCameraOpen: false,
      isPhotoTaken: false,
      isShotPhoto: false,
      isLoading: false,
      snaps: [],
    };
  },
  mounted: function() {
    this.toggleCamera();
  },
  components: {},
  methods: {
    toggleCamera() {
      if (this.isCameraOpen) {
        this.isCameraOpen = false;
        this.isPhotoTaken = false;
        this.isShotPhoto = false;
        this.stopCameraStream();
      } else {
        this.isCameraOpen = true;
        this.createCameraElement();
      }
    },

    createCameraElement() {
      this.isLoading = true;

      const constraints = (window.constraints = {
        audio: false,
        video: true,
      });

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((stream) => {
          this.isLoading = false;
          this.$refs.camera.srcObject = stream;
        })
        .catch((error) => {
          this.isLoading = false;
          console.log(error);
          alert("May the browser didn't support or there is some errors.");
        });
    },

    stopCameraStream() {
      let tracks = this.$refs.camera.srcObject.getTracks();

      tracks.forEach((track) => {
        track.stop();
      });
    },

    takePhoto() {
      if (!this.isPhotoTaken) {
        this.isShotPhoto = true;

        const FLASH_TIMEOUT = 50;

        setTimeout(() => {
          this.isShotPhoto = false;
        }, FLASH_TIMEOUT);
      }

      this.isPhotoTaken = !this.isPhotoTaken;

      const context = this.$refs.canvas.getContext("2d");
      context.drawImage(this.$refs.camera, 0, 0, 1000, 750);
    },

    saveImage() {
      const canvas = document
        .getElementById("photoTaken")
        .toDataURL("image/jpeg")
        .replace("image/jpeg", "image/octet-stream");
      this.snaps.unshift(canvas);
    },

    downloadGallery() {
      console.log("printing the page");
      window.print();
    },
  },
};
</script>

<style scoped>
body {
  display: flex;
  justify-content: center;
}

.web-camera-container {
  margin-top: 2rem;
  margin-bottom: 2rem;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 500px;
}
.web-camera-container .camera-button {
  margin-bottom: 2rem;
}
.web-camera-container .camera-box .camera-shutter {
  opacity: 0;
  width: 450px;
  height: 337.5px;
  background-color: #fff;
  position: absolute;
}
.web-camera-container .camera-box .camera-shutter.flash {
  opacity: 1;
}
.web-camera-container .camera-shoot {
  margin: 1rem 0;
}
.web-camera-container .camera-shoot button {
  height: 60px;
  width: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 100%;
}
.web-camera-container .camera-shoot button img {
  height: 35px;
  object-fit: cover;
}
.web-camera-container .camera-loading {
  overflow: hidden;
  height: 100%;
  position: absolute;
  width: 100%;
  min-height: 150px;
  margin: 3rem 0 0 -1.2rem;
}
.web-camera-container .camera-loading ul {
  height: 100%;
  position: absolute;
  width: 100%;
  z-index: 999999;
  margin: 0;
}
.web-camera-container .camera-loading .loader-circle {
  display: block;
  height: 14px;
  margin: 0 auto;
  top: 50%;
  left: 100%;
  transform: translateY(-50%);
  transform: translateX(-50%);
  position: absolute;
  width: 100%;
  padding: 0;
}
.web-camera-container .camera-loading .loader-circle li {
  display: block;
  float: left;
  width: 10px;
  height: 10px;
  line-height: 10px;
  padding: 0;
  position: relative;
  margin: 0 0 0 4px;
  background: #999;
  animation: preload 1s infinite;
  top: -50%;
  border-radius: 100%;
}
.web-camera-container .camera-loading .loader-circle li:nth-child(2) {
  animation-delay: 0.2s;
}
.web-camera-container .camera-loading .loader-circle li:nth-child(3) {
  animation-delay: 0.4s;
}

.sidebar .img {
  margin: 5px;
}
@keyframes preload {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.4;
  }
  100% {
    opacity: 1;
  }
}
@media print {
  body .non-printable {
    display: none;
  }

  body .printable {
    display: block;
  }
}
</style>
