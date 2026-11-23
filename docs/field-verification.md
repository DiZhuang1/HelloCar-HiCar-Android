# 海豹06 HiCar 车机软件 · 验证阶段操作手册

> 目标：在正式移植成车机 APK 之前，先验证 HelloCar（全网唯一公开的 HiCar 车机端逆向实现）的协议链路真实可用——即用您的荣耀 Magic7 Pro 成功连上运行 HelloCar 的 Linux 主机，看到手机 HiCar 界面输出到主机。
> 更新日期：2026-10-29

---

## 一、验证目标与判据

验证链路（HiCar 无线连接四阶段，全通即验证成功）：

| 阶段 | 内容 | 验证判据 |
|---|---|---|
| 1 | BLE 发现 | 手机 HiCar 搜索到名为 `HelloCar` 的设备 |
| 2 | 蓝牙配对 | 手机与主机完成蓝牙配对（PIN 来自 `show-pin`） |
| 3 | 认证 | 配对后进入 HiCar 会话（HiChain 认证通过） |
| 4 | Wi-Fi 热点 + 投屏 | 主机出现 5GHz 热点（SSID HelloCar），手机 HiCar 桌面投出，`record_media.py` 落盘 H.264 视频与 PCM 音频 |

**成功判据**：第 4 阶段落盘出可播放的 `video.h264`（手机桌面画面）。若只到第 2/3 阶段即断，记录断点与日志（见第八节），用于判断是 HelloCar 协议问题还是 MagicOS 11 兼容问题。

**重要风险预告知**：HelloCar 仅声明目标 HarmonyOS 4.2；您的 MagicOS 11 属荣耀系新版本，HiCar 手机端实现可能有差异。若验证失败，需先区分「HelloCar 本身不可用」与「MagicOS 11 不兼容」——前者意味着整条逆向路线不成立（转方案），后者可尝试借鸿蒙 4.2 华为手机复测。

---

## 二、硬件采购清单（¥50–90 左右）

验证主机是 Linux 虚拟机，虚拟机的 USB 直通需要两个外接适配器（内置 Wi-Fi 走 PCIe 无法直通、内置蓝牙可先尝试直通，不稳再外接）：

| 设备 | 要求 | 选购关键词 | 参考价 |
|---|---|---|---|
| USB Wi-Fi 适配器 | 双频（支持 5GHz）+ 支持 AP 模式（SoftAP）；Linux 驱动成熟 | `8812au usb 网卡`（RTL8812AU 芯片，Linux 有 rtl8812au-dkms 驱动）或 `mt7612u usb 网卡` | ¥30–80 |
| USB 蓝牙适配器（备用） | BR/EDR + BLE 双模（CSR 芯片，BlueZ 原生支持） | `csr 4.0 蓝牙适配器 usb` | ¥10–20 |

> 说明：内置 MediaTek 蓝牙通常走 USB 内部总线，VirtualBox 可尝试直通（先到设备管理器把「MediaTek Bluetooth Adapter」禁用/解除占用再直通）；不行就插 USB 蓝牙适配器。

---

## 三、第一步（现在就能做）：GitHub Actions 出包

您有 GitHub 账号，走 fork 出包最快，本机不用装任何 Linux 工具：

1. 登录 GitHub，打开 <https://github.com/Aki894/HelloCar> → 右上角 **Fork**（保持默认即可）
2. 进入您账号下的 `HelloCar` 仓库 → **Actions** 标签页 → 找到 workflow **Linux receiver build and validation**（fork 后默认禁用）→ 点 **Enable workflow** 启用
3. 点 **Run workflow** → 分支选默认分支 → 绿色 **Run workflow** 按钮
4. 等待约 10–20 分钟构建完成（run 状态变绿色 ✓）
5. 打开该 run → 底部 **Artifacts** → 下载 **hellocar-linux-receivers**（约 9.4MB，30 天有效）
6. 解压后找到 `hellocar-linux-x86_64.tar.gz`，放到本项目目录（`D:\软件\AI Class\制作apk\海豹06-HiCar可行性调研\`）

> 备选路径（不用 GitHub）：在本机启用 WSL2（`wsl --install`，需重启系统）→ 装 Ubuntu → 在 Ubuntu 里执行 `GITHUB_ACTIONS=true bash tools/build.sh` 构建。需要重启，等您方便时再说。

---

## 四、虚拟机环境搭建（VirtualBox + Ubuntu）

1. **安装 VirtualBox**：<https://www.virtualbox.org/wiki/Downloads>（Windows 版 + 同页下载 **Extension Pack**，USB 直通必需；注意 Extension Pack 版本号必须与 VirtualBox 完全一致）
2. **下载 Ubuntu 24.04 LTS 镜像**：<https://ubuntu.com/download/desktop>（或服务器版，均可）
3. **创建虚拟机**：名称随意（如 `hicar-verify`）→ 内存 ≥ 4GB → 磁盘 ≥ 40GB → 完成创建后进「设置」：
   - 系统 → 处理器 ≥ 2 核
   - 网络 → 桥接模式（后续若需联网装包）
   - 显示 → 显存 ≥ 64MB（投屏画面要能看）
4. **安装 Ubuntu**：挂载 ISO 启动 → 按向导安装（用户名密码记住）
5. **安装增强功能与扩展包**：设备 → 安装增强功能（Guest Additions）→ 重启；确认 VirtualBox 主界面「全局设定 → 扩展」里 Extension Pack 已启用
6. **USB 直通**：插上 USB Wi-Fi / 蓝牙适配器 → 虚拟机设置 → USB → 添加对应设备过滤器（每次启动自动直通）；运行中也可在 设备 → USB 菜单里勾选直通
7. **Ubuntu 内验证硬件**：
   ```bash
   lsusb                    # 应看到 8812au / CSR 等 USB 设备
   iw list | grep -A5 "Supported interface modes"   # 应包含 AP 模式
   iw list | grep -A3 "5745 MHz"                    # 应支持 5GHz 信道
   hciconfig -a             # 应看到 hci0（蓝牙）
   ```
   - 若 `iw list` 看不到 8812au：`sudo apt install -y dkms build-essential` 后装驱动：`sudo apt install -y rtl8812au-dkms`（或按网卡型号装对应驱动）

---

## 五、HelloCar 部署（在 Ubuntu 虚拟机内）

```bash
# 1. 依赖
sudo apt-get update
sudo apt-get install -y bluez network-manager libdbus-1-3 iproute2 iw ffmpeg python3 python3-venv

