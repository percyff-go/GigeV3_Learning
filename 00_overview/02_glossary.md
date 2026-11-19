# GigE Vision 3.0 术语表（定义、需求术语与缩略语）

> **覆盖范围**：§1.3 Definitions and Acronyms——§1.3.1 定义（PDF p.17–19）、§1.3.2 需求术语（p.19–20）、§1.3.3 缩略语（p.21–24）。
>
> **一句话总结**：本表收录规范 §1.3.1 的全部 **25 条定义**与 §1.3.3 的全部 **91 个缩略语**（中英对照 + 中文解释），并概述 §1.3.2 的需求术语（MUST / SHOULD、R / CR / O / CO 与后缀）。
>
> **本文目录**：使用说明 → 1.3.1 定义（25 条）→ 1.3.2 需求术语速览 → 1.3.3 缩略语（91 个，按字母序）→ 易混淆项 → 相关章节
>
> **使用说明**：“解释”列首先忠实转述规范原文定义；本知识库补充的背景或延伸以 “💡” 标出。缩略语表中，规范 §1.3.3 **只给出英文全称**，中文名与“说明”列均为本知识库补充（其中引用的章节/页码均已在 3.0 正文中核对）；“3.0 正文未出现”表示在第 1 章以外的 3.0 提取文本中未检索到该缩略语。

---

## 核心要点

- “Device” 与 “Application” 是全书最基础的两个词：Device = **可被控制**的 GigE Vision 设备（通常是相机）；Application = 主机上的 GigE Vision **控制应用**（也可以是 FPGA 微码）（p.17–18）。
- Primary application（读/写）与 Secondary application（只读）区分了对设备的访问权；Switchover request 是第三方应用试图接管“允许切换”的主应用控制权的状况（p.19）。
- Standard GVCP port = **UDP 3956**（p.19）。
- Persistent IP（非易失、上电复用）与 Static IP（易失、掉电丢失、由应用“强制”写入）是两种不同概念（p.19）。
- 规范中大写的规范性词只有 **MUST / MUST NOT / SHOULD**（配合 IF 表示条件；本知识库对 3.0 全文检索的结果）；§1.3.2 没有定义 MAY / SHALL 等词。
- 缩略语中 msb/lsb（位）与 MSB/LSB（字节）**大小写含义不同**；规范按 MSB-first 给位编号（bit 0 = 最高位）。

---

## 1.3.1 定义（Definitions, p.17–19）

规范原文按字母序排列，共 25 条：

