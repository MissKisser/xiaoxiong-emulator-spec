# 小熊模拟器 · 共享规格层

本仓库是**小熊模拟器**（Xiaoxiong Emulator）的共享规格层，**不含任何可执行代码**。

两端在运行时和测试中直接读取本仓库的契约文件，以规格而非共享代码来保证产品一致性。

## 三个仓库的关系

| 仓库 | 角色 | 技术栈 |
|---|---|---|
| **xiaoxiong-emulator-spec**（本仓库） | 共享规格 | JSON 契约，无可执行代码 |
| [xiaoxiong-windows](https://github.com/MissKisser/xiaoxiong-windows) | Windows 端 | C# / .NET 8 + WPF，QEMU + WHPX |
| [xiaoxiong-android](https://github.com/MissKisser/xiaoxiong-android) | Android 端 | Kotlin + Jetpack Compose，系统级容器 |

## 项目要做什么

在 Windows 和 Android 上提供一个可批量运行安卓实例的模拟器。用户在宿主上创建多个互相隔离的安卓环境，各自有独立的数据、身份和网络边界；对宿主上的应用而言，这些实例表现为正常的独立安卓设备，而不是可被轻易识别的虚拟机或被 root 的设备。

## 架构概览

两端的技术路线差异达数量级，没有可共享的实现代码：

**Windows 端 —— 硬件虚拟化**
以 QEMU 承载安卓系统镜像，通过 WHPX 调用 Windows Hypervisor Platform 获得接近原生的运行性能。每个实例是独立的虚拟机，拥有自己的虚拟硬件与磁盘镜像。这是**对抗性**路线：宿主确实存在虚拟化痕迹，需要主动隐藏。

**Android 端 —— 系统级容器**
在安卓框架层重定向系统服务，让容器内的安卓应用看到一套完整的系统环境。宿主上并不存在传统的虚拟机或虚拟设备节点。这是**结构性**路线：痕迹不是被藏起来，而是从未产生，因此无从检测。

## 为什么是三个仓库

共享代码会牺牲两端各自的最优解：抽象层会阻碍单端的调试链路，并让一端拖累另一端的迭代速度。真正让人认出「同一个产品」的是设计令牌、术语表、数据契约和品牌资产——这几类文件合计只有一两千行，约束力更强、成本更低。

因此两端**共享规格，不共享代码**，跨端约束通过 Schema 双向兼容测试保证：各端都必须能解析对方提供的样例文件。

## 规格目录

```
spec/
├─ tokens/
│  └─ design-tokens.json        设计令牌：品牌色 / 间距 / 字号 / 圆角 / 动效
├─ terminology.json             双端统一术语表，含禁用近义词
├─ version.json                 产品版本与规格版本，及各自的递增规则
├─ baseline.json                两端共同的性能基线；目标值与实测值分列
├─ brand/                       品牌资产：标识、应用图标、配色映射与使用约束
│  ├─ logo-mark.svg             主标识（纯图形，浅色底）
│  ├─ logo-mark-inverse.svg     主标识反色版（深色底）
│  ├─ logo-horizontal.svg       横版标识（图形 + 中英文字，浅色底）
│  ├─ logo-horizontal-inverse.svg 横版标识反色版（深色底）
│  ├─ icon-app.svg              方形应用图标，窗口与任务栏
│  ├─ palette.json              语义名到设计令牌键的映射与实测对比度
│  └─ README.md                 用途、最小尺寸、留白与深浅背景约束
├─ icons/                       功能图标集：纯 SVG，色值一律取自设计令牌
│  ├─ README.md                 图标清单、用途与适用场景
│  └─ *.svg                     实例、快照、镜像、交互、传输、状态与风险类图标
└─ schema/
   ├─ instance.schema.json      实例配置契约（含 display 显示设置）
   ├─ image.schema.json         镜像清单契约（含 boot 推荐引导配置）
   ├─ snapshot.schema.json      快照元数据契约
   ├─ projection.schema.json    投屏会话契约
   ├─ filetransfer.schema.json  文件传输任务契约
   ├─ module.schema.json        模块清单与安装状态契约
   ├─ application.schema.json   应用管理契约
   ├─ terminology.schema.json   术语表结构定义
   └─ fixtures/                 双向兼容测试样例
      ├─ instance.windows.json  Windows 端样例
      ├─ instance.android.json  Android 端样例
      ├─ image.json             镜像样例
      ├─ snapshot-minimal.json  快照样例（仅必填字段）
      ├─ snapshot-full.json     快照样例（覆盖全部可选字段）
      ├─ projection-minimal.json 投屏样例（仅必填字段）
      ├─ projection-full.json   投屏样例（覆盖全部可选字段）
      ├─ filetransfer-minimal.json 传输样例（仅必填字段）
      ├─ filetransfer-full.json 传输样例（覆盖全部可选字段）
      ├─ module-minimal.json    模块样例（仅必填字段）
      ├─ module-full.json       模块样例（覆盖全部可选字段）
      ├─ application-minimal.json 应用样例（仅必填字段）
      └─ application-full.json  应用样例（覆盖全部可选字段）
```

## 两端如何引用

```bash
git submodule add https://github.com/MissKisser/xiaoxiong-emulator-spec.git spec
git commit -m "chore: 引入共享规格层"
```

克隆含子模块的仓库时使用 `git clone --recursive`，或对已有仓库执行 `git submodule update --init`。

## 硬性规则

1. 两端代码**不得**写字面量颜色、字号、间距，一律引用 `design-tokens.json` 的键
2. 同一个概念在两端**必须**使用 `terminology.json` 中的同一个词，且不得使用其中列出的禁用近义词
3. 任何一端新增字段，另一端必须能解析且不丢字段——由双向兼容测试在 CI 中保证
4. `image.json` 的 `verified` 字段只填实测结论，`untested` 不得臆测改为 `pass`
5. 规格变更必须先改本仓库，再更新两端代码；不允许某端私自扩展字段语义
6. `baseline.json` 的 `measured` 只填真实实测值，**不得由 `target` 推导或估算**；无法测量时保持 `null` 并写明 `blockedBy`
7. 版本号一律从 `version.json` 读取，两端代码与关于页**不得硬编码任何版本字符串**；产品版本与规格版本是两个独立序列

## 目标能力基线

| 能力 | 要求 |
|---|---|
| 可获取 root | 每个实例内可获得 root，且重启后保持 |
| 系统可写 | 系统分区可写，可刷入模块与修改 |
| 设备标识独立 | 每实例的序列号、Android ID、IMEI 强制独立，不可复用 |
| 网络隔离 | 默认仅回环暴露，跨网段暴露需阻断式确认 |
| 多开 | 常规桌面级宿主上支持多个实例并行 |

性能指标与更细的能力边界在两端各自仓库中定义，跨端一致性由本仓库的契约保证。

## 许可证

本项目采用 **Apache License 2.0**。完整条款见根目录 `LICENSE`。

与依赖的关系：QEMU 以独立进程方式调用，不构成链接，其 GPLv2 不影响本项目授权。Bliss OS 基于 AOSP（Apache-2.0）。若日后将隐藏栈模块（SUSFS / PathMask，GPL 系）以源码形式并入本仓库，需重新评估授权策略。