# 2. 把 hellocar-linux-x86_64.tar.gz 拷进虚拟机（VBox 共享文件夹或 scp），解包
tar xzf hellocar-linux-x86_64.tar.gz
cd hellocar-linux-x86_64

# 3. 身份与 PIN（一次性初始化）
sudo install -d -m 700 /var/lib/hellocar /run/hellocar
sudo python3 tools/identity.py init
sudo python3 tools/identity.py show-pin      # 记下 6 位 PIN，手机连接时要用

# 4. 启用 5GHz 信道（中国区法规）
sudo iw reg set CN
```

---

## 六、真机连接验证（荣耀 Magic7 Pro）

打开两个终端：

```bash
# 终端 1：媒体消费者（录制投屏流）
sudo python3 examples/record_media.py --output /var/tmp/hc --duration 120

# 终端 2：启动 HelloCar 接收端（wlan0 换成您的 Wi-Fi 接口名，用 `iw dev` 查）
sudo bin/hellocar run --network-interface wlan0 --pair 120
```

手机侧操作：
1. 手机开启 蓝牙 + WLAN
2. 打开 荣耀车联 / HiCar（MagicOS 11 上入口为「荣耀车联」或系统设置里的「车联」）
3. 搜索设备 → 应出现 **HelloCar** → 选择连接
4. 按提示输入第 5 节 `show-pin` 的 6 位 PIN
5. 观察终端 2 日志与终端 1 是否落盘 `video.h264` / `audio.s16le`

**验证结果记录**（无论成败都要记录）：
- 走到哪个阶段（发现/配对/认证/投屏）
- 手机侧报错原文（截图）
- 终端 2 日志尾部（保存文件）

---

## 七、排错速查

| 现象 | 可能原因 | 处理 |
|---|---|---|
| 手机搜不到 HelloCar | BLE 广播未发出；蓝牙未直通 | `hciconfig hci0 up`；检查 `lsusb`/`dmesg` 蓝牙是否在虚拟机内 |
| 配对失败 | PIN 输入问题；SDP 服务记录差异 | 重新 `show-pin`；`sdptool browse local` 看服务记录 |
| 认证/会话中断 | HiChain 认证失败（鸿蒙 4.2 vs MagicOS 11） | 保存完整日志（第八节），这是关键判定信息 |
| 热点未建立 | 网卡不支持 5GHz AP；被占用 | `iw list` 查 AP 模式；`iw dev wlan0 info`；确保 wlan0 未被 NetworkManager 当客户端占用 |
| 投屏黑屏/无数据 | 视频流未到消费者 | 检查终端 1 是否有数据；`ffprobe video.h264` |

---

## 八、日志采集（验证失败时必做）

无论成败，请收集以下内容发回，用于判定「HelloCar 协议问题」还是「MagicOS 11 兼容问题」：

1. 终端 2 完整日志：`sudo bin/hellocar run --network-interface wlan0 --pair 120 2>&1 | tee /var/tmp/hc.log`
2. 手机侧报错/现象截图
3. `record_media.py` 落盘文件大小（`ls -la /var/tmp/hc/`）
4. 系统信息：`uname -a`、`hciconfig -a`、`iw dev`

---

## 九、验证结论的分支走向

| 验证结果 | 结论 | 下一步 |
|---|---|---|
| 全链路通，落盘视频可播 | HelloCar 协议真实可用 | 进入移植阶段：认证库 NDK 交叉编译 → Rust 协议层移植 → 平台层重写 → 车机 APK |
| 卡在认证/会话（3 阶段） | 疑似 MagicOS 11 不兼容 | 借鸿蒙 4.2 华为手机复测；仍失败则判定 HelloCar 协议与现行手机端不兼容，路线不可行，转评估替代方案 |
| 卡在发现/配对（1-2 阶段） | 环境问题为主 | 按第七节排错；排除环境因素后再判定 |
