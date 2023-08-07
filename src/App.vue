<script setup lang="ts">
import { ref, computed } from "vue";
import DrawTool from "@/components/DrawTool/DrawTool.vue";
const drawTool = ref();
console.log(drawTool);
const area = computed(() => {
  return drawTool.value?.polygons;
});
</script>

<template>
  <DrawTool ref="drawTool" />
  <div>{{ area }}</div>
</template>
<style scoped>
body {
  margin: 0;
}
</style>
