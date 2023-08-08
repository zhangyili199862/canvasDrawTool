<template>
  <div
    class="draw-tool"
    :style="`width:${canvasWidth + 'px'};height:${canvasHeight + 'px'}`"
  >
    <div
      class="draw-area"
      id="drawArea"
      :style="`background-image: url('${picSrc}');`"
    >
      <canvas
        id="canvas"
        :width="canvasWidth"
        :height="canvasHeight"
        ref="canvas"
        @click="handleClick"
      ></canvas>
    </div>
    <div
      v-if="selectedPolygon !== null"
      class="selection-box"
      ref="selectedBox"
    >
      <el-button
        type="danger"
        circle
        :icon="Close"
        class="closeButton"
        @click="deleteSelectedPolygon"
        size="small"
      ></el-button>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, Ref, onMounted, nextTick } from "vue";
import { Close } from "@element-plus/icons-vue";
import { ElMessage, ElMessageBox } from "element-plus";

const props = defineProps({
  isReadonly: {
    type: Boolean,
    required: false,
    default: false,
  },
  picSrc: {
    type: String,
    required: false,
    default: "/src/assets/img/video-card-bg.png",
  },
});

const canvas = ref<HTMLCanvasElement | null>();
const ctx = ref<CanvasRenderingContext2D | null>(null);
const polygons = ref<Array<Array<{ x: number; y: number }>>>([]);
const currentPolygon = ref<Array<{ x: number; y: number }>>([]);

const selectedPolygon = ref<number | null>(null);
const selectedBox = ref<HTMLDivElement>();
/**
 * 绘制是否开启
 */
const isDrawing = ref(false);
const drawClick = () => {
  if (!isDrawing.value) {
    startMode();
  } else {
    stopMode();
  }
};
const startMode = () => {
  isDrawing.value = true;
};
const stopMode = () => {
  isDrawing.value = false;
  currentPolygon.value = [];
  redrawer();
};
/**
 * 保存currentPolygon
 */
const savePolygon = () => {
  ElMessage.success("保存成功");
  closeDrawing();
};
const handleClick = (event: MouseEvent) => {
  if (props.isReadonly) {
    return;
  }
  if (canvas.value) {
    const rect = canvas.value.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;

    if (!isDrawing.value) {
      const clickedIndex = findPolygon(x, y);
      if (clickedIndex !== null) {
        selectedPolygon.value = clickedIndex;
        nextTick(() => {
          updateSelectedBox();
        });
      } else {
        selectedPolygon.value = null;
      }
    } else {
      // 继续绘制当前多边形
      currentPolygon.value.push({ x, y });
    }

    // 清空画布
    const ctx = canvas.value.getContext("2d");
    if (ctx) {
      ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
      drawNowPolygons();
      // 绘制当前多边形
      if (isDrawing.value) drawPolygon(ctx, currentPolygon.value);

      // 绘制当前多边形的填充区域
      if (currentPolygon.value.length > 2) {
        fillPolygon(ctx, currentPolygon.value);
      }
    }
  }
};
const drawNowPolygons = () => {
  if (canvas.value) {
    const ctx = canvas.value.getContext("2d");
    if (ctx) {
      // 绘制已存在的多边形
      for (const polygon of polygons.value) {
        drawPolygon(ctx, polygon);
        fillPolygon(ctx, polygon);
      }
    }
  }
};
/**
 * 结束当前绘制
 */
const closeDrawing = () => {
  if (isDrawing.value && currentPolygon.value.length > 2) {
    // 完成当前多边形绘制
    isDrawing.value = false;

    // 连接第一个顶点和最后一个顶点
    const firstPoint = currentPolygon.value[0];
    // const lastPoint = currentPolygon.value[currentPolygon.value.length - 1];
    currentPolygon.value.push({ x: firstPoint.x, y: firstPoint.y });

    // 添加当前多边形到多边形列表
    polygons.value.push([...currentPolygon.value]);
    currentPolygon.value = [];
  }
};

const drawPolygon = (
  ctx: CanvasRenderingContext2D,
  polygon: Array<{ x: number; y: number }>
) => {
  ctx.strokeStyle = "#00F4AB";
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(polygon[polygon.length - 1].x, polygon[polygon.length - 1].y);
  for (const point of polygon) {
    ctx.lineTo(point.x, point.y);
  }
  ctx.stroke();
};

