# OMO_Hardware_PA


> **目标（Role & Goal）**  
> 。请依据以下**完整规格**，为“Waybox”交付：可量产的 **CM4 载板**（Compute Module 4 Baseboard）、**系统镜像与驱动配置**、**产测方案与文档**。交付后，插入 **SIM/eSIM** 上电即可联网；屏幕仅显示**表情动画**；手机端承担主 OS 与复杂 UI/算力，通过 **BLE** 与设备同步。

---

## 产品定位（必须遵守）
- 设备端：表情显示（仅表情动画，无复杂 GUI）、语音 I/O（拾音/播音）、**双路摄像头采集**与**视频推流**、BLE/Wi-Fi/蜂窝连接；本地承担主要交互逻辑与 AI 算力。
- 手机端（伴侣应用层）：主要负责 **账号登录/认证**、**用户查看历史状态**、**服务订阅与付费管理**；作为用户入口与管理控制台。设备与手机通过 **BLE** 进行控制与状态同步。
- 量产：小/中批量直接使用 **CM4 + 自研载板**；大批量阶段再评估全定制 SoC（本项目不展开）。


## 设备端 vs 手机端定位对比

| 模块 | 设备端（Waybox 硬件） | 手机端（Companion App / 管理端） |
|------|----------------------|---------------------------------|
| **定位** | 主执行单元（AI 主机 / 外设） | 用户入口 & 管理控制台 |
| **主要职责** | - 表情显示（仅表情动画）<br>- 语音 I/O（拾音/播音）<br>- 双摄像头采集 & 视频推流<br>- 本地 AI 算力与实时交互 | - 账号登录 / 认证<br>- 用户查看历史状态 / 数据<br>- 订阅服务 / 付费管理<br>- 设备状态展示与基础控制 |
| **通信方式** | Wi-Fi / 蜂窝：与云端交互<br>BLE：与手机实时同步 | BLE：下发控制命令 & 接收设备状态 |
| **算力分工** | 执行复杂交互、实时多模态处理（语音/视频/表情/推流） | 不承担主算力，仅做管理和展示 |
| **用户交互** | 语音 + 表情动画（实时） | App 界面（历史数据、设置、订阅） |
| **角色类比** | “外设 + 本地 AI 大脑” | “遥控器 + 账户中心” |


---

## 硬件需求总表（必须实现）
| 模块 | 选型 / 接口 | 软件支持 | 备注 |
|---|---|---|---|
| **主控（SoM）** | Raspberry Pi Compute Module 4（CM4，1~4GB RAM，**32GB eMMC**） | Raspberry Pi OS / Yocto / Buildroot | 工业级 SoM，长供货周期；双摄、DPI、I²S、USB、BLE/Wi‑Fi |
| **存储（Storage）** | **32GB eMMC**（随 CM4 机型集成） | eMMC 驱动 + Linux FS | 系统镜像、日志、缓存、用户数据 |
| **显示（Display）** | RGB666 TFT（4.3″~7″），**DPI 并口** | KMS / `vc4-kms-dpi-generic`；LVGL/Qt | 仅表情动画；提供 `config.txt`/DT overlay 与时序参数 |
| **摄像头（Camera）** | 2 × MIPI‑CSI（CAM0 2‑lane + CAM1 4‑lane） | libcamera / V4L2 / FFmpeg / GStreamer | 双路采集 + **H.264 硬编**；支持 RTSP/WebRTC 推流 |
| **音频输入（Mic In）** | **PDM 双麦阵列（DMIC）** 或 PDM→I²S/PCM 转换前端 | ALSA + VAD/唤醒词 | 若无 PDM 前端，可选 I²S 数字麦 |
| **音频输出（Spk Out）** | I²S → 数字功放（如 MAX98357A） → 喇叭 | ALSA / PipeWire | 提示音/语音；功放与数字/射频地分区 |
| **无线连接（BLE/Wi‑Fi）** | CM4 自带 | BlueZ / wpa_supplicant | BLE：控制/OTA；Wi‑Fi：调试/局域网 |
| **蜂窝（Cellular，可选 Pro）** | USB 4G/5G（Quectel EC25/EG25/EC200/RM5xx 等） | ModemManager / `qmi_wwan`/`cdc_mbim`/`uqmi` | **同时预留 nano‑SIM + eSIM 焊盘**；USB 口供电能力充足 |
| **电源管理（PMIC）** | 1 节锂电 + 路径管理（如 BQ25895）+ 升压 5V | 电池/电量驱动 | USB‑C 充电；电量检测与 NTC；背光恒流可调 |
| **扩展接口（I/O）** | USB 2.0 Host / GPIO / I²C / UART | Linux 驱动齐全 | 触控/传感器/调试；预留产测接口与关键测试点 |
| **安全/OTA** | BLE OTA + **A/B rootfs** | 自研 OTA 管理 | BLE 分块传输；系统失败自动回滚与日志保留 |

