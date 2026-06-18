# Android 17 全面解读：开发者与用户都该知道的变化

> 版本信息：Android 17 / API Level 37
> 稳定版发布日期：2026 年 6 月 16 日
> 首批适配：Pixel 6 ~ Pixel 10 系列（含 Pixel 10 Pro Fold、Pixel Fold、Pixel Tablet），三星 / 荣耀 / 一加 / OPPO / 小米 / 摩托罗拉等厂商陆续跟进。

---

## 一、一句话总结

Android 17 是一个"承上启下"的版本：用户侧的可感知更新相对克制（更大的变化会放到 Android 17 QPR1 及后续季度更新），但**对开发者而言是一次方向性的转折**——官方正式宣布 **Compose 优先（Compose-First）**、**大屏自适应强制化（Adaptive-First）**，并把系统重新定位为"**智能系统（Intelligence System）**"，让 App 能力可以被 AI Agent（Gemini）直接调用。

如果你只想记住三件事：

1. **Compose 优先**：所有新 API/库/工具只面向 Jetpack Compose，传统 View 体系进入维护模式。
2. **大屏强制可调整大小**：targetSDK 37 的 App 在大屏（sw ≥ 600dp）上不能再拒绝分屏/旋转/自由窗口（游戏除外）。
3. **隐私与安全收紧**：本地网络访问、后台音频、短信 OTP、内存使用等都有强制约束。

---

## 二、发布流程的变化（先了解游戏规则）

Android 17 这一代彻底改变了预览发布方式：

- **废弃传统的 Developer Preview**，改为常态化的 **Canary 通道**（always-on）。
- 特性一旦通过内部测试即合入 Canary，开发者能更早拿到；同时通过早期"实战测试"提升稳定性。
- Canary 支持 OTA 更新，方便接入 CI 流程做自动化验证。
- 后续走"季度更新"节奏：Android 17 → QPR1 → ... Q4 会有一次带新 API 的小版本 SDK 发布。
- 配套工具推荐使用最新 Canary 版 **Android Studio（Quail 版本线）**。

> 实践建议：把 targetSDK 升级和兼容性回归纳入持续集成，跟随 Canary 提前发现行为变化。

---

## 三、开发者最需要关注的三大方向性变化

### 1. Compose 优先（Compose-First）

- 从 Android 17 起，**所有新的 Android API、Jetpack 库、工具与官方指南都只面向 Jetpack Compose 构建**。
- 传统 View 体系（`android.widget` 包）以及基于 View 的 Jetpack 库（**Fragments、RecyclerView、ViewPager** 等）进入**维护模式**：只修关键 Bug，不再有新功能。
- 影响：新项目应直接 Compose；存量项目要规划渐进式迁移路径，关键 UI 优先。

### 2. 大屏自适应强制化（Adaptive-First）

- 大屏设备装机量已超 5.8 亿台，Google 把"自适应"从建议变为强制。
- **targetSDK 37 的 App 在 sw ≥ 600dp 的设备上，系统将忽略其方向（orientation）、可调整大小（resizability）和宽高比（aspect ratio）限制，且无法再 opt-out**（Android 16 还是可选，Android 17 起强制；**游戏除外**）。
- 也就是说，App 必须允许用户自定义窗口大小 / 分屏配置。
- 配套能力：**App Bubbles（所有应用可气泡化）**、平板上的 **Bubble Bar**、桌面环境下可交互的 **画中画（PiP）窗口**。

### 3. 智能系统（Intelligence System）/ AppFunctions

- 通过给类添加 **`@AppFunction`** 注解，开发者可以把 App 的工作流暴露为"可被发现的工具"，供 Gemini 等 AI Agent 调用，代用户完成任务自动化。
- 这是 Android 把自身定位为 AI Agent 运行底座的关键一步。
- 部分高端设备今夏起还会获得 **Gemini Intelligence**（端侧 AI 能力套件）。

---

## 四、安全相关变化

### 全部应用（不论 targetSDK）

- **跨 Profile 回环流量默认禁止**（Block cross-profile loopback traffic）。
- **每个 App 的 Keystore 密钥创建数量受限**，超限会抛异常。
- **限制隐式 URI 授权**：应显式预分配 URI 权限，不要依赖系统自动授予。

### 仅 targetSDK 37+ 应用

- **默认开启 Certificate Transparency（CT）**：所有 TLS 连接强制启用（Android 16 是可选）。
- **更安全的本地动态代码加载（DCL）**：DCL 模块必须标记为只读、不可被无感知地覆写。
- **限制访问 CP2（ContactsContract）中的部分 PII 字段**。
- **CP2 强制严格 SQL 校验**：在无 `READ_CONTACTS` 权限下访问 `ContactsContract.Data` 时强制严格校验。