| English term | 中文 | 解释（中文） |
|---|---|---|
| **Aggregator** | 聚合器 | 在 IEEE 802.1AX 中，执行“让多条物理链路作为一条聚合链路工作”所需操作的实体（p.17）。💡 详见第 3 章链路聚合（sLAG / dLAG）。 |
| **Application** | 应用（控制应用） | 运行在主机上的 GigE Vision 控制应用软件。通常是运行在 PC 上的软件应用，也可以是其他形式，例如运行在 FPGA 中的微码（p.17）。 |
| **Control protocol** | 控制协议 | GigE Vision 控制协议（GVCP），定义 GigE Vision 兼容设备所支持的命令（p.18）。 |
| **Conversation** | 会话（有序报文集） | 按 IEEE 802.1AX 的定义：从一个设备发往另一个设备的一组报文，这些报文构成一个有序序列，且通信双方要求在这组报文中保持顺序（p.18）。💡 链路聚合时，同一 conversation 的报文需要保序，这影响聚合器如何把流量分配到各条链路。 |
| **Device** | 设备 | GigE Vision 兼容的**可控**设备。通常是相机，也可以是其他形式，例如运行在 PC 上的软件视频服务器（p.18）。 |
| **Event Generator** | 事件发生器 | 按 GigE Vision 规范产生事件消息的实体（p.18）。 |
| **Event Receiver** | 事件接收器 | 接收并能解释 GigE Vision 事件消息的实体（p.18）。 |
| **Gratuitous ARP** | 免费 ARP（无故 ARP） | 由设备发出的一种 ARP 请求或应答：源 IP 与目的 IP 都设为发出该包的设备自己的 IP，目的 MAC 为广播地址。可用于检查是否有其他设备在使用相同 IP（p.18）。💡 FORCEIP 设置新 IP 后，设备应发 gratuitous ARP 通告，让其他主机更新 ARP 缓存（§16.2.1, p.124）。 |
| **GVSP Receiver** | GVSP 接收端 | 按 GigE Vision Streaming Protocol 接收并能解封装数据流的实体（p.18）。 |
| **GVRSP Receiver** | GVRSP 接收端 | 按 GigE Vision RDMA Streaming Protocol 接收并能解封装数据流的实体（p.18）。 |
| **GVSP Transmitter** | GVSP 发送端 | 按 GigE Vision Streaming Protocol 产生数据流的实体（p.18）。 |
| **GVRSP Transmitter** | GVRSP 发送端 | 按 GigE Vision RDMA Streaming Protocol 产生数据流的实体（p.18）。 |
| **Link** | 链路 | 在源与目的之间传送报文的物理连接（p.18）。 |
| **Load Balancing** | 负载均衡 | 把流量分配到多条链路上，以均匀分摊负载、最大化总体吞吐量的做法（p.18）。 |
| **Management Entity** | 管理实体 | 负责系统中已部署设备的配置与控制的应用（p.18）。 |
| **Multihomed** | 多宿主 | 主机拥有多个 IP 地址时称为 multihomed（p.18）。💡 多宿主主机做广播发现时要注意从哪个接口发出（§5.1.1, p.49 讨论了这一问题）。 |
| **Network Interface** | 网络接口 | 为设备提供接入通信链路手段的子系统（p.18）。 |
| **Partner** | 链路对端 | 点对点链路另一端的设备，可以是终端站（例如网卡），也可以是以太网交换机端口等（p.18）。 |
| **Persistent IP** | 持久 IP | 硬编码在设备**非易失存储**中的 IP 地址；在 Persistent IP 被启用时，设备上电后会重新使用它（p.19）。 |
| **Primary application** | 主应用 | 对设备拥有 exclusive（独占）或 control（控制）访问权（读/写）的应用，包括“启用 switchover 的 control 访问”（p.19）。 |
| **Primary application Switchover request** | 主应用切换请求 | 另一个应用试图接管某设备控制权的状况，该设备当前由一个以“启用 switchover 的 control 访问”方式获得控制权的主应用控制（p.19）。💡 机制详见 §14.4 Primary Application Switchover（p.109 起）。 |
| **Secondary application** | 从应用（次级应用） | 对设备只有 monitoring（监视、只读）访问权的应用（p.19）。 |
| **Server** | 服务器 | 提供 IP 服务器功能（例如 DHCP）以分配 IP 地址的服务器；它可以选择性地包含在应用中，但多数情况下是独立的（p.19）。 |
| **Standard GVCP port** | 标准 GVCP 端口 | 设备上用于接收 GVCP 命令的 UDP 端口。IETF 已分配端口号 **3956** 作为标准 GVCP 端口；对于多服务设备，指附属于第一个服务的端口（p.19）。💡 对应需求 [R-042c]：设备的第一个 GVCP 端口必须使用 UDP 3956（§8.1, p.60）。 |
| **Static IP** | 静态 IP | 由应用写入设备**易失存储**的 IP 地址，掉电即丢失。主要用于应用“强制”设备使用某个 IP 地址（例如设备中记忆了一个无效的 Persistent IP 时）（p.19）。💡 典型手段是 FORCEIP_CMD 的 static IP 字段（§16.2.1, p.124）。 |

> 💡 解读：注意 Device 定义中的 “controllable”——GigE Vision 的 Device 一定是**可被 GVCP 控制**的实体；而 GVSP/GVRSP Transmitter/Receiver 是**流角色**，Device 与 Application 都可能承担（见 [引言与架构 §1.7](01_introduction_and_architecture.md)）。

---

## 1.3.2 需求术语速览（Requirements Terminology, p.19–20）

| 规范用词 | 出现形式 | 含义 |
|---|---|---|
| **MUST** | Absolute Requirement `[R-…]` | 产品必须支持；为保证互操作而强制 |
| **MUST**（IF 另一特性存在） | Conditional Requirement `[CR-…]` | 当另一特性被支持时，必须支持 |
| **SHOULD** | Absolute Objective `[O-…]` | 推荐支持，但非必需 |
| **SHOULD**（IF 另一特性存在） | Conditional Objective `[CO-…]` | 当另一特性被支持时，推荐支持 |