---

## 电气/版图设计要求（关键点必须满足）
1. **CSI‑2（MIPI‑CSI）**：100 Ω 差分、对内等长、过孔最少、连续参考地；CAM0 2‑lane、CAM1 4‑lane；统一 22‑pin FFC 座，方向与丝印一致。  
2. **DPI（RGB666）**：18 条 RGB 数据 + HS/VS/DE/CLK；同组等长并控制时钟相对延迟；背光恒流独立供电与地分区。  
3. **USB 2.0（蜂窝）**：90 Ω 差分，近端 ESD 保护，线长最短、避开噪声区；供电支路满足发射峰值电流（4G 可达 2 A）。  
4. **SIM/eSIM**：nano‑SIM 卡座 + eUICC 焊盘并存；SIM_DATA/SIM_CLK/SIM_RST 近模组布线、ESD 保护；支持 1.8 V/3.0 V。  
5. **功耗与供电**：蜂窝模组独立低阻电源路径（建议带开关以便硬断电），输入大电容（≥470 µF 低 ESR）+ 局部去耦。  
6. **射频/天线（RF/Antenna）**：u.FL/IPEX，Π 型匹配位；天线净空≥10 mm，避金属与大电流回流路径；提供驻波/回损测试。  
7. **音频/EMI**：功放电源加大电流退耦与 LC 滤波；音频与射频/视频数字区隔离，单点汇接；I²S 走线等长+地护。  
8. **调试/测试点**：USB D+/D-、PWRKEY/RESET/STATUS、主 UART、蜂窝支路电流取样电阻（0.01–0.02 Ω）测试点。  
9. **机械/热**：提供 3D 结构、模组与天线位置、散热路径与热仿简报；外壳对天线衰减评估。  

---

## 系统与驱动（必须预装/可运行）
- **OS**：Raspberry Pi OS 64‑bit（注明版本）；可选 Yocto/Buildroot（需提供构建脚本）。  
- **组件**：`ModemManager`、`NetworkManager`、`mmcli`、`nmcli`、`qmicli/uqmi`、`ffmpeg`、`gstreamer`、`libcamera`、`arecord/aplay`、`tcpdump`、`iperf3`、`jq`、`curl`。  
- **内核模块**：`qmi_wwan` / `cdc_mbim` / `cdc_wdm` / `usbnet` 等已启用；`lsmod` 可见。  
- **显示**：提供 `config.txt` / DT overlay 以启用 `vc4-kms-dpi-generic` 输出 **RGB666**，含分辨率/像素时钟/引脚映射范例。  
- **摄像头**：`libcamera` + 硬编 H.264 管线；提供**双路并发推流脚本**（RTSP 或 WebRTC 其一）。  
- **音频**：ALSA 设备正确枚举；PDM→I²S/PCM 前端对应驱动/设备树已配置；提供本地录放测试脚本。  
- **BLE/Wi‑Fi**：BlueZ 可扫描/配对；Wi‑Fi 可联网与 OTA；默认禁用不必要后台服务以节能。  
- **OTA/日志**：A/B 分区回滚；`journalctl`、应用日志与蜂窝诊断日志写入 `/var/log`，带轮转。

---

## 蜂窝网络：一键可用（脚本/服务已内置模板）
- **NetworkManager 路线（推荐）**：在 `/etc/NetworkManager/system-connections/` 预置 `cell0.nmconnection`（APN 参数化、开机自动连接）。  
- **脚本（源码）**：  
  1) `/usr/local/sbin/cellular_connect.sh <APN>`：识别模组 → 连接 → 打印 `wwan0` IP → 验证 `ping`。  
  2) `/usr/local/sbin/cellular_diag.sh`：导出 `lsusb/lsmod/mmcli/ip route/ping` 等体检信息到 `/var/log/cellular/`。  
