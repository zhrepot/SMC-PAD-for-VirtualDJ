# SMC-PAD for VirtualDJ（中文说明）

用于在 **VirtualDJ** 中使用 **M-VAVE SMC-PAD**（16 键无线 MIDI 打击垫控制器）的配置文件。

[English version](README.md)

---

## 目录

1. [项目介绍](#1-项目介绍)
2. [实现原理与方法](#2-实现原理与方法)
3. [如何选择配置文件与使用](#3-如何选择配置文件与使用)
4. [问答](#4-问答)
   - [4-1 硬编码按键问题](#4-1-硬编码按键问题部分按键不发送-midi)
   - [4-2 识别为多设备问题](#4-2-识别为多设备问题)
5. [目录结构](#5-目录结构)
6. [许可证](#6-许可证)

---

## 1. 项目介绍

本仓库提供 M-VAVE **SMC-PAD** 控制 VirtualDJ 所需的 **Device 定义文件** 与 **Mapper 映射文件**。

SMC-PAD 是一款 16 键无线 MIDI 打击垫控制器，具备：

- 16 个 RGB 背光打击垫（力度 + 触后）
- 8 个 360° 无极编码器
- 5 个可分配按钮：`LEFT` `RIGHT` `PLAY` `STOP` `RECORD`
- 硬件按钮（`BT`、`PAD BANK`、`KNOB BANK`、`SHIFT`、`Note Repeat`）——**不可映射**（见问答）

> 注意：RGB 灯光**无法通过 MIDI 控制**（待机颜色由 MidiSuite 软件设定，按压时变白）。这是硬件限制，不是映射问题。

---

## 2. 实现原理与方法

VirtualDJ 使用两类 XML 文件：

| 文件 | 位置 | 作用 |
| --- | --- | --- |
| **Device 定义** | `Documents\VirtualDJ\Devices\` | 定义硬件元素（打击垫、编码器、按钮）及设备识别方式（VID/PID）。 |
| **Mapper 映射** | `Documents\VirtualDJ\Mappers\` | 把定义的元素映射到 VDJ 动作。 |

本项目基于 SMC-PAD 的 **默认预设**（8 个预设槽位中的槽位 3–8）。

**所用 MIDI 布局（默认预设）：**

| 控件 | 类型 | 通道 | 取值 |
| --- | --- | --- | --- |
| 打击垫 1–16 | Note On/Off | CH10 | 36–51（按 DJ 习惯垂直镜像重排） |
| 打击垫力度 | Note On 力度 | CH10 | 0–127 → 0–100% 滑杆值 |
| 编码器 1–8 | 相对 CC | CH1 | 30–37（重排） |
| 按钮 | CC Push | CH1 | 25–29（`LEFT`–`RECORD`），按下=127，释放=0 |

**关键实现点：**

- 打击垫做了**垂直镜像**：使 1 号垫位于左上（DJ 习惯），而非 SMC-PAD 出厂的左下。
- 编码器做了**顺序重排**，匹配常见 DJ 旋钮排布。
- 打击垫**力度**通过为每个 Pad 额外定义 `<slider>` 元素暴露，敲击越重可驱动连续参数（滤波、效果量等）。

---

## 3. 如何选择配置文件与使用

### 3.1 选择

1. **Device 定义** —— 在 `devices/` 中根据你偏好的 deck 布局选择（如 `decks4` = 双面双层，`decks2` = 双 deck）。详见 `devices/README.md`。
2. **Mapper 映射** —— 在 `mappers/<相同布局>/` 中根据你偏好的按键功能布局选择。每个 mapper 文件夹内都有各自的 README 说明按键功能布局。

### 3.2 使用

1. 把选定的 **Device** XML 复制到：
   ```
   Documents\VirtualDJ\Devices\
   ```
2. 把选定的 **Mapper** XML 复制到：
   ```
   Documents\VirtualDJ\Mappers\
   ```
3. 重启 VirtualDJ。
4. 打开 **设置 → 控制器**，找到 SMC-PAD 并选择对应 Mapper。

---

## 4. 问答

### 4-1 硬编码按键问题（部分按键不发送 MIDI）

以下按键是**硬件编码**的，**不发送任何 MIDI 消息**，因此**无法映射**：

- `BT` —— 蓝牙开关
- `PAD BANK` —— 切换打击垫音高范围（36–51 ↔ 52–67）
- `KNOB BANK` —— 切换编码器 CC 范围（30–37 ↔ 38–45）
- `SHIFT` —— 用于预设 / 力度曲线 / 移调 / 八度的修饰键
- `Note Repeat` —— 内部音符重复功能

只有 **16 个打击垫**、**8 个编码器** 和 **5 个按钮**（`LEFT` `RIGHT` `PLAY` `STOP` `RECORD`）可映射。本项目有意**不定义**第二组 bank（音高 52–67 / CC 38–45）。

### 4-2 识别为多设备问题

**现象**：VirtualDJ 显示三个设备，例如 `SMC PAD`、`SMC PAD (MIDI IN 1)`、`SMC PAD (MIDI IN 2)`。

**原因**：SMC-PAD 暴露了**多个 MIDI 端口**，且所有端口共享同一 USB **VID/PID**。VirtualDJ 把每个 MIDI 端口当作独立控制器，因此同一份定义会匹配所有端口。

**解决**：在 **设置 → 控制器** 中，把多余端口的 **Mapping（映射）** 设为 **`None`**（或点击每个条目旁的电源开关），只保留承载数据的那个端口。这是多端口 MIDI 设备的正常现象。

---

## 5. 目录结构

```
ForGithub/
├── README.md                    ← 英文主页
├── README.zh-CN.md              ← 中文版（本文件）
├── LICENSE                      ← MIT 许可证
├── devices/                     ← Device 定义（多套）
│   ├── README.md                ← 如何选择 Device 定义
│   ├── SMC-PAD_decks4.xml
│   ├── SMC-PAD_decks2.xml
│   └── ...
├── mappers/                     ← Mapper 文件（按 device 布局分组）
│   ├── README.md                ← 如何选择 Mapper
│   ├── decks4/
│   │   ├── README.md            ← 该套按键功能布局说明
│   │   ├── SMC-PAD_decks4_basic.xml
│   │   └── ...
│   └── decks2/
│       ├── README.md
│       └── ...
└── docs/                        ← 补充文档
    └── layout.md                ← 打击垫/旋钮/按钮布局参考
```

---

## 6. 许可证

依据 **MIT License** 发布，见 [LICENSE](LICENSE)。
