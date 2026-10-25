# Grabette Raspberry Pi 4 安装与复刻笔记

记录日期：2026-10-24（America/Los_Angeles）。依据本次用户终端输出及确认整理。

## 目录

- [当前进度与设备](#当前进度与设备)
- [README 对照](#readme-对照)
- [从刷卡到后台服务](#从刷卡到后台服务)
- [故障与修复](#故障与修复)
- [最终验证](#最终验证)
- [日常操作](#日常操作)
- [下次继续](#下次继续)

## 当前进度与设备

**基础软件安装、后台服务和重启自启已验证；采集硬件尚未装齐，未验证录制。**

| 项目 | 本次配置或实测结果 |
| --- | --- |
| 主机 | Raspberry Pi 4，无显示器，通过 Windows PowerShell SSH 操作 |
| 已连接硬件 | Pi 4 + HAT，PiSugar 供电；相机、角度传感器等外设尚未连接 |
| 系统选择 | Raspberry Pi OS Lite（64-bit，无桌面） |
| 系统输出 | Debian GNU/Linux 13 (trixie)，DEBIAN_VERSION_FULL=13.7 |
| 内核 | 6.18.50+rpt-rpi-v8 |
| 架构 | aarch64 |
| 主机名 / 用户 | R-grabette / wenbin |
| 系统 Python | /usr/bin/python3，3.13.5 |
| uv | 0.13.5 |
| 树莓派代码目录 | /home/wenbin/grabette |
| 安装工作目录 | /home/wenbin/grabette/packages/grabette |
| 虚拟环境 | /home/wenbin/grabette/.venv（位于仓库根目录） |
| 手别 | right |
| 最终深度相机配置 | gemini305；SDK pyorbbecsdk2 2.1.2 已安装，设备未连接验证 |
| 网页 | http://R-grabette.local:8000/ |

没有保存密码、Wi-Fi 密码等凭据。版本为本次记录，不代表今后必须固定为这些版本。

## README 对照

参考：[上游 Grabette README（develop）](https://github.com/pollen-robotics/grabette/tree/develop/packages/grabette)。

| README 位置 | 本次操作 | 状态 |
| --- | --- | --- |
| Install → Raspberry Pi → Prerequisites → 1. Flash the SD card | Lite 64-bit、用户名、网络、主机名、SSH | 完成 |
| README 未单列的准备操作 | 安装 git/curl/make、克隆仓库、进入包目录 | 完成 |
| Prerequisites → 2. Install uv… | uv、启动配置、音频 overlay、网络权限、重启 | 完成 |
| 补充排查 | i2c-dev 加载及持久化、Python 开发头文件 | 完成 |
| One-shot bringup | 安装右手设备环境、手动启动 | 完成 |
| systemd (auto-start on boot) | 主服务及蓝牙服务安装、自启 | 完成，已重启验证 |
| Usage | Windows 浏览器访问 dashboard | 完成 |
| Calibration | 机械结构与传感器安装后校准 | 未执行 |

**版本差异：**树莓派克隆的上游 develop 比 Windows 本地副本更新。树莓派 Makefile 支持 SERVICE_USER，且安装结果带 CAMERA 配置；不要照着旧本地副本直接改写用户名。develop 会变化，本次未记录实际 commit SHA，尚不具备精确版本复现依据。

## 从刷卡到后台服务

### 1. 刷卡与 SSH

在 Windows Raspberry Pi Imager 中选择 Pi 4、Raspberry Pi OS Lite 64-bit，配置主机名 R-grabette、用户 wenbin、自己的密码与 Wi-Fi，并启用 SSH 密码认证。写入并校验后，安全弹出卡，断电插卡再开机。首次启动等待几分钟。

Windows PowerShell 中执行：

```powershell
ssh wenbin@R-grabette.local
```

作用：通过网络登录树莓派，无需显示器。首次确认主机身份后输入 yes，再输入密码（输入时不显示字符）。本次首次密码验证失败，第二次成功，无后续影响。

登录后，以下命令均在树莓派执行，除非另有说明：

```bash
cat /etc/os-release
uname -m
hostname
```

作用：依次确认系统版本、CPU/系统架构与主机名。本次结果见前表。

### 2. 安装基础工具和下载代码

```bash
sudo apt update
sudo apt install -y git curl make
git clone --branch develop https://github.com/pollen-robotics/grabette.git
cd ~/grabette/packages/grabette
```

作用：刷新软件包索引，安装 Git、下载工具和 make；下载上游 develop 分支，再进入含 Makefile 的目录。`-y` 自动确认安装。该 clone 用于新设备，已有同名目录时不要直接重复执行。

注意：本次树莓派克隆的是上游仓库，并非 Windows 本地 fork 的自动同步副本。

### 3. 安装 uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env"
uv --version
pwd
python3 --version
```

作用：下载并运行 uv 安装脚本；将 uv 加入当前 shell 的 PATH；检查 uv、当前位置与 Python 版本。安装位置为 ~/.local/bin。uv 管理项目 Python 环境和依赖；无需另装一套 Python。

### 4. 检查账号及启动配置

```bash
grep -n 'rasp' Makefile systemd/*.service
cat config/config.txt
diff -u /boot/firmware/config.txt config/config.txt
```

作用：找出默认账号引用（`-n` 显示行号）、读取项目启动配置、比较当前与项目配置。diff 有差异时返回非零状态属于正常行为。

本次树莓派版本支持 `SERVICE_USER=wenbin`。systemd 模板保留 rasp 路径/账号占位，安装目标会替换相关项；不是发现 rasp 就需要全局替换。

本次配置差异主要为启用 I²C/I²S、关闭板载模拟音频、添加 V2 HAT overlay。删除的 pi5 专属配置不影响 Pi 4。关键配置如下：

```ini
dtparam=i2c_arm=on
dtparam=audio=off
dtparam=i2s=on
[all]
dtoverlay=i2c3,pins_4_5
dtoverlay=i2c4,pins_8_9
dtoverlay=tlv320aic3104
```

以上仅为说明关键项，不是完整 config.txt。

### 5. 备份、应用配置、安装 overlay 与权限

```bash
sudo cp -a /boot/firmware/config.txt "/boot/firmware/config.txt.before-grabette-$(date +%Y%m%d-%H%M%S)"
sudo cp config/config.txt /boot/firmware/config.txt
make install-audio SERVICE_USER=wenbin
make install-netdev SERVICE_USER=wenbin
sudo reboot
```

作用依次为：用时间戳备份现有启动配置；应用已比较过的项目配置；编译安装 HAT 音频 overlay、音频初始化服务并授予 audio 组权限；配置 netdev 组和 Wi-Fi 管理 polkit 规则；重启生效。任何安装命令报错应先处理，再重启。

`make install-audio` 应在重启前执行，因为 tlv320aic3104 不是系统自带的 overlay。首次安装时尚无 aic3104 声卡提示，不等于安装失败。

重启断开 SSH 后，在 Windows 用上述 ssh 命令重新连接，再执行：

```bash
cd ~/grabette/packages/grabette
id
ls -l /dev/i2c-*
```

作用：返回包目录、查看当前账号的组权限、检查 I²C 设备节点。

### 6. 加载 I²C 用户态接口

本次重启后首次检查没有 /dev/i2c-*，按以下方式处理：

```bash
sudo modprobe i2c-dev
ls -l /dev/i2c-*
echo i2c-dev | sudo tee /etc/modules-load.d/i2c-dev.conf
```

作用：当前启动加载 i2c-dev 模块，检查节点，再将模块名称写入开机加载配置。`|` 把左侧输出传给右侧命令；tee 以管理员权限写文件并显示内容。最终重启已验证持久化有效。

总线节点出现不依赖传感器已连接，也不证明传感器通信成功。项目中 i2c-3 对应 distal、i2c-4 对应 proximal。

### 7. One-shot bringup：安装依赖

```bash
make install-rpi HAND=right SERVICE_USER=wenbin
```

作用：安装系统依赖、设备访问规则、时间同步和音频配置，创建仓库根目录的虚拟环境并安装 Python 依赖，检查导入并写设备配置。HAND 指定右手，SERVICE_USER 指定账号。不要在整条 make 命令前加 sudo；任务内部会按需使用 sudo。

**本次第一次执行因 Python.h 缺失失败，按下一节补装 python3.13-dev 后重跑成功。**

最终配置文件 /etc/grabette/env 实测为：

```ini
GRABETTE_HAND=right
GRABETTE_DEPTH_CAMERA=gemini305
```

上文 HAND-only 命令是会话中给出的指令；用户实际成功执行的完整重跑命令未完整贴回，不能据此断言其默认 CAMERA 值。实际安装输出确认了 Gemini 305 SDK 及配置。下次复刻先查看该版本 `make help`、Makefile 中 CAMERA 参数的支持方式，显式选择实际相机；不要盲目沿用相机默认值。

### 8. 手动启动与网页验证

```bash
uv run --package grabette python -m grabette
```

作用：通过项目虚拟环境运行 Grabette 包。`--package` 指定工作区中的包，`python -m` 运行包入口。无需手动激活 .venv。程序在前台持续输出日志，按 Ctrl+C 停止。

本次日志确认 `RPi hardware detected, using RpiBackend`、`Application startup complete` 及监听 0.0.0.0:8000。Windows 浏览器成功访问 http://R-grabette.local:8000/。

### 9. 安装后台服务

先在手动运行程序的终端按 Ctrl+C，等提示符回来，再执行：

```bash
make install-systemd SERVICE_USER=wenbin
systemctl status grabette grabette-bluetooth --no-pager -l
systemctl is-enabled grabette grabette-bluetooth
```

作用：安装并立即启动主服务与蓝牙 Wi-Fi 配置服务，启用开机自启并设置 BLE 模式；查看详细状态（不分页、不截断长行）；检查开机自启。预期分别为 active (running) 和两行 enabled。

安装后程序在后台运行，不再依赖 SSH 终端保持打开。

## 故障与修复

### pidng 编译缺少 Python.h

依赖链：grabette[rpi] → picamera2 0.3.36 → pidng 4.0.9。编译器已能运行，但报错：

```text
fatal error: Python.h: No such file or directory
```

修复：

```bash
sudo apt install -y python3.13-dev
ls -l /usr/include/python3.13/Python.h
make install-rpi HAND=right SERVICE_USER=wenbin
```

作用：安装当前系统 Python 对应的开发头文件，检查文件存在，重新执行安装。若将来系统 Python 版本变化，需用匹配的开发包。本次修复后 `all imports OK`，并确认 home=/usr/bin、version_info=3.13.5、include-system-site-packages=true。

安装目标会重建项目虚拟环境；不要把手工安装但未记录的依赖当作永久配置。下载缓存一般可复用。无需为本次报错降级 Python或重刷系统。

### 没有 I²C 节点

修复方法见第 6 步。未连接外设不是节点缺失的原因；节点存在与外设响应是两项不同检查。

### 启动警告与当前缺件状态

| 日志 | 本次解释与验证边界 |
| --- | --- |
| Gradio 参数迁移/弃用、SSR 提示 | 没有阻止此次网页启动，未做代码修复 |
| RGB camera unusable / list index out of range | 相机未连接，录制禁用；未验证真实相机 |
| distal sensor does not answer on /dev/i2c-3 | 传感器未连接，未验证真实通信 |
| Angle sensors not calibrated | 尚未校准，待硬件与机械结构安装后处理 |
| Speaker ready on plughw:CARD=aic3104,DEV=0 | 软件找到音频设备；未确认实际听到声音 |

## 最终验证

为确认不是只在当前会话有效，执行了再次重启：

```bash
sudo reboot
```

重启后未手动运行程序，网页仍可访问。SSH 登录后检查：

```bash
systemctl is-active grabette grabette-bluetooth
ls -l /dev/i2c-*
```

结果：两个服务均输出 active；存在 /dev/i2c-1、3、4、20、21，属主 root、组 i2c，权限 crw-rw----。账号 wenbin 已属于 audio、netdev、gpio、i2c 等组。

用户确认网页可显示电量、温度、存储信息。电量数值准确度、PiSugar 管理服务/关机联动未单独测试。

## 日常操作

正常使用：供电 → 等待启动 → Windows 浏览器打开 http://R-grabette.local:8000/。无需先 SSH 或运行 uv。

以下为维护参考命令，不代表本次全部执行过：

| 命令（在树莓派执行） | 用途 |
| --- | --- |
| `systemctl status grabette grabette-bluetooth --no-pager -l` | 查看服务状态 |
| `journalctl -u grabette -n 100 --no-pager` | 查看主服务最近 100 条日志 |
| `journalctl -u grabette -f` | 持续跟踪日志；Ctrl+C 只退出日志查看 |
| `journalctl -u grabette-bluetooth -n 100 --no-pager` | 查看蓝牙配置服务日志 |
| `sudo systemctl restart grabette` | 配置/代码变更后重启主服务 |
| `sudo systemctl stop grabette` | 临时停止主服务，便于手动调试；不取消开机自启 |
| `sudo systemctl start grabette` | 恢复启动主服务 |
| `sudo poweroff` | 正常关闭 Linux；不保证 PiSugar 随即切断供电 |

后台服务运行时不要再手动运行同一程序，以免端口和硬件访问冲突。接插相机排线、HAT 和传感器前，先正常关机并确认 PiSugar 也停止供电。

## 下次继续

1. 先记录树莓派实际代码版本：`git -C ~/grabette rev-parse HEAD`（读取 commit SHA，补全本笔记的精确复现依据）。
2. 确认实际相机型号与最终接线方案；当前深度相机配置为 gemini305，不能把配置成功当作设备已测试。
3. 对照实际 HAT、角度板和排线，断电检查方向、引脚与供电，再分步连接 RGB 相机、角度传感器、按钮/LED、深度相机。
4. 分别验证相机、角度读数、按钮/LED、声音；软件检测到声卡不等于完成声学测试。
5. 机械结构、磁铁与传感器装好后，按 README Calibration 要求完全张开夹爪、两个关节伸直，再校准。现在不要提前写入无效零位。
6. 完成单次录制、回放、文件检查后，才确认采集功能通过；后处理与上传另行验证。

继续遵守项目硬件交接记录：已打样、订单生产资料、实际到货/连接、功能测试和历史替代料结论分开记录。此次软件安装成功不能证明未测试硬件或历史替代件兼容。
