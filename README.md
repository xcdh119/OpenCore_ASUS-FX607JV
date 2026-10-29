# OpenCore ASUS FX607JV

⚠️使用前请仔细阅读

这是 华硕天选5Pro酷睿版 （具体型号ASUS FX607JV）  笔记本的 OpenCore EFI 配置，针对以下硬件和 macOS 使用场景整理。配置处于持续测试阶段，适合已经了解 OpenCore 和黑苹果基本操作的用户参考。

> 对于其他设备：EFI 只建议在相同或相近硬件上测试。使用前请备份原系统 EFI 和重要数据。

<img width="2560" height="1600" alt="IMG_20261023_211359" src="https://github.com/user-attachments/assets/eb06505a-2ba6-4c35-9439-2fcc9b8ec741" />

## 硬件信息

| 项目 | 配置 |
| --- | --- |
| 机型 | ASUS FX607JV |
| CPU | Intel Core i9-13980HX |
| 独立显卡 | NVIDIA GeForce RTX 4060 Laptop GPU |
| 内存 | 英睿达 DDR5-5600 |
| 固态硬盘 | 宏碁 GM7（联芸主控） |
| 无线网卡 | Intel Wi-Fi 6E AX201 |
| 有线网卡 | Realtek RTL8111 |
| 声卡 | Realtek ALC256/ALC3246 |
| 触控板 | 具体设备请在设备管理器中确认 |
| 内置屏幕 | 友达 B160QAN02.7 |

## 当前配置

- SMBIOS：`MacBookPro16,1`。
- 已包含 CPUFriend、CPUFriendDataProvider、CpuTopologyRebuild 和 FX607HWP 等 CPU 相关组件。
- 已包含 VoodooI2C、VoodooPS2Controller 及 FX607PanelSize 等触控板和输入设备组件。
- 无线网络使用 `itlwm.kext`，有线网络使用 `RealtekRTL8111.kext`。
- 已包含 USB 定制、NVMeFix、AppleALC、IntelBluetoothFirmware 等常用组件。
- `SimpleHDA.kext` 主要用于 macOS 26 的声音修复；macOS 15 及更低版本尚未充分测试，使用该驱动是否会影响声音功能目前尚不确定，请按系统版本自行验证。
- `OpenCore.efi` 和配置文件均来自本仓库当前版本，升级组件前请先验证兼容性。

如果声音受到影响，请使用 **OpenCore Configurator.app** 或 ProperTree 编辑 `config.plist`，在 `Kernel → Add` 中找到 `SimpleHDA.kext`，将其 `Enabled` 设为 `False`（取消启用），保存后重启。

## 已知问题

### 睡眠

10月27日提交：睡眠没完全修复，能进入睡眠状态，但尝试从睡眠状态唤醒时就会出现秒重启的问题，如果连睡眠状态都进不去直接秒重启的话，在 macOS 终端中输入这些命令并回车：
```bash
sudo pmset autopoweroff 0
sudo pmset powernap 0
sudo pmset standby 0
sudo pmset proximitywake 0
sudo pmset tcpkeepalive 0
sudo pmset hibernatemode 0
sudo pmset tcpkeepalive 0
sudo pmset ttyskeepawake 0
sudo pmset womp 0
```

### 解码

硬件解码存在问题，NVIDIA 驱动原因，部分软件无法使用硬件解码。需要硬件解码的应用可能退回软件解码或出现功能异常。

### 隔空投送

隔空投送可以打开，如果无线网卡使用 `itlwm.kext + HeliPort.app` 方案可能不能用；目前已经支持原生 Wi-Fi 但是能不能用还没来得及测试，先把更新推送了。

### Apple 服务

iMessage.app、FaceTime/电话.app无法使用。即使添加了 `en0` 网卡，待研究。

### CPU 睿频

CPU 最高睿频似乎卡在 4.4 GHz 的固定频率，可能与大小核调度有关。待研究。

### Wi-Fi

