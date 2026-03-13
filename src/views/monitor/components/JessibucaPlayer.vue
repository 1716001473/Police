<template>
  <div class="jessibuca-wrapper" ref="containerRef">
    <div id="jessibuca-container" class="player-container"></div>

    <div class="player-toolbar" v-if="isPlaying">
      <el-button link type="primary" size="small" @click="screenshot"> 📸 截图 </el-button>
      <el-button link type="danger" size="small" @click="destroyPlayer"> ⏹ 停止 </el-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from "vue";
import { ElMessage } from "element-plus";

// 接收父组件（多宫格大屏）传来的视频流地址
const props = defineProps({
  videoUrl: {
    type: String,
    default: ""
  }
});

const containerRef = ref(null);
let jessibuca: any = null;
const isPlaying = ref(false);

// 🛠️ 初始化播放器实例
const initPlayer = () => {
  // 确保全局已经引入了 Jessibuca 的 JS 文件
  if (!window.Jessibuca) {
    ElMessage.error("未检测到 Jessibuca 核心库，请在 index.html 中引入");
    return;
  }

  jessibuca = new window.Jessibuca({
    container: containerRef.value,
    videoBuffer: 0.2, // 极低延迟配置
    isResize: false,
    text: "", // 可以在这里加公安平台的水印
    loadingText: "正在接入公安视频流...",
    debug: false,
    showBandwidth: true, // 显示网速
    operateBtns: {
      fullscreen: true,
      play: true,
      audio: true
    },
    forceNoOffscreen: true,
    isNotMute: false // 默认静音，防止多宫格同时发声刺耳
  });

  // 监听各种底层事件（报错、断流等）
  jessibuca.on("error", (error: any) => {
    console.error("视频流播放错误:", error);
    isPlaying.value = false;
  });
};

// ▶️ 播放逻辑
const playVideo = (url: string) => {
  if (!jessibuca) initPlayer();
  if (url) {
    jessibuca.play(url);
    isPlaying.value = true;
  }
};

// 📸 截图功能（完美解决安防 M2 阶段的抓拍需求）
const screenshot = () => {
  if (jessibuca) {
    // 这行代码会直接触发浏览器下载截图
    jessibuca.screenshot("监控抓拍证据", "png", 0.5);
    ElMessage.success("抓拍成功！");
  }
};

// 💥 终极核武器：防止 16宫格 内存泄露的销毁机制
const destroyPlayer = () => {
  if (jessibuca) {
    jessibuca.destroy();
    jessibuca = null;
    isPlaying.value = false;
  }
};

// 监听 URL 变化，只要父组件传了新 URL，立刻播放
watch(
  () => props.videoUrl,
  newUrl => {
    if (newUrl) {
      playVideo(newUrl);
    } else {
      destroyPlayer();
    }
  }
);

onMounted(() => {
  initPlayer();
  if (props.videoUrl) playVideo(props.videoUrl);
});

// 极其重要：组件销毁前必须干掉播放器，否则浏览器必卡死
onBeforeUnmount(() => {
  destroyPlayer();
});
</script>

<style scoped lang="scss">
.jessibuca-wrapper {
  width: 100%;
  height: 100%;
  position: relative;
  background-color: #000;
  display: flex;
  flex-direction: column;

  .player-container {
    flex: 1;
    width: 100%;
    height: 100%;
  }

  .player-toolbar {
    position: absolute;
    bottom: 0;
    width: 100%;
    height: 30px;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    justify-content: flex-end;
    align-items: center;
    padding: 0 10px;
    z-index: 10;
  }
}
</style>