### 新能力

- **Android Advanced Protection Mode（AAPM）**：面向用户的高级保护模式。
- **PQC APK 签名**：支持使用后量子加密（Post-Quantum Crypto）密钥签名的 APK。
- **Android Keystore 新增 ML-DSA**（NIST 标准化的格基数字签名算法），提供抗量子签名能力。
- **OS 校验（Android OS verification）**：帮助确认设备运行的是官方、广泛分发的 Android 构建，对抗被篡改的修改版系统。

---

## 五、隐私相关变化

### 全部应用

- **限制访问端到端加密消息**：大多数 App 无法再读取 E2EE 消息。

### 仅 targetSDK 37+ 应用

- **标准短信 OTP 保护**：非优先级 App 即便有短信权限，也**不能立即读取一次性验证码，会被延迟约 3 小时**。请迁移到 **SMS Retriever API** 或 **SMS User Consent API**。
- **本地网络访问需权限**：访问本地网络设备需要"附近设备"权限组（强制，Android 16 为可选）。建议优先使用隐私友好的系统选择器；需要广泛、持久访问时使用新的 **`ACCESS_LOCAL_NETWORK`** 权限。
- **默认启用 ECH（Encrypted Client Hello）**。
- **物理键盘不再回显最后输入的密码字符**。

### 新能力

- **系统级联系人选择器（Contacts Picker）**：无需申请 `READ_CONTACTS` 即可让用户挑选联系人。
- **EyeDropper 取色 API**：无需敏感的屏幕截图权限即可精确取色。

---

## 六、核心运行时与性能

### 全部应用

- **App 内存上限（App memory limits）**：根据设备总 RAM 设定上限。Android 17 阶段阈值偏保守，主要用于**狙击极端内存泄漏 / 异常占用**，避免拖垮系统造成 UI 卡顿。若因此被杀，`ApplicationExitInfo.getDescription()` 会包含 **"MemoryLimiter"**。
- **`android:usesCleartextTraffic` 计划废弃**：未来版本将默认阻止明文流量，请迁移到 **Network Security Configuration**。

### 仅 targetSDK 37+ 应用

- **NPU 访问需声明特性**：直接访问 NPU 的 App 必须在清单声明 **`android.hardware.npu`（FEATURE_NEURAL_PROCESSING_UNIT）**，否则会被阻止。涵盖使用 LiteRT NPU delegate、厂商 SDK 以及已废弃的 NNAPI 的场景。
- **通知自定义视图的内存限制更严格**。
- **`static final` 字段不可再被修改**，强行修改抛异常。
- **MessageQueue 改为无锁实现**：提升帧率与启动速度。
- **配置变更行为优化**：键盘 / 导航类配置变更不再重启 Activity。

### 性能整体改进

- **分代垃圾回收（Generational GC）**：降低 CPU 占用、减少 UI 卡顿。
- **无锁 MessageQueue**：改善帧率与冷启动。
- **R8 优化器**：新增配置分析器。

### 新调试能力

- **新的 ProfilingManager 触发器**：系统可触发收集性能调试数据。
- **JobDebugInfo API**：用于调试 JobScheduler 任务。

---

## 七、媒体与影像

### 全部应用

- **后台音频收紧（Background audio hardening）**：App 在非可见状态且无前台服务时，不能播放音频、请求音频焦点或调用音量等干扰性 API。Beta 2 后的调整：按 targetSDK 进行门控、while-in-use 前台服务约束、为闹钟音频豁免。

### 新格式与能力

- **VVC / H.266 平台级集成**：在支持的芯片上硬件解码，画质等同 H.265/HEVC 但码率约低一半。
- **RAW14 图像格式**：新增 `ImageFormat.RAW14`，支持 14-bit/像素 RAW 拍摄，给专业相机 App 更高动态范围。
- **Eclipsa Video HDR**：自适应显示渲染标准。
- **Extended HE-AAC 编码器**：面向低带宽音频。
- **`BYPASS_CONCURRENT_RECORD_AUDIO_RESTRICTION` 权限**：用于通话等敏感场景下的并发录音。

---

## 八、连接性

- **蓝牙绑定丢失后自动重新配对**：系统自动重建绑定，免去手动解绑 / 重连。
- **RFCOMM 的 `BluetoothSocket.read()` 行为对齐标准 Java InputStream**（targetSDK 37+）。
- **受限卫星网络（Constrained satellite networks）**：支持低带宽卫星网络能力。
- **蓝牙 LE Audio 助听器类别**：支持更细粒度的音频路由。

---

## 九、图形