- **systemd**：`modem-connect@.service`（断线自动重连，失败重试；可扩展 GPIO **PWRKEY** 以硬复位模组）。

---

## 产测（ATE）与量产
- **ATE**：摄像头/显示/音频/蜂窝/按键/LED/USB 自动化脚本与判定；IMEI/SN 绑定流程与烧录工具。  
- **量产镜像**：版本号/校验和、首次开机初始化脚本、默认网络策略（IPv4 优先、IPv6 可选）。  
- **返修/复位 SOP**：eMMC 重刷/恢复、蜂窝模块硬复位、日志抓取打包。

---

## 验收用例（样机到手即可执行）
1) **上电识别**  
```bash
uname -a
lsusb | grep -i -E "Quectel|Simcom|Fibocom"
dmesg | tail -n 100 | egrep -i "cdc|qmi|mbim|ttyUSB|wwan"
```
**期望**：模组被识别，`/dev/cdc-wdm0` 或 `/dev/ttyUSB*` 存在，无持续错误。

2) **摄像头**
```bash
libcamera-vid -t 5000 -n -o cam0.h264 --camera 0 --width 1280 --height 720 --framerate 30
libcamera-vid -t 5000 -n -o cam1.h264 --camera 1 --width 1280 --height 720 --framerate 30
```
**期望**：两路均成功生成 H.264 文件（可回放）。

3) **显示**
- 上电显示测试图/表情动画 Demo；亮度可调；无撕裂/异常闪烁。

4) **音频**
```bash
arecord -l && aplay -l
arecord -d 5 -f S16_LE -r 16000 /tmp/mic.wav && aplay /tmp/mic.wav
```
**期望**：设备枚举正确，可录可放；无明显底噪/啸叫。

5) **蜂窝联网**
```bash
sudo /usr/local/sbin/cellular_connect.sh <APN>
ip addr show wwan0
ping -I wwan0 -c 3 8.8.8.8
curl -4 https://ifconfig.me
```
**期望**：`wwan0` 获取 IPv4 地址；`ping` 成功；能获得外网 IPv4（或 NAT IP）。

6) **稳定性（20 分钟）**
- `iperf3` 上/下行各 2 分钟，不掉线/无 USB 反复断连；模组温升在规格内。

7) **自恢复**
- 拔/插天线或进入弱信号区，1 分钟内自动恢复在线（systemd 服务重连成功）。

---

## 交付物清单（不可缺项）
1. **硬件**：原理图（PDF+源文件）、PCB（Gerber+源文件）、叠层、关键网络走线截图（USB/SIM/CSI/DPI/RF）、BOM、3D 结构、天线与电源完整性测试报告。  
2. **固件/软件**：系统镜像（版本与校验）、`config.txt`/DT overlay、`cellular_connect.sh`、`cellular_diag.sh`、`modem-connect@.service` 源码与安装脚本、**双摄推流脚本**（RTSP 或 WebRTC）、音视频/显示/蜂窝最小 Demo。  
3. **文档**：Bring‑up 指南、调试手册、ATE 与量产流程、返修/复位 SOP、变更记录（Changelog）。  
4. **质量门槛**：≥10 台样机通过全部验收用例；连续 72 小时在线稳定运行；问题 5 个工作日内提供可复现与修复补丁。

---

## 输出格式（请按此结构提交）
* 《硬件设计说明书》（含约束与仿真截图）  
* 《系统与驱动配置说明》（含所有脚本与配置项）  
* 《产测与验收手册》（逐条测试指令与判定）  
* 《镜像与源码包下载链接 + SHA256》  
* 《问题清单与修复计划》（如有）  

> **术语（中英）**：PDM/DMIC、I²S、DPI/RGB666、MIPI‑CSI、A/B rootfs、ModemManager/QMI/MBIM、RTSP/WebRTC、KMS/DRM、ALSA。

---

# 附录 A：双摄推流脚本模板（RTSP，H.264 硬编）目前经供参考

> **说明**：使用 **libcamera-vid**（分别指定 `--camera 0/1`）硬编 H.264，经 **FFmpeg RTSP muxer** 推送到 **MediaMTX**（rtsp-simple-server）。先启动 `mediamtx`。

