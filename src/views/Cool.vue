<template>
  <div>
    <input @change="mainChange" type="file" ref="mainRef" />
  </div>
</template>

<script>
export default {
  name: "Cool",
  setup() {
    const mainChange = (event) => {
      const file = event.target.files[0];
      console.log(file);
    };
    return { mainChange };
  },
};
</script>

<style scoped lang="postcss">
</style>