| 后缀 | 适用对象 |
|---|---|
| `ca` | 仅应用软件（control application） |
| `cd` | 仅设备（control device） |
| `c` | 应用和设备 |
| `st` | 仅流发送端（stream transmitter） |
| `sr` | 仅流接收端（stream receiver） |
| `s` | 流发送端和接收端 |
| 无后缀 | 应用、设备、流发送端与接收端全部 |

关于规范性用词的补充说明：

- §1.3.2 只用 **MUST** 和 **SHOULD** 来定义四类条目，并用 “IF … is present” 表示条件；**没有**像 RFC 2119 那样定义 MAY、SHALL、OPTIONAL 等词。
- 💡 本知识库对 3.0 全文的检索结果：大写 **MUST NOT**（及写法 “MUST not”）用于表示禁止，例如 [R-041c] 设备和应用不得在 GVCP 的 IP 数据报中使用任何 IP 选项（Ch8, p.60）；未检索到大写的 MAY、SHALL、SHOULD NOT。正文与寄存器描述中大量使用小写 must / should / shall / may，它们没有编号，不进入附录 A 的需求表，但同样描述了预期行为，实现时不应忽视。
- 💡 只有带方括号编号的条目才会出现在附录 A 的需求参考表中，也才是合规矩阵逐条核对的对象（第 39 章, p.367）。
- 详细解读与真实例子见 [引言与架构 · 1.3.2 需求术语](01_introduction_and_architecture.md)。

---

## 1.3.3 缩略语（Acronyms, p.21–24）

规范按字母序列出 91 个缩略语。为便于查找，下表保持原顺序（注意原表中大小写不同的条目，如 msb/MSB、lsb/LSB、sLAG/dLAG）。

