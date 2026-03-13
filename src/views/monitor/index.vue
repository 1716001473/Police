<template>
  <div class="monitor-container">
    <div class="left-tree card">
      <div class="tree-header">
        <span>📍 监控点位列表</span>
      </div>
      <el-input v-model="filterText" placeholder="搜索摄像头名称" clearable class="mb-4" />
      <el-tree
        ref="treeRef"
        class="camera-tree"
        :data="cameraData"
        :props="defaultProps"
        :filter-node-method="filterNode"
        default-expand-all
        @node-click="handleNodeClick"
      >
        <template #default="{ node, data }">
          <span class="custom-tree-node">
            <span class="status-dot" :class="data.status === 'online' ? 'bg-success' : 'bg-danger'" v-if="!data.children"></span>
            <span class="ml-1">{{ node.label }}</span>
          </span>
        </template>
      </el-tree>
    </div>

    <div class="right-video card">
      <div class="video-controls">
        <div class="title">🖥️ 实时监控大屏</div>
        <el-button-group>
          <el-button :type="gridCount === 1 ? 'primary' : 'default'" @click="changeGrid(1)">单屏</el-button>
          <el-button :type="gridCount === 4 ? 'primary' : 'default'" @click="changeGrid(4)">4宫格</el-button>
          <el-button :type="gridCount === 9 ? 'primary' : 'default'" @click="changeGrid(9)">9宫格</el-button>
          <el-button :type="gridCount === 16 ? 'primary' : 'default'" @click="changeGrid(16)">16宫格</el-button>
          <el-button type="danger" plain icon="FullScreen">全屏</el-button>
        </el-button-group>
      </div>

      <div class="video-grid-wrapper">
        <div class="video-grid" :class="`grid-${gridCount}`">
          <div
            v-for="index in 16"
            :key="index"
            v-show="index <= gridCount"
            class="video-box"
            :class="{ 'active-box': activeIndex === index }"
            @click="activeIndex = index"
          >
            <div class="placeholder-text">
              <el-icon size="24" class="mb-2"><VideoCamera /></el-icon>
              <span>通道 {{ index }} (等待推流...)</span>
            </div>
            <div class="video-tag" v-if="activeIndex === index">当前选中</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts" name="monitor">
import { ref, watch } from "vue";
import { VideoCamera } from "@element-plus/icons-vue";

// --- 左侧树形数据逻辑 ---
const filterText = ref("");
const treeRef = ref();

// 模拟后端返回的点位树状结构
const cameraData = ref([
  {
    id: 1,
    label: "东大门区域",
    children: [
      { id: 11, label: "东大门进出口抓拍", status: "online", stream: "url_1" },
      { id: 12, label: "东大门人脸识别", status: "offline", stream: "url_2" }
    ]
  },
  {
    id: 2,
    label: "周界防范区",
    children: [
      { id: 21, label: "北墙全景摄像头", status: "online", stream: "url_3" },
      { id: 22, label: "南墙红外摄像头", status: "online", stream: "url_4" }
    ]
  }
]);

const defaultProps = { children: "children", label: "label" };

watch(filterText, val => {
  treeRef.value!.filter(val);
});

const filterNode = (value: string, data: any) => {
  if (!value) return true;
  return data.label.includes(value);
};

// 点击摄像头节点
const handleNodeClick = (data: any) => {
  if (!data.children) {
    console.log(`准备将摄像头 [${data.label}] 的流播放到第 ${activeIndex.value} 个窗口`, data.stream);
    // 这里未来写对接播放器的逻辑
  }
};

// --- 右侧多宫格逻辑 ---
const gridCount = ref(4); // 默认4宫格
const activeIndex = ref(1); // 当前选中的窗口焦点

const changeGrid = (count: number) => {
  gridCount.value = count;
  if (activeIndex.value > count) {
    activeIndex.value = 1; // 如果切回少宫格，焦点重置
  }
};
</script>

<style scoped lang="scss">
/* 整体左右布局 */
.monitor-container {
  display: flex;
  height: 100%;
  gap: 16px;
  padding: 0;

  .card {
    background: var(--el-bg-color);
    border: 1px solid var(--el-border-color-light);
    border-radius: 8px;
    padding: 16px;
    display: flex;
    flex-direction: column;
  }
}

/* 左侧点位树 */
.left-tree {
  width: 280px;
  min-width: 280px;

  .tree-header {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 16px;
    color: var(--el-text-color-primary);
  }

  .camera-tree {
    flex: 1;
    overflow-y: auto;
  }

  .custom-tree-node {
    display: flex;
    align-items: center;
    font-size: 14px;

    .status-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      display: inline-block;
    }
    .bg-success {
      background-color: #67c23a;
    }
    .bg-danger {
      background-color: #f56c6c;
    }
  }
}

/* 右侧视频区 */
.right-video {
  flex: 1;
  overflow: hidden;

  .video-controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;

    .title {
      font-size: 18px;
      font-weight: bold;
      color: var(--el-text-color-primary);
    }
  }

  .video-grid-wrapper {
    flex: 1;
    background-color: #000; /* 安防经典黑底 */
    padding: 2px;
    border-radius: 4px;
    overflow: hidden;
  }

  /* CSS Grid 魔法核心 */
  .video-grid {
    display: grid;
    width: 100%;
    height: 100%;
    gap: 2px; /* 窗口之间的缝隙 */
    background-color: #333; /* 缝隙颜色 */
  }

  /* 动态行列配置 */
  .grid-1 {
    grid-template-columns: 1fr;
    grid-template-rows: 1fr;
  }
  .grid-4 {
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: repeat(2, 1fr);
  }
  .grid-9 {
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
  }
  .grid-16 {
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(4, 1fr);
  }

  /* 单个视频占位框 */
  .video-box {
    position: relative;
    background-color: #1a1a1a;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    border: 2px solid transparent;
    transition: all 0.2s;

    .placeholder-text {
      display: flex;
      flex-direction: column;
      align-items: center;
      color: #666;
      font-size: 14px;
    }

    .video-tag {
      position: absolute;
      top: 0;
      right: 0;
      background-color: var(--el-color-primary);
      color: #fff;
      font-size: 12px;
      padding: 2px 6px;
      border-bottom-left-radius: 4px;
    }
  }

  /* 选中状态的高亮边框 */
  .active-box {
    border: 2px solid var(--el-color-primary);
  }
}
</style>
