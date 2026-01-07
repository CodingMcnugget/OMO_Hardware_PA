# BOM

**版本/Version**：v0.2（整理版）

**日期/Date**：2026-01-06（America/Los_Angeles）

**范围/Scope**：电子料 BOM（Electronics BOM）估算 + 整机 COGS（Cost of Goods Sold）估算 + Plan B 备选 + 全球供应/中美风险 + 良率（Yield/FPY）风险 + 毛利率（Gross Margin）估算

**计价口径/Pricing Basis**：深圳供应链（LCSC/嘉立创等）公开价锚点 + 中国 B2B 常见报价区间（用于估算；最终以 RFQ/合同价为准）

**需求约束/Constraints**：单路摄像头（Single camera）；单 SoC + 高性能 MCU（SoC + High-performance MCU）；保留语音 I/O（Audio I/O）；保留 DPI（RGB 并口）屏幕（DPI/RGB Parallel）；默认保留 4G 蜂窝通信（LTE module）

---

## 变更记录（Change Log）

- **v0.2**：显示子系统档位定义更新为“**4.3" 固定**（按分辨率/亮度分档）”，并在**成本/毛利**部分给出对应的**修正口径与敏感性说明**（见第 5、8 节）。
    - 说明：文中仍保留原先“4.3/5/7 英寸”分档的内容作为历史假设与对照，不删除任何信息，仅在成本模型处补充“以新档位为准”的修正说明。

---

## 0. 结论摘要（Executive Summary）