| 缩略语 | English（规范原文） | 中文 | 说明（💡 本知识库补充；章节/页码为 3.0 正文出处） |
|---|---|---|---|
| ARB | Arbitrary (as defined in IEEE 1588) | 任意时标 | IEEE 1588 的 ARB 时标：网络中没有可溯源到 UTC 的时钟时，时间仅在本网络内相对有效（§14.2.2, p.99–100）。 |
| BOOTP | Bootstrap Protocol | 引导协议 | DHCP 的前身（RFC951）；3.0 正文仅在 DHCP 需求处顺带提及（§4.3, p.43）。 |
| CCP | Control Channel Privilege (a bootstrap register) | 控制通道权限（bootstrap 寄存器） | 地址 0x0A00；应用写入以申请 exclusive / control 访问；3.0 新增 `connection_count` 字段（§36.46, p.312–313）。 |
| CFA | Color Filter Array | 彩色滤光阵列 | 如 Bayer 阵列；与像素格式/布局相关（Ch25 p.192–193，Ch26 p.204）。 |
| CID | Chunk Identifier | Chunk 标识符 | Chunk 数据中每个 chunk 的 ID（§24.4, p.179–181）。 |
| CM | Communication Management | 通信管理 | InfiniBand 连接管理；GVRSP 用 CM REQ / REP / REJ 建立 RoCEv2 连接（Ch31, p.223 起）。 |
| DHCP | Dynamic Host Configuration Protocol | 动态主机配置协议 | 设备必须支持（[R-021cd]，§4.3, p.43）。 |
| dLAG | Dynamic Link Aggregation Group | 动态链路聚合组 | 与 sLAG 的区别在于使用 IEEE 802.1AX 的 LACP（§3.3.5, p.38）。 |
| DMA | Direct Memory Access | 直接内存访问 | RDMA/RoCEv2 背景（Ch23 p.167；Ch28 p.208–209）。 |
| DNS | Domain Name System | 域名系统 | mDNS / DNS-SD 的基础（§5.2）。 |
| DNS-SD | Domain Name System - Service Discovery | 基于 DNS 的服务发现 | 可选的 Zeroconf 设备发现（§5.2, p.51 起）。 |
| EOC | End Of Codestream | 码流结束 | 💡 JPEG 2000 码流结束标记。3.0 正文未出现（JPEG 2000 payload 细节仅在 2.2 版定义，见 Ch25 注释 p.186–187）。 |
| EOI | End Of Image | 图像结束 | 💡 JPEG 结束标记。3.0 正文未出现。 |
| FMO | Flexible Macroblock Ordering | 灵活宏块排序 | 💡 H.264 编码工具。3.0 正文未出现。 |
| GenDC | Generic Data Container | 通用数据容器 | GenICam 的传输无关数据容器；GVSP GenDC payload 与 GVRSP 唯一 payload（第 38 章；版本 1.2，§1.4）。 |
| GenICam | Generic Interface for Cameras | 相机通用接口 | EMVA 标准（版本 2021.02）；设备 XML 描述文件、SFNC、PFNC 等（第 37 章）。 |
| GenTL | Generic Transport Layer | 通用传输层 | GenICam 传输层接口（版本 1.6）；GVRSP 兼容旧接收端的 GenTL Producer 方案（§35.2, p.243）。 |
| GEV | GigE Vision | GigE Vision | 常用前缀，如 `GEV_STATUS_*`、`GEV_EVENT_*`。 |
| GIF | Graphic Interchange Format | 图形交换格式 | 3.0 正文未出现。 |
| GigE | Gigabit Ethernet | 千兆以太网 | 标准名称来源；但规范适用于任意以太网速率（§1.1, p.15）。 |
| GMT | Greenwich Mean Time | 格林尼治标准时间 | 用于表述 63 位 PTP 时间戳的溢出时刻（§14.2.2, p.100）。 |
| GPS | Global Positioning System | 全球定位系统 | 作为可溯源到 UTC 的时间源举例（§14.2.2, p.99）。 |
| GVCP | GigE Vision Control Protocol | GigE Vision 控制协议 | 基于 UDP 的控制与消息协议（PART 2）。 |
| GVSP | GigE Vision Streaming Protocol | GigE Vision 流协议 | 基于 UDP 的流协议（PART 3）。 |
| GVRSP | GigE Vision RDMA Streaming Protocol | GigE Vision RDMA 流协议 | 3.0 新增，基于 RoCEv2，只传 GenDC（PART 4）。 |
| IANA | Internet Assigned Numbers Authority | 互联网号码分配局 | 已登记 GVCP 标准端口 3956（§8.1, p.60）。 |
| IB | InfiniBand | InfiniBand | RoCEv2 基于 IB 规范：用 UDP 承载格式化的 IB 包（§27.1, p.206）。 |
| IBA | InfiniBand Architecture | InfiniBand 体系结构 | 见 Ch28（p.209–210）；参考文档 “InfiniBand Architecture Specification”。 |
| ID | Identifier | 标识符 | 如 `block_id64`、`packet_id32`、`req_id`。 |
| IDR | Instantaneous Decoding Refresh | 即时解码刷新 | 💡 H.264 中可独立解码的关键帧。3.0 正文未出现。 |
| IEC | International Electrotechnical Commission | 国际电工委员会 | ⚠️ 勿与 GVCP Capability 寄存器 bit 15 的缩写 “(IEC)” = `IEEE1588_extended_capabilities` 混淆（§36.29, p.294）。 |
| IEEE | Institute of Electrical and Electronics Engineers | 电气电子工程师学会 | IEEE 802.3、802.1AX、1588 等标准的发布者。 |
| IETF | Internet Engineering Task Force | 互联网工程任务组 | 发布 RFC；§1.1 要求实现遵循相关 RFC（p.15）。第 1 章以外未出现。 |
| IP | Internet Protocol | 网际协议 | GVCP/GVSP 基于 UDP/IPv4。 |
| IPv4 | Internet Protocol version 4 | 网际协议第 4 版 | 当前版本的传输基础（§7.1, p.58；§22.1, p.164）。 |
| IPv6 | Internet Protocol version 6 | 网际协议第 6 版 | 3.0 中仅在 mDNS 语境中出现（AAAA 记录、mDNS 的 IPv6 多播地址 FF02::FB，§5.2.1, p.52）。 |
| ISO | International Organization for Standardization | 国际标准化组织 | 第 1 章以外未出现（§1.4 中 UTF-8 即 ISO 10646 的转换格式）。 |
| ITU | International Telecommunication Union | 国际电信联盟 | ITU-T T.81（JPEG）、T.800（JPEG 2000）、H.264。 |
| JFIF | JPEG File Interchange Format | JPEG 文件交换格式 | 3.0 正文未出现。 |
| JPEG | Joint Photographic Experts Group | 联合图像专家组 | JPEG payload 类型 0x0006（表 25-1, p.186）。 |
| LAG | Link Aggregation Group | 链路聚合组 | 第 3 章（p.34–39）。 |
| LFSR | Linear Feedback Shift Register | 线性反馈移位寄存器 | 用于生成 GVSP test packet 的测试数据（§24.5, p.183–184；GVCP Capability 寄存器 `test_data` 位，§36.29, p.293）。 |
| LLA | Link-Local Address | 链路本地地址 | 设备必须支持（[R-028cd]）；地址范围 169.254.xxx.xxx，可用 169.254.1.0–169.254.254.255（§4.4, p.47）。 |
| lsb | Least Significant Bit | 最低有效位 | 规范 MSB-first 编号下，32 位字段的 lsb 是 bit 31（§15.3, p.115）。 |
| LSB | Least Significant Byte | 最低有效字节 | 注意与 lsb（位）区分。 |
| MAC | Media Access Control | 媒体访问控制 | 以太网 MAC 层 / MAC 地址。 |
| mDNS | Multicast Domain Name System | 多播 DNS | 可选的 Zeroconf 发现（§5.2.1, p.51–52）。 |
| ML | Multiple Links | 多链路 | 第 3 章链路配置之一（p.34 起）。 |
| MPEG | Moving Picture Experts Group | 动态图像专家组 | 3.0 正文未出现。 |
| msb | Most Significant Bit | 最高有效位 | 规范中 bit 0 即 msb（§15.3, p.115）。 |
| MSB | Most Significant Byte | 最高有效字节 | 注意与 msb（位）区分；big-endian 下最先发送。 |
| MTU | Maximum Transmission Unit | 最大传输单元 | 避免 IP 分片的 IP MTU 为 576 字节，GVCP 默认最大包长即 576（§8.1.1, p.61）。 |
| NAL | Network Abstraction Layer | 网络抽象层 | 💡 H.264 码流组织单元。3.0 正文未出现。 |
| NAT | Network Address Translation | 网络地址转换 | 3.0 正文未出现。 |
| NDSPI | NetworkDirect Service Provider Interface | NetworkDirect 服务提供者接口 | Windows 上的 RDMA 接口；见 RoCEv2 操作系统/硬件支持表（§35.1, p.242）。 |
| NIC | Network Interface Card | 网卡 | GVRSP 依赖支持 RoCEv2 的网卡（§11.1, p.82）。 |
| OSI | Open Systems Interconnection | 开放系统互连（参考模型） | §23.1.4 Flow Control 引用 OSI 模型说明 PAUSE 机制处于数据链路层（p.167）。 |
| OUI | Organizationally Unique Identifier | 组织唯一标识符 | 💡 MAC 地址中标识厂商的部分。3.0 正文未出现。 |
| PC | Personal Computer | 个人计算机 | |
| PFNC | GenICam Pixel Format Naming Convention | GenICam 像素格式命名约定 | 版本 2.4（§1.4）；像素格式见 §26.3（p.203）。 |
| PSN | Packet Sequence Number | 包序列号 | RoCEv2 包序号；CM REQ/REP 中的 Starting PSN（§31.2, p.225）。 |
| PTP | Precision Time Protocol | 精确时间协议 | 即 IEEE 1588（§14.2；§37.7）。 |
| QP | Queue Pair | 队列对 | RDMA 的发送/接收队列对；GVRSP 每个 GenDC flow 连接一个独立 QP（[CR-639sr]，Ch31, p.228）。 |
| QPN | Queue Pair Number | 队列对编号 | 3.0 正文未以缩写形式出现。 |
| RC | Reliable Connection | 可靠连接 | GVRSP 使用的 RoCEv2 传输服务类型（[CR-634sr]，§31.1, p.223）。 |
| RDMA | Remote Direct Memory Access | 远程直接内存访问 | GVRSP 名称中的 “R”。 |
| RFC | Request For Comments | 请求评论（IETF 文档） | §1.4 列出的各项 RFC。 |
| RoCE | RDMA over Converged Ethernet | 融合以太网上的 RDMA | |
| RoCEv2 | RoCE version 2 | RoCE 第 2 版 | GVRSP 的传输层：可靠、按序、可由网卡硬件加速（§27.1, p.206）。 |
| ROI | Region of Interest | 感兴趣区域 | Width / Height 即 ROI 尺寸（§37.5.1, p.359）。 |
| RRoCE | Routable RoCE, informal acronym for RoCEv2 used for example in Wireshark | 可路由 RoCE | RoCEv2 的非正式名称，例如 Wireshark 中使用。3.0 正文未出现。 |
| RTP | Real-time Transport Protocol | 实时传输协议 | 3.0 正文未出现（§1.4 中 RFC2435/3984/5371 均为 RTP 负载格式）。 |
| SC | Stream Channel | 流通道 | 寄存器缩写前缀（SCPx、SCPSx、SCCx …）；Ch31 p.231–233 等。 |
| SDP | Session Description Protocol | 会话描述协议 | 3.0 正文未出现。 |
| SL | Single Link | 单链路 | 第 3 章链路配置之一。 |
| sLAG | Static Link Aggregation Group | 静态链路聚合组 | 第 3 章（§3.3.5, p.38）。 |
| SOC | Start Of Codestream | 码流开始 | 💡 JPEG 2000 标记。3.0 正文未出现。 |
| SOD | Start Of Data | 数据开始 | 💡 JPEG 2000 标记。3.0 正文未出现。 |
| SOI | Start Of Image | 图像开始 | 💡 JPEG 标记。3.0 正文未出现。 |
| SOT | Start Of Tile | 图块开始 | 💡 JPEG 2000 标记。3.0 正文未出现。 |
| SPI | Stateful Packet Inspection | 状态包检测 | 💡 防火墙技术，可能影响 GVCP/GVSP 报文通行。3.0 正文未以缩写出现。 |
| TAI | International Atomic Time | 国际原子时 | IEEE 1588 时间戳以 1970-01-01 00:00:00 TAI 为纪元（§14.2.2, p.99）。 |
| TCP | Transmission Control Protocol | 传输控制协议 | GVRSP 连接的 ServiceID 取自 IANA 为 GigE Vision 登记的 TCP 端口 3956（§31.1, p.223）。 |
| UCS | Universal Character Set | 通用字符集 | 💡 即 ISO 10646。3.0 正文未以缩写出现。 |
| UDP | User Datagram Protocol | 用户数据报协议 | GVCP、GVSP 均基于 UDP；RoCEv2 也用 UDP 承载 IB 包。 |
| UTC | Coordinated Universal Time | 协调世界时 | §14.2.2（p.99）。 |
| UTF | UCS Transformation Format | UCS 转换格式 | 如 UTF-8：Device Mode 寄存器 `character_set_index`（§36.2, p.271）。 |
| UTP | Unshielded Twisted Pair | 非屏蔽双绞线 | 3.0 正文未出现。 |
| VCL | Video Coding Layer | 视频编码层 | 💡 H.264 概念。3.0 正文未出现。 |
| WC | Work Completion | 工作完成项 | 💡 RDMA 完成队列中的完成通知；见流控示意图（§33.1, p.239）。 |
| WQE | Work Queue Entry (Work Queue Element) | 工作队列项 | 💡 投递到 QP 的发送/接收请求；Ch28（p.209–210）、Ch29、§33.1。 |

