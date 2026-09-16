# Android17‑API37（CinnamonBun 肉桂卷）开发要点
> 更新时间：2026‑08‑26
> 平台版本：Android 17 API‑37
> IDE最低适配版本：Android Studio Koala +

## 一、平台新特性 & Breaking‑Change 破坏性变更
### 1. 大屏 / 折叠屏适配（重点迁移项）
- targetSdk = 37，**sw≥600dp大屏设备禁止硬锁定横竖屏**
  - 移除Activity `android:screenOrientation` 硬编码锁方向；系统强制任意方向适配
- Bubbles悬浮气泡升级：非聊天类App也可以使用悬浮气泡；折叠设备新增气泡专用栏
- Screen Reactions 原生反应录屏：录屏时可叠加前置摄像头画面，系统原生支持，无需自己实现相机叠加

### 2. 隐私安全
- 一次性精准位置授权：位置弹窗新增临时一次性授权，使用完自动回收权限
- Mark as lost 设备丢失保护：设备标记丢失后，即使知道PIN码，解锁强制生物识别；自动清空Google钱包支付卡片
- 诈骗来电校验：系统层识别仿冒金融诈骗电话
- 后台内存回收策略收紧，系统更激进杀掉高内存后台应用

### 3. 相机 & Media媒体
- PhotoPicker UI自定义 `PhotoPickerUiCustomizationParams`，支持自定义网格布局（9:16纵向）
- ImageFormat.RAW14：14bit RAW图像格式，面向专业相机App
- 相机设备区分API：区分内置摄像头 / USB外接摄像头 / 虚拟摄像头
- 延续Android16 APV视频编码支持；Media3 1.11稳定版

### 4. 端侧AI（本年度最重要方向）
#### AppFunctions(Android MCP) 【预览版】
> App对外暴露函数接口，供系统Gemini Agent调用；App变成AI Agent的工具
- App作为MCP服务端向系统注册业务工具
- 系统智能助手跨App直接调用本App内部逻辑
- 当前处于私密预览阶段，配套Jetpack库同步更新

#### AICore / Gemini‑Nano4 端侧大模型
- ML‑Kit GenAI Prompt API 新增**前缀缓存**，减少重复Prompt推理耗时，提升速度
- 支持直接输出结构化数据，序列化映射Kotlin对象
- Gemma4本地运行；优先调用系统AICore服务，不要把模型打包进APK

#### Continue‑On 跨设备接力Handoff
手机暂停任务，平板继续运行同一个App会话；Android多设备协同API

### 5. ART虚拟机运行时变更
- ART分代GC优化；Handler MessageQueue无锁优化，消息调度性能提升
- `static final` 编译期强不可变；**反射修改static final常量不再生效！老项目会踩坑**

## 二、Jetpack 2026‑08 稳定版本清单
|组件|版本|备注|
|---|---|---|
|Compose BOM|2026.08.01|Compose UI 1.12.1 Stable|
|Material3|1.4.0|Material3‑Expressive，毛玻璃动态配色|
|Navigation3|1.1.7|新一代状态路由导航，替代旧Navigation|
|Room3|3.0.2|K2编译器完全支持，新版SQLite绑定|
|Hilt|1.4.0|DI依赖注入|
|Lifecycle|2.11.0|生命周期库|
|Paging3|5.1|分页库|
|Fragment|1.9.0|Fragment维护版本|

> ✅ 新项目技术栈建议：**Kotlin + Compose + Navigation3 + Room3 + Hilt**

## 三、Android Studio & 工具更新
1. Android Studio Koala+ 完整支持API‑37 SDK，内置AppFunctions预览调试工具
2. Vibe‑Coding：IDE内置AI，自然语言生成Compose代码，直接导出工程
3. WebGPU Jetpack（实验）：Kotlin上层API；底层ANGLE将OpenGL转Vulkan；适合高性能视频编辑场景

## 四、2026 Android开发整体趋势
1. **端侧AI优先**：AICore + Gemini‑Nano4 + AppFunctions(MCP)，应用向AI‑Agent工具化演进
2. **大屏优先开发**：一套Compose布局同时适配手机 / 平板 / 折叠屏；放弃硬锁屏幕方向
3. Kotlin + Jetpack Compose 官方首选；View仍然维护，但是全部新特性只供给Compose
4. 隐私权限持续收紧：照片、位置、后台运行大量新增单次临时授权模式
5. 跨设备协同：Continue‑On接力，打通手机‑平板‑Wear OS

## 五、升级迁移重点清单（targetSdk37）
1. ❗ 删除Activity硬编码 `android:screenOrientation`，解决大屏设备崩溃
2. ❗ 废弃反射修改 `static final` 的旧hack写法，ART已封锁
3. 端侧AI开发优先ML‑Kit GenAI(AICore)，禁止APK内嵌入大模型文件
4. 老View项目逐步评估迁移Compose；Fragment仍然可用，但优先单Activity+Compose架构

## 六、后续可拓展Demo方向
- AppFunctions(MCP) 注册示例
- Navigation3基础路由Demo
- Room3迁移升级指南
