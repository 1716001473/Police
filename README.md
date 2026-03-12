# 公安智能监控平台（Police Intelligent Monitoring Platform）

> **前后端统一规划文档** — 整合前端全景概览 + 后端架构设计讨论

---

## 一、项目定位

基于 **Geeker-Admin**（Vue 3 + TypeScript + Vite 5 + Element Plus）构建的政企安防监控后台，后端使用 **MySQL + Python**，核心能力覆盖：实时视频监控、多场景告警闭环、工作流编排。

---

## 二、技术栈

### 前端

| 层级       | 技术选型              | 备注                               |
| ---------- | --------------------- | ---------------------------------- |
| 框架       | Vue 3.4 + TypeScript  | Composition API + `<script setup>` |
| 构建       | Vite 5                | gzip/brotli 已配置                 |
| UI 库      | Element Plus          | 政企项目标配                       |
| 状态管理   | Pinia + 持久化插件    |                                    |
| 路由       | Vue Router 4          | 动态路由 + 按钮级权限              |
| HTTP       | Axios 二次封装        | 请求拦截/取消/Token 注入           |
| 图表       | ECharts               | M3 统计分析                        |
| 视频播放器 | **Jessibuca**（锁定） | 后端返回 WebRTC/HLS 地址，前端渲染 |

### 后端

| 层级       | 技术选型                  | 备注                                   |
| ---------- | ------------------------- | -------------------------------------- |
| 结构化存储 | MySQL                     | 主数据 / 状态 / 审计                   |
| 流媒体转码 | ZLMediaKit（建议）        | RTSP → WebRTC/HLS，前端不直接接触 RTSP |
| 告警推送   | SSE（Server-Sent Events） | 单向推送，浏览器自动重连               |
| 后端框架   | 待确认                    |                                        |

> **注意**：Nginx 反向代理需配置 `proxy_buffering off`，否则 SSE 消息会被缓存，导致实时性差。

---

## 三、里程碑规划

### M1（第 1-4 周）：系统基础 + 视频展示

| 验收项                    | 前端状态            | 后端接口                                        |
| ------------------------- | ------------------- | ----------------------------------------------- |
| 登录/退出/会话管理        | ✅ 框架自带         | —                                               |
| 角色权限控制              | ✅ 框架自带         | —                                               |
| 点位（视频流）列表 + 筛选 | 🔲 `views/point/`   | `GET /api/streams`                              |
| 点位在线/离线状态刷新     | 🔲 状态标签映射     | `GET /api/streams`（status 字段）               |
| 新增 / 编辑 / 删除点位    | 🔲 弹窗表单         | `POST` / `PATCH` / `DELETE /api/streams/{id}`   |
| 手动立即检查              | 🔲 按钮             | `POST /api/streams/{id}/check`                  |
| 单路视频播放              | 🔲 `views/monitor/` | `GET /api/streams/{id}/preview`（返回播放地址） |
| 多宫格视频（4/9/16）      | 🔲 CSS Grid 布局    | 子码流 / 主码流切换                             |
| 视频切换 / 全屏 / 静音    | 🔲 播放器封装       | —                                               |
| 时间轴回放                | 🔲                  | 需后端提供录像时间片段列表                      |

#### 后端视频流状态机（已确认）

```
新增流 → checking → available ⇄ unavailable
                           ↘              ↙
                          disabled（人工禁用，跳过所有任务）
```

#### 后端核心数据模型 `video_stream` 主要字段

`id` / `name` / `source_url`（加密）/ `protocol` / `status` / `last_check_time` / `last_error` / `consecutive_failures`

---

### M2（第 5-8 周）：场景管理 + 告警闭环

| 验收项                | 说明                                                |
| --------------------- | --------------------------------------------------- |
| 13 个监控场景统一管理 | 启用/停用、绑定点位（`video_stream`）、参数配置     |
| 实时告警接入          | SSE 推送，前端 `EventSource` 监听                   |
| 告警列表筛选          | 按时间/场景/点位/级别                               |
| 告警详情 + 证据       | 关联视频片段 + 关键帧截图（后端生成 URL，前端展示） |
| 告警处置闭环          | 确认/误报/备注，状态可追溯                          |

> ⚠️ 后端场景管理 & 报警系统接口设计**待补充**，纳入催办清单。

---

### M3（第 9-12 周）：工作流编排 + 平台扩展

| 验收项                         | 说明                                                      |
| ------------------------------ | --------------------------------------------------------- |
| 工作流编排                     | ⚠️ 需产品出原型图（表单配置可控 / 拖拽画布至少需 1 个月） |
| 动态新增场景（无需改前端代码） | 引入动态表单引擎（form-create / formily）                 |
| 操作日志 / 统计分析            | ECharts 可视化                                            |
| 🧠 公安智库 AI 助手（附加项）  | 预留悬浮窗，基于 RAGFlow 定制版 iframe 接入               |

---

## 四、当前文件结构

```
src/views/
├── home/          ✅ 首页
├── login/         ✅ 登录
├── system/        ✅ 系统管理
├── dashboard/     ✅ 数据可视化
├── dataScreen/    ✅ 数据大屏
├── proTable/      ✅ ProTable 示例（开发参考）
├── monitor/       🆕 视频与监控（M1）
├── point/         🆕 点位与通道管理（M1）
├── alarm/         🆕 告警与处置（M2）
└── workflow/      🆕 工作流编排占位（M3）
```

---

## 五、下一步行动建议

1. **Mock 先行**：用本地 Mock JSON 先完成 `point/index.vue` 点位列表的 ProTable 开发，不等接口联调
2. **视频播放器集成**：向后端索取测试用 WebRTC/HLS 拉流地址，在 `monitor/index.vue` 集成 Jessibuca
3. **多宫格布局**：用 CSS Grid 搭好 4/9/16 宫格占位框架
4. **催后端补充**：场景管理 & 报警系统的接口设计（对应 M2，越早越好）

---

## 六、运行方式

```bash
pnpm install   # 安装依赖
pnpm dev       # 启动开发服务器
pnpm build:pro # 生产环境打包
```

**代码仓库**：https://github.com/1716001473/Police
