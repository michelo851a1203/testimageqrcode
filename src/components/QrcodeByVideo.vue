<template>
  <div>
    <video ref="videoRef"></video>
  </div>
</template>


<script>
// TODO : add error handler solution.
import { ref, onMounted } from "vue";
import { BrowserQRCodeReader } from "@zxing/library";
export default {
  name: "qrcodebyvideo",
  setup() {
    const videoRef = ref(null);
    const codereader = new BrowserQRCodeReader();
    onMounted(() => {
      const video = videoRef.value;
      // ============================================================================
      video.src = "QrcodeSrc";
      // ============================================================================
      codeReader.decodeFromVideo(video).then((qrcodeObject) => {
        if (!qrcodeObject || qrcodeObject.text === "") {
          return;
        }
        console.group(`%c qrcodeObject.text`, "color:yellow");
        console.log(qrcodeObject.text);
        console.groupEnd();
      });
    });

    return { videoRef };
  },
};
</script>

<style scoped lang="postcss">
</style>