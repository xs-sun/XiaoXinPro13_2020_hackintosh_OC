# 💻 联想小新 Pro 13 2020 (Intel) OpenCore 黑苹果 EFI

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.0+-blue.svg)](https://github.com/acidanthera/OpenCorePkg)
[![macOS](https://img.shields.io/badge/macOS-Ventura%20|%20Sonoma%20|%20Sequoia-brightgreen.svg)](https://www.apple.com/macos/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

本仓库提供适用于 **联想小新 Pro 13 2020 款（Intel 处理器）** 的 OpenCore 引导配置。

特别优化了 **2.5K HiDPI 屏幕色深与 SwiftUI 系统设置卡顿问题**、解锁了 **外接 4K@60Hz 视频输出**，并附带华星光电 2.5K 屏幕专属校色 ICC 文件。

---

## 📋 电脑硬件配置

| 组件 | 详细型号 / 参数 | 驱动状态 |
| :--- | :--- | :--- |
| **设备型号** | 联想小新 Pro 13 2020 (Lenovo XiaoXin Pro 13 2020 Intel) | 完美兼容 |
| **处理器 (CPU)** | Intel Core i5-10210U / i7-10510U / i5-1035G4 | 完美变频 (XCPM) |
| **集成显卡 (iGPU)**| Intel Iris Plus Graphics 655 (仿射 `0x3EA50009`) | 完美 Metal 3 加速 |
| **独立显卡 (dGPU)**| NVIDIA GeForce MX350 | ❌ 已通过 `-wegnoegpu` 屏蔽 (省电) |
| **显示屏 (Display)**| 13.3 英寸 2.5K (2560x1600) 16:10 QHD IPS | 完美 HiDPI / 24-bit 色深 |
| **声卡 (Audio)** | Realtek ALC257 (`layout-id = 11`) | 扬声器 / 耳机 / 麦克风正常 |
| **无线 / 蓝牙** | Intel Wireless-AC / AX200 / AX201 | Wi-Fi 与蓝牙均正常运行 |
| **触控板 (Trackpad)**| I2C 触摸板 (VoodooI2C + SSDT 优化) | 多指手势流畅 |
| **电池管理** | 联想专属 ACPI 补丁 (`SSDT-OCBAT1`) | 剩余电量显示 / 充电正常 |

---

## 🛠️ 工作状态 (What's Working)

- [x] **图形加速**：Intel Iris Plus Graphics 655 显卡硬解与 Metal 3 加速全开。
- [x] **屏幕显示**：2.5K 屏幕 2x HiDPI 正常缩放，调整亮度快捷键正常。
- [x] **性能与流畅度优化**：注入 `-no30kexts` 强行锁定 24 位 (8-bit) 色深，彻底消除 macOS Ventura/Sonoma/Sequoia“系统设置”页面及 SwiftUI 界面点击顿挫卡顿。
- [x] **视频输出**：通过 Type-C 扩展坞输出 HDMI / DisplayPort 支持 **4K @ 60Hz**。
- [x] **音频输出**：内建扬声器、3.5mm 耳机接口切换、板载麦克风正常。
- [x] **网络与蓝牙**：Intel Wi-Fi 与 Intel 蓝牙驱动正常，网络挂载为内建 `en0`。
- [x] **系统升级**：配合 `RestrictEvents` 与 `revpatch=sbvmm` 参数，支持在线 OTA 收到 macOS 系统更新推送。
- [x] **触控板**：支持 macOS 官方全套多指手势操作。
- [x] **睡眠唤醒**：合盖睡眠、开盖唤醒及电源管理状态正常。
- [x] **双系统兼容**：完全兼容 Windows 11 / macOS 双系统引导。

---

## ⚡ 核心优化亮点

### 1. 2.5K 屏幕 & 4K 外接屏 24-bit (8-bit) 色深修复
在 macOS 新版（Ventura / Sonoma / Sequoia）中，由于系统默认将 2.5K 及外接高分屏误识别为 30 位彩色 (10-bit 色深)，导致 Intel 核显吞吐量爆表、系统设置界面点击有极强的顿挫卡顿感。
* 本 EFI 在 `boot-args` 中添加了 **`-no30kexts`** 强制全屏锁定 24 位 (8-bit) 标准色深，显卡渲染负担大幅减轻，界面极其顺滑。

### 2. 华星光电 2.5K 屏 ICC 校色文件
仓库附带小新 Pro 13 华星光电面板专属校色文件：`Lenovo Pro13 2019-1.icc`。
* **安装方法**：将该 `.icc` 文件复制到 `~/Library/ColorSync/Profiles/` 目录下，然后在 **系统设置 -> 显示器 -> 色彩描述文件** 中勾选 `Lenovo Pro13 2019-1` 即可。

### 3. 解锁 HDMI 4K@60Hz 输出
在 `DeviceProperties` 中保留并优化了 `enable-hdmi20` 和 `enable-max-pixel-clock-override`，支持通过 Type-C 转换器外接 4K 显示器并跑满 60Hz 刷新率。

---

## ⚙️ 启动参数 (boot-args) 说明

```xml
<key>boot-args</key>
<string>-wegnoegpu keepsyms=1 forceRenderStandby=0 -no30kexts revpatch=sbvmm</string>
```

| 参数 | 作用说明 |
| :--- | :--- |
| `-wegnoegpu` | 屏蔽 NVIDIA MX350 独显，防止发热与耗电 |
| `keepsyms=1` | 保留崩溃日志符号表，方便排错 |
| `forceRenderStandby=0` | 禁用 Intel 核显 RC6 省电待机，防止闪屏/假死 |
| `-no30kexts` | 禁用 10-bit 深彩色，强制 24 位 (8-bit) 色深，修复系统设置卡顿 |
| `revpatch=sbvmm` | 配合 RestrictEvents 欺骗苹果升级服务器，允许在线 OTA 升级 macOS |

---

## ⚠️ 使用须知与注意事项 (IMPORTANT)

1. **重新生成三码 (SMBIOS)**：
   * 出于隐私和 Apple 服务校验安全考虑，**请勿直接使用 EFI 中的默认序列号**。
   * 请使用 [OpenCore Auxiliary Tools (OCAT)](https://github.com/icooky/OCAT) 或 **ProperTree** 打开 `EFI/OC/config.plist`，在 `PlatformInfo -> Generic` 中重新生成属于你自己的三码：
     * `SystemProductName`: `MacBookPro15,4`
     * `SystemSerialNumber`
     * `SystemUUID`
     * `MLB`
2. **首次安装/升级建议**：
   * 建议在安装或升级系统前先备份好个人数据与原始 EFI 分区。

---

## 👏 致谢 & 参考项目 (Credits & References)

- [lovesakuratears/XiaoXinPro13_hackintosh_OC](https://github.com/lovesakuratears/XiaoXinPro13_hackintosh_OC) 提供基础 EFI 架构与配置参考。
- [Acidanthera](https://github.com/acidanthera) 提供 OpenCore, Lilu, WhateverGreen, AppleALC, VirtualSMC, RestrictEvents 等卓越开源驱动。
- [OpenCore Legacy Patcher (OCLP)](https://github.com/dortania/OpenCore-Legacy-Patcher) 项目团队。
- [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) 团队提供触摸板驱动支持。

---
*欢迎 Star 🌟 支持！祝你黑苹果体验愉快！*
