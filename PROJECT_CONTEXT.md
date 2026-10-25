# 开源UMI：项目背景与工作交接

更新时间：2026-10-20（America/Los_Angeles）。

## 用户目标与当前阶段

- 本项目是在复刻开源 GRABETTE/Gripette 项目。
- 用户已将相关仓库放入本地工作目录，并明确表示已经在旧聊天指导下打样了项目中的 PCB。
- 已打样是用户确认的进度；具体哪些板、数量、订单状态、是否收货和上电验证，当前尚未取得记录。
- 原工作聊天标题为 **Grabette_PCB打样**，聊天 ID：`6aa83140-b628-83e8-82f0-54d24d1b3d79`。
- 用户已要求把旧工作背景接入当前项目。原聊天保留，当前以本地交接文档延续工作。

## Raspberry Pi 软件进度（2026-10-24）

用户已在 Raspberry Pi 4 上完成 Lite 64-bit / Trixie 安装，通过 Windows SSH 操作（`wenbin@R-grabette.local`）。当前 Pi 4 + HAT，PiSugar 供电，尚未连接相机、角度传感器等外设。主服务及蓝牙服务已安装并通过重启自启验证，I²C 1/3/4 等设备节点重启后存在，网页可访问并显示电量、温度、存储信息。右手配置，最终深度相机配置为 `gemini305`；不代表相机已连接或验证。声卡被软件识别，未确认实际声音。采集、录制及角度校准尚未验证。

详见 [Raspberry Pi 4 安装与复刻笔记](docs/hardware/rpi4-setup-notes-2026-10-24.md)，包含 README 对照、命令用途、`Python.h` 缺失及 `i2c-dev` 修复、最终验证和下次继续事项。树莓派使用上游 develop，比本地软件副本更新，支持 `SERVICE_USER=wenbin`；具体 commit SHA 待补录。后续给用户命令时，同时解释命令用途及对应 README 的步骤。

## 本地仓库列表

| 路径 | 用途 |
| --- | --- |
| `D:\GitHub\grabette-fork` | 主软件、装配说明、模型、数据采集与后处理 |
| `D:\GitHub\elec_grabette-fork` | Grabette/Gripette 各电子板、原理图、PCB、生产输出 |
| `D:\GitHub\elec_RPI_Robot_HAT-fork` | Raspberry Pi Robot HAT 原理图、PCB、生产输出 |

## PCB 历史背景入口

详见 [PCB 打样交接记录](docs/hardware/pcb-handoff-2026-10-20.md)，包括已读取的旧问题、器件与本地文件的对应关系、附件及尚缺资料。

旧聊天接口目前只返回了五轮用户消息，助手回复均为内容引用占位，不能还原技术结论。后续不要把“曾询问能否替代”当作“已经批准或实际使用了替代件”。本地 revised/revise 文件名也不等于最终下单版本。

## 后续工作的依据

2026-10-20 已解析用户下载的 HAT 生产包 `D:\GitHub\elec_RPI_Robot_HAT-fork\JLC_Orders\PCB01186-C1_elec_RPI_Robot_HAT_PCB_Y122.zip`。工单 8562074A_Y122，四层、1.6 mm，单板约 65 × 30.9 mm，加工框约 75 × 70.9 mm。包内原始 Gerber 与本地 production ZIP 的 15 个对应文件字节一致；包中未找到可用的最终贴装 BOM/CPL，实际替代料号仍未知。详情见相邻硬件仓库 `JLC_Orders/PCB01186-C1_Y122_analysis/解析报告.md`。

优先结合实际收到的板、最终下单 BOM/CPL、对应原理图及用户补充信息推进。后续确定的板卡版本、替代件和实测结果，应同步补充到交接记录。

同日解析 `D:\GitHub\elec_grabette-fork\JLC_Orders` 中四个订单：Y116 角度传感器（2层，约20×20 mm）、Y115 HMI（2层，约38×20 mm）、Y117 Bulkhead（4层，约26×38 mm）、Y119 Gripette hub/motor（4层，约65×30 mm）。均为1.6 mm、绿油白字，元数据记录为有铅喷锡；共56个原始生产文件与本地对应 production ZIP 字节一致。该目录已由仓库 `.gitignore` 忽略。最终贴装BOM/CPL仍未找到，实际替代料号未确认。详情：`D:\GitHub\elec_grabette-fork\JLC_Orders\解析报告.md`。Y117 的下载文件名为B2、顶面装配丝印为ASE01199-B1，此标识差异与本地生产资料一致，后续需区分记录。