**安装（示例，Raspberry Pi OS Bookworm 64‑bit）**
```bash
sudo apt-get update && sudo apt-get install -y ffmpeg
# MediaMTX：到其 Releases 下载对应架构二进制，放到 /usr/local/bin 并赋 executable 权限
# mediamtx &   # 或 systemd 管理；默认 RTSP 端口 8554
```

**脚本：`/usr/local/sbin/dual_cam_rtsp.sh`**
```bash
#!/usr/bin/env bash
# 双摄 RTSP 推流（libcamera-vid → FFmpeg → MediaMTX）
# Usage: sudo /usr/local/sbin/dual_cam_rtsp.sh [--w 1280] [--h 720] [--fps 30] [--bitrate 4000000] [--gop 60] [--cam0 0] [--cam1 1] [--host 127.0.0.1] [--port 8554]
set -Eeuo pipefail

W=1280; H=720; FPS=30; BITRATE=4000000; GOP=60
CAM0=0; CAM1=1
HOST=127.0.0.1; PORT=8554

while [[ $# -gt 0 ]]; do
  case "$1" in
    --w) W="$2"; shift 2;;
    --h) H="$2"; shift 2;;
    --fps) FPS="$2"; shift 2;;
    --bitrate) BITRATE="$2"; shift 2;;
    --gop) GOP="$2"; shift 2;;
    --cam0) CAM0="$2"; shift 2;;
    --cam1) CAM1="$2"; shift 2;;
    --host) HOST="$2"; shift 2;;
    --port) PORT="$2"; shift 2;;
    *) echo "Unknown arg: $1" >&2; exit 2;;
  esac
done

command -v libcamera-vid >/dev/null || { echo "libcamera-vid 未找到"; exit 1; }
command -v ffmpeg >/dev/null || { echo "ffmpeg 未找到"; exit 1; }

pids=()
cleanup(){ for p in "${pids[@]:-}"; do kill "$p" 2>/dev/null || true; done; }
trap cleanup EXIT

# Camera 0
(libcamera-vid --camera "$CAM0" -t 0 --inline -n --width "$W" --height "$H" --framerate "$FPS" \
               --codec h264 --bitrate "$BITRATE" --intra "$GOP" -o - \
 | ffmpeg -hide_banner -loglevel error -f h264 -i - -c copy -f rtsp -rtsp_transport tcp \
          "rtsp://${HOST}:${PORT}/cam0") &
pids+=("$!")

# Camera 1
(libcamera-vid --camera "$CAM1" -t 0 --inline -n --width "$W" --height "$H" --framerate "$FPS" \
               --codec h264 --bitrate "$BITRATE" --intra "$GOP" -o - \
 | ffmpeg -hide_banner -loglevel error -f h264 -i - -c copy -f rtsp -rtsp_transport tcp \
          "rtsp://${HOST}:${PORT}/cam1") &
pids+=("$!")

echo "RTSP: rtsp://<IP>:${PORT}/cam0  (Camera ${CAM0})"
echo "RTSP: rtsp://<IP>:${PORT}/cam1  (Camera ${CAM1})"
echo "VLC 示例：vlc rtsp://<IP>:${PORT}/cam0"

wait
```

> 备注：`--inline` 使每个关键帧带 SPS/PPS/SEI，便于 RTSP 随机接入；如需音频合流，可在 FFmpeg 端追加 `-f alsa -i hw:0` 并 `-map` 合流。

---

# 附录 B：DPI 屏 `config.txt` 示例（RGB666 / KMS / `vc4-kms-dpi-generic`）

**/boot/config.txt（关键片段）**
```ini
# 启用 KMS
dtoverlay=vc4-kms-v3d

# 通用 DPI 面板，800x480 @ 60Hz，RGB666（pad-hi），像素时钟 32MHz
dtoverlay=vc4-kms-dpi-generic
dtparam=clock-frequency=32000000
dtparam=hactive=800,hfp=16,hsync=1,hbp=46
dtparam=vactive=480,vfp=7,vsync=3,vbp=23
dtparam=rgb666-padhi
# 可选：背光控制 GPIO 与物理尺寸
dtparam=backlight-gpio=19
dtparam=width-mm=154,height-mm=86

# 如面板要求，可启用极性/时钟翻转：
# dtparam=hsync-invert,vsync-invert
# dtparam=pixclk-invert
```

> 提示：不同面板**时序**以其数据手册为准，上述为 800×480 常见参考值。

