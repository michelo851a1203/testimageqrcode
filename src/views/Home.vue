<template>
  <div class="container mx-auto">
    <button
      @click="cameraClick"
      class="bg-blue-500 focus:outline-none hover:bg-blue-700 text-white font-medium py-2 px-4 rounded"
    >test scan</button>
    <p>result : {{ oData }}</p>
    <p>is error : {{ errorRef }}</p>
    <img :src="imageSrc" ref="imageRef" alt />
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
    const imageRef = ref(null);
    const imageSrc = ref("");
    const errorRef = ref("");
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
          errorRef.value = "fileReader error";
          reject(`fileReader error`);
        };
      });

    const fileToPreview = (file) =>
      new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => {
          const getReader = reader.result;
          if (!getReader || getReader === "") {
            errorRef.value = "getReader error";
            reject("getReader error");
            return;
          }
          errorRef.value = `imageSrc get ${getReader}`;
          imageSrc.value = getReader;
          resolve(imageRef.value);
        };
        reader.onerror = () => reject("onload fail");
      });
    // get file to
    const getCameraChange = async (event) => {
      const mainFileList = event.target.files;
      if (mainFileList.length === 0 || inputRef.value === "") return;

      const typeBoolean =
        mainFileList[0].type === "image/jpg" ||
        mainFileList[0].type === "image/jpeg" ||
        mainFileList[0].type === "image/png";
      errorRef.value = `${typeBoolean}`;
      if (typeBoolean) {
        fileToPreview(mainFileList[0])
          .then((imgElement) => {
            errorRef.value = "imgElement is here";
            return codeReader.decodeFromImage(imgElement);
          })
          .then((decodeData) => {
            oData.value = decodeData.text;
            inputRef.value = "";
          })
          .catch((err) => {
            inputRef.value = "";
            errorRef.value = `image error error ${err}`;
            console.error("image error :" + err);
          });
        return;
      }

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
        })
        .catch((err) => {
          inputRef.value = "";
          videoRef.value.src = null;
          codeReader.reset();
          errorRef.value = `getCameraChange error ${err}`;
          console.log("getCameraChange error", err);
        });
    };

    return {
      videoRef,
      inputRef,
      imageRef,
      imageSrc,
      oData,
      errorRef,
      cameraClick,
      getCameraChange,
    };
  },
};
</script>


