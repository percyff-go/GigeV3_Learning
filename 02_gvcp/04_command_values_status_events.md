# GVCP 命令/确认值、状态码、事件与 ICMP（Ch18–21 + 附录 B）

> **覆盖范围**：Chapter 18 Command and Acknowledge Values（PDF p.154–155）；Chapter 19 Status Codes（PDF p.156–159）；Chapter 20 Events（PDF p.160）；Chapter 21 ICMP（PDF p.161–162）；Appendix B Status Codes Explained（§B.1–B.21，PDF p.402–406）。
>
> **一句话总结**：本文是 GVCP 的“数值字典”——列出全部命令/确认报文值（ACK = CMD + 1，PACKETRESEND 与 PENDING_ACK 例外）、全部标准状态码（16 位：bit 0 严重级别、bit 1 设备自定义标志、低 12 位取值）及附录 B 的逐条解释、标准事件标识符的取值区间，以及设备必须支持的 ICMP 子集。
>
> **约定提醒**：
> - 规范的位编号是 **MSB-first**：16 位 status 字段的 bit 0 是最高位（掩码 0x8000），bit 15 是最低位（0x0001）。
> - 所有数值均为大端序传输；本文中的十六进制值即线路上 16 位字段的数值。
> - 页码均为 PDF 页码（打印页码 = PDF 页码 − 1）。
>
> **本文目录**
> - 核心要点
> - 18 命令与确认值（全部报文值总表、命令特性表）
> - 19 状态码（字段结构、总表、出现场景、附录 B 逐条详解）
> - 20 事件（标准事件标识符）
> - 21 ICMP
> - 实现要点与易错点
> - 相关章节
> - 自测题

---

## 核心要点

- **报文值空间**：0–32767（0x0000–0x7FFF，最高位为 0）保留给 GVCP 标准报文；32768–65535（0x8000–0xFFFF，最高位为 1）供设备自定义报文使用（p.154）。
- **命令值最低位为 0，确认值最低位为 1，ACK = CMD + 1**（[R-238c]）。两个例外：PACKETRESEND_CMD（0x0040）没有 ACK，0x0041 被标为“无效”；PENDING_ACK（0x0089）没有对应的命令。
- 标准报文一共 20 个有效值（另有 1 个被标为无效的 0x0041）：DISCOVERY（0x0002/0x0003）、FORCEIP（0x0004/0x0005）、PACKETRESEND_CMD（0x0040）、READREG（0x0080/0x0081）、WRITEREG（0x0082/0x0083）、READMEM（0x0084/0x0085）、WRITEMEM（0x0086/0x0087）、PENDING_ACK（0x0089）、EVENT（0x00C0/0x00C1）、EVENTDATA（0x00C2/0x00C3）、ACTION（0x0100/0x0101）。其中只有 EVENT/EVENTDATA 走消息通道。
- **状态码是 16 位**：bit 0 = severity（0 信息 / 1 错误，掩码 0x8000），bit 1 = device_specific_flag（掩码 0x4000），bit 2–3 保留，bit 4–15 = status_value（p.156）。标准错误码都落在 0x8001–0x8FFF；信息类只有 SUCCESS（0x0000）和 PACKET_RESEND（0x0100）。
- 设备应返回**信息量最大**的状态码（[O-239cd]）；实在给不出更具体的码时，**至少**返回 `GEV_STATUS_ERROR`（0x8FFF）（[R-240cd]）；GVSP 发送端同理（[O-241st]/[R-242st]）。
- **已废弃的值**：0x8008–0x800B 与 0x800F（2.0 起不再使用）。v3.0 中“extended status code”的能力/使能位已废弃并恒为 1，即扩展状态码总是可用（§36.29/§36.35）。
- **状态码不仅出现在 GVCP ACK 中，也出现在 GVSP 包头中**：PACKET_RESEND、PACKET_UNAVAILABLE、DATA_OVERRUN、PACKET_NOT_YET_AVAILABLE、PACKET_(AND_PREV_)REMOVED_FROM_MEMORY、PACKET_TEMPORARILY_UNAVAILABLE、LEADER_TRAILER_OVERFLOW 主要用于流通道。
- **事件标识符区间**：0x0000–0x0006 废弃；0x0007 PRIMARY_APP_SWITCH、0x0008 LINK_SPEED_CHANGE、0x0009 ACTION_LATE；0x000A–0x8000 保留；0x8001–0x8FFF 为 `GEV_EVENT_ERROR_xxx`（值即对应的状态码）；0x9000–0xFFFF 为设备自定义事件（p.160）。带数据的事件必须用 EVENTDATA 发送（[CR-243cd]）。
- **ICMP**：设备在所有支持的网络接口上必须至少支持 Echo（接收）、Echo Reply（发送）——均针对 ≤ 576 字节的包——以及 Destination Unreachable（接收）；其余 ICMP 报文均可选（[R-244cd]，p.161）。

---

<a id="ch18"></a>
## 18 命令与确认值（Ch18，p.154–155）

规范规则（p.154）：

- 规范保留 0–32767（最高位清零）给 GVCP；32768–65535（最高位置位）可用于设备自定义报文。
- 确认报文的值总是对应命令值加 1，因此**命令报文的最低位为 0，确认报文的最低位为 1**。
- 设备和应用必须使用下表中的值；确认值总是对应命令值 + 1（当存在对应命令时），PACKETRESEND_CMD 和 PENDING_ACK 例外（[R-238c]）。

> ⚠️ 与 §15.1 的差异：§15.1 写 “Commands from 0 to 32765 are reserved for GigE Vision”（p.113），与本章 “0 to 32767” 不一致；按本章“以最高位区分”的规则，标准区间应为 0x0000–0x7FFF（💡 判断 32765 为笔误）。

### 18.1 全部报文值总表（按数值排序）

