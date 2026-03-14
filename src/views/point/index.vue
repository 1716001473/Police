<template>
  <div class="table-box">
    <ProTable
      ref="proTableRef"
      :columns="columns"
      :request-api="getStreamList"
      :init-param="initParam"
      :data-callback="dataCallback"
      id="streamId"
    >
      <!-- 表格 header 按钮 -->
      <template #tableHeader="scope">
        <el-button type="primary" icon="CirclePlus" @click="openDrawer('新增')">新增摄像头</el-button>
        <el-button type="danger" icon="Delete" plain :disabled="!scope.isSelected" @click="batchDelete(scope.selectedListIds)">
          批量删除
        </el-button>
      </template>

      <!-- 状态列：在线/离线 标签高亮 -->
      <template #status="scope">
        <el-tag :type="scope.row.status === 'online' ? 'success' : 'danger'">
          {{ scope.row.status === "online" ? "在线" : "离线" }}
        </el-tag>
      </template>

      <!-- 操作列 -->
      <template #operation="scope">
        <el-button type="primary" link icon="View" @click="handlePreview(scope.row)">预览视频</el-button>
        <el-button type="primary" link icon="EditPen" @click="openDrawer('编辑', scope.row)">编辑</el-button>
        <el-button type="primary" link icon="Delete" @click="deleteStream(scope.row)">删除</el-button>
      </template>
    </ProTable>

    <!-- 新增/编辑 摄像头的 Drawer 表单弹窗 -->
    <StreamDrawer ref="drawerRef" />
  </div>
</template>

<script setup lang="tsx" name="streamManage">
import { ref, reactive } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import ProTable from "@/components/ProTable/index.vue";
import { type ColumnProps, type ProTableInstance } from "@/components/ProTable/interface";
import StreamDrawer from "./components/StreamDrawer.vue";

// ⚠️ M1阶段前期：暂时先写死 Mock 数据接口，后端给真实的后直接替换！
const mockData = [
  {
    streamId: "1",
    streamName: "东大门进出口抓拍",
    streamUrl: "http://192.168.1.100/live.flv",
    status: "online",
    location: "东大门",
    remark: "带人脸抓拍功能"
  },
  {
    streamId: "2",
    streamName: "北墙全景摄像头",
    streamUrl: "http://192.168.1.101/main.flv",
    status: "online",
    location: "周界防范区",
    remark: ""
  },
  {
    streamId: "3",
    streamName: "南墙红外摄像头",
    streamUrl: "http://192.168.1.102/sub.flv",
    status: "offline",
    location: "周界防范区",
    remark: "夜间模式已开启"
  }
];

// 模拟获取列表的接口
const getStreamList = async (params: any) => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        data: {
          list: mockData,
          pageNum: 1,
          pageSize: 10,
          total: 3
        }
      });
    }, 500);
  });
};

const proTableRef = ref<ProTableInstance>();

// 初始化查询参数
const initParam = reactive({ status: "" });

// 数据请求后的回调
const dataCallback = (data: any) => {
  return {
    list: data.list,
    total: data.total,
    pageNum: data.pageNum,
    pageSize: data.pageSize
  };
};

// 表格配置项
const columns: ColumnProps<any>[] = [
  { type: "selection", fixed: "left", width: 80 },
  { type: "index", label: "#", width: 80 },
  { prop: "streamName", label: "点位(摄像头)名称", search: { el: "input" } },
  { prop: "streamUrl", label: "拉流地址", width: 300, isShow: false }, // 默认隐藏拉流地址让界面干净点
  {
    prop: "location",
    label: "归属区域",
    search: {
      el: "select",
      // 这相当于高级查询表单里的下拉框
      options: [
        { label: "全部", value: "" },
        { label: "东大门", value: "东大门" },
        { label: "周界防范区", value: "周界防范区" }
      ]
    }
  },
  {
    prop: "status",
    label: "点位状态",
    search: {
      el: "select",
      options: [
        { label: "全部", value: "" },
        { label: "在线", value: "online" },
        { label: "离线", value: "offline" }
      ]
    }
  },
  { prop: "remark", label: "备注说明" },
  { prop: "operation", label: "操作", fixed: "right", width: 260 }
];

// 删除单条
const deleteStream = async (row: any) => {
  await ElMessageBox.confirm(`确认删除摄像头【${row.streamName}】?`, "温馨提示", { type: "warning" });
  ElMessage.success("删除成功 (模拟)");
  proTableRef.value?.getTableList();
};

// 批量删除
const batchDelete = async (ids: string[]) => {
  await ElMessageBox.confirm(`确认批量删除选中的 ${ids.length} 个点位?`, "温馨提示", { type: "warning" });
  ElMessage.success("删除成功 (模拟)");
  proTableRef.value?.clearSelection();
  proTableRef.value?.getTableList();
};

// 预览视频
const handlePreview = (row: any) => {
  ElMessage.info(`跳转大屏预览：${row.streamName}`);
  // 后续这里可以携带 streamUrl 跳转路由到 monitor 页面
};

// ====================== 抽屉相关 ======================
const drawerRef = ref<InstanceType<typeof StreamDrawer> | null>(null);

// 打开新增 / 编辑抽屉弹窗
const openDrawer = (title: string, row: Partial<any> = {}) => {
  const params = {
    title,
    isView: title === "查看",
    row: { ...row },
    // 目前使用假数据测试，所以不传真实后端的 API
    api: undefined,
    getTableList: proTableRef.value?.getTableList
  };
  drawerRef.value?.acceptParams(params);
};
</script>
