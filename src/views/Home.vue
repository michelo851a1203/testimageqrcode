<template>
  <div class="container mx-auto">
    <button
      @click="cameraClick"
      class="bg-blue-500 focus:outline-none hover:bg-blue-700 text-white font-medium py-2 px-4 rounded"
    >test scan</button>
    <span>{{ oData }}</span>
    <video class="w-full h-full" ref="videoRef"></video>
    <input
      ref="inputRef"
      class="hidden"
      type="file"
      @change="getCameraChange"
      accept="image/*;capture=camera"
    />
  </div>
</template>

<script>
// @ is an alias to /src

import { ref } from "vue";
import { BrowserQRCodeReader } from "@zxing/library";
export default {
  name: "Home",
  setup() {
    const videoRef = ref(null);
    const inputRef = ref(null);
    const codeReader = new BrowserQRCodeReader();
    const oData = ref("");

    const cameraClick = () => {
      inputRef.value.click();
    };

    // fileReader to Video
    const filereaderToVideo = (file) =>
      new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsArrayBuffer(file);
        reader.onload = (event) => {
          const buffer = event.target.result;
          const videoBlob = new Blob([new Uint8Array(buffer)], {
            type: "video/mp4",
          });
          const url = window.URL.createObjectURL(videoBlob);
          videoRef.value.src = url;
          resolve(videoRef.value);
        };
        reader.onerror = () => {
          reject(`fileReader error`);
        };
      });

    // get file to
    const getCameraChange = async (event) => {
      const mainFileList = event.target.files;
      if (mainFileList.length === 0 || inputRef.value === "") return;
      filereaderToVideo(mainFileList[0])
        .then((video) => {
          return codeReader.decodeFromVideo(video);
        })
        .then((qrcodeObject) => {
          if (!qrcodeObject || qrcodeObject.text === "") return;
          oData.value = qrcodeObject.text;
          if (oData.value !== "") {
            inputRef.value = "";
            videoRef.value.src = null;
            codeReader.reset();
          }
        });
    };

    return { videoRef, inputRef, oData, cameraClick, getCameraChange };
  },
};
</script>