| 值 | 报文 | Ch18 分组 | 通道 | 方向 | 支持要求 / 能力位（GVCP Capability 0x0934） | 格式详见 |
|---|---|---|---|---|---|---|
| 0x0002 | DISCOVERY_CMD | Discovery Protocol Control | Control | App → Dev | 必须（[R-139cd]） | [16.1](03_message_formats.md#discovery) |
| 0x0003 | DISCOVERY_ACK | Discovery Protocol Control | Control | Dev → App | 必须 | [16.1](03_message_formats.md#discovery) |
| 0x0004 | FORCEIP_CMD | Discovery Protocol Control | Control | App → Dev（广播） | 必须（[R-147cd]） | [16.2](03_message_formats.md#forceip) |
| 0x0005 | FORCEIP_ACK | Discovery Protocol Control | Control | Dev → App | 必须 | [16.2](03_message_formats.md#forceip) |
| 0x0040 | PACKETRESEND_CMD | Streaming Protocol Control | Control | App → Dev | GVSP 发送端 SHOULD（[O-196cd]）；bit 29 | [16.7](03_message_formats.md#packetresend) |
| 0x0041 | *Invalid. Resent packet must be on the stream channel* | Streaming Protocol Control | Control | — | 设备绝不能发送（[CR-201cd]） | [16.7.2](03_message_formats.md#packetresend-response) |
| 0x0080 | READREG_CMD | Device Memory Access | Control | App → Dev | 必须（[R-157cd]）；拼接需 bit 31 | [16.3](03_message_formats.md#readreg) |
| 0x0081 | READREG_ACK | Device Memory Access | Control | Dev → App | 必须 | [16.3](03_message_formats.md#readreg) |
| 0x0082 | WRITEREG_CMD | Device Memory Access | Control | App → Dev | 必须（[R-166cd]）；拼接需 bit 31 | [16.4](03_message_formats.md#writereg) |
| 0x0083 | WRITEREG_ACK | Device Memory Access | Control | Dev → App | 必须 | [16.4](03_message_formats.md#writereg) |
| 0x0084 | READMEM_CMD | Device Memory Access | Control | App → Dev | 必须（[R-176cd]） | [16.5](03_message_formats.md#readmem) |
| 0x0085 | READMEM_ACK | Device Memory Access | Control | Dev → App | 必须 | [16.5](03_message_formats.md#readmem) |
| 0x0086 | WRITEMEM_CMD | Device Memory Access | Control | App → Dev | 必须（[R-185cd]）；bit 30 已废弃恒为 1 | [16.6](03_message_formats.md#writemem) |
| 0x0087 | WRITEMEM_ACK | Device Memory Access | Control | Dev → App | 必须 | [16.6](03_message_formats.md#writemem) |
| 0x0089 | PENDING_ACK | （表中列于 WRITEMEM_ACK 之后） | Control | Dev → App | v3.0 起必须（p.142）；bit 26 已废弃恒为 1 | [16.8](03_message_formats.md#pending) |
| 0x00C0 | EVENT_CMD | Asynchronous Events | **Message** | Dev → App | SHOULD（[O-228cd]）；bit 28 | [17.1](03_message_formats.md#event) |
| 0x00C1 | EVENT_ACK | Asynchronous Events | **Message** | App → Dev | 随 EVENT | [17.1](03_message_formats.md#event) |
| 0x00C2 | EVENTDATA_CMD | Asynchronous Events | **Message** | Dev → App | 可选；bit 27 | [17.2](03_message_formats.md#eventdata) |
| 0x00C3 | EVENTDATA_ACK | Asynchronous Events | **Message** | App → Dev | 随 EVENTDATA | [17.2](03_message_formats.md#eventdata) |
| 0x0100 | ACTION_CMD | Miscellaneous | Control | App → Dev（单播/广播） | 可选；bit 25（Scheduled Action：bit 14） | [16.9](03_message_formats.md#action) |
| 0x0101 | ACTION_ACK | Miscellaneous | Control | Dev → App（单播） | 随 ACTION | [16.9](03_message_formats.md#action) |

> 💡 解读：
> - 0x0088 未分配——PENDING_ACK 是“只有确认、没有命令”的报文，它的 `ack_id` 指向正在执行的那条命令的 `req_id`。
> - 值的高字节大致对应功能分组：0x0002–0x0005 发现/IP，0x0040 流控制，0x0080–0x0089 存储访问，0x00C0–0x00C3 异步事件，0x0100–0x0101 杂项（ACTION）。
> - 抓包快速判断：`value & 0x0001` 为 1 → 确认报文；`value & 0x8000` 为 1 → 设备自定义报文。命令报文首字节固定为 0x42，可作为区分命令与确认的辅助判据——但设备自定义的信息类状态码 0x42xx 也会让 ACK 首字节恰好为 0x42，因此仍应结合方向/端口与 req_id/ack_id 判断。

```c
/* 💡 示例：报文值分类（C 伪代码） */
static inline bool gvcp_is_ack(uint16_t v)        { return (v & 0x0001u) != 0; }
static inline bool gvcp_is_dev_specific(uint16_t v){ return (v & 0x8000u) != 0; }
static inline uint16_t gvcp_ack_of(uint16_t cmd)  { return (uint16_t)(cmd + 1u); } /* 不适用于 0x0040 */
```

### 18.2 命令特性汇总（Table 10-1，资料性，p.76–77）

Ch10 的 Table 10-1 给出了各命令的特性摘要（规范注明“仅供参考，以正文需求为准”）：

| 命令 | 强制 | 发送方 | CMD 单播/广播 | ACK 单播/广播 | 复位心跳 | 可产生 PENDING_ACK |
|---|---|---|---|---|---|---|
| DISCOVERY | 是 | 主、次应用 | 单播或广播 | 单播或广播 | 否 | 否 |
| FORCEIP | 是 | 次应用 | 广播 | 单播或广播 | 否 | 否 |
| READREG | 是 | 主、次应用 | 单播为主，可广播 | 单播 | 是 | 是 |
| WRITEREG | 是 | 主应用 | 单播为主，不建议广播 | 单播 | 是 | 是 |
| READMEM | 是 | 主、次应用 | 单播为主，可广播 | 单播 | 是 | 是 |
| WRITEMEM | 是 | 主、次应用（见下方 ⚠️） | 单播为主，不建议广播 | 单播 | 是 | 是 |
| PACKETRESEND | 否 | 主、次应用 | 单播 | 不适用：结果在 GVSP 流通道上 | 否 | 否 |
| PENDING | 是 | 主、次应用 | 不适用 | 单播 | 不适用 | 不适用 |
| ACTION | 否 | 主、次应用 | 单播或广播 | 单播 | 否 | 是 |

> ⚠️ Table 10-1 中 WRITEMEM 的发送方写成 “Primary and secondary”（p.77），但 §16.6 明确规定只有主应用可以发 WRITEMEM（switchover 请求除外），设备对次应用必须返回 ACCESS_DENIED（[R-191cd]，p.135）。以正文需求为准。另：“复位心跳”只对主应用发来的命令有效（[R-075cd]，p.75）。

<a id="ch19"></a>
## 19 状态码（Ch19，p.156–159；附录 B，p.402–406）

状态码出现在**确认报文**的 status 字段或 **GVSP 包头**的 status 字段中。规范定义两类状态码（p.156）：

1. **标准状态码**：由本规范定义（附加解释见附录 B——原文写作 “Refer to appendix 2”，实为 Appendix B）；
2. **设备自定义状态码**。

规范性要求：

- 设备应返回能尽可能多地说明错误来源的状态码（[O-239cd]）；若无法返回更具体的状态码，设备**至少必须**返回 `GEV_STATUS_ERROR`（[R-240cd]）。
- GVSP 发送端应返回能尽可能说明流传输状态的状态码（[O-241st]）。例如 GVSP 发送端内存/数据溢出时，应在数据尾包（data trailer）中把 status 置为 `GEV_STATUS_DATA_OVERRUN`；若流错误发生时给不出更具体的码，**至少必须**返回 `GEV_STATUS_ERROR`（[R-242st]）。
- 设备或 GVSP 发送端不要求支持所有状态码；有些状态码适合由应用使用（p.156）。

> 💡 解读：“适合由应用使用”的典型场景是消息通道——应用回 EVENT_ACK / EVENTDATA_ACK 时同样要填 status 字段（例如对不支持的消息回 `GEV_STATUS_NOT_IMPLEMENTED`）。

<a id="status-layout"></a>
### 19.1 status 字段结构（p.156）

```
  bit:  0          1                  2   3   4                                   15
      +----------+------------------+-------+--------------------------------------+
      | severity | device_specific_ |reserved|            status_value             |
      |          |      flag        |  (2)  |              (12 bits)               |
      +----------+------------------+-------+--------------------------------------+
 掩码:   0x8000        0x4000         0x3000                 0x0FFF
```

| 位 | 名称 | 含义 |
|---|---|---|
| 0 | severity | 0 = info（信息），1 = error（错误） |
| 1 | device_specific_flag | 0 = 标准 GigE Vision 状态码，1 = 设备自定义状态码 |
| 2–3 | reserved | 发送置 0，接收忽略 |
| 4–15 | status_value | 状态码的实际取值 |

> 💡 解读：由此可推出四个区间——标准信息码 0x0000–0x0FFF（规范只定义了 0x0000 与 0x0100）、标准错误码 0x8000–0x8FFF、设备自定义信息码 0x4000–0x4FFF、设备自定义错误码 0xC000–0xCFFF。应用判断成败应先看 severity 位，而不是简单地 “status == 0”：例如重传包的 `GEV_STATUS_PACKET_RESEND`（0x0100）就是“成功”的一种。

```c
/* 💡 示例：status 字段解码 */
bool     sev_error  = (status & 0x8000u) != 0;   /* bit 0  */
bool     dev_spec   = (status & 0x4000u) != 0;   /* bit 1  */
uint16_t value      =  status & 0x0FFFu;          /* bit 4-15 */
```

<a id="status-table"></a>
### 19.2 标准状态码总表（Table 19-1，p.157–159，已用 pdfplumber 对照原表核实）

| 名称 | 值 | 级别 | 主要出现在 | 含义（Table 19-1） | 附录 B |
|---|---|---|---|---|---|
| GEV_STATUS_SUCCESS | 0x0000 | 信息 | GVCP ACK、GVSP | 命令执行成功 | B.1 |
| GEV_STATUS_PACKET_RESEND | 0x0100 | 信息 | GVSP（重传包） | 仅用于被重传的包；发送端发送重传包时优先使用它而非 SUCCESS，便于接收端监控重传；使用 block_id64 时重传包的 GVSP flag bit 15 必须置位 | B.2 |
| GEV_STATUS_NOT_IMPLEMENTED | 0x8001 | 错误 | GVCP ACK | 设备不支持该命令 | B.3 |
| GEV_STATUS_INVALID_PARAMETER | 0x8002 | 错误 | GVCP ACK | 命令中至少有一个参数对设备无效（或越界） | B.4 |
| GEV_STATUS_INVALID_ADDRESS | 0x8003 | 错误 | GVCP ACK | 试图访问不存在的地址空间 | B.5 |
| GEV_STATUS_WRITE_PROTECT | 0x8004 | 错误 | GVCP ACK | 所寻址的寄存器不可写 | B.6 |
| GEV_STATUS_BAD_ALIGNMENT | 0x8005 | 错误 | GVCP ACK | 地址偏移或数据大小未正确对齐 | B.7 |
| GEV_STATUS_ACCESS_DENIED | 0x8006 | 错误 | GVCP ACK | 试图访问当前/暂时不可访问的地址；取决于设备当前状态，特别是应用当前的权限 | B.8 |
| GEV_STATUS_BUSY | 0x8007 | 错误 | GVCP ACK | 服务该请求所需的资源当前不可用，可稍后重试 | B.9 |
| Deprecated | 0x8008–0x800B | — | — | GigE Vision 2.0 起不再使用（表中印作 “0x8008 to 0x0800B”，0x0800B 为笔误） | — |
| GEV_STATUS_PACKET_UNAVAILABLE | 0x800C | 错误 | GVSP（重传错误包） | 请求的包已不可得 | B.10 |
| GEV_STATUS_DATA_OVERRUN | 0x800D | 错误 | GVSP（数据尾包） | GVSP 发送端内部存储溢出（通常发生在图像采集中） | B.11 |
| GEV_STATUS_INVALID_HEADER | 0x800E | 错误 | GVCP ACK | 报文头无效，某些字段不符合规范 | B.12 |
| Deprecated | 0x800F | — | — | GigE Vision 2.0 起不再使用 | — |
| GEV_STATUS_PACKET_NOT_YET_AVAILABLE | 0x8010 | 错误 | GVSP（重传错误包） | 请求的包尚未采集；可用于行触发速率慢于应用超时的线扫相机 | B.13 |
| GEV_STATUS_PACKET_AND_PREV_REMOVED_FROM_MEMORY | 0x8011 | 错误 | GVSP（重传错误包） | 请求的包及之前所有包都已不可得、已从发送端内存丢弃；接收端不应再请求重传这些包 | B.14 |
| GEV_STATUS_PACKET_REMOVED_FROM_MEMORY | 0x8012 | 错误 | GVSP（重传错误包） | 请求的包已从发送端内存丢弃；但接收端仍可继续对更早的未到包运行重传算法——这并不表示更早的数据一定可得，只是不应假设更早的都不可得 | B.15 |
| GEV_STATUS_NO_REF_TIME | 0x8013 | 错误 | ACTION_ACK | 设备未同步到可作时间参考的主时钟；典型用于 IEEE 1588 未锁定、Scheduled Action 无法安排在未来时刻执行 | B.18 |
| GEV_STATUS_PACKET_TEMPORARILY_UNAVAILABLE | 0x8014 | 错误 | GVSP（重传错误包） | 因临时带宽问题暂时无法重传，应稍后再请求 | B.16 |
| GEV_STATUS_OVERFLOW | 0x8015 | 错误 | ACTION_ACK | 设备队列或包数据溢出；典型用于 Scheduled Action 队列已满、无法接收新请求 | B.17 |
| GEV_STATUS_ACTION_LATE | 0x8016 | 错误 | ACTION_ACK | 请求的 Scheduled Action 时间已过去；仅自 GEV 2.0 起、且指定了 action_time 时可用 | B.19 |
| GEV_STATUS_LEADER_TRAILER_OVERFLOW | 0x8017 | 错误 | GVSP（数据头/尾包）；打开流通道时 | 数据头包或数据尾包的包大小不足以容纳传输该块所需的全部信息 | B.20 |
| GEV_STATUS_ERROR | 0x8FFF | 错误 | 任意 | 通用错误；应尽量避免，改用上表中更具体的状态码 | B.21 |

<a id="status-usage"></a>
### 19.3 状态码 → 典型出现的报文 / 场景（汇总自全文条款）

> 💡 本表把散落在各章的条款按状态码归类，方便实现时“对号入座”；标注了依据的为规范明文，未标依据的为一般适用的推断。

| 状态码 | 典型出现的报文 / 场景 | 依据 |
|---|---|---|
| SUCCESS（0x0000） | 所有 ACK 的成功情形；GVSP 正常数据包 | B.1 |
| PACKET_RESEND（0x0100） | GVSP 重传包（同时置 flag bit 15） | Table 19-1，[CR-204st] |
| NOT_IMPLEMENTED（0x8001） | 任何不被支持的命令（若请求了 ACK）：ACK 值 = 命令值 + 1，无载荷 | [R-132c]，p.112 |
| INVALID_PARAMETER（0x8002） | WRITEREG/WRITEMEM 写入不支持的值（寄存器保持原值）；流通道关闭时请求测试包（也可忽略）；streaming_protocol 写入不支持的协议 | [O-427cd] p.248；§11 p.84–85；Table 36-2 p.305–306 |
| INVALID_ADDRESS（0x8003） | READREG/WRITEREG/READMEM/WRITEMEM 访问不支持的 bootstrap 寄存器；访问不支持的网络接口的寄存器；不支持的可选字符串寄存器（READMEM 时 length = 0，WRITEMEM 时 index = 0）；非 GVRSP 模式下访问 GVRSP 寄存器（也可用 ACCESS_DENIED） | [R-421cd]、[CR-425cd]、[CR-426cd] p.247；[R-431cd] 等 p.271；§11.12 p.87 |
| WRITE_PROTECT（0x8004） | WRITEREG/WRITEMEM 写只读寄存器（如 Version）；PTP 驯服时钟下写 Timestamp Control 复位时间戳 | [R-423cd] p.247；[CR-124cd] p.100 |
| BAD_ALIGNMENT（0x8005） | READMEM/WRITEMEM 字节数不是 4 的倍数或地址未 32 位对齐；READREG/WRITEREG 地址未对齐 | [R-178cd]、[R-179c]、[R-188cd]、[R-189c]；B.7 |
| ACCESS_DENIED（0x8006） | 权限不足的 READREG/WRITEREG/READMEM/WRITEMEM（ACK 只含报文头，length = 0）；读只写寄存器（如 Action Device Key）；无权写 CCP；GVRSP 模式下访问 SCPx、GVSP 模式下访问 RSCCTLx | [R-138cd]、[R-162cd]、[R-171cd]、[R-181cd]、[R-191cd]；[R-423cd]；§10.4 p.74；[R-671cd] p.322；[CR-673cd] p.341 |
| BUSY（0x8007） | 设备暂时无法执行（可作为 PENDING_ACK 的替代）；正常流传输期间写 SCPx / SCDAx 等；任一流通道打开时写 Streaming Configuration；流传输期间请求测试包；PENDING_ACK 期间收到新命令时的一种处理方式 | [O-428cd] p.248；[R-586cd] p.322；[R-667cd] p.305；§11 p.85；§10.7 p.80 |
| PACKET_UNAVAILABLE（0x800C） | 只含头部的 GVSP 重传错误包 | [CR-205st]，Table 16-2 |
| DATA_OVERRUN（0x800D） | GVSP 数据尾包（块内有数据包因内存不足被丢弃） | [R-249st] p.169；[O-241st] |
| INVALID_HEADER（0x800E） | 任意 ACK：报文头字段非法（如需要载荷的命令 length = 0） | [O-134c]；B.12 |
| PACKET_NOT_YET_AVAILABLE（0x8010） | GVSP 重传错误包（线扫、行触发慢） | Table 16-2；§16.7.3 |
| PACKET_AND_PREV_REMOVED_FROM_MEMORY（0x8011） | GVSP 重传错误包（可逐包发送，也可用 RESEND_RANGE_ERROR 按范围发送） | Table 16-2；[CO-206st] |
| PACKET_REMOVED_FROM_MEMORY（0x8012） | GVSP 重传错误包 | Table 16-2 |
| NO_REF_TIME（0x8013） | ACTION_ACK：参考时间未同步，Scheduled Action 不入队 | [CR-128cd] p.106 |
| PACKET_TEMPORARILY_UNAVAILABLE（0x8014） | GVSP 重传错误包，附带 hold-off 字 | [CR-208st] |
| OVERFLOW（0x8015） | ACTION_ACK：Scheduled Action 队列已满，不入队 | [CR-127cd] p.105 |
| ACTION_LATE（0x8016） | ACTION_ACK：action_time 已过，动作被立即执行 | [CR-221cd] |
| LEADER_TRAILER_OVERFLOW（0x8017） | multi-part 流：每个无法传全部 part 的块；以及打开流通道时（💡 即写 SCPx 的 WRITEREG_ACK）设备已知当前配置无法传输 | B.20 |
| ERROR（0x8FFF） | 任何无法给出更具体原因的情形 | [R-240cd]、[R-242st] |

<a id="appendix-b"></a>
### 19.4 逐条详解（合并 Table 19-1 与附录 B）

以下按状态码数值排序；括号中为附录 B 的小节号。

#### GEV_STATUS_SUCCESS — 0x0000（附录 B.1，p.402）
- 操作成功执行时由设备返回。对 GVCP 命令表示命令已成功完成；对 GVSP 表示数据传输包含正确数据。

#### GEV_STATUS_PACKET_RESEND — 0x0100（附录 B.2，p.402；Table 19-1，p.157）
- 出现在因重传请求而发送的 GVSP 包中。由于不允许存在 PACKETRESEND_ACK，它帮助 GVSP 接收端确认 PACKETRESEND_CMD 已被正确处理。
- Table 19-1：发送重传包时它优于 SUCCESS；使用 block_id64 时重传 GVSP 包的 flag bit 15（`GEV_FLAG_PACKET_RESEND`）必须置位。
- 💡 它的 severity 位为 0，属于“信息”而非错误；接收端不能把非 0 的 status 一律当作失败。

#### GEV_STATUS_NOT_IMPLEMENTED — 0x8001（附录 B.3，p.402）
- 当设备不支持所指定的 GVCP 命令时在 ACK 中返回，提示应用可能存在 GigE Vision 版本不匹配。
- 配合 [R-132c]（p.112）：ACK 值 = 命令值 + 1，且不附载荷。

#### GEV_STATUS_INVALID_PARAMETER — 0x8002（附录 B.4，p.402）
- 请求中提供的数据对当前情况无效（或越界）时返回；也可用于提供了过多参数的情形。例如向 bootstrap 寄存器写入不支持的值。
- 💡 相关：[O-427cd] 要求此时寄存器保持原值（p.248）。

#### GEV_STATUS_INVALID_ADDRESS — 0x8003（附录 B.5，p.402）
- GVCP 请求访问的地址上不存在寄存器时返回。例如不支持的 bootstrap 寄存器或设备未实现的可选寄存器；也可用于访问厂商自定义寄存器空间中（按 XML 描述文件）未映射任何寄存器的区域。

#### GEV_STATUS_WRITE_PROTECT — 0x8004（附录 B.6，p.402–403）
- 试图写只读寄存器、只读存储区或其他不能写的寄存器时返回，例如写 Version 寄存器。
- **当寄存器只有部分字段只读、其他字段可写时，不应使用 WRITE_PROTECT。**

#### GEV_STATUS_BAD_ALIGNMENT — 0x8005（附录 B.7，p.403）
- 存储访问未按适当边界对齐（地址或数据）时返回。例如 READMEM_CMD/WRITEMEM_CMD 要求数据量按 32 位对齐，字节数不是 4 的倍数时返回该码；读写访问的地址未 32 位对齐时同样返回。

#### GEV_STATUS_ACCESS_DENIED — 0x8006（附录 B.8，p.403）
试图访问不可访问的地址时返回，原因可能是当前权限或设备配置。典型情形：
1. **控制权限不足**：应用没有所需的 control 或 exclusive access。另一应用已持有 Exclusive Access 时，读请求被拒绝；另一应用已持有控制权（原文 “Exclusive Control Access”，💡 应理解为 exclusive 或 control access）时，写请求被拒绝。
2. **访问只写位置**：例如读 Action Device Key——该位置存在，但内容不可读。
3. **寄存器当前不可用**：因当前配置或工作模式被禁用，例如设备工作在 GVRSP 模式时访问仅用于 GVSP 的寄存器。

- 💡 相关：按 [R-138cd] 此时 ACK 只含报文头（length = 0），应用应忽略可能附带的任何数据（p.119）。

#### GEV_STATUS_BUSY — 0x8007（附录 B.9，p.403）
- 设备因当前状态不对而暂时无法执行 GVCP 请求时返回，告诉应用稍后重试。例如设备正在执行某个使存储位置暂时不可访问的操作：异步的相机标定正在进行时应用要求采集图像，设备可回 BUSY 让应用稍后重试。
- 💡 相关：[O-428cd] 指出它可作为 PENDING_ACK 的替代方案（p.248）。

#### 已废弃：0x8008–0x800B、0x800F（Table 19-1，p.157–158）
- “Not used starting with GigE Vision 2.0”。新实现不得使用；应用收到时可按通用错误处理（💡）。

#### GEV_STATUS_PACKET_UNAVAILABLE — 0x800C（附录 B.10，p.403–404）
- 当 PACKETRESEND_CMD 请求的包已不在设备内存中（通常已被新数据覆盖）时，在 GVSP 包中设置。
- 该码出现得早于 NOT_YET_AVAILABLE、AND_PREV_REMOVED_FROM_MEMORY、REMOVED_FROM_MEMORY，因此**建议优先使用后三者**，以便在重传时提供更好的可用性反馈。Table 16-2 也规定：只有其他状态码都不能更好描述原因时才用它（p.139）。

#### GEV_STATUS_DATA_OVERRUN — 0x800D（附录 B.11，p.404）
- 由于内部缓冲不足，载荷数据不得不被截断时，必须在数据尾包中设置该状态；例如采集速度快于 GVSP 传输速度。
- **整块被丢弃时应改用 `GEV_FLAG_PREVIOUS_BLOCK_DROPPED` 标志**（GVSP flag bit 14），而不是 DATA_OVERRUN（参见 [R-249st]/[R-250st]，p.169）。

#### GEV_STATUS_INVALID_HEADER — 0x800E（附录 B.12，p.404）
- 校验 GVCP 报文头时发现了**除 key 无效（非 0x42）和命令不支持以外**的问题时返回。例如很多命令需要在报文头之后携带数据，若 length 为 0，设备应返回 INVALID_HEADER 表示该命令格式错误。
- 💡 区分：key 错误 → 静默丢弃（[R-131c]）；命令不支持 → NOT_IMPLEMENTED；其他头部问题 → INVALID_HEADER。

#### GEV_STATUS_PACKET_NOT_YET_AVAILABLE — 0x8010（附录 B.13，p.404）
- 与重传逻辑相关：被请求重传的包尚未被设备采集。例如外部行触发速率低于应用 GVSP 超时的线扫相机。

#### GEV_STATUS_PACKET_AND_PREV_REMOVED_FROM_MEMORY — 0x8011（附录 B.14，p.404）
- 与重传逻辑相关：请求的包**及其之前所有包**都已从 GVSP 发送端内存中移除，均无法重传。常见于采集快于传输、应用发现丢包时该包已被覆盖的情形。

#### GEV_STATUS_PACKET_REMOVED_FROM_MEMORY — 0x8012（附录 B.15，p.404）
- 与重传逻辑相关：请求的包已从发送端内存中移除，无法重传。与 0x8011 不同，它**对之前的包不表态**，应用不应假设之前的包也不可得。该码放在数据包的 GVSP 头中。

#### GEV_STATUS_NO_REF_TIME — 0x8013（附录 B.18，p.405）
- 设备没有参考时间，因而无法把动作命令安排在未来的精确时刻执行。IEEE 1588 时钟状态可通过 IEEE 1588 Status 寄存器获得。
- 💡 相关：[CR-128cd]——此时 ACTION_ACK 返回该码且不得入队（p.106）。

#### GEV_STATUS_PACKET_TEMPORARILY_UNAVAILABLE — 0x8014（附录 B.16，p.405）
- 与重传逻辑相关：请求的包**暂时**不可得。
- 用于那些为重传预留了有限带宽、且不允许重传超出该带宽的相机。它让应用可以“明智地等一会儿再请求”，而不是误以为重传请求丢失（如果相机直接忽略请求，就会出现这种误判）。
- GVSP 发送端给出一个 hold-off 时间作为接收端再次发出重传请求的参考。hold-off 的格式参照以太网流控 PAUSE 帧建模；该建议值不必精确，可以是常数，属于实现质量。厂商也可以把它与内部带宽预留机制挂钩，精确指出在不超出带宽限制的前提下何时能发下一个包。
- 💡 相关：格式见 [03 文件 Figure 16-14](03_message_formats.md#packetresend-response)（[CR-208st]）。

#### GEV_STATUS_OVERFLOW — 0x8015（附录 B.17，p.405）
- 设备内部队列无法再容纳新内容、不得不丢弃请求时返回；由应用负责采取措施避免队列溢出。
- 用于 Scheduled Action Command 队列溢出：此时 ACTION_ACK 返回 OVERFLOW，该 ACTION_CMD 不入队。队列大小见 Scheduled Action Command Queue Size 寄存器（0x0970）。

#### GEV_STATUS_ACTION_LATE — 0x8016（附录 B.19，p.405）
- 用于 Scheduled Action Command：要求执行动作的时刻（action_time 字段）已经过去。它提示发出 ACTION_CMD 的应用可能出现了失步（mis-synchronization）。
- 💡 相关：[CR-221cd]——设备立即执行该动作并返回 ACTION_LATE，并应发出 `GEV_EVENT_ACTION_LATE` 事件（若已启用）。

#### GEV_STATUS_LEADER_TRAILER_OVERFLOW — 0x8017（附录 B.20，p.405–406）
- 表示数据头包或数据尾包的最大包大小被超出。只用于 multi-part 场景：数据头包含 multi-part 信息，其包大小限制了一个块中可传输的 part 数量。
- 使用时机：(a) 打开流通道时，若设备已知按当前配置无法传输；(b) 在每一个未能传输全部所需 part 的块中。
- 💡 相关：SCCFGx 的 `large_leader_trailer_enable` 置位后，数据头包最大可达 SCPSx 指定的大小，而不再局限于 576 字节（[R-261st]，p.175）。

#### GEV_STATUS_ERROR — 0x8FFF（附录 B.21，p.406）
- 当没有其他状态码能恰当描述问题原因时使用的默认状态码。由于它几乎不提供错误性质的信息，应尽量避免使用。

### 19.5 v3.0 中扩展状态码总是可用

- GVCP Capability（0x0934）bit 9（原 `extended_status_code_for_1_1`）与 bit 13（原 `extended_status_code_for_2_0`）在 v3.0 中已废弃、恒为 1（§36.29，p.293–294）；GVCP Configuration（0x0954）bit 29 与 bit 13 同样废弃、恒为 1（§36.35，p.301–302）。
- 💡 解读：从这些位的旧名称可知，早期版本中 1.1/2.0 新增的状态码曾需要显式启用；对 v3.0 设备不再需要任何启用操作。但应用若要兼容旧设备，仍应能处理“退回通用码”（如 `GEV_STATUS_ERROR`、`GEV_STATUS_PACKET_UNAVAILABLE`）的情况。

<a id="ch20"></a>
## 20 事件（Ch20，p.160）

- 支持消息通道时，**带数据的事件必须用 EVENTDATA 命令发送**（若设备支持 EVENTDATA）（[CR-243cd]）。
- 事件标识符就是 EVENT_CMD / EVENTDATA_CMD 中每个事件的 16 位 `event_identifier` 字段（格式见 [03 文件 17.1](03_message_formats.md#event)）。

**Table 20-1：标准事件列表**（p.160，已用 pdfplumber 对照原表核实）

| 事件标识符 | 值 | 描述 | 附带数据 |
|---|---|---|---|
| Deprecated | 0x0000–0x0006 | 已过时，由旧版规范定义 | — |
| GEV_EVENT_PRIMARY_APP_SWITCH | 0x0007 | 已批准主应用切换（primary application switchover） | 无 |
| GEV_EVENT_LINK_SPEED_CHANGE | 0x0008 | 链路速率已改变 | 无 |
| GEV_EVENT_ACTION_LATE | 0x0009 | Scheduled Action Command 的执行晚了 | 无 |
| Reserved | 0x000A–0x8000 | — | — |
| GEV_EVENT_ERROR_xxx | 0x8001–0x8FFF | 设备上发生了异步错误；**事件标识符的值就是该异步错误的状态码**（参见状态码列表） | 无 |
| Device-specific Event | 0x9000–0xFFFF | 参见 XML 设备描述文件 | 见设备文档 |

各事件说明：

- **0x0000–0x0006（废弃）**：由旧版规范定义，v3.0 不再定义其含义；附录 D 记载 v3.0 “Removed the older Asynchronous Events section”（p.416）。💡 新设备不应再使用这些值；曝光/传输等事件应作为设备自定义事件（0x9000–0xFFFF）在 XML 中描述。
- **GEV_EVENT_PRIMARY_APP_SWITCH（0x0007）**：用于 primary application switchover（§14.4，p.110）。设备可选地被配置为发送此事件，让原主应用知道自己失去了控制权；设备在把访问权授予新主应用**之前**发送它，前提是原主应用已打开消息通道并启用了该事件。发送该事件的设备**不应请求 ACK**，因为原主应用回 ACK 时消息通道可能已归新主应用控制。主应用切换不会关闭消息通道，由新主应用负责管理。
- **GEV_EVENT_LINK_SPEED_CHANGE（0x0008）**：支持链路聚合时，设备应提供由聚合组总速率变化触发的该事件（[CO-012cd]，§3.3.6，p.39）；在聚合组中有物理链路 up/down 导致总速率变化时发出。启用方法由 SFNC 定义，实际速率见 Link Speed 寄存器。
- **GEV_EVENT_ACTION_LATE（0x0009）**：支持 Scheduled Action 的设备，收到时间已过的有效动作命令时，若消息通道已打开且该事件已启用，应发出此事件（[CR-221cd] 第 2 条，p.146）。
- **0x000A–0x8000（保留）**：不得使用。
- **GEV_EVENT_ERROR_xxx（0x8001–0x8FFF）**：用事件报告异步错误，标识符取值与标准错误状态码一一对应。💡 例如 event_identifier = 0x800D 可理解为异步发生的 `GEV_STATUS_DATA_OVERRUN`；具体会发哪些由设备决定。
- **设备自定义事件（0x9000–0xFFFF）**：应在 XML 中描述（[CO-230cd]）；若附带数据，其内容也应在 XML 中说明（[CO-235cd]）。

> 💡 解读：标准事件都“无数据”，因此既可用 EVENT_CMD 也可用 EVENTDATA_CMD 发送；有数据的自定义事件则必须走 EVENTDATA（[CR-243cd]）。设备必须提供启用/禁用事件生成的方法（[CR-226cd]），在 GenICam 中通常体现为 SFNC 的 Event Control 类特征（如 EventSelector/EventNotification），具体以设备 XML 为准。

示例：设备在主应用切换前发送 PRIMARY_APP_SWITCH（不请求 ACK → flag = 0x10），req_id = 7：

```
42 10 00 C0 00 18 00 07   00 18 00 07   FF FF 00 00   00 00 00 00 00 00 00 00   <8 字节 timestamp>
```

<a id="ch21"></a>
## 21 ICMP（Ch21，p.161–162）

- 设备必须在**其所有支持的网络接口上**至少支持 Table 21-1 中规定的 ICMP 子集（[R-244cd]）。
- 每种 ICMP 报文分为强制或可选，**只有 Echo Reply、Echo 和 Destination Unreachable 是强制的**（p.161）。
- 例如：设备必须支持包大小不超过 576 字节的 ping，即必须能正确接收 ICMP Echo 并正确发送 ICMP Echo Reply。建议设备至少对其支持的最大控制通道包大小（CCMPS）也能回应 ping（p.161）。
- **Destination Unreachable**（p.161 NOTE）：必须按 RFC 的建议处理。该报文可能源于路由的暂态变化，因此只是“目的不可达的提示，而非证据”。RFC1122 给出了更多细节，并根据报文中的错误码给出处理建议：例如收到**软错误（soft error）时不得中断连接**，收到**硬错误（hard error）时则应中断**。若设备因通信错误决定关闭其通道，必须遵循 [R-072cd]（停止采集与流传输、复位所有连接状态等，p.74）。

**Table 21-1：ICMP 报文**（p.161–162）

| 类型码 | 报文 | 设备接收 | 设备发送 |
|---|---|---|---|
| 0 | Echo Reply | 可选 | **强制**（包大小 ≤ 576 字节；> 576 不要求） |
| 3 | Destination Unreachable | **强制** | 可选 |
| 4 | Source Quench | 可选 | 可选 |
| 5 | Redirect | 可选 | 可选 |
| 8 | Echo | **强制**（包大小 ≤ 576 字节；> 576 不要求） | 可选 |
| 9 | Router Advertisement | 可选 | 可选 |
| 10 | Router Solicitation | 可选 | 可选 |
| 11 | Time Exceeded | 可选 | 可选 |
| 12 | Parameter Problem | 可选 | 可选 |
| 13 | Timestamp | 可选 | 可选 |
| 14 | Timestamp Reply | 可选 | 可选 |
| 15 | Information Request | 可选 | 可选 |
| 16 | Information Reply | 可选 | 可选 |
| 17 | Address Mask Request | 可选 | 可选 |
| 18 | Address Mask Reply | 可选 | 可选 |

> 💡 解读：
> - 规范没有明确 “packet size 576” 指的是 IP 数据报总长还是 ICMP 载荷长度；结合 GVCP 中 576 字节一律包含 IP 头的用法，按 IP 数据报总长理解较为一致（用 Windows `ping -l` 或 Linux `ping -s` 测试时，指定的是 ICMP 数据长度，需另加 IP 20 + ICMP 8 字节）。
> - 规范允许 GVSP 发送端在启用 unconditional streaming 时，不受控制通道状态或收到的 ICMP 报文（如 Destination Unreachable）影响而继续推流（§10.4 NOTE，p.74；详见 §11.13）。

---

## 实现要点与易错点

### 设备端

- **报文值**：ACK 一律用“命令值 + 1”；绝不能发送 0x0041；PENDING_ACK 用 0x0089，其 ack_id 等于原命令的 req_id。
- **状态码选择**：尽量返回最具体的状态码，兜底用 0x8FFF；不要使用已废弃的 0x8008–0x800B、0x800F。
- **ACCESS_DENIED 只回报文头**（length = 0）；READMEM 任何失败也都 length = 0。
- **WRITE_PROTECT 的边界**：整个寄存器只读时才用；只有部分字段只读的寄存器不要用它（附录 B.6）。
- **设备自定义状态码**要把 bit 1 置位：错误为 `0xC000 | value`，信息为 `0x4000 | value`，否则会与标准码冲突。
- **GVSP 侧**：重传包用 0x0100 + flag bit 15；块内数据被截断 → 尾包 DATA_OVERRUN；整块丢弃 → 下一个数据头包置 PREVIOUS_BLOCK_DROPPED，不要用 DATA_OVERRUN；无法重传时优先使用 0x8010/0x8011/0x8012/0x8014，而不是笼统的 0x800C。
- **事件**：PRIMARY_APP_SWITCH 不请求 ACK、且在授予新主应用访问权之前发出；Scheduled Action 迟到时既回 ACTION_LATE，也发 ACTION_LATE 事件（若已启用）；带数据的事件必须用 EVENTDATA。
- **ICMP**：每个网络接口都要能回应 ≤ 576 字节的 ping（最好到 CCMPS）；收到 Destination Unreachable 时区分软/硬错误，不要因为一次软错误就关闭通道。

### 应用端

- **判断成败看 severity 位（0x8000）**，不要写成 `status == 0`；0x0100 是成功的重传包。
- **按状态码采取行动**：BUSY → 稍后重试；ACCESS_DENIED → 检查 CCP/权限，或是否读了只写寄存器、是否处于错误的流模式；NOT_IMPLEMENTED → 设备不支持（先查 GVCP Capability）；INVALID_ADDRESS → 寄存器不存在或接口不存在；BAD_ALIGNMENT → 应用自身的地址/长度计算有误；INVALID_HEADER → 报文构造错误。
- **设备自定义状态码**（bit 1 = 1）无法通用解析，应查设备文档或 XML。
- **事件分发**：按区间处理——0x0007–0x0009 标准事件；0x8001–0x8FFF 视为异步错误并映射到状态码名称；0x9000–0xFFFF 交给 GenICam/XML 解码；0x0000–0x0006 与保留区间的值视为异常或旧设备行为。

### 流接收端（GVSP receiver）

- 重传错误码要区别对待：0x8011 → 不再请求该包及其之前的包；0x8012 → 只放弃该包，之前的仍可请求；0x8014 → 按 hold-off 等待；0x8010 → 稍后重试；0x800C → 放弃。
- 尾包 status = DATA_OVERRUN → 该块数据不完整；数据头包 flag 有 PREVIOUS_BLOCK_DROPPED → 用 block_id64 的跳变量计算丢了几块。
- multi-part 流收到 LEADER_TRAILER_OVERFLOW → 该块的 part 不全，可考虑启用 `large_leader_trailer_enable` 或减少 part 数量。

---

## 相关章节

- [03_message_formats.md](03_message_formats.md)：所有报文的字节级格式、标志位与时序（Ch15–17）
- [01_transport_and_channels.md](01_transport_and_channels.md)：控制/消息通道、权限、心跳、PENDING_ACK 使用（Ch7–13）
- [02_advanced_concepts.md](02_advanced_concepts.md)：Action Command、Scheduled Action、IEEE 1588、primary application switchover（Ch14）
- [../03_gvsp/01_transport_and_data_block.md](../03_gvsp/01_transport_and_data_block.md)：GVSP 包头 status/flag、DATA_OVERRUN 与 PREVIOUS_BLOCK_DROPPED（Ch22–24）
- [../03_gvsp/02_payload_types_and_pixel_layouts.md](../03_gvsp/02_payload_types_and_pixel_layouts.md)：multi-part 与 LEADER_TRAILER_OVERFLOW 的上下文（Ch25–26）
- [../05_bootstrap_registers/02_device_and_network_registers.md](../05_bootstrap_registers/02_device_and_network_registers.md)：GVCP Capability / Configuration、Pending Timeout、Scheduled Action Queue Size、IEEE 1588 Status 等寄存器
- [../01_device_discovery/device_discovery.md](../01_device_discovery/device_discovery.md)：链路聚合与 LINK_SPEED_CHANGE 事件的背景（Ch3）
- [../reference/requirements_index.md](../reference/requirements_index.md)：全部需求编号索引

---

## 自测题

1. READMEM_CMD 对应的确认值是多少？为什么 PENDING_ACK 是 0x0089 而 0x0088 没有定义？
   <details><summary>答案</summary>

   READMEM_ACK = 0x0084 + 1 = 0x0085。PENDING_ACK 是“只有确认、没有命令”的报文，取最低位为 1 的确认值 0x0089；不存在对应的命令，所以 0x0088 未分配（[R-238c] 明确把 PENDING_ACK 列为例外）。
   </details>

2. 设备返回 status = 0xC123，这代表什么？
   <details><summary>答案</summary>

   bit 0（0x8000）= 1 → 错误；bit 1（0x4000）= 1 → 设备自定义状态码；status_value = 0x123。具体含义需查设备文档或 XML，规范不定义（p.156）。
   </details>

3. GVSP 包头 status = 0x0100 表示出错了吗？
   <details><summary>答案</summary>

   不是。0x0100 是 `GEV_STATUS_PACKET_RESEND`，severity 位为 0（信息），表示这是一个响应 PACKETRESEND 的重传包；同时 flag bit 15（0x0001）应置位（Table 19-1，[CR-204st]）。
   </details>

4. 对 GVSP 接收端来说，0x8011 与 0x8012 的处理有什么不同？
   <details><summary>答案</summary>

   0x8011（PACKET_AND_PREV_REMOVED_FROM_MEMORY）：该包及之前的所有包都已丢弃，不应再请求它们。0x8012（PACKET_REMOVED_FROM_MEMORY）：只说明该包已丢弃，对之前的包不表态，接收端可以继续对更早的未到包运行重传算法（p.158，附录 B.14/B.15）。
   </details>

5. 某寄存器高 16 位只读、低 16 位可写。应用写入整个寄存器时，设备能否返回 `GEV_STATUS_WRITE_PROTECT`？
   <details><summary>答案</summary>

   不应该。附录 B.6 明确规定：当寄存器只有部分字段只读、其他字段可写时，不应使用 WRITE_PROTECT（p.402–403）。
   </details>

6. 应用在 EVENT_CMD 中收到 event_identifier = 0x8006，含义是什么？收到 0x0003 呢？
   <details><summary>答案</summary>

   0x8006 落在 GEV_EVENT_ERROR_xxx 区间（0x8001–0x8FFF），表示设备发生了异步错误，其状态码为 0x8006（GEV_STATUS_ACCESS_DENIED）。0x0003 落在 0x0000–0x0006 的废弃区间，由旧版规范定义，v3.0 不再定义其含义（p.160）。
   </details>

7. 设备必须支持哪些 ICMP 报文？
   <details><summary>答案</summary>

   接收 Echo（类型 8）并发送 Echo Reply（类型 0），对 ≤ 576 字节的包为强制；接收 Destination Unreachable（类型 3）为强制。其余均可选，且要求适用于所有支持的网络接口（[R-244cd]，Table 21-1）。
   </details>

8. 数据尾包 status = DATA_OVERRUN 与数据头包 flag 置位 PREVIOUS_BLOCK_DROPPED，分别说明了什么？
   <details><summary>答案</summary>

   DATA_OVERRUN：当前块已发出数据头，但块内有数据包因发送端缓冲不足被丢弃，块不完整（[R-249st]）。PREVIOUS_BLOCK_DROPPED：在当前块之前有一个或多个完整的块被整体丢弃（连数据头都没发），可用 block_id64 的跳变量计算丢了几块（[R-250st]，附录 B.11）。
   </details>