---

# 附录 C：蜂窝联网一键化（文件与脚本模板）

## 1) NetworkManager 预置连接：`/etc/NetworkManager/system-connections/cell0.nmconnection`
> 替换其中 `YOUR_APN`，注意权限 `600`，并 `nmcli connection reload`。
```ini
[connection]
id=cell0
type=gsm
autoconnect=true
autoconnect-retries=-1

[gsm]
apn=YOUR_APN

[ipv4]
method=auto

[ipv6]
method=ignore
```

## 2) 连接脚本：`/usr/local/sbin/cellular_connect.sh`
```bash
#!/usr/bin/env bash
# 一键蜂窝连接（NetworkManager 优先，失败回退到 mmcli --simple-connect）
# Usage: sudo cellular_connect.sh <APN>
set -Eeuo pipefail

APN="${1:-}"
[[ -z "${APN}" ]] && { echo "用法: $0 <APN>"; exit 2; }

log(){ echo "[$(date '+%F %T')] $*"; }

command -v nmcli >/dev/null || { echo "缺少 nmcli (NetworkManager)"; exit 1; }
command -v mmcli >/dev/null || { echo "缺少 mmcli (ModemManager)"; exit 1; }

# 确认 Modem 存在并启用
MID="$(mmcli -L 2>/dev/null | sed -n 's!.*/Modem/\([0-9]\+\).*!\1!p' | head -n1 || true)"
if [[ -z "${MID}" ]]; then
  log "未发现 Modem；lsusb/内核驱动是否就绪？"; exit 1
fi
mmcli -m "${MID}" --enable >/dev/null || true

# 如果没有 cell0，则创建；若存在则更新 APN
if ! nmcli -t -f NAME,TYPE connection show | grep -q '^cell0:gsm$'; then
  log "创建 NetworkManager 连接：cell0"
  nmcli connection add type gsm ifname "*" con-name "cell0" apn "${APN}" ipv6.method ignore
else
  log "更新 APN -> ${APN}"
  nmcli connection modify "cell0" gsm.apn "${APN}" ipv6.method ignore
fi
nmcli connection modify "cell0" connection.autoconnect yes

# 尝试激活
if ! nmcli -t -f NAME,DEVICE connection show --active | grep -q '^cell0:'; then
  log "激活 cell0 ..."
  if ! nmcli connection up "cell0"; then
    log "NM 失败，回退 mmcli --simple-connect"
    mmcli -m "${MID}" --simple-connect="apn=${APN}" || { log "连接失败"; exit 1; }
  fi
fi

# 解析 WWAN 接口名（如 wwan0）
WWAN_IF="$(ip -o link | awk -F': ' '/wwan[0-9]+/{print $2; exit}')"
[[ -z "${WWAN_IF}" ]] && WWAN_IF="$(nmcli -t -f GENERAL.DEVICES con show cell0 | tail -n1 | cut -d: -f2 || true)"
[[ -z "${WWAN_IF}" ]] && { log "未找到 WWAN 接口"; exit 1; }

sleep 2
log "IP 地址："
ip -4 addr show "${WWAN_IF}" | sed 's/^[[:space:]]*//'

log "连通性测试："
ping -I "${WWAN_IF}" -c 3 8.8.8.8 || true

if command -v curl >/dev/null; then
  EXT_IP="$(curl -4s --max-time 10 https://ifconfig.me || true)"
  [[ -n "${EXT_IP}" ]] && log "外网 IPv4: ${EXT_IP}"
fi
```

