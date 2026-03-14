<template>
  <el-drawer v-model="drawerVisible" :destroy-on-close="true" size="450px" :title="`${drawerProps.title}摄像头`">
    <el-form
      ref="ruleFormRef"
      label-width="100px"
      label-suffix=" :"
      :rules="rules"
      :disabled="drawerProps.isView"
      :model="drawerProps.row"
      :hide-required-asterisk="drawerProps.isView"
    >
      <el-form-item label="设备名称" prop="streamName">
        <el-input v-model="drawerProps.row!.streamName" placeholder="请填写摄像头名称（如：南门人脸抓拍）" clearable></el-input>
      </el-form-item>
      <el-form-item label="归属区域" prop="location">
        <el-select v-model="drawerProps.row!.location" placeholder="请选择该设备所属的物理防区" clearable>
          <el-option label="东大门" value="东大门" />
          <el-option label="周界防范区" value="周界防范区" />
          <el-option label="内部办公区" value="内部办公区" />
        </el-select>
      </el-form-item>
      <el-form-item label="推流地址" prop="streamUrl">
        <el-input v-model="drawerProps.row!.streamUrl" placeholder="RTSP/RTMP/FLV 真实的推流地址" clearable></el-input>
      </el-form-item>
      <el-form-item label="上线状态" prop="status">
        <el-switch
          v-model="drawerProps.row!.status"
          active-value="online"
          inactive-value="offline"
          active-text="在线"
          inactive-text="离线"
        ></el-switch>
      </el-form-item>
      <el-form-item label="详细备注" prop="remark">
        <el-input
          v-model="drawerProps.row!.remark"
          type="textarea"
          :rows="3"
          placeholder="例如：带红外夜视、支持云台..."
          clearable
        ></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="drawerVisible = false">取消</el-button>
      <el-button v-show="!drawerProps.isView" type="primary" @click="handleSubmit">确定</el-button>
    </template>
  </el-drawer>
</template>

<script setup lang="ts" name="StreamDrawer">
import { ref, reactive } from "vue";
import { ElMessage, FormInstance } from "element-plus";

const rules = reactive({
  streamName: [{ required: true, message: "设备名称不能为空" }],
  location: [{ required: true, message: "请选择归属区域" }],
  streamUrl: [{ required: true, message: "推流地址必须填写，否则无法播放" }]
});

export interface DrawerProps {
  title: string;
  isView: boolean;
  row: Partial<any>;
  api?: (params: any) => Promise<any>;
  getTableList?: () => void;
}

const drawerVisible = ref(false);
const drawerProps = ref<DrawerProps>({
  isView: false,
  title: "",
  row: {}
});

// 接收父组件（列表页）的参数以渲染抽屉
const acceptParams = (params: DrawerProps) => {
  drawerProps.value = params;
  drawerVisible.value = true;
};

const ruleFormRef = ref<FormInstance>();

// 提交表单（这里 mock 请求）
const handleSubmit = () => {
  ruleFormRef.value!.validate(async valid => {
    if (!valid) return;
    try {
      ElMessage.success({ message: `${drawerProps.value.title}点位成功！(Mock记录)` });
      // drawerProps.value.getTableList!(); 这里通常会去刷新外面的表格
      drawerVisible.value = false;
    } catch (error) {
      console.log(error);
    }
  });
};

// 暴露给父组件调用
defineExpose({
  acceptParams
});
</script>
