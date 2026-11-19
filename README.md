# GigE Vision 3.0 规范知识库

> 基于 **《GigE Vision™ Specification, Version 3.0.00》**（A3 Vision & Imaging，416 页，封面日期印作 “2026-24-4”，即 2026 年 4 月）整理的**中文学习笔记与工程参考**。
> 正文用中文讲解，报文名、字段名、寄存器名、状态码、GenICam feature 名保留英文，便于与规范、SDK 和抓包工具对照。

---

## 这个知识库包含什么

| 类别 | 内容 |
|---|---|
| 📘 章节笔记（16 篇） | 覆盖规范全部 39 章与 4 个附录：核心要点 → 按章节逐节讲解（附 `§章节, p.页码` 出处）→ 报文/寄存器字段表（字节偏移、位掩码）→ mermaid 时序图/流程图 → 实现要点与易错点（分设备端/应用端）→ 相关章节 → 自测题 |
| 🧭 综合文档（2 篇） | **端到端流程（从上电到出图）**、**速查表**（端口、命令码、状态码、常用寄存器与关键位） |
| 📑 参考索引 | **521 条现行需求**的编号/类型/适用对象/章节/页码/引入版本 + 中文要点（Markdown 与 CSV）、**197 条已废弃需求**、Bootstrap 寄存器地址总表（223 行） |
| ⚠️ 存疑汇总 | 精读时发现的规范笔误与前后矛盾（见[文末](#规范存疑点汇总)） |

> **版权说明**：规范版权归 Association for Advancing Automation（A3）所有，封面声明禁止将其复制到电子检索系统。本知识库**不包含规范原文**，只包含学习笔记（意译、总结、解读）、事实性数据（地址、数值、字段布局）和极少量单句引用。需要原文时，请按页码查阅你本地的 PDF（方法见[如何查原文](#如何查原文)）。

---

## 目录结构

```text
knowledge_base/
├── README.md                                   ← 本文件：索引、学习路线、约定
├── 00_overview/
│   ├── 01_introduction_and_architecture.md     第 1 章：目的、需求编号规则、参考文档、系统架构
│   ├── 02_glossary.md                          术语表：25 条定义 + 91 个缩略语（中英对照）
│   ├── 03_version_history_and_v3_changes.md    附录 D：1.0→3.0 版本历史，3.0 新增/变更/废弃，2.x→3.0 迁移清单
│   ├── 04_end_to_end_walkthrough.md            端到端流程：上电→IP→发现→XML→控制权→配置→出流→断开
│   └── 05_quick_reference.md                   速查表：端口、报文头、命令码、状态码、常用寄存器、关键位
├── 01_device_discovery/
│   └── device_discovery.md                     Part 1（Ch2–6）：物理链路/LAG、IP 配置、DISCOVERY、mDNS/DNS-SD
├── 02_gvcp/
│   ├── 01_transport_and_channels.md            Ch7–13：UDP 可靠性、控制/流/消息通道、权限、心跳、包长协商
│   ├── 02_advanced_concepts.md                 Ch14：XML 获取（URL/Manifest）、IEEE 1588、ACTION_CMD、主应用切换
│   ├── 03_message_formats.md                   Ch15–17：报文头与全部命令/确认/事件报文的字节级格式
│   └── 04_command_values_status_events.md      Ch18–21 + 附录 B：命令码、状态码详解、事件、ICMP
├── 03_gvsp/
│   ├── 01_transport_and_data_block.md          Ch22–24：GVSP 传输、数据块、20 字节包头、leader/payload/trailer、chunk、测试包
│   └── 02_payload_types_and_pixel_layouts.md   Ch25–26：负载类型（Image/Chunk/GenDC）、像素布局与填充
├── 04_gvrsp/                                   ★ 3.0 新增的 RoCEv2 流协议
│   ├── 01_rocev2_and_streaming_model.md        Ch27–30：RoCEv2 基础、GenDC 流模型、buffer layout table、流通道开关
│   └── 02_connection_and_flow_control.md       Ch31–35 + 附录 C：CM 连接/释放、上行消息、credit 流控、主机注意事项
├── 05_bootstrap_registers/
│   ├── 01_overview_and_register_map.md         Ch36 总则 + 按功能分组的寄存器映射
│   ├── 02_device_and_network_registers.md      §36.1–36.45：设备、网络、能力、时间戳、1588 等寄存器位定义
│   └── 03_channel_registers.md                 §36.46–36.81：CCP、消息通道、GVSP/GVRSP 流通道、Manifest、Action
├── 06_interoperability/
│   └── interoperability_and_testing.md         Ch37–39：相机强制 feature、GenDC 流、测试与合规
└── reference/                                  （脚本从 PDF 自动提取，人工核对）
    ├── requirements_index.md / .csv            521 条现行需求 + 中文要点 + 引入版本（CSV 可在 Excel 筛选）
    ├── deprecated_requirements.md              197 条已废弃需求（按废弃版本分组）
    └── bootstrap_register_map.md               Table 36-1 地址索引（地址/名称/M-O/访问/长度/页码）
```

---

## 规范结构 ↔ 知识库文件

| 规范部分 | 章节 | PDF 页 | 知识库文件 |
|---|---|---|---|
| 引言 | Ch1 | 15–30 | [01_introduction_and_architecture](00_overview/01_introduction_and_architecture.md) · [02_glossary](00_overview/02_glossary.md) |
| Part 1 设备发现 | Ch2–6 | 31–56 | [device_discovery](01_device_discovery/device_discovery.md) |
| Part 2 GVCP | Ch7–13 | 57–93 | [01_transport_and_channels](02_gvcp/01_transport_and_channels.md) |
| | Ch14 | 94–111 | [02_advanced_concepts](02_gvcp/02_advanced_concepts.md) |
| | Ch15–17 | 112–153 | [03_message_formats](02_gvcp/03_message_formats.md) |
| | Ch18–21、附录 B | 154–162、402–406 | [04_command_values_status_events](02_gvcp/04_command_values_status_events.md) |
| Part 3 GVSP | Ch22–24 | 163–185 | [01_transport_and_data_block](03_gvsp/01_transport_and_data_block.md) |
| | Ch25–26 | 186–204 | [02_payload_types_and_pixel_layouts](03_gvsp/02_payload_types_and_pixel_layouts.md) |
| Part 4 GVRSP | Ch27–30 | 205–222 | [01_rocev2_and_streaming_model](04_gvrsp/01_rocev2_and_streaming_model.md) |
| | Ch31–35、附录 C | 223–244、407–408 | [02_connection_and_flow_control](04_gvrsp/02_connection_and_flow_control.md) |
| Part 5 Bootstrap 寄存器 | Ch36 总则与总表 | 245–269 | [01_overview_and_register_map](05_bootstrap_registers/01_overview_and_register_map.md) · [bootstrap_register_map](reference/bootstrap_register_map.md) |
| | §36.1–36.45 | 269–311 | [02_device_and_network_registers](05_bootstrap_registers/02_device_and_network_registers.md) |
| | §36.46–36.81 | 312–352 | [03_channel_registers](05_bootstrap_registers/03_channel_registers.md) |
| Interoperability | Ch37–39 | 353–368 | [interoperability_and_testing](06_interoperability/interoperability_and_testing.md) |
| 附录 A 需求表 | — | 369–401 | [requirements_index](reference/requirements_index.md) · [deprecated_requirements](reference/deprecated_requirements.md) |
| 附录 D 版本历史 | — | 409–416 | [03_version_history_and_v3_changes](00_overview/03_version_history_and_v3_changes.md) |

---

## 推荐学习路线

```mermaid
flowchart LR
    A["1 引言与架构"] --> B["2 端到端流程"]
    B --> C["3 设备发现"]
    C --> D["4 GVCP 通道与报文"]
    D --> E["5 Bootstrap 寄存器"]
    E --> F["6 GVSP"]
    F --> G["7 GVCP 高级概念<br/>1588 / Action / 切换"]
    G --> H["8 GVRSP"]
    H --> I["9 互操作与测试<br/>版本变化"]
```

**快速入门（约 1 小时）**：[引言与架构](00_overview/01_introduction_and_architecture.md) → [端到端流程](00_overview/04_end_to_end_walkthrough.md) → [速查表](00_overview/05_quick_reference.md)

**系统学习**：[设备发现](01_device_discovery/device_discovery.md) → [GVCP 通道](02_gvcp/01_transport_and_channels.md) → [GVCP 报文](02_gvcp/03_message_formats.md) → [命令码/状态码](02_gvcp/04_command_values_status_events.md) → [寄存器总览](05_bootstrap_registers/01_overview_and_register_map.md) → [GVSP 数据块](03_gvsp/01_transport_and_data_block.md) → [负载类型](03_gvsp/02_payload_types_and_pixel_layouts.md) → [GVCP 高级概念](02_gvcp/02_advanced_concepts.md) → [GVRSP 模型](04_gvrsp/01_rocev2_and_streaming_model.md) → [GVRSP 连接与流控](04_gvrsp/02_connection_and_flow_control.md) → [寄存器详解 A](05_bootstrap_registers/02_device_and_network_registers.md) / [B](05_bootstrap_registers/03_channel_registers.md) → [互操作与测试](06_interoperability/interoperability_and_testing.md) → [版本变化](00_overview/03_version_history_and_v3_changes.md)

**按角色**

| 角色 | 重点阅读 | 需求索引筛选 |
|---|---|---|
| 主机 / SDK 开发（应用、流接收端） | 端到端流程、GVCP 报文、GVSP 两篇（接收端易错点）、GVRSP 连接与 credit、心跳与重传 | 适用 = 应用 / 流接收端 / 流收发两端 |
| 相机固件开发（设备、流发送端） | 设备发现（IP 配置）、GVCP 通道（权限/心跳/关闭复位）、寄存器三篇、GVSP 发送端规则、互操作（强制 feature） | 适用 = 设备 / 流发送端 |
| 从 2.x 升级到 3.0 | 版本变化与迁移清单、已废弃需求、GVRSP 两篇、CCP connection_count、PENDING_ACK 强制 | 引入版本 = 3.0 |
| 测试 / 合规 | 需求索引（CSV）、互操作与测试（Ch39）、状态码 | 按章节或适用对象 |

---

## 阅读约定

| 约定 | 说明 |
|---|---|
| 页码 | 一律为 **PDF 页码**（阅读器显示的页码）；规范页脚印刷的页码 = PDF 页码 − 1 |
| 出处格式 | `（§16.7.1, p.137）` = 第 16.7.1 节，PDF 第 137 页 |
| 位编号 | 与规范一致，**MSB-first：bit 0 = 最高位**（32 位寄存器的 bit 31 = 最低位）；寄存器位表附 “LSB0 掩码” 列，如 spec bit 31 → `0x00000001` |
| 字节序 | GVCP/GVSP 报文头、bootstrap 寄存器均为大端（network byte order） |
| 需求编号 | `[R-421cd]` = 第 421 号、仅适用于设备（cd）的绝对需求（R = MUST）；`CR` 条件需求、`O` 目标（SHOULD）、`CO` 条件目标；后缀 ca/cd/c/st/sr/s 见[需求索引](reference/requirements_index.md) |
| 💡 解读 | 笔记作者的补充理解、推导或背景知识，**不是规范原文**，使用前请自行判断 |
| ⚠️ | 规范存疑点、前后不一致或易错点 |

---

## 如何查原文

本知识库不含规范原文。需要核对原文时：

```powershell
$pdf = "D:\GigeV3_Learning\GigE Vision Specification version 3.0.pdf"

# 1) 查看某一页（PDF 页码）的原文，例如 p.137
pdftotext -f 137 -l 137 -layout -enc UTF-8 $pdf -

# 2) 查找某个需求编号或关键字出现在哪些页
python -c "import pypdf; r=pypdf.PdfReader(r'$pdf'); print([i+1 for i,p in enumerate(r.pages) if 'R-421cd' in (p.extract_text() or '')])"
```

- `pdftotext` 随 Git for Windows 提供（`mingw64\bin`）；`pypdf` 可用 `pip install pypdf` 安装。
- 也可以直接在 PDF 阅读器里搜索需求编号（如 `R-421cd`）或报文名（如 `PACKETRESEND_CMD`）。

---

## 规范存疑点汇总

以下是精读时发现的规范笔误或前后不一致（详细讨论见对应笔记，建议对照 PDF 原文确认）：

| # | 位置 | 问题 | 详见 |
|---|---|---|---|
| 1 | p.42 [R-017cd] | 把 0x0024 写作 “Persistent IP Address”，但寄存器表中 0x0024 是 Current IP Address | [device_discovery](01_device_discovery/device_discovery.md) |
| 2 | p.36 vs p.323 | SCPx 的 network_interface_index：§3.2 写 bit 12–15，§36.57 定义为 bit 14–15 | [device_discovery](01_device_discovery/device_discovery.md)、[速查表](00_overview/05_quick_reference.md) |
| 3 | p.50 | “sLAG or mLAG” 疑为 dLAG | [device_discovery](01_device_discovery/device_discovery.md) |
| 4 | p.113 vs p.154 | 保留给 GVCP 的命令值范围：§15.1 写 0–32765，Ch18 写 0–32767 | [03_message_formats](02_gvcp/03_message_formats.md) |
| 5 | p.122、p.126 | 图 16-2 / 16-4 的 acknowledge 字段画成了 CMD 值（应为 ACK 值） | [03_message_formats](02_gvcp/03_message_formats.md) |
| 6 | p.157 | 状态码表 “0x8008 to 0x0800B” 应为 0x800B | [04_command_values_status_events](02_gvcp/04_command_values_status_events.md) |
| 7 | p.77 vs [R-191cd] | 表 10-1 允许次应用发 WRITEMEM，与“仅主应用”的要求矛盾 | [01_transport_and_channels](02_gvcp/01_transport_and_channels.md) |
| 8 | [R-132c] | 标为 R（MUST），正文却是 SHOULD | [03_message_formats](02_gvcp/03_message_formats.md) |
| 9 | p.259 vs p.316–317 | CCMPS/CCTRA：总表标 R/W，详细定义写 Read | [03_channel_registers](05_bootstrap_registers/03_channel_registers.md) |
| 10 | p.263、p.267–268 | Support 列印成数字 “0”（应为 “O”）；“0x8CF0 Reserved” 与 SCMPC511 地址重复 | [01_overview_and_register_map](05_bootstrap_registers/01_overview_and_register_map.md) |
| 11 | p.257/298、p.304、p.307 | 总表与章节定义不一致：Timestamp Value 的 M/O、Streaming Configuration 的适用范围、Physical Link Configuration 的访问类型 | [02_device_and_network_registers](05_bootstrap_registers/02_device_and_network_registers.md) |
| 12 | p.334–337 | SCMPCx/SCEBAx/SCMBSx/SCCx/SCCFGx 的“可选/不适用”标注与对应强制需求冲突 | [03_channel_registers](05_bootstrap_registers/03_channel_registers.md) |
| 13 | p.100 | 时间戳溢出日期写 “Nov 4th, 2262”，按 2^63 ns 推算应为 2262-04-11 | [02_advanced_concepts](02_gvcp/02_advanced_concepts.md) |
| 14 | p.107–108 | 图 14-5 中 ACTION_2 的 key 为 0x12431244，表格为 0x12341244；示例用了不存在的 “GEV_STATUS_OK” | [02_advanced_concepts](02_gvcp/02_advanced_concepts.md) |
| 15 | p.96–97 vs [R-118ca] | XML 扩展名大小写：要求写 `.XML`，字段表却说区分大小写的小写 `xml`/`zip` | [02_advanced_concepts](02_gvcp/02_advanced_concepts.md) |
| 16 | p.140 | 图 16-14 中 HO_shift / hold_off 的位宽与字段表（4 + 12 位）不一致 | [03_message_formats](02_gvcp/03_message_formats.md) |
| 17 | p.178 | 图 24-4 列出 packet_format 6，但字段表只定义 3、5、7、8 | [01_transport_and_data_block](03_gvsp/01_transport_and_data_block.md) |
| 18 | p.173 vs p.176 | [R-256s] 要求至少一个 payload 包，§24.3.2 却允许没有；还提到表 25-1 中已不存在的 Raw/File 类型 | [01_transport_and_data_block](03_gvsp/01_transport_and_data_block.md) |
| 19 | p.181–182 | chunk 的 XML 示例与字节序示例表的解码结果不一致 | [01_transport_and_data_block](03_gvsp/01_transport_and_data_block.md) |
| 20 | p.164 | §22.3 提到的非 PFNC 像素格式列表在 3.0 中已不存在 | [02_payload_types_and_pixel_layouts](03_gvsp/02_payload_types_and_pixel_layouts.md) |
| 21 | p.407 | 附录 C 表 42-1 仍用旧的 bit 1 名称；`gvrsp_mode_supported` 放错到 0x0934 行（实为 0x092C bit 5） | [02_connection_and_flow_control](04_gvrsp/02_connection_and_flow_control.md) |
| 22 | p.29 vs p.291 | §1.7 允许接收端使用 GVRSP 流，[CR-663cd] 却要求 GVRSP 设备必须是发送端 | [01_introduction_and_architecture](00_overview/01_introduction_and_architecture.md) |
| 23 | p.20 | 写 “Appendix 1”，实际是附录 A | [01_introduction_and_architecture](00_overview/01_introduction_and_architecture.md) |
| 24 | p.1、p.414–415 | 日期格式错误：“2026-24-4”、“2022-22-06”、“2026-15-4” | [03_version_history_and_v3_changes](00_overview/03_version_history_and_v3_changes.md) |

---

## 生成与校验说明

- **生成方式**：按章节由 AI 逐页精读规范后撰写中文笔记；需求索引与寄存器总表由脚本（pdfplumber）从 PDF 表格提取，并与正文交叉核对（521 条现行需求全部匹配到章节与附录 A 版本信息）；需求“中文要点”为逐条意译。
- **自动校验**（生成时全部通过）：所有相对链接可达；笔记中引用的需求编号均存在于现行或废弃列表；`§章节, p.页码` 引用经脚本与 PDF 书签的章节页范围比对，脚本报出的例外项已人工复核；全部 83 张 mermaid 图均通过 mermaid v11 语法解析。
- **局限**：笔记难免有理解偏差；涉及实现决策时请以规范原文为准，尤其是标有 💡 与 ⚠️ 的内容。相关配套标准（GenICam GenApi/SFNC/GenTL/GenDC/PFNC、IEEE 1588、InfiniBand/RoCEv2）不在本知识库范围内。
- 生成日期：2026-11-16；规范版本：3.0.00。