- **目标成本（Target Cost）**：在“**带 4G（LTE）+ 单摄 + 4.3" DPI 屏**”前提下，**电子料 BOM（Electronics BOM）做到约 28~31 美金**是合理可达的；若把**PCB/SMT、整机组装、外壳、附件、包装**也算进 COGS，则整机 **COGS 约 42~60 美金（按低/中/高配置）**。
- **核心选型方向（Key Selections）**：
    - 主控 **SoC（System-on-Chip）**：Allwinner **V833**（Linux/视频编解码/CSI/LCD）——LCSC 可查到 **100+ 单价约 $4.1037** 的锚点。 [LCSC Electronics](https://www.lcsc.com/product-detail/C3036462.html?utm_source=chatgpt.com)
    - 高性能 **MCU（Microcontroller Unit）**：Espressif **ESP32-S3（BLE/Wi-Fi/协处理/低功耗守护）**——LCSC 可查到 **1000+ 单价约 $2.2053** 的锚点。 [LCSC Electronics](https://www.lcsc.com/product-detail/C2913196.html?utm_source=chatgpt.com)
    - 蜂窝 **4G 模组（LTE Cat.1 module）**：Quectel **EC200U**（USB、3.3~4.3V、Linux 驱动生态）——深圳嘉立创商城参考价 **¥59.98**（作为成本锚点；以实际询价为准）。
- **Plan B**：关键件（4G 模组/PMIC/摄像头/屏幕/存储）提供**可替代、可切换、可量产**的备选路线，并给出 4G 模组“**同封装/兼容封装迁移**”思路（EC200U 与 EC25/EG25 等家族兼容）。
- **中美供应链风险（重点在 4G 模组）**：美国监管对通信设备供应链的审查与限制持续存在，FCC Covered List 为公开基准。 [Federal Communications Commission](https://www.fcc.gov/supplychain/coveredlist?utm_source=chatgpt.com) Reuters 报道过 FCC 主席曾要求评估对 Quectel/Fibocom 的限制可能性。 [Reuters](https://www.reuters.com/technology/us-fcc-chair-asks-agencies-consider-restrictions-quectel-fibocom-2023-09-06/?utm_source=chatgpt.com) Reuters 亦报道 FCC 在 2025 年提出进一步收紧对中国相关设备限制的措施。 [Reuters](https://www.reuters.com/business/media-telecom/us-fcc-vote-tighten-restrictions-chinese-equipment-2025-10-06/?utm_source=chatgpt.com)
- **定价与毛利（Pricing & Gross Margin）**：若硬件单价按 **$199~$399**，在 COGS 约 $42~$60 情况下，硬件毛利率约 **70%~89%**（取决于售价与配置）。显示档位改动可能影响中/高配 COGS 与毛利（见第 8 节“修正说明”）。

---

## 1. 需求基线与本次调整点（Requirements Baseline & Updates）

### 1.1 原始需求基线（来自 README）

README 的硬件需求表明确包含：

- **显示（Display）**：RGB666 TFT **4.3"~7"**，**DPI 并口（DPI/RGB666）** README
- **摄像头（Camera）**：原方案是 **2× MIPI-CSI**（双摄） README
- **音频（Audio）**：PDM 双麦阵列（DMIC）/I²S，输出 I²S→数字功放 MAX98357A→喇叭 README
- **蜂窝（Cellular）**：USB 4G/5G（Quectel EC25/EG25/EC200…），并强调 SIM/eSIM、USB 供电能力 Waybox Teaser_追光资本
- **电源管理（PMIC）**：1 节锂电 + 路径管理（如 BQ25895）+ 升压 5V Waybox Teaser_追光资本
    - 此外版图要求里强调：USB（蜂窝）差分、**4G 峰值电流可达 2A**、模组电源路径与大电容（≥470µF 低 ESR）等 Waybox Teaser_追光资本。

### 1.2 本次调整与采用的解释（This Revision Assumptions）

- **不要双路摄像头**：改为 **单路摄像头（Single camera）**
- **主板：单 SoC + 高性能 MCU**：主控 SoC 负责 Linux/视频/蜂窝，MCU 做 BLE/低功耗守护/电源与外设管理
- **仍需语音 I/O**：保留双麦或单麦 + 扬声器链路
- **仍需 DPI 屏幕**：维持 RGB 并口（TTL/RGB）为主流低功耗方案
- **仍需蜂窝通信（4G 模组）**：默认保留；同时在策略层给出**可选 Wi-Fi-only SKU**（不影响“仍需 4G”的主方案结论）

---

## 2. 推荐硬件架构（Architecture：Single SoC + High-perf MCU）

### 2.1 系统分工（SoC vs MCU）

- **SoC（System-on-Chip）= 主算力/多媒体主机**
    - Linux（Buildroot/Yocto）、视频采集/编码（H.264/H.265）、屏幕输出（RGB/LCD）、USB Host 接 4G 模组
- **高性能 MCU（MCU）= 低功耗守护/外设与产测控制器**
    - **BLE（Bluetooth Low Energy）**：手机控制/配网/OTA 协助（符合 README 对 BLE 同步定位） README
    - 电源与电池管理（Power/Battery management）：开关机序列、背光调光、低电量保护、模组硬断电（Hard power cut）
    - 产测/治具接口（ATE fixtures）：串口日志、GPIO 自检、蜂窝模组 PWRKEY/RESET 控制

### 2.2 主控 SoC 选型：Allwinner V833（Plan A）

- **价格锚点（Price Anchor）**：LCSC 上 V833 **100+ 单价约 $4.1037**。 [LCSC Electronics](https://www.lcsc.com/product-detail/C3036462.html?utm_source=chatgpt.com)
- **能力锚点（Capability Anchor，用于单摄行车记录仪级别）**：文中以“支持视频编解码与摄像头接口”等作为选择理由（具体以 SoC datasheet/SDK 验证为准）。

> 注释（Comment）：你原文中出现过一个 LCSC 链接编号与 V833 不一致的情况；此处不删除任何原文信息，仅将“V833 的价格锚点”用 LCSC V833 条目补充为可核对的公开锚点。 LCSC Electronics
> 

### 2.3 MCU 选型：ESP32-S3（Plan A）

- **价格锚点**：LCSC 可查 ESP32-S3FN8 **1000+ 单价约 $2.2053**。 [LCSC Electronics](https://www.lcsc.com/product-detail/C2913196.html?utm_source=chatgpt.com)
- 价值：BLE/Wi-Fi 一体，量产生态成熟；做“副控/低功耗/产测控制”适配。

---

## 3. BOM 总览（深圳供应链价格口径）+ 三档配置（Configurations）

### 3.1 计价口径说明（Pricing Notes）

- 币种：优先 **USD**，同时给 **RMB（¥）**大致换算（按 1USD≈7.2RMB 仅用于估算）。
- 价格来源：**深圳嘉立创/立创商城（szlcsc）、LCSC**为主；屏/摄像头模组用中国供应链常见 B2B 报价区间（Made-in-China/Alibaba 等）作为锚点。
- 三档含义（原版定义，保留不删）：
    - **低配（Low）**：4.3" 屏 + 2MP 单摄 + LTE Cat.1 + 基础存储
    - **中配（Mid）**：5" 屏 + 更好镜头/更高亮度 + LTE Cat.1 + 更稳电源与散热
    - **高配（High）**：7" 屏 + 4MP 单摄/更好 ISP 方案 + LTE Cat.4（可选） + 更大电池/更强散热
- 目标：**电子料 BOM（不含 PCB/SMT/外壳包装）尽量贴近 $30**，整机 COGS 由此推导。

> 注释（Comment）：显示档位在第 4.2 节已更新为“4.3" 固定分档”；上面“中配=5" /高配=7"”作为历史假设保留。成本/毛利的“以新档位为准”的修正见第 5、8 节。
> 

---

## 4. 详细 BOM（Plan A / Plan B）——按子系统拆解（Subsystem Breakdown）

> 注释（Comment）：表格里所有“Plan B”以可量产替换为准：要么脚位/封装兼容（footprint compatible），要么通过“子板/模组化（daughterboard）”保证不改主板也能切换。
> 

### 4.1 核心计算与连接（Compute & Connectivity）

| 子系统 | 关键件（Key Part） | Plan A（主推） | Plan B（备选） | 低配单价（USD） | 中配单价（USD） | 高配单价（USD） | 供应链/风险要点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 主控 SoC（Main SoC） | Allwinner **V833** | V833（LFBGA-273） | V831/V853 或同类车载视频 SoC（需评估 ISP/SDK） | 4.10 | 3.80 | 3.50 | V833 LCSC 价格锚点约 $4.1037（100+）。 [LCSC Electronics](https://www.lcsc.com/product-detail/C3036462.html?utm_source=chatgpt.com) |
| MCU（High-perf MCU） | **ESP32-S3** | ESP32-S3FN8（BLE/Wi-Fi） | ESP32-C3/S3-WROOM 模组（Module） | 2.30 | 2.20 | 2.10 | LCSC 单价锚点：1000+ 约 $2.2053。 [LCSC Electronics](https://www.lcsc.com/product-detail/C2913196.html?utm_source=chatgpt.com) |
| 蜂窝 4G（LTE module） | **LTE Cat.1** | Quectel **EC200U** | SIMCom **A7670C** 或 China Mobile **ML307S** | 7.5~8.5 | 7.5~9.0 | 12~18（Cat4） | EC200U 深圳参考价 ¥59.98；A7670C LCSC ~$7.23（10+）；ML307S ¥53。 |
| SIM（SIM/eSIM） | nano-SIM座 + eSIM焊盘（可选） | nano-SIM（必备） | eSIM（eUICC）上高配/企业版 | 0.25 | 0.35 | 0.6~2.5 | README 建议 nano-SIM + eSIM 并存。Waybox Teaser_追光资本 |
| 天线（Antenna） | LTE 主天线 + 可选分集（DIV） | 1× LTE | 2× LTE（MIMO/DIV） | 0.8 | 1.2 | 1.8 | README 要求 u.FL/IPEX 与 π 匹配位。Waybox Teaser_追光资本 |

**关于“4G 是否必须”的结论（保留原文信息，不删）**：

- 你的需求是“仍需 4G”，按此做主方案。
- 策略层建议准备 **Wi-Fi only SKU**（去掉 LTE 模组与 SIM/天线），可降低 **$7~$10** 级别 BOM，并降低部分美国合规与舆情风险（见第 6 节）。

---

### 4.2 显示（Display, DPI/RGB Parallel）

### 4.2.1 显示成本估算区间（原版成本区间，保留不删）

| 子系统 | 关键件 | Plan A | Plan B | 低配（USD） | 中配（USD） | 高配（USD） | 供应链/风险要点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| LCD（TFT） | 4.3"~7" RGB（TTL） | 4.3" 480×272 RGB | 5" 800×480 或 7" 1024×600 RGB | 3.0~4.5 | 4.5~7.0 | 7.0~11.0 | 4.3" TTL/RGB 屏在中国 B2B 报价常见 **$2.50~$5.00** 区间（不含触控）。 |
| 背光（Backlight） | LED driver（恒流）+ PWM/I²C | 简单 PWM+升压/恒流 | 专用 LED driver（更稳条纹噪声） | 0.3 | 0.5 | 0.8 | README 建议背光恒流独立供电与地分区，避免条纹。Waybox Teaser_追光资本 |

> 注释（Comment）：上表为“尺寸分档”的原成本区间，用于与下方“4.3 固定分档”的新规格对照；成本/毛利修正见第 5、8 节。
> 

### 4.2.2 显示档位定义（新版：4.3" 固定，按分辨率/亮度分档）

| 档位 | 尺寸（Size） | 分辨率（Resolution） | 面板类型（Panel） | 亮度（Luminance） | 接口（Interface） | 关键意义（Why it’s a tier） |
| --- | --- | --- | --- | --- | --- | --- |
| 低配（Low） | 4.3" 固定 | **480×272** | IPS/TN 均可（优先稳定供货） | 典型 350~500 nits（厂商级别差异） | RGB 24-bit | **像素少 → 帧缓存/带宽/时钟压力更低**；功耗与 EMI 更容易控。Winstar 4.3" 480×272、RGB 24-bit 的规格示例可参考。 [Winstar+1](https://www.winstar.com.tw/products/tft-lcd/module/tft-lcd-4_3.html?utm_source=chatgpt.com) |
| 中配（Mid） | 4.3" 固定 | **800×480** | IPS（视角与颜色更一致） | 典型 600 nits | RGB 24-bit | UI/表情细节更好，仍保持同尺寸结构不变。Winstar 4.3" 800×480、RGB、600 nits 的规格示例可参考。 [Winstar](https://www.winstar.com.tw/products/tft-lcd/ips-tft/wf43xtwagdnn0.html?utm_source=chatgpt.com) |
| 高配（High） | 4.3" 固定 | **800×480**（不变） | IPS（车内可视更稳） | **更高亮度版本**（例如 high-brightness） | RGB 24-bit | **不靠尺寸区分，而靠“可视性/车载强光可读/宽温一致性/供货等级”区分**。Winstar 高亮度 4.3" 800×480（例：1100 nits）规格示例可参考。 [Winstar](https://www.winstar.com.tw/products/tft-lcd/ips-tft/wf43xswagdnn0.html?utm_source=chatgpt.com) |

---

### 4.3 摄像头（Camera）——单路，行车记录仪规格匹配

| 子系统 | 关键件 | Plan A | Plan B | 低配（USD） | 中配（USD） | 高配（USD） | 供应链/风险要点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Camera module | 2MP/1080P MIPI 模组 | GC2053/OV2710 类 2MP MIPI | USB UVC 1080P（标准摄像头模组） | 4.0~6.0 | 5.0~7.5 | 8.0~15.0 | 市面常见摄像头模组报价区间可参考：GC2053/OV4689 等 DVP/MIPI 模组 **$3.90~$9.90**。 |
| 备用高阶镜头 | 更大光圈/更低畸变 | 6G 镜头 MIPI 模组 | 更换镜头供应商 | — | +1~2 | +4~7 | 深圳渠道中，带更高规格镜头的 MIPI 模组可见到 **¥68~¥75** 区间。 |

**为什么准备 Plan B 的 “USB UVC 摄像头”（保留原文信息，不删）**：

- MIPI-CSI（MIPI Camera Serial Interface）量产风险常集中在**模组 pinout、FPC、信号完整性**与**模组供货**；USB UVC 的优势是“**快速换供应商**”，代价是 USB 资源与功耗略升。

---

### 4.4 语音 I/O（Audio：Mic & Speaker）

| 子系统 | 关键件 | Plan A | Plan B | 低配（USD） | 中配（USD） | 高配（USD） | 供应链/风险要点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 麦克风（Mic） | PDM/DMIC 双麦（2-mic array） | TDK **MMICT390200012**（PDM）×2 | 低成本模拟 MEMS（Analog MEMS）×2 + codec/前端 | 0.9~1.2 | 1.0~1.6 | 1.2~2.5 | 原文保留：LCSC 可查 PDM 麦 “Price from $0.4373”；模拟 MEMS 也有更低价位选择。 |
| 功放（Amp） | I²S 数字功放（Digital amp） | **MAX98357A**（I²S→SPK） | NS4168/TPA 系列或 Class-D 替代 | 1.0~1.3 | 1.0~1.6 | 1.2~2.0 | 原文保留：MAX98357A LCSC “Price from $1.0504”。 |
| 喇叭（Speaker） | 4Ω 3W | 3525BOX-4Ω3W | 40mm/8Ω 2W（视结构） | 0.4~0.8 | 0.5~1.0 | 0.8~1.5 | 原文保留：LCSC 可见 4Ω3W 喇叭约 $0.40（1000+）。 |

---

### 4.5 电源管理与电池（Power, PMIC, Battery）

| 子系统 | 关键件 | Plan A | Plan B | 低配（USD） | 中配（USD） | 高配（USD） | 供应链/风险要点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 充电/路径管理（Charger + Power Path） | PMIC（Power Management IC） | TI **BQ25895**（路径管理） | SGM41542（SG Micro）/ Injoinic IP 系列 | 0.8~1.8 | 1.0~2.2 | 1.5~3.0 | README 点名 BQ25895 作为路径管理示例。Waybox Teaser_追光资本 |
| 电源大电容 | ≥470µF 低 ESR | 470~1000µF | 同规格多家替代 | 0.15 | 0.25 | 0.35 | README 强调 4G 峰值电流与大电容/低阻路径。Waybox Teaser_追光资本 |
| 电池（Battery） | 1S 锂电（Li-ion/LiPo） | 18650 2000mAh（或 LiPo 2000mAh） | 2500~3000mAh（高配） | 1.0~2.0 | 1.5~2.8 | 2.2~4.0 | 深圳渠道 18650 2000mAh 电芯可见 **¥5.00（≥500只）**级别锚点（不含 PCM/包装）。 |
| 4G 供电策略（LTE Power） | 独立电源开关（load switch） | 模组电源可硬断电 | 同 | 0.15 | 0.25 | 0.35 | README 建议蜂窝模组独立低阻电源路径并可硬断电。Waybox Teaser_追光资本 |

**续航 3~4 小时的电池估算（原文保留，不删）**：

- 典型工作功耗（粗估）：屏幕 0.5~1.2W + SoC 1~2W + LTE 平均 0.5~2W（峰值更高但不连续）
- 若按平均 2.0~2.5W 计算，4 小时需要 8~10Wh；对应 1S 3.7V 电池约 2200~2700mAh。
- 所以：低配 2000mAh；中配 2500mAh；高配 3000mAh。

---

### 4.6 其他通用项（PCB/连接器/结构/散热/制造）

| 项目 | 低配（USD） | 中配（USD） | 高配（USD） | 备注 |
| --- | --- | --- | --- | --- |
| PCB + SMT（4L 量产） | 2.5~3.5 | 3.0~4.0 | 3.5~5.0 | BGA SoC + LTE 模组对制程要求略高 |
| 连接器/小料（FPC/USB-C/u.FL/SIM） | 1.0~1.8 | 1.2~2.2 | 1.5~2.8 | 双 u.FL/更大屏 FPC 会增加 |
| 外壳/结构件（Plastic + screws + gasket） | 2.5~4.0 | 3.0~5.0 | 4.0~7.0 | 车载温度与卡扣强度要注意 |
| 散热（石墨/导热垫/小散热片） | 0.6~1.2 | 0.8~1.6 | 1.2~2.5 | Teaser 强调被动散热结构 Waybox Teaser_追光资本 |
| 包装/附件（Packaging & accessories） | 1.5~3.0 | 1.8~3.5 | 2.0~4.0 | 线材/支架/车充是否标配影响大 |

---

## 5. 成本汇总（三档）——电子料 BOM vs 整机 COGS（Cost Summary）

> 注释（Comment）：成本拆为两层：电子料 BOM（Electronics BOM） vs 整机 COGS（含制造装配）。下列为估算区间与点估算并存的呈现方式。
> 

### 5.1 电子料 BOM（Electronics BOM）估算（原文保留）

- **低配（4.3"+2MP+Cat1）**：**$28~31**（目标贴近 $30）
- **中配（5"+更稳电源/镜头）**：**$32~36**
- **高配（7"+Cat4+更高阶摄像头）**：**$42~52**

### 5.2 整机 COGS（制造后可出货，原文保留）

- **低配 COGS**：约 **$42**（电子料 + PCB/SMT + 外壳 + 附件 + 包装 + 小幅良率摊销）
- **中配 COGS**：约 **$48**
- **高配 COGS**：约 **$60**

### 5.3 显示档位更新后的“成本模型修正说明”（仅针对屏幕变更带来的影响）

**问题来源（保留信息并客观表述）**：

- 文中同时存在两套显示分档：
    1. 原版按尺寸（4.3/5/7 英寸）分档（见 3.1 与 4.2.1）；
    2. 新版按“4.3 固定，分辨率/亮度分档”（见 4.2.2）。
- 若以**新版显示分档**为准，则中配/高配不再因“屏幕尺寸”产生明显增量；因此 **COGS 与毛利率**可能需要下修显示相关成本增量。

**修正口径（推算，明确标注为估算）**：

- 基于 4.2.1 中给出的显示模组成本区间（4.3"：$3.0~4.5；5"：$4.5~7.0；7"：$7.0~11.0），仅把“尺寸增量”替换为 4.3" 固定后，可得到显示项对 COGS 的潜在下修区间：
    - 中配显示项下修：**0~4 美金**（(4.5~7.0) − (3.0~4.5)）
    - 高配显示项下修：**2.5~8 美金**（(7.0~11.0) − (3.0~4.5)）
- 为便于单点演示（示例值，非最终报价）：可采用 **中配下修 $2、 高配下修 $5**，用于第 8 节毛利表“修正版”。

---

## 6. Plan B 供应保障策略（Supply Continuity Plan）

### 6.1 4G 模组（LTE module）Plan B（最关键）

**风险来源（原文保留）**：

- 4G 模组受：认证（FCC/PTCRB/运营商）、地缘政治、芯片供给周期、以及渠道波动影响最大。

**Plan A：Quectel EC200U（Cat.1 bis）**

- 深圳渠道参考价可查到 ¥59.98，且接口/驱动生态适配 Linux。

**Plan B-1：同厂“兼容封装迁移”（原文保留）**

- Quectel 官方说明 EC200U 与 EC25/EG25 等系列在封装上具备兼容迁移思路（便于同 PCB 设计切换）。
    - 工程实现建议：在 PCB 上预留“同一 LCC 家族”的关键引脚兼容，或者做一个小型 modem daughterboard（模组子板）统一对外 USB/SIM/RF，主板只认 USB + 控制脚。

**Plan B-2：SIMCom A7670C（Cat.1）**

- LCSC 可查到 A7670C-LASE 单价 $7.2284（10+）。
    - 用途：当 EC200U 交期/价格不稳定时快速切换；注意不同模组的 USB PID/驱动枚举差异，软件用 ModemManager/MBIM/QMI 做抽象（README 也以此为验收路径）。 README

**Plan B-3：China Mobile ML307S（Cat.1）**

- 深圳嘉立创可查到参考价 ¥53。
    - 用途：国内供货更稳、更便宜的备选；海外频段/认证要单独评估。

### 6.2 中美/美国市场合规风险（Compliance Notes）

- FCC 有官方的 **Covered List（受限清单）**作为公开基准。 [Federal Communications Commission](https://www.fcc.gov/supplychain/coveredlist?utm_source=chatgpt.com)
- Reuters 报道过 FCC 主席要求相关机构评估对 **Quectel/Fibocom** 等中国蜂窝模组厂商的限制可能性。 [Reuters](https://www.reuters.com/technology/us-fcc-chair-asks-agencies-consider-restrictions-quectel-fibocom-2023-09-06/?utm_source=chatgpt.com)
- Reuters 亦报道 FCC 在 2025 年拟进一步收紧对中国企业相关设备的限制措施（更广泛的合规环境趋严）。 [Reuters](https://www.reuters.com/business/media-telecom/us-fcc-vote-tighten-restrictions-chinese-equipment-2025-10-06/?utm_source=chatgpt.com)

**可控策略（原文保留并客观化）**：

1. 默认准备 **双 SKU 策略**：
    - **Global SKU**：EC200U / A7670C（成本优、供应链成熟）
    - **US Compliance SKU（备选）**：必要时切换到“非敏感供应商”模组（如 Telit Cinterion/Thales 等，成本更高但面向特定渠道/企业订单）
2. 结构与电气上做 **模组化/可替换**：USB 接入 + SIM/RF 统一，避免主板大改。
3. 认证路径提前锁定：PTCRB 数据库与 FCC ID 流程在 EVT/DVT 即启动；PTCRB Certified Devices 数据库为公开入口之一。 [PTCRB](https://www.ptcrb.com/certified-devices/?utm_source=chatgpt.com)

---

## 7. 良品率（Yield）与量产风险分析（Manufacturing & FPY）

> 注释（Comment）：器件晶圆良率通常不可获得公开精确值；此处关注整机制造良率（FPY, First Pass Yield）与爬坡（Ramp）风险。
> 

### 7.1 预期良率（原文保留）

- EVT → DVT：整机 FPY 常见 **92%~97%**
- PVT 稳定后：整机 FPY **97%~99%**
- 稳态量产：整机 FPY **98.5%~99.5%**
    - 说明：取决于测试覆盖率与供应商稳定性；逻辑为通过 ATE 覆盖与关键件来料 IQC 把 FPY 拉到 98%+。

### 7.2 关键失效模式（Top Failure Modes）与对策（原文保留）

| 风险点 | 常见问题 | 对策（可写进量产计划） |
| --- | --- | --- |
| SoC BGA 焊接（BGA soldering） | 虚焊、锡珠、翘曲 | 4L/6L 叠层与回流曲线、X-Ray 抽检、BGA underfill（可选） |
| MIPI/屏线（FPC/FFC） | 接触不良、插反、ESD | 选带锁扣连接器；ESD 器件；结构防呆 |
| 摄像头模组（Camera module） | 对焦偏差、灰尘、暗角 | 模组供应商做出厂光学测试；整机端做上电画面自检 |
| 4G 模组（LTE module） | 峰值掉电、USB 反复重连 | 低阻供电路径 + ≥470µF 低 ESR；模组可硬断电复位（PWRKEY/Load switch）Waybox Teaser_追光资本 |
| 音频（Audio） | 底噪/啸叫/串扰 | 功放与 RF/数字地分区、单点汇接（README 也强调）Waybox Teaser_追光资本 |
| 电池与充电（Battery/Charging） | 过热、鼓包、充电异常 | 选用带 NTC；充电策略限流；UN38.3/MSDS 资料齐全 |

### 7.3 产测（ATE）覆盖建议（原文保留）

- 将验收脚本固化为工厂 ATE（Automated Test Equipment）流程：
    - Camera capture → 生成 H.264 文件 → hash 校验
    - Display 上电测试图 → 背光 PWM 分档
    - Audio 录放自检
    - Cellular attach/ping/iperf3 稳定性（20 分钟）README

---

## 8. 毛利率（Gross Margin）估算：按 $199~$399 定价区间

### 8.1 硬件毛利率（不含 Kickstarter 手续费/支付费，原文保留）

| 方案 | COGS（USD） | 售价 $199 | 售价 $299 | 售价 $399 |
| --- | --- | --- | --- | --- |
| 低配（Low） | 42.3 | **78.7%** | **85.9%** | **89.4%** |
| 中配（Mid） | 48.0 | **75.9%** | **83.9%** | **87.0%** |
| 高配（High） | 60.0 | **69.8%** | **79.9%** | **85.0%** |

### 8.2 考虑 Kickstarter/支付综合费率 10% 后的“贡献毛利率”（原文保留）

| 方案 | COGS（USD） | 售价 $199 | 售价 $299 | 售价 $399 |
| --- | --- | --- | --- | --- |
| 低配（Low） | 42.3 | **68.7%** | **75.9%** | **79.4%** |
| 中配（Mid） | 48.0 | **65.9%** | **73.9%** | **78.0%** |
| 高配（High） | 60.0 | **59.8%** | **69.9%** | **75.0%** |

> 备注（原文保留）：若把运费单独向用户收取，毛利率更好；若包邮，需把国际物流与关税计入 COGS。
> 

### 8.3 显示档位更新后的毛利“修正版”（仅修正屏幕变更对 COGS/毛利的影响）

**修正依据**：第 5.3 节给出显示尺寸增量下修区间；这里按“示例值”演示（非最终报价）：

- 中配 COGS：**48.0 − 2.0 = 46.0**
- 高配 COGS：**60.0 − 5.0 = 55.0**

### 8.3.1 修正版：硬件毛利率（示例）

| 方案 | COGS（USD） | 售价 $199 | 售价 $299 | 售价 $399 |
| --- | --- | --- | --- | --- |
| 低配（Low） | 42.3 | **78.7%** | **85.9%** | **89.4%** |
| 中配（Mid, 4.3" 固定分档示例） | 46.0 | **76.9%** | **84.6%** | **88.5%** |
| 高配（High, 4.3" 固定分档示例） | 55.0 | **72.4%** | **81.6%** | **86.2%** |

### 8.3.2 修正版：贡献毛利率（扣 10% 平台/支付费，示例）

| 方案 | COGS（USD） | 售价 $199 | 售价 $299 | 售价 $399 |
| --- | --- | --- | --- | --- |
| 低配（Low） | 42.3 | **68.7%** | **75.9%** | **79.4%** |
| 中配（Mid, 示例） | 46.0 | **66.9%** | **74.6%** | **78.5%** |
| 高配（High, 示例） | 55.0 | **62.4%** | **71.6%** | **76.2%** |

> 注释（Comment）：上述“修正版”只对屏幕尺寸分档→4.3 固定分档引起的增量做演示性修正；其他子系统（蜂窝 Cat1/Cat4、摄像头规格、电池容量、散热/结构件等）的差异仍按原模型保留。最终以 RFQ 与 DVT BOM 冻结为准。
> 

---

## 9. 供应链与量产控制要点（Key Control Points）

1. **成本锚点可核对**：关键大件（SoC/MCU 等）可通过公开渠道价格锚点进行复核（如 LCSC）。 [LCSC Electronics+1](https://www.lcsc.com/product-detail/C3036462.html?utm_source=chatgpt.com)
2. **Plan B 的可切换机制**：4G 模组、PMIC、摄像头路径等不仅提供备选料号，也给出“兼容迁移/子板化”的切换方式，降低断供或合规变化时的工程返工成本。
3. **合规与地缘风险可分层处理**：以 FCC Covered List 等公开信息为外部基准，并通过“双 SKU + 模组可替换”将风险从“单点失效”转为“成本与认证路径差异”。 [Federal Communications Commission+2Reuters+2](https://www.fcc.gov/supplychain/coveredlist?utm_source=chatgpt.com)
4. **量产卡点提前工程化**：4G 峰值电流、电源低阻路径与大电容、RF 匹配、BGA 工艺、光学良率、ATE 覆盖等均作为量产计划的硬约束项，目标 FPY 拉升至 98%+（按原文良率模型）。

---

## 可选导出格式（Optional Deliverables，保留原文意图）

- 一页 “**BOM & COGS Summary（含三档配置）**”
- 一页 “**Plan B & 地缘风险矩阵**”
- 一页 “**良率/产测流程图**”
    
    （中英对照同样保留；可直接用于上会 PPT）
    

---

### 你提到的“屏幕改动”已处理的范围说明（Summary of What Was Corrected）

- 我没有删除任何段落、表格或数字，只做了：
    1. 全文去“非客观话术”的表达方式（例如“谁最爱问/最喜欢”之类措辞改为客观标题与表述）；
    2. **新增**“显示档位更新对成本/毛利影响”的**修正口径与示例表**（第 5.3、8.3），以保证文档内部逻辑在“4.3 固定分档”下依然自洽。

ChatGPT can make mistakes. Check important info.