const fillPolygon = (
  ctx: CanvasRenderingContext2D,
  polygon: Array<{ x: number; y: number }>
) => {
  ctx.fillStyle = "rgba(25, 204, 150, 0.30)";
  ctx.beginPath();
  ctx.moveTo(polygon[0].x, polygon[0].y);
  for (const point of polygon) {
    ctx.lineTo(point.x, point.y);
  }
  ctx.closePath();
  ctx.fill();
};
const findPolygon = (x: number, y: number): number | null => {
  for (let i = 0; i < polygons.value.length; i++) {
    const polygon = polygons.value[i];
    const path = new Path2D();
    path.moveTo(polygon[polygon.length - 1].x, polygon[polygon.length - 1].y);
    for (const point of polygon) {
      path.lineTo(point.x, point.y);
    }
    if (canvas.value) {
      const ctx = canvas.value.getContext("2d");
      if (ctx!.isPointInPath(path, x, y)) {
        return i;
      }
    }
  }
  return null;
};
const updateSelectedBox = () => {
  if (selectedPolygon.value !== null && selectedBox.value && canvas.value) {
    const polygon = polygons.value[selectedPolygon.value];
    const minX = Math.min(...polygon.map((point) => point.x));
    const minY = Math.min(...polygon.map((point) => point.y));
    const maxX = Math.max(...polygon.map((point) => point.x));
    const maxY = Math.max(...polygon.map((point) => point.y));

    const rect = canvas.value.getBoundingClientRect();
    const boxLeft = minX;
    const boxTop = minY + rect.top;
    const boxWidth = maxX - minX;
    const boxHeight = maxY - minY;
    selectedBox.value.style.left = boxLeft + "px";
    selectedBox.value.style.top = boxTop + "px";
    selectedBox.value.style.width = boxWidth + "px";
    selectedBox.value.style.height = boxHeight + "px";
  }
};
const deleteSelectedPolygon = () => {
  if (selectedPolygon.value !== null) {
    polygons.value.splice(selectedPolygon.value, 1);
    selectedPolygon.value = null;
    updateSelectedBox();
    // 绘制已存在的多边形
    redrawer();
  }
};
/**
 * 通过showDrawings来控制预览按钮的图表
 *
 * toggleDrawingVisibility 是用来切换是否绘制路径的方法
 */
const showDrawings = ref(true); // 控制绘制区域的可见性
const toggleDrawingsVisibility = () => {
  if (isDrawing.value) {
    ElMessageBox.confirm("正在绘制中，请确认是否停止绘制", "提示", {
      confirmButtonText: "确认",
      cancelButtonText: "取消",
    }).then(async () => {
      closeDrawing();
      toggleDrawingsVisibility();
    });
  } else {
    showDrawings.value = !showDrawings.value;
    const ctx = canvas.value!.getContext("2d");
    if (ctx) {
      ctx.clearRect(0, 0, canvas.value!.width, canvas.value!.height);
    }
    if (showDrawings.value) {
      redrawer();
    }
  }
};
// 清空画布并重绘
const redrawer = () => {
  const ctx = canvas.value!.getContext("2d");
  if (ctx) {
    ctx.clearRect(0, 0, canvas.value!.width, canvas.value!.height);
    drawNowPolygons();
  }
};
const clearCanvas = () => {
  if (isDrawing.value) {
    ElMessageBox.confirm("正在绘制中，请确认是否清空画布", "提示", {
      confirmButtonText: "确认",
      cancelButtonText: "取消",
    }).then(async () => {
      closeDrawing();
      clearCanvas();
    });
  } else {
    const ctx = canvas.value!.getContext("2d");
    if (ctx) {
      ctx.clearRect(0, 0, canvas.value!.width, canvas.value!.height);
      polygons.value = [];
    }
  }
};
const canvasWidth: Ref<number> = ref(800),
  canvasHeight: Ref<number> = ref(432);
//计算图片的宽度
const calcPicWidthAndHeight = () => {
  const picDom = document.getElementById("drawArea");
  if (picDom) {
    const style: CSSStyleDeclaration = window.getComputedStyle(picDom);
    const backgroundImage: string = style.backgroundImage;
    //获取图片路径
    const imageUrl: string = backgroundImage
      .replace(/^url\(["']?/, "")
      .replace(/["']?\)$/, "");

    //创建Image对象获取高度和宽度
    const img = new Image();
    img.src = imageUrl;

    img.onload = () => {
      const width = img.width;
      const height = img.height;
      canvasWidth.value = width;
      canvasHeight.value = height;
    };
  }
};

const initCanvas = (initArray?: Array<Array<{ x: number; y: number }>>) => {
  showDrawings.value = true;
  if (initArray) {
    polygons.value = initArray;
  }
  redrawer();
};
onMounted(() => {
  nextTick(() => {
    if (canvas.value) {
      ctx.value = canvas.value.getContext("2d");
    }
    calcPicWidthAndHeight();
  });
});
defineExpose({
  polygons,
  drawClick,
  clearCanvas,
  toggleDrawingsVisibility,
  savePolygon,
  initCanvas,
  stopMode,
});
</script>
<style lang="scss">
@import "./DrawTool.scss";
</style>