这个 EFI 里放了两套无线网络方案：
1. 使用 `itlwm.kext + HeliPort.app` ，但不要把 HeliPort 的可用误认为 macOS 原生 Wi-Fi。勾选 `itlwm.kext` 并取消勾选 `Airportltlwm.kext` ，启动参数删除 `itlwm_he=1` ，在系统内安装 app 即可。`HeliPort.app` 已收录在 [SoftwarePackage 发布页](https://github.com/xcdh119/OpenCore_ASUS-FX607JV/releases/tag/SoftwarePackage)
2. 使用原生 Wi-Fi ，和方案1相反，但速率可能存在问题，以下是使用[测速网](https://www.speedtest.cn/)的测速结果（两台测速设备均在同一固定位置）：

| 测速类型 | 地区 | 节点 | 上传,下载 (Mbps) |
| --- | --- | --- | --- |
| macOS 以太网 | 石家庄移动 | 河北省石家庄移动 | 112,1083 |
| macOS Wi-Fi | 石家庄移动 | 河北省石家庄移动 | 120,36 |
| macOS Wi-Fi | 石家庄电信 | 九天网络-电信 | 111,338 |
| Win11 Wi-Fi | 石家庄移动 | 河北省石家庄移动 | 117,269 |
| Win11 Wi-Fi | 石家庄电信 | 九天网络-电信 | 86,291 |
| vivo Wi-Fi | 石家庄移动 | 河北省石家庄移动 | 116,667 |
| vivo Wi-Fi | 石家庄电信 | 九天网络-电信 | 116,583 |

## 使用前准备

1. 备份原系统 EFI 和重要数据。首次测试建议准备一个可用的 Windows 或 macOS 恢复启动盘。
2. 使用 GenSMBIOS 等工具生成自己的 `SystemSerialNumber`、`MLB`、`SystemUUID` 和 `ROM`，不要直接使用仓库中的设备信息。
3. 按下方说明设置独显直连、关闭安全启动与 VMD，并准备安装文件。
4. 出现无法启动时，使用备份 EFI 恢复。
5. 相近的笔记本，可能需要重新定制ACPI，睿频补丁也可能需要重新生成。
6. `-v`debug跑码，仓库中的plist默认不启用跑码

> **警告：不要使用 OCAT（OpenCore Auxiliary Tools）编辑或升级此 EFI。OCAT 可能造成配置数据丢失；即使数据库中的 OpenCore 版本已更新到与当前 EFI 相同的版本，也仍可能发生问题。**

## 安装前的 BIOS 设置

1. 在奥创控制中心、G-Helper 或 BIOS 中，将显示模式切换为 **独显直连（only dGPU）** （此笔记本支持MUX）。

2. 在 BIOS 中进行以下设置：
- 使用 UEFI 启动模式。
- 关闭安全启动（Secure Boot）。
- 关闭 VMD。

### 如果关闭 VMD 后 Windows 无法启动

操作前请备份重要数据；启用 BitLocker 或设备加密时，请先保存恢复密钥，或直接关闭BitLocker。
原有 Windows 如果依赖 VMD 存储控制器，直接关闭 VMD 可能导致无法启动。请先恢复 VMD，让 Windows 正常启动，再通过安全模式切换存储控制器：

1. 重新开启 VMD，进入 Windows，打开设备管理器，找到存储控制器，把VMD相关的卸载掉（注意是卸载设备，不要卸载驱动！！！）。
2. 在开始菜单的电源选项中，按住 Shift 并点击“重启”，进入高级启动菜单。
3. 依次选择“疑难解答”→“高级选项”→“启动设置”→“重启”。
4. 重启时进入 BIOS，关闭 VMD，保存并重启。使用机型对应的 BIOS 快捷键。
5. 在启动设置中选择安全模式。成功进入 Windows 后，正常重启，再检查存储控制器是否已切换为标准 NVM Express 控制器。

如果切换后仍无法启动，请恢复原来的 VMD 设置，并尝试使用其他方法...

## 安装与使用

本仓库提供的是 **OC 文件夹内容**，不是完整的安装盘。macOS 在线恢复文件和配套的 `BOOTx64.efi` 已放在 [SoftwarePackage 发布页](https://github.com/xcdh119/OpenCore_ASUS-FX607JV/releases/tag/SoftwarePackage)，安装时还需要将本仓库内容放入 `EFI/OC/`。

### 文件结构

U 盘根目录或用于无 U 盘安装的 EFI 分区根目录，按以下层级放置文件：

```text
安装介质根目录/
├── com.apple.recovery.boot/
│   ├── BaseSystem.chunklist
│   └── BaseSystem.dmg
└── EFI/
    ├── BOOT/
    │   └── BOOTx64.efi
    └── OC/                    ← 本仓库内容放在这里
        ├── ACPI/
        ├── Drivers/
        ├── Kexts/
        ├── Resources/
        ├── Tools/
        ├── OpenCore.efi
        └── config.plist
```

`com.apple.recovery.boot` 与 `EFI` 文件夹同级，不要将恢复镜像放进 `EFI/OC/`。恢复镜像和 `BOOTx64.efi`（如果没有这个.efi文件的话） 可从 [SoftwarePackage 发布页](https://github.com/xcdh119/OpenCore_ASUS-FX607JV/releases/tag/SoftwarePackage) 获取。

### U 盘安装

1. 准备一个容量至少 8 GB、使用 FAT32 文件系统的 U 盘。
2. 按上方结构放入恢复镜像、`BOOTx64.efi`，并将本仓库内容放入 `EFI/OC/`。
3. 从 BIOS 启动菜单选择 U 盘的 UEFI 启动项，再进入 macOS 恢复环境安装系统。

### 无 U 盘安装

可将安装文件放到电脑硬盘的 EFI 分区，层级与上方结构相同。该方案通常需要将 EFI 分区扩大到约 2 GB，以容纳恢复镜像和引导文件。

> **不建议为此直接扩展现有 EFI 分区：调整分区存在损坏现有引导文件、导致系统无法启动的风险。优先使用 U 盘安装。**

如果仍选择无 U 盘安装，请先备份 EFI 分区和重要数据，再准备足够空间，将 `com.apple.recovery.boot` 放在分区根目录，与 `EFI` 文件夹同级，并在 BIOS 中添加对应的 UEFI 启动项。

### 安装完成后

系统安装完成后，先将本仓库 OC 内容放入电脑硬盘 EFI 分区的 `EFI/OC/`，保留配套的 `EFI/BOOT/BOOTx64.efi`，并在 BIOS 中添加对应启动项。确认可以从硬盘正常进入 macOS 后，再进行显卡驱动安装。

### NVIDIA 显示驱动

本机黑苹果显示驱动由 [nvidia-macos-driver](https://github.com/nullmoth/nvidia-macos-driver/releases) 项目提供。驱动有两种安装方式：

1. **DMG 安装**：在 macOS 中打开对应版本的 DMG 安装包。使用 DMG 打驱动前，必须先把 OC 放到电脑硬盘的 EFI 分区；如果 OC 只放在 U 盘或其他外部介质中，驱动无法正常安装或打入系统。
2. **手动安装**：下载 `nullmoth-nvidia-{版本}.tar.gz`，解压后按照项目说明手动复制和安装驱动文件。文件名中的 `{版本}` 替换为实际版本号。

驱动版本请从 [nvidia-macos-driver Releases](https://github.com/nullmoth/nvidia-macos-driver/releases) 选择。安装或更新驱动前请保留可用的硬盘 EFI 和备用启动介质，避免驱动异常时无法进入系统。

**10月29日：从 [NVIDIA driver 2.6.3](https://github.com/nullmoth/nvidia-macos-driver/releases/tag/v2.6.3) 版本起需要关闭 FX607PanelSize.kext 这个驱动才能正常使用独显内屏，这个驱动原本是用于修复 HiDPI 和显示器尺寸识别的，现在 nullmoth 已修复此问题，启用这个驱动并使用了 2.6.3 及以后版本的驱动会出现内屏卡 AppleLogo 的问题。**

## Wi-Fi、EN0 与 Apple 服务

[OpenCore_HUANANZHI-X79-Q67_2690V2_RX570：EN0 网卡设置说明](https://github.com/xcdh119/OpenCore_HUANANZHI-X79-Q67_2690V2_RX570)

即使完成 `en0` 设置并更换了设备序列号，Apple 服务仍可能无法使用。

## 文件说明

- `ACPI/`：处理电源管理、设备识别、USB 和平台兼容性的 SSDT。
- `Kexts/`：网络、声卡、蓝牙、触控板、CPU、电源管理和其他硬件驱动。
- `Drivers/`：OpenCore 启动所需的 UEFI 驱动。
- `Tools/`：OpenCore 工具组件。
- `config.plist`：本 EFI 的 OpenCore 配置文件。

修改 `config.plist` 可使用 OpenCore Configurator.app 或 ProperTree，其中 **OpenCore Configurator.app** 更适合本项目的图形化操作。修改前请备份原配置，并在修改后使用 `ocvalidate` 检查配置。更新 Kext 或 OpenCore 后，应重新验证配置和启动参数。

## 免责声明

这是一份个人测试配置，适用所有 i9-13980HX 版本的天选5Pro机型，不保证适用于除FX607JV之外其他所有天选5系列或其他天选相近型号。睡眠、硬件解码、隔空投送、Apple 服务和 CPU 调度问题目前仍未完全解决。`SimpleHDA.kext` 适用于 macOS 26 的声音修复，在 macOS 15 及更低版本上的影响也尚未确认，请在使用前备份原配置并自行测试。

本文档编写较为仓促，部分内容由 AI 协助整理，可能存在疏漏或错误，请谅解，并以实际硬件、系统表现和项目最新说明为准。

## 相关项目

- [OpenCore](https://github.com/acidanthera/OpenCorePkg)
- [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [itlwm](https://github.com/OpenIntelWireless/itlwm)
- [HeliPort](https://github.com/OpenIntelWireless/HeliPort)
- [NVIDIA macOS Driver](https://github.com/nullmoth/nvidia-macos-driver)
- [SimpleHDA](https://github.com/laobamac/SimpleHDA)
- [itlwm (AirportItlwm 原生 Wi-Fi)](https://github.com/laobamac/itlwm)