---

## 易混淆项

| 易混淆 | 区分 |
|---|---|
| msb / lsb vs MSB / LSB | 小写是**位**，大写是**字节**。规范位编号 MSB-first：bit 0 = msb。 |
| Persistent IP vs Static IP | Persistent：非易失存储，上电复用；Static：易失存储，掉电丢失，由应用“强制”写入（p.19）。 |
| Primary vs Secondary application | Primary：exclusive / control 访问（读写）；Secondary：monitoring 访问（只读）（p.19）。 |
| Device vs Transmitter | Device 指“可被控制”的实体；Transmitter / Receiver 是流角色或设备类别（§1.7）。 |
| sLAG vs dLAG | 区别在于 dLAG 使用 IEEE 802.1AX LACP（§3.3.5, p.38）。 |
| IEC（组织）vs (IEC) 寄存器位 | 缩略语表中是国际电工委员会；GVCP Capability 寄存器 bit 15 的 (IEC) 指 IEEE 1588 extended capabilities。 |
| SC vs SCx 寄存器 | SC = Stream Channel；寄存器名中的 x 表示通道索引（如 SCPS0）。 |

---

## 相关章节

- [引言与系统架构（含需求术语详解）](01_introduction_and_architecture.md)
- [版本历史与 3.0 变化](03_version_history_and_v3_changes.md)
- [设备发现（DHCP / LLA / Persistent IP / mDNS）](../01_device_discovery/device_discovery.md)
- [GVCP：传输与通道](../02_gvcp/01_transport_and_channels.md)
- [GVRSP：RoCEv2 与流模型](../04_gvrsp/01_rocev2_and_streaming_model.md)
- [Bootstrap 寄存器总览](../05_bootstrap_registers/01_overview_and_register_map.md)
- [全部需求索引](../reference/requirements_index.md)