- **图形 API 底线提升到 Vulkan 1.4**，并强制支持 **ANGLE**。
- **WebGPU 登陆 Android**；**OpenGL ES 进入维护模式**，新特性集中在 Vulkan + WebGPU。

---

## 十、系统 UI、输入与无障碍

### 系统 UI / 交互

- **Bubbles 面向所有 App 开放**：长按应用图标即可变成可移动的悬浮小窗，浮于其他任务之上；大屏 / 折叠屏上气泡集中到底部的 **Bubble Bar**，可一键切换、缩放、最大化。
- **Assistant 独立音量流**：与媒体音量分离，可单独控制。
- **MetricStyle 通知模板**：适配健康、健身、计时器、秒表、出行等场景。
- **Live Update 语义颜色 API**：提供安全 / 危险 / 警示语义色。
- **Handoff（Continue On）**：可在另一台设备（如平板）上继续当前任务，系统通过 `CompanionDeviceManager` 同步状态，并在附近设备的启动器中展示接续建议。Google I/O 2026 已用 **"Continue On"** 作为面向用户的品牌名。
- 其他：旋转后恢复默认 IME 可见性；触控板在指针捕获时默认发送相对事件（类鼠标）。

### 用户可见的小功能

- 系统级**手柄按键重映射**。
- 主屏可**隐藏应用名称**。
- 更多**深色主题自定义**选项。
- **家长控制扩展到所有 Android 设备**。

### WebView / 无障碍

- **targetSDK 37+ 的 WebView 默认 User-Agent 字符串缩短**（UA reduction）。
- **无障碍增强**：IME 在物理键盘复杂输入时传递更多文本变更信息，改善读屏反馈。

---

## 十一、迁移与升级 Checklist（给团队的行动清单）

升级 targetSDK 到 37 前，建议逐项验证：

- [ ] **大屏适配**：确认 App 在分屏 / 自由窗口 / 横竖屏切换下布局正常（已无法再锁定方向）。
- [ ] **本地网络**：梳理局域网访问场景，按需申请 `ACCESS_LOCAL_NETWORK` 或改用系统选择器。
- [ ] **短信 OTP**：迁移到 SMS Retriever / SMS User Consent，避免 3 小时延迟影响登录验证。
- [ ] **后台音频**：检查后台播放 / 音频焦点 / 音量调用，补齐前台服务或调整时机。
- [ ] **内存占用**：排查内存泄漏，关注 `ApplicationExitInfo` 中的 "MemoryLimiter"。
- [ ] **明文流量**：用 Network Security Configuration 替换 `usesCleartextTraffic`。
- [ ] **NPU**：使用 NPU / LiteRT / NNAPI 的 App 在清单声明 `android.hardware.npu`。
- [ ] **TLS**：确认服务端兼容默认开启的 CT 与 ECH。
- [ ] **联系人 / URI**：改用系统联系人选择器，显式声明 URI 权限。
- [ ] **反射 / 字段修改**：移除对 `static final` 字段的运行时修改。
- [ ] **Compose 规划**：评估 View → Compose 的迁移优先级。
- [ ] **图形**：确认渲染栈兼容 Vulkan 1.4 / ANGLE，关注 OpenGL ES 维护状态。

---

## 十二、写在最后

Android 17 本体偏"基础设施与方向定调"：**Compose 优先、大屏强制自适应、系统智能化（AppFunctions）**是三条主线，配合一连串隐私 / 安全 / 性能的强制收紧。真正面向用户的体验跃迁会在 **QPR1 及后续季度更新**中陆续到来。

对开发团队来说，现在就是动手的时间点——**提前在 Canary 上跑兼容性回归，按上面的 Checklist 逐项排雷**，避免在 targetSDK 升级时被强制行为变化打个措手不及。

---

### 参考资料

- [Android 17 features and changes list — Android Developers](https://developer.android.com/about/versions/17/summary)
- [Release notes — Android Developers](https://developer.android.com/about/versions/17/release-notes)
- [Android Developers Blog: Android 17 is here](https://android-developers.googleblog.com/2026/06/Android-17.html)
- [Android Developers Blog: The First Beta of Android 17](https://android-developers.googleblog.com/2026/02/the-first-beta-of-android-17.html)
- [Android Developers Blog: The Fourth Beta of Android 17](https://android-developers.googleblog.com/2026/04/the-fourth-beta-of-android-17.html)
- [This is every new feature in Android 17 — 9to5Google](https://9to5google.com/2026/06/16/this-is-every-new-feature-in-android-17-video/)
- [Android 17: Confirmed features, codename, leaks — Android Authority](https://www.androidauthority.com/android-17-3561251/)
