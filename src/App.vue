<script setup lang="ts">
import { ref, computed, onMounted, reactive, nextTick } from "vue";
import DrawTool from "@/components/DrawTool/DrawTool.vue";
const drawTool = ref();
console.log(drawTool);
const area = computed(() => {
  return drawTool.value?.polygons;
});
let drawToolFunc = reactive({
  drawClick: () => {},
  showDrawings: false,
  clearCanvas: () => {},
  toggleDrawingsVisibility: () => {},
  savePolygon: () => {},
});
const setDrawToolFunc = () => {
  nextTick(() => {
    if (drawTool.value) {
      const {
        drawClick,
        showDrawings,
        clearCanvas,
        toggleDrawingsVisibility,
        savePolygon,
      } = drawTool.value;
      drawToolFunc = Object.assign(drawToolFunc, {
        drawClick,
        showDrawings,
        clearCanvas,
        toggleDrawingsVisibility,
        savePolygon,
      });
    }
  });
};
onMounted(() => {
  setDrawToolFunc();
});
</script>

<template>
  <div class="tools">
    <el-row>
      <el-button @click="drawToolFunc.drawClick" type="primary" link>
        绘制</el-button
      >
      <el-button @click="drawToolFunc.clearCanvas" type="danger" link>
        清空</el-button
      >
      <el-button
        @click="drawToolFunc.toggleDrawingsVisibility"
        link
        type="success"
        >预览</el-button
      >
      <el-button @click="drawToolFunc.savePolygon" type="primary" link
        >确认</el-button
      >
    </el-row>
  </div>
  <DrawTool ref="drawTool" />
  <div v-for="item in area">{{ item }}</div>
</template>
<style scoped>
.tools {
  padding: 8px 0;
}
</style>
