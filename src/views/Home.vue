<template>
  <div class="container mx-auto">
    <button
      @click="cameraClick"
      class="bg-blue-500 focus:outline-none hover:bg-blue-700 text-white font-medium py-2 px-4 rounded"
    >test scan</button>
    <p>============================================</p>
    <p>step1:{{ s1 }}</p>
    <p>============================================</p>
    <p>step2:{{ s2 }}</p>
    <p>============================================</p>
    <p>step3:{{ s3 }}</p>
    <p>============================================</p>
    <br />
    <span>result : {{ oData }}</span>
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
import { ref, reactive, toRefs } from "vue";
import { BrowserQRCodeReader } from "@zxing/library";
export default {
  name: "Home",
  setup() {
    const videoRef = ref(null);
    const inputRef = ref(null);
    const codeReader = new BrowserQRCodeReader();
    const oData = ref("");
    const tmpUrl = ref("");

    const step = reactive({
      s1: "",
      s2: "",
      s3: "",
    });

    const cameraClick = () => {
      step.s1 = "";
      step.s2 = "";
      step.s3 = "";
      inputRef.value.click();
    };

    // fileReader to Video
    const filereaderToVideo = (file) =>
      new Promise((resolve, reject) => {
        step.s2 = "fileReader start ... ";
        const reader = new FileReader();
        reader.readAsArrayBuffer(file);
        reader.onload = (event) => {
          step.s2 = "fileReader onload ... ";
          const buffer = event.target.result;
          const videoBlob = new Blob([new Uint8Array(buffer)], {
            type: "video/mp4",
          });
          if (videoBlob) {
            step.s2 += ` (videoBlob OK : ${typeof videoBlob} ) `;
          }
          tmpUrl.value = window.URL.createObjectURL(videoBlob);
          if (tmpUrl.value) {
            step.s2 += ` (url OK : ${typeof tmpUrl.value} ) `;
          }
          videoRef.value.src = tmpUrl.value;
          // window.URL.revokeObjectURL(url);
          resolve(videoRef.value);
        };
        reader.onerror = () => {
          step.s2 = "fileReader error ... ";
          reject(`fileReader error`);
        };
      });

    // get file to
    const getCameraChange = async (event) => {
      step.s1 = "getCameraChange start ... ";
      const mainFileList = event.target.files;
      if (mainFileList.length === 0 || inputRef.value === "") return;
      filereaderToVideo(mainFileList[0])
        .then((video) => {
          step.s3 = "getvideo ... ";
          return codeReader.decodeFromVideo(video);
        })
        .then((qrcodeObject) => {
          step.s3 = "getvideo qrcodeObject ... ";
          if (!qrcodeObject || qrcodeObject.text === "") return;
          oData.value = qrcodeObject.text;
          step.s3 = "qrcodeObject OK ... ";
          if (oData.value !== "") {
            inputRef.value = "";
            videoRef.value.src = null;
            codeReader.reset();
            // if (tmpUrl.value !== "") {
            //   window.URL.revokeObjectURL(tmpUrl.value);
            // }
            step.s3 = "qrcodeObject OK ... (reset all) ";
          }
        })
        .catch((err) => {
          step.s3 = "qrcodeObject Error ... ";
          if (tmpUrl.value !== "") {
            window.URL.revokeObjectURL(tmpUrl.value);
          }
          inputRef.value = "";
          videoRef.value.src = null;
          codeReader.reset();
          console.log("getCameraChange error", err);
        });
    };

    return {
      videoRef,
      inputRef,
      oData,
      cameraClick,
      getCameraChange,
      tmpUrl,
      ...toRefs(step),
    };
  },
};
</script>


