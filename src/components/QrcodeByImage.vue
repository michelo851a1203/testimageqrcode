<template>
  <div>
    <img ref="imageRef" :src="mainSrc" />
    {{ oResult }}
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { BrowserQRCodeReader } from "@zxing/library";
export default {
  name: "qrcodebyimage",
  setup() {
    const imageRef = ref(null);
    const img = require("@/assets/01.png");
    const mainSrc = ref(img);
    const codeReader = new BrowserQRCodeReader();
    const oResult = ref("");

    onMounted(() => {
      if (!imageRef.value.src || imageRef.value.src === "") return;
      const imageElement = imageRef.value;
      codeReader
        .decodeFromImage(imageElement)
        .then((result) => {
          oResult.value = result.text;
        })
        .catch((err) => {
          console.error(`image qrcode error ${err}`);
        });
    });
    return { imageRef, mainSrc, oResult };
  },
};
</script>

<style scoped lang="postcss">
</style>