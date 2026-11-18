# ZTools Flutter 像素级复刻开发计划

> 目标应用：[ZToolsCenter/ZTools](https://github.com/ZToolsCenter/ZTools)（开源 uTools 实现，Electron 41 + Vue 3 + TypeScript，支持 macOS / Windows）
> 本项目：使用 Flutter 对 ZTools 进行像素级一比一复刻，主目标平台为 **Windows 桌面端**（次目标 macOS）
> 文档版本：v1.0 ｜ 编制日期：2026-11-18 ｜ 状态：待评审

---

## 目录

1. [项目整体概述与复刻范围](#1-项目整体概述与复刻范围)
2. [技术栈选型与 Flutter 开发环境配置规范](#2-技术栈选型与-flutter-开发环境配置规范)
3. [页面拆解与开发排期](#3-页面拆解与开发排期)
4. [资源处理规范](#4-资源处理规范)
5. [测试与验收标准](#5-测试与验收标准)
6. [交付物清单与里程碑节点](#6-交付物清单与里程碑节点)

---

## 1. 项目整体概述与复刻范围

### 1.1 项目背景

ZTools 是一款键盘优先的效率启动器与插件平台：通过全局快捷键（Windows 默认 `Alt+Z`）呼出主面板，在同一输入框内搜索应用、文件与插件，内置剪贴板历史、插件市场、主题定制等能力。原应用基于 Electron 构建，本项目使用 Flutter 复刻其 **全部 UI 表现与交互逻辑**，并逐步实现对应的系统能力（搜索、剪贴板、窗口管理等）。

### 1.2 复刻目标与原则

| 原则 | 说明 |
| --- | --- |
| 像素级一致 | 所有页面在默认窗口尺寸（含 DPI 缩放 100%/125%/150%）下与原应用截图逐像素比对，UI 元素位置/尺寸/颜色偏差 ≤ 1px（颜色 ΔE ≤ 2） |
| 交互等价 | 键盘操作（↑↓←→ / Enter / Esc / Ctrl+K / Tab）、鼠标 hover、焦点流转、动画时长与缓动曲线与原应用一致 |
| 分层复刻 | 第一阶段完成全部 UI + 模拟数据；第二阶段接入真实系统能力（应用扫描、剪贴板监听、全局热键）；第三阶段实现插件系统与插件市场 |
| 可追溯 | 每个页面对应原应用截图基线（Baseline）与差异报告（Diff Report），纳入版本管理 |

### 1.3 需完整还原的功能模块清单

| # | 模块 | 原应用能力（来源：README / 官网） | 复刻范围 |
| --- | --- | --- | --- |
| M1 | 主面板（悬浮搜索窗） | 全局快捷键呼出/隐藏；单输入框即搜即用；拼音首字母搜索；正则匹配；历史记录；置顶应用；结果分组（应用/文件/插件/系统指令）；键盘导航 | UI + 搜索交互 + 本地应用扫描 |
| M2 | 搜索引擎 | Fuse.js 模糊搜索 + 拼音支持；权重排序；高亮匹配片段 | 算法行为还原（排序结果与原应用一致） |
| M3 | 工作台（悬浮条模式） | 官网展示的"悬浮条 工作台"模式，内置工具入口（计算器、JSON 格式化、时间戳转换等） | UI 还原 + 内置工具入口跳转 |
| M4 | 内置工具面板 | 计算器（科学函数/单位换算/汇率/日期推算）、JSON 格式化（校验/美化/压缩/生成 TS/Go/Rust 类型）、时间戳转换 | UI + 功能逻辑还原 |
| M5 | 剪贴板管理 | 历史记录、搜索、图片支持、原生监听（C++ 实现 → Flutter 侧用平台通道重写） | UI + 历史存储 + Windows 剪贴板监听 |
| M6 | 插件市场 | 在线插件列表、搜索、分类、安装/卸载/更新 | UI 还原 + 对接原官方插件源 API |
| M7 | 插件运行容器 | UI 插件（Web 界面）、无头插件（headless）；独立窗口运行插件 | 第一阶段仅 UI 占位，第三阶段实现 WebView 容器 + plugin.json 解析 + `ztools` API |
| M8 | 设置中心 | 通用设置、主题定制（系统/浅色/深色 + 6 主题色）、快捷键自定义、数据管理 | UI + 功能还原 |
| M9 | 主题系统 | 浅色/深色/跟随系统；6 种主题色切换；全局实时生效 | 完整还原 |
| M10 | 窗口管理 | 无边框悬浮窗、失焦自动隐藏、位置记忆、多显示器适配、系统托盘 | 完整还原（Windows 优先） |

### 1.4 页面/视图清单（复刻 UI 清单）

| 页面 ID | 页面名称 | 窗口形态 | 基线截图来源 |
| --- | --- | --- | --- |
| P01 | 主面板-默认态（空输入） | 无边框悬浮窗（约 640×560，以实测为准） | `.github/assets/main-light.png` / `main-dark.png` + 实机截图 |
| P02 | 主面板-搜索态（有输入/有结果） | 同 P01 | 实机截图 + `demo.gif` 逐帧 |
| P03 | 主面板-无结果态 | 同 P01 | 实机截图 |
| P04 | 主面板-Ctrl+K 动作面板 | 主面板内叠加层 | 官网演示 + 实机截图 |
| P05 | 工作台/悬浮条模式 | 细长悬浮条 | 官网 index.html 演示图 |
| P06 | 设置-通用 | 独立主窗口 | `.github/assets/settings.png` |
| P07 | 设置-主题定制 | 同 P06 | settings.png + 实机截图 |
| P08 | 设置-快捷键 | 同 P06 | 实机截图 |
| P09 | 设置-数据/关于 | 同 P06 | 实机截图 |
| P10 | 插件市场-首页/推荐 | 独立窗口或主窗口内 | `.github/assets/plugin-market.png` |
| P11 | 插件市场-搜索/分类列表 | 同 P10 | 实机截图 |
| P12 | 插件详情页 | 同 P10 | 实机截图 |
| P13 | 剪贴板面板 | 悬浮窗 | 实机截图 |
| P14 | 插件运行窗口（容器壳） | 独立无边框窗口 | 原应用插件运行截图 |
| P15 | 托盘菜单 + 右键菜单 | 系统级 | 实机截图 |

> **基线采集规范**：P01–P15 每页需采集 3 套基线——浅色主题、深色主题、6 主题色各抽查 1 色；分辨率统一 1920×1080 @100% 与 2560×1440 @150% 两档。基线图存放于 `assets/baseline/`（仅开发用，不打包进发布产物）。

### 1.5 关键交互细节还原清单（必须逐项验收）

- 全局快捷键 `Alt+Z` 呼出/隐藏主面板；失焦后自动隐藏（动画约 120–200ms，以实测帧为准）
- 输入框：自动聚焦、占位文案、光标样式、IME（中文输入法）兼容、拼音首字母匹配（如输入 `js` 命中「计算器/JSON」等）
- 结果列表：↑↓ 循环导航、Enter 执行首要动作、鼠标 hover 高亮、右侧动作提示文案、分组标题吸顶行为
- `Ctrl+K` 唤起动作面板：面板展开动画、子动作列表、Esc 逐级返回
- `Tab` 键补全 / 右侧预览行为（以实机为准）
- 设置页：主题色切换即时生效、开关控件动效、列表 hover 态
- 插件市场：卡片 hover 抬升/边框态、安装按钮状态机（未安装 → 下载中 → 已安装/更新）
- 剪贴板：文本/图片/文件三类条目渲染、置顶、删除、双击回贴
- 窗口：拖拽、位置记忆、多显示器边界约束、DPI 缩放模糊规避

---

## 2. 技术栈选型与 Flutter 开发环境配置规范

### 2.1 技术栈选型总览

| 层 | 选型 | 说明 |
| --- | --- | --- |
| 框架 | Flutter 3.35+（Dart SDK ^3.13） | 与当前仓库 `pubspec.yaml` 的 `sdk: ^3.13.7` 保持一致；锁定 Flutter 版本到 minor 级别 |
| 平台 | Windows 桌面（优先）、macOS（次期） | 复刻原应用的双平台支持；Android/iOS/Web 目录保留但不在本期范围 |
| 状态管理 | **Riverpod 3.x**（flutter_riverpod + riverpod_annotation） | 与原应用 Pinia 的 store 划分一一对应（每 store 一个 NotifierProvider），支持测试与热更新 |
| 路由 | go_router 15.x + 自研窗口路由表 | 多窗口（悬浮面板/主窗口/插件窗）各自独立 Navigator，go_router 用于窗口内页面 |
| 搜索 | 自实现 Dart 版 Fuse.js 等价算法 + `pinyin` 库 | 保证排序结果与原应用 Fuse.js 配置一致；配置项（阈值/权重/字段）从源码 `renderer` 目录对齐 |
| 本地存储 | LMDB 不适用 Dart → 选用 **Hive CE 4.x**（结构化）+ 原生 LMDB 仅作兼容迁移备选 | 键值结构与原 LMDB 键命名保持一致，便于数据迁移 |
| 剪贴板/原生能力 | 平台通道 + C++ 插件（`win32clipboard`、`super_native_extensions`、自写 Node-API 等价 FFI） | 原应用 C++ 剪贴板监听逻辑以 Dart FFI + Win32 API 重写 |
| 全局热键 | `hotkey_manager` / 自写 Win32 `RegisterHotKey` FFI | 支持用户自定义快捷键（设置页 M8） |
| 无边框窗口 | `window_manager` + `bitsdojo_window`（二选一，验证后锁定） | 实现悬浮窗、失焦隐藏、拖拽区域、位置记忆 |
| WebView（插件容器） | `webview_windows`（Windows） | M7 插件容器阶段引入 |
| 网络与市场 API | dio 5.x | 对接插件市场源 |
| 主题 | 自研 `ZTheme`（ThemeExtension + Token 体系） | 见 2.4 |
| 字体 | 与原应用一致的自托管字体（见 4.3） | 禁用系统默认字体回退差异 |

### 2.2 依赖库版本约束（pubspec.yaml 规范）

```yaml
environment:
  sdk: ^3.13.7

dependencies:
  flutter: { sdk: flutter }
  # 状态管理
  flutter_riverpod: ^3.0.0
  riverpod_annotation: ^3.0.0
  # 路由
  go_router: ^15.0.0
  # 桌面窗口能力
  window_manager: ^0.5.0
  # 本地存储
  hive_ce: ^4.0.0
  # 网络
  dio: ^5.7.0
  # 拼音搜索
  lpinyin: ^2.0.0
  # 剪贴板/拖拽等原生扩展
  super_native_extensions: ^0.9.0
  # 工具
  collection: ^1.19.0
  equatable: ^2.0.7
  intl: ^0.20.0

dev_dependencies:
  flutter_test: { sdk: flutter }
  flutter_lints: ^6.0.0
  riverpod_generator: ^3.0.0
  build_runner: ^2.4.0
  golden_toolkit / alchemist: ^0.11.0   # Golden 测试（二选一锁定）
  mocktail: ^1.0.4
```

**版本管理纪律**：

1. 所有第三方依赖使用 `^` 允许 patch 级升级，minor/major 升级需在 PR 中说明并在 `CHANGELOG-DEPS.md` 记录；
2. `pubspec.lock` 必须提交，CI 中使用 `flutter pub get --enforce-lockfile`；
3. 引入任何新依赖前需在计划评审中登记用途，禁止为单一 UI 细节引入大型 UI 库（当前仓库的 `material_ui: ^1.11.1` 若未使用则移除，复刻以自绘/定制 Material 组件为主）。

### 2.3 开发环境配置规范

| 项 | 规范 |
| --- | --- |
| Flutter SDK | 通过 fvm（Flutter Version Management）锁定：`fvm use 3.35.x`，仓库提交 `.fvmrc` |
| IDE | VS Code / Android Studio 均可，统一启用 `flutter_lints`（已有 analysis_options.yaml） |
| 代码生成 | `dart run build_runner build --delete-conflicting-outputs`，生成的 `.g.dart` 全部提交 |
| 提交规范 | Conventional Commits：`feat(P01): 主面板默认态布局`、`fix(clipboard): ...`，页面 ID 作为 scope |
| 分支策略 | `main` 受保护；`feat/PXX-页面名` 单页面单分支；合并需 Golden Diff 通过 |
| 目录结构 | 见下 |

**目录结构规范**（与原应用 renderer 结构映射）：

```
lib/
├── main.dart
├── app.dart                    # MaterialApp / 多窗口入口
├── core/
│   ├── search/                 # M2 搜索引擎（Fuse.js 等价 + 拼音）
│   ├── storage/                # Hive 封装、键名映射表
│   ├── native/                 # 平台通道/FFI（剪贴板、热键、窗口）
│   └── plugin/                 # M7 plugin.json 解析、ztools API
├── theme/
│   ├── z_theme.dart            # 主题入口（浅/深/系统）
│   ├── tokens/                 # 颜色/字号/间距 Token（见 4.2）
│   └── theme_colors.dart       # 6 主题色定义
├── features/
│   ├── launcher/               # P01–P05 主面板与工作台
│   ├── clipboard/              # P13
│   ├── market/                 # P10–P12
│   ├── settings/               # P06–P09
│   └── plugin_host/            # P14
├── widgets/                    # 跨页面复用组件（ZInput、ZListTile、ZSwitch…）
└── l10n/                       # 中英文案（与原应用 i18n 文案逐字一致）
```

### 2.4 主题配置规范

- 原 ZTools：系统 / 浅色 / 深色 三模式 × 6 主题色。Flutter 侧建立 **三态 Token**：
  - `mode`：`light | dark | system`（system 监听 `MediaQuery.platformBrightness` + Windows 深色模式注册表）
  - `accent`：6 枚主题色枚举（具体色值从原应用源码 `renderer` 的 CSS 变量/主题文件提取，记录进 `theme/tokens/colors.dart` 注释中标注来源版本 tag）
- 所有 UI 组件 **禁止硬编码颜色/尺寸**，一律引用 Token；Lint 层面通过自定义 lint 或 code review 强制；
- 主题切换必须 **无重建闪烁**：使用 `MaterialApp(theme/darkTheme/themeMode)` + `ThemeExtension`，切换动画时长与原应用一致；
- 每个页面 Golden 测试需覆盖 浅色/深色 × 至少 2 个主题色 = 4 组基线。

---

## 3. 页面拆解与开发排期

### 3.1 开发阶段总览（按优先级）

| 阶段 | 名称 | 包含页面/模块 | 优先级 | 出口条件 |
| --- | --- | --- | --- | --- |
| S0 | 基线采集与设计还原 | 全部 P01–P15 截图、色值/字号/间距提取 | P0 | 基线库 + Token 表建立完成 |
| S1 | 应用骨架与主题系统 | 多窗口骨架、无边框窗、托盘、热键、主题切换（M9/M10） | P0 | Alt+Z 呼出/隐藏可用，主题 4 组 Golden 通过 |
| S2 | 主面板 UI 全量还原 | P01–P05（模拟数据） | P0 | 逐页像素 Diff ≤ 1px |
| S3 | 搜索引擎真实接入 | M1/M2：应用扫描、文件搜索、拼音、历史、置顶 | P0 | 与原应用相同输入得到相同排序（抽样 30 组用例） |
| S4 | 设置中心 | P06–P09（M8） | P1 | 设置项全部可持久化并生效 |
| S5 | 内置工具 | P05 工作台 + M4 计算器/JSON/时间戳 | P1 | 功能用例通过 |
| S6 | 剪贴板管理 | P13（M5） | P1 | 文本/图片/文件监听与回贴 |
| S7 | 插件市场 | P10–P12（M6） | P1 | 拉取官方源、安装流程可用 |
| S8 | 插件容器 | P14（M7）：WebView、plugin.json、ztools API 子集 | P2 | 官方任一 UI 插件可运行 |
| S9 | 联调打磨与验收 | 全量回归、多机型（多分辨率/DPI）验证 | P0 | 第 5 章验收标准全通过 |

### 3.2 页面级拆解与还原精度要求

每页按下表格式登记（此处列出关键页示例，全部页面在 `docs/pages/PXX.md` 逐页登记）：

**P01 主面板-默认态**

| 项 | 要求 |
| --- | --- |
| 还原精度 | 布局：±1px；字号字重：完全一致；圆角/阴影：阴影用原 CSS box-shadow 换算（blur/spread/offset/alpha）；间距网格化对齐 |
| 结构 | 顶部输入框区（图标 + TextField + 快捷键提示）、结果列表区、底部状态栏 |
| 交互 | 启动即聚焦输入框；Esc 隐藏窗口；无输入时展示历史/置顶分组 |
| 验收 | Golden Diff（100% DPI）≤1px 差异像素占比 < 0.1% |

**P02 主面板-搜索态**

| 项 | 要求 |
| --- | --- |
| 还原精度 | 同 P01；匹配高亮颜色/背景完全一致 |
| 交互 | 输入防抖（与原应用一致，实测约 100–200ms）；↑↓ 导航滚动自动跟随；Enter 执行；Ctrl+K 叠加 P04 |
| 验收 | 30 组搜索用例（中文全拼/首字母/正则）结果排序与原应用一致 |

**P06–P09 设置中心**

| 项 | 要求 |
| --- | --- |
| 还原精度 | 左侧导航 + 右侧表单布局 ±1px；所有控件（开关/下拉/热键录入框）自绘以匹配原样式 |
| 交互 | 主题色切换即时生效全局；热键录制冲突检测；设置即时保存 |

**P10–P12 插件市场**

| 项 | 要求 |
| --- | --- |
| 还原精度 | 卡片网格尺寸、封面图占位、评分/下载量排版一致 |
| 交互 | 安装按钮状态机（未装/下载中%/已装/可更新）、下载进度条、错误重试 |

**P13 剪贴板面板**

| 项 | 要求 |
| --- | --- |
| 还原精度 | 文本条目两行截断省略、图片条目缩略图比例一致 |
| 交互 | 新条目顶部插入动画；双击回贴并隐藏面板；置顶/删除/搜索 |

### 3.3 排期（按周迭代，工作量以"页面"为单位滚动排布）

| 周次 | 任务 | 交付 |
| --- | --- | --- |
| W1–W2 | S0 基线采集（原应用实机截图 + 源码色值提取）；S1 骨架：多窗口、无边框窗、热键、托盘 | 基线库、Token 表、骨架 Demo |
| W3–W5 | S2：P01 → P02 → P03 → P04 → P05 顺序开发，每页 Golden 验收 | 主面板 UI 100% 还原 |
| W6–W7 | S3 搜索引擎（Fuse 等价实现、拼音索引、应用/文件扫描、历史与置顶持久化） | 搜索闭环 |
| W8–W9 | S4 设置中心 P06–P09 + 主题系统联调 | 设置闭环 |
| W10 | S5 内置工具（计算器、JSON、时间戳）+ P05 工作台 | 工具闭环 |
| W11–W12 | S6 剪贴板（FFI 监听、图片缩略图、存储上限策略） | 剪贴板闭环 |
| W13–W14 | S7 插件市场 P10–P12（对接官方 API、安装器） | 市场闭环 |
| W15–W17 | S8 插件容器（WebView 壳、plugin.json、ztools API 子集：通知/存储/剪贴板/窗口） | 插件可运行 |
| W18–W19 | S9 全量回归、性能优化（冷启动 < 300ms、呼出 < 100ms）、验收报告 | v1.0 发布候选 |

> 排期按"每个工作日可完成 1 个中型页面"估算，若并行 2 名开发者可压缩约 40%；每次迭代结束进行一次里程碑评审（见第 6 章）。

---

## 4. 资源处理规范

### 4.1 切图资源适配方案

| 资源类型 | 来源 | 处理规范 |
| --- | --- | --- |
| 应用图标/Logo | 原仓库 `.github/assets/icon.png` 及实机资源 | 从原应用安装包 `resources/` 提取原始 PNG/SVG；SVG 优先直接使用 |
| 图标字体/矢量图标 | 原应用 UI 图标 | 逐个从源码提取 SVG，转为 Flutter 自绘（`flutter_svg` 8.x）或 IconFont；禁止用 Material Icons 近似替代 |
| 位图切图 | 原应用静态图 | 提供 1x/2x/3x 三档（`assets/images/2.0x/` `3.0x/`），Windows 下按 DPI 缩放自动匹配 |
| 插件封面图 | 市场在线资源 | 走 CDN URL + 内存/磁盘二级缓存（`cached_network_image` 等价实现），本地仅缓存不进包 |
| 基线截图 | 开发比对用 | 存放 `assets/baseline/`，在 `pubspec.yaml` 中 **不注册**，仅测试引用 |

**命名规范**：`{page}_{component}_{state}@{scale}.png`，如 `p01_input_normal@2x.png`；全小写下划线。

### 4.2 颜色系统映射规则

1. **提取来源**：以原应用源码为准（renderer 目录中 CSS 变量 / Vue 主题文件 / Tailwind config），以实机取色器采样交叉校验；
2. 建立 **三层 Token**：
   - `Primitive`：原始色板（如 gray-50…900、accent-6 色 × 10 阶）
   - `Semantic`：`surface / background / textPrimary / textSecondary / highlight / border / hover / accent` 等，浅色与深色各一套映射表
   - `Component`：组件级（输入框背景、结果项选中背景…）引用 Semantic
3. 每个颜色标注：HEX 值、来源文件路径、对应原 CSS 变量名、浅/深色对照；
4. 验收容差：色值 ΔE(CIE2000) ≤ 2，即肉眼不可分辨；禁止动态计算透明度替代原设计给定的带 alpha 色值。

### 4.3 字体与文字系统映射规则

| 项 | 规范 |
| --- | --- |
| 字体族 | 与原应用渲染字体一致（Electron 下默认为系统 UI 字体栈，需实测：Windows 为 Segoe UI 系 / 中文 Microsoft YaHei UI）；将字体文件随包分发以消除系统差异 |
| 字号阶梯 | 从源码/DevTools 提取全部使用字号（如 12/13/14/16px），建立 `TypeScale` Token；Flutter 侧按逻辑像素 1:1 使用 |
| 字重 | Regular/Medium/SemiBold 与原一致；中文字体注意 Medium 是否存在，缺失时以描边/降级策略固定，禁止系统 faux bold |
| 行高与字距 | 原应用 CSS line-height / letter-spacing 换算为 `TextStyle(height, letterSpacing)`，逐条记录映射 |
| 光标/选区 | 输入框光标宽度、选区高亮色、IME 下划线样式对齐 |

### 4.4 布局与尺寸换算规则

- 原 CSS px（逻辑像素）→ Flutter 逻辑像素 **1:1**；窗口缩放由系统 DPI 处理；
- CSS `border-radius`、`padding`、`gap` 逐值映射进 `ZDimens` Token；
- CSS box-shadow → `BoxShadow(color, blurRadius=blur*2, offset, spreadRadius=spread)` 并以 Golden 比对微调；
- 动画：CSS transition 时长/缓动 → `Duration` + `Curves` 对照表（如 `cubic-bezier(0.4,0,0.2,1)` → `Curves.easeInOutCubicEmphasized` 近似并实测校正）。

---

## 5. 测试与验收标准

### 5.1 像素级对齐校验方法

1. **Golden 测试**：使用 `flutter test --update-goldens` 生成基线，`flutter test` 比对；每页覆盖 浅色/深色 × 2 主题色；
2. **实机截图比对**：Windows 下以 `window_manager` 固定窗口尺寸 + 100% DPI 运行，用自动化截图（PowerShell/`flutter screenshot`）与原应用同条件截图做 Diff（工具：pixelmatch / ImageMagick `compare`）；
3. **合格线**：
   - 差异像素占比 < 0.1%，且无 ≥2px 的连续错位块；
   - 文本渲染因字体光栅化产生的亚像素差异可豁免，但需人工确认视觉不可区分；
4. **DPI 校验**：125%/150% 缩放下无模糊、无错切（截图比对各一轮）；
5. 每个 PR 附 Diff 报告图片，未达标不得合入。

### 5.2 多机型/多环境适配要求

| 维度 | 要求 |
| --- | --- |
| 分辨率 | 1366×768 / 1920×1080 / 2560×1440 / 3840×2160 下布局不塌陷、不出现横向滚动 |
| DPI 缩放 | 100% / 125% / 150% / 200% 截图比对通过 |
| 多显示器 | 主面板跟随最近使用显示器居中；跨屏拖动边界约束正确 |
| 深浅系统色 | Windows 深色模式下 `themeMode: system` 生效 |
| OS 版本 | Windows 10 21H2+ 与 Windows 11 全部验收项通过 |
| 数据规模 | 剪贴板 1000 条、应用列表 500+、插件 100+ 时列表滚动帧率 ≥ 55fps |
| 性能 | 冷启动 ≤ 3s（首启含索引）；热键呼出响应 ≤ 100ms；搜索输入到结果更新 ≤ 50ms |

### 5.3 功能一致性验证标准

- **搜索一致性**：构造 30 组标准用例（中文全拼、首字母、英文前缀、正则、文件名片段），与原应用逐一比对排序 Top5 与高亮位置；
- **交互脚本测试**：`integration_test` 编写核心链路自动化（呼出 → 搜索 → Enter 执行；设置切换主题 → 全局生效；剪贴板复制 → 面板出现新条目）；
- **功能用例覆盖率**：M1–M8 每模块编写验收用例表（Given/When/Then），覆盖率 100% 的 P0 用例必须全过；
- **回归**：每个里程碑跑一次全量 Golden + Integration + 单元测试，输出回归报告。

---

## 6. 交付物清单与里程碑节点

### 6.1 交付物清单（全程可追溯）

| 类别 | 交付物 | 存放位置 |
| --- | --- | --- |
| 计划类 | 本开发计划、逐页还原登记表（P01–P15）、风险登记册 | 根目录 / `docs/pages/` |
| 设计基线 | 原应用截图基线库、色值/字体/间距 Token 表、动效对照表 | `assets/baseline/` / `lib/theme/` |
| 代码 | Flutter 源码（含生成代码）、平台通道 C++/FFI 代码 | `lib/` / `windows/runner` 等 |
| 测试 | Golden 基线、单元测试、Integration 测试、Diff 报告、回归报告 | `test/` / `docs/qa/` |
| 文档 | 架构说明、搜索算法对齐说明（Fuse.js 参数映射）、数据键名映射表（原 LMDB → Hive）、发布说明 | `docs/` |
| 产物 | Windows 安装包/便携包（对齐原应用分发形态）、macOS 包（次期） | CI 产物 |

### 6.2 里程碑节点

| 里程碑 | 时间点 | 验收内容（可验证的出口标准） |
| --- | --- | --- |
| **M0 基线就绪** | W2 末 | 基线库 15 页 × 2 主题齐备；Token 表评审通过；Alt+Z 呼出空窗口可用 |
| **M1 主面板还原** | W5 末 | P01–P05 Golden 全过（≤1px）；键盘导航交互用例全过 |
| **M2 搜索可用** | W7 末 | 30 组搜索一致性用例 ≥ 95% 一致；本地应用可被搜索并启动 |
| **M3 设置与工具** | W10 末 | P06–P09 + 内置工具功能用例全过；主题切换 Golden 全过 |
| **M4 剪贴板与市场** | W14 末 | 剪贴板三条目类型闭环；市场可安装官方插件 |
| **M5 插件容器** | W17 末 | 至少 1 个官方 UI 插件 + 1 个 headless 插件可运行 |
| **M6 v1.0 验收发布** | W19 末 | 第 5 章全部验收标准通过；安装包 + 验收报告交付 |

### 6.3 可追溯机制

- 每个页面建立 `docs/pages/PXX.md`：包含基线截图、Token 引用清单、交互用例、Golden 结果、变更记录；
- 所有里程碑评审产出签署记录（验收人 + 日期 + Diff 报告编号）；
- Git tag 对齐里程碑（`M1-main-panel` 等），发布候选从 tag 构建；
- 每周输出进度简报：已完成页面 / Golden 通过率 / 偏离计划项与应对。

---

## 附：风险与对策（节选）

| 风险 | 对策 |
| --- | --- |
| 原应用 UI 数值无法从源码完全提取（运行时计算样式） | 以 DevTools 实测 + 截图测量双通道校准 |
| Windows 无边框窗口 + IME/失焦隐藏的原生行为差异 | S1 阶段优先做窗口原型验证，必要时自写 Win32 原生插件 |
| Fuse.js 排序结果难以逐字节一致 | 以"Top5 结果一致"为验收线，参数对照表记录偏差 |
| 插件生态依赖 Electron 专属 API | M7 仅实现 ztools API 可行子集，不兼容项登记并在文档声明 |
| 中文字体跨机器渲染差异 | 自托管字体文件随包分发 |