## 3) 诊断脚本：`/usr/local/sbin/cellular_diag.sh`
```bash
#!/usr/bin/env bash
# 采集蜂窝诊断信息到 /var/log/cellular/
set -Eeuo pipefail
TS="$(date '+%Y%m%d_%H%M%S')"
OUT="/var/log/cellular/$TS"
mkdir -p "$OUT"

copy(){ cmd="$1"; fn="$2"; echo ">>> $cmd" > "$OUT/$fn"; bash -lc "$cmd" >> "$OUT/$fn" 2>&1 || true; }

copy "uname -a" "sys.txt"
copy "lsusb" "lsusb.txt"
copy "lsmod | egrep 'cdc|qmi|mbim|wwan|usbnet'" "lsmod.txt"
copy "mmcli -L" "mm_list.txt"

MID="$(mmcli -L 2>/dev/null | sed -n 's!.*/Modem/\([0-9]\+\).*!\1!p' | head -n1 || true)"
if [[ -n "$MID" ]]; then
  copy "mmcli -m $MID" "mm_modem.txt"
fi

copy "nmcli device status" "nm_dev.txt"
copy "nmcli -f all connection show cell0" "nm_cell0.txt"
copy "ip addr" "ip_addr.txt"
copy "ip route" "ip_route.txt"
copy "ping -c 3 8.8.8.8" "ping.txt"
copy "curl -4 --max-time 10 -s https://ifconfig.me" "ext_ip.txt"
journalctl -u ModemManager -n 500 > "$OUT/journal_ModemManager.txt" 2>&1 || true
journalctl -u NetworkManager -n 500 > "$OUT/journal_NetworkManager.txt" 2>&1 || true

echo "日志导出目录: $OUT"
```

## 4) systemd 自启动：`/etc/systemd/system/modem-connect@.service`
```ini
[Unit]
Description=Waybox Cellular Auto Connect (%i as APN)
After=network.target ModemManager.service
Wants=ModemManager.service

[Service]
Type=simple
ExecStart=/usr/local/sbin/cellular_connect.sh %i
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**安装与启用**
```bash
sudo chmod +x /usr/local/sbin/cellular_connect.sh /usr/local/sbin/cellular_diag.sh
sudo systemctl daemon-reload
# 把 APN 作为实例参数启用：
sudo systemctl enable --now modem-connect@internet.service
# （将 internet 替换为你的 APN 名）
```

---

# 附录 D：音频与 ATE 快速脚本

## 1) 音频自检：`/usr/local/sbin/audio_quick_test.sh`
```bash
#!/usr/bin/env bash
set -Eeuo pipefail
arecord -l || true
aplay -l || true
echo "录音 5 秒..."
arecord -d 5 -f S16_LE -r 16000 /tmp/mic.wav
echo "回放..."
aplay /tmp/mic.wav
```

## 2) 产测冒烟：`/usr/local/sbin/ate_smoke_test.sh`
```bash
#!/usr/bin/env bash
set -Eeuo pipefail
ok(){ echo "[OK] $*"; }
fail(){ echo "[FAIL] $*"; exit 1; }

# 摄像头
libcamera-vid -t 2000 -n -o /tmp/cam0.h264 --camera 0 --width 640 --height 480 --framerate 15 && ok "cam0"
libcamera-vid -t 2000 -n -o /tmp/cam1.h264 --camera 1 --width 640 --height 480 --framerate 15 && ok "cam1"

# 显示（仅检查 KMS 设备存在）
[[ -e /dev/dri/card0 ]] && ok "KMS present" || fail "KMS"

# 音频
arecord -d 2 -f S16_LE -r 16000 /tmp/mic.wav && aplay /tmp/mic.wav && ok "audio rec/play"

# 蜂窝（如配好 APN）
if ip link | grep -q wwan; then
  ping -I "$(ip -o link | awk -F': ' '/wwan[0-9]+/{print $2; exit}')" -c 2 8.8.8.8 && ok "cellular ping"
else
  echo "跳过蜂窝：未发现 wwan 接口"
fi

echo "冒烟测试完成"
```

---

# 附录 E：DPI 背光与亮度建议
- 若 `dtparam=backlight-gpio=<N>` 已设置，系统会导出对应 **GPIO 背光（Backlight）** 设备；也可通过外置 **恒流驱动（LED driver）** 用 I²C/PWM 调光。  
- 注意将功放与背光电源域与数字/RF 分区，单点汇接，避免条纹/噪声耦合到显示与音频通道。

---

# 交付验收摘要（一页版）
- **硬件**：CSI‑2 等长/阻抗、DPI 时序/阻抗、USB（蜂窝）差分/ESD、SIM/eSIM 走线与电压、射频天线净空与匹配、电源完整性与热设计、调试测试点。  
- **系统**：KMS `vc4-kms-dpi-generic` 输出 RGB666；双摄 H.264 硬编推流；ALSA 录放通；NetworkManager/ModemManager 蜂窝一键联网（含自启动/诊断）。  
- **产测**：ATE 自动化脚本通过；≥10 台样机全项 PASS；72 小时在线稳定；日志归档与复现补丁。

---

