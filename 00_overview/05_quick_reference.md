# GigE Vision 3.0 速查表（Quick Reference）

> **覆盖范围**：汇总 Ch4、5、8、10、11、12、15、18、19、24、25、31、36 中最常查的常量与格式（PDF p.40–352）。所有数值均已对照规范核对，页码为 PDF 页码（印刷页码 = PDF 页码 − 1）。
>
> **一句话总结**：写代码、调试、抓包（Wireshark）时随手可查的端口、报文头、命令码、状态码、寄存器地址与关键位。
>
> **本文目录**：[1 基本约定](#1-基本约定) · [2 GVCP 报文头](#2-gvcp-报文头ch15) · [3 命令/确认码](#3-命令确认码ch18) · [4 状态码](#4-状态码ch19) · [5 常用寄存器](#5-常用-bootstrap-寄存器ch36) · [6 关键位定义](#6-关键位定义) · [7 GVSP 报文头](#7-gvsp-报文头ch24) · [8 GVSP 负载类型](#8-gvsp-负载类型ch25) · [9 最小操作序列](#9-最小操作序列速记)

---

## 1. 基本约定

| 项目 | 取值 / 规则 | 出处 |
|---|---|---|
| GVCP 设备端端口 | 设备第一个 GVCP 端口**必须**为 **UDP 3956**（IANA 注册）；应用用任意动态端口作为源端口 | §8.1, p.60（[R-042c]） |
| GVRSP 连接的 ServiceID | 由固定前缀 + IP 协议号 + 端口构成，GigE Vision 服务使用 **TCP 3956**；传输服务类型为 Reliable Connection | §31.1, p.223（[CR-634sr]） |
| 流/消息通道源端口 | 建议使用动态端口（IANA 动态端口范围 49152–65535） | §11.5, p.83（[O-086st]）；§12.2, p.90 |
| 字节序 | GVCP、GVSP 头部/leader/trailer、bootstrap 寄存器一律**大端（network byte order）** | §15.3, p.115（[R-137c]）；§24.2, p.173（[R-257s]）；§36, p.246 |
| **位编号** | **MSB-first：bit 0 = 最高位**（32 位寄存器 bit 31 = 最低位）。看到 “bit 31” 对应掩码 `0x00000001` | §15.3, p.115 |
| GVCP 报文大小 | 一条命令/确认必须在单个 UDP 包内，不允许 IP 分片；包长上限 = Control Channel Max Packet Size（**默认 576 字节**，含 IP 20 + UDP 8 + GVCP 头 8 → 默认最大 GVCP 负载 540 字节） | §8.1.1–8.1.2, p.60–61（[R-043c] [R-044c]） |
| GVSP leader / trailer 大小 | 默认必须放进 **576 字节**的单包内（含 IP/UDP/GVSP 头）；置位 SCCFGx 的 `large_leader_trailer_enable` 后上限改为 SCPSx 包长 | §24.3, p.175, 178（[R-261st] [R-267st]） |
| IP 配置顺序 | ① Persistent IP（若支持且启用）→ ② DHCP（若启用）→ ③ LLA；出厂默认：Persistent IP 关、DHCP 开，LLA 始终开启 | §4.1, p.40–41（[R-015cd] [R-016cd]） |
| LLA 地址范围 | 169.254.1.0 – 169.254.254.255（RFC3927），设备必须支持且不能关闭 | §4.4, p.47（[R-028cd] [R-029cd]） |
| 广播发现 | DISCOVERY_CMD 发往 255.255.255.255（limited broadcast，不跨路由器）或子网定向广播；设备完成 IP 配置前不得应答 | §5.1, p.48–49（[R-030cd] [R-031cd]） |
| 心跳超时 | Heartbeat Timeout 寄存器 `0x0938`，默认 3000 ms；超时未收到主应用的有效命令，设备关闭控制通道 | §10.5, p.75（[R-076cd]）；Table 36-1, p.257 |
| block_id64 | 从 1 开始递增；0 保留给测试包；流通道打开时复位为 1 | §24.2, p.174（[R-259st]） |
| 需求编号 | `[R/CR/O/CO-<序号><后缀>]`，后缀 ca/cd/c/st/sr/s，见 [需求索引](../reference/requirements_index.md) | §1.3.2, p.19–20 |

---

## 2. GVCP 报文头（Ch15）

命令头与确认头均为 **8 字节**（不含 Ethernet/IP/UDP 头），32 位自然对齐、大端。

```text
命令头 Command header（§15.1, p.113）
 0               7 8            15 16                             31
+-----------------+---------------+--------------------------------+
|   0x42 (key)    |     flag      |            command             |
+-----------------+---------------+--------------------------------+
|             length              |             req_id             |
+---------------------------------+--------------------------------+

确认头 Acknowledge header（§15.2, p.114）
 0                              15 16                             31
+---------------------------------+--------------------------------+
|             status              |          acknowledge           |
+---------------------------------+--------------------------------+
|             length              |             ack_id             |
+---------------------------------+--------------------------------+
```

| 字段 | 说明 |
|---|---|
| `0x42` | 固定识别码；首字节不是 0x42 的包不视为 GVCP 报文，必须静默丢弃（[O-130c] [R-131c]） |
| `flag` | bit 0–3：各命令自定义；bit 4–6：保留；**bit 7 = ACKNOWLEDGE**（置 1 要求对端回确认；清 0 则即使该命令通常要回确认也**不回**） |
| `command` / `acknowledge` | 报文码（见 §3）；确认码 = 命令码 + 1（PACKETRESEND_CMD 与 PENDING_ACK 例外） |
| `length` | 头部之后的负载字节数（不含 8 字节头） |
| `req_id` / `ack_id` | 应用生成、**不得为 0**、逐条递增，回绕时跳过 0 回到 1；设备把它原样拷贝到 `ack_id` |

---

## 3. 命令/确认码（Ch18）

规范保留 0–32767（最高位为 0）给 GVCP，32768–65535（最高位为 1）给厂商自定义报文（§18, p.154）。

| 报文 | 通道 | 值 | 报文 | 通道 | 值 |
|---|---|---|---|---|---|
| DISCOVERY_CMD | Control | `0x0002` | DISCOVERY_ACK | Control | `0x0003` |
| FORCEIP_CMD | Control | `0x0004` | FORCEIP_ACK | Control | `0x0005` |
| PACKETRESEND_CMD | Control | `0x0040` | （`0x0041` 无效：重传包走流通道） | — | `0x0041` |
| READREG_CMD | Control | `0x0080` | READREG_ACK | Control | `0x0081` |
| WRITEREG_CMD | Control | `0x0082` | WRITEREG_ACK | Control | `0x0083` |
| READMEM_CMD | Control | `0x0084` | READMEM_ACK | Control | `0x0085` |
| WRITEMEM_CMD | Control | `0x0086` | WRITEMEM_ACK | Control | `0x0087` |
| — | — | — | PENDING_ACK | Control | `0x0089` |
| EVENT_CMD | Message | `0x00C0` | EVENT_ACK | Message | `0x00C1` |
| EVENTDATA_CMD | Message | `0x00C2` | EVENTDATA_ACK | Message | `0x00C3` |
| ACTION_CMD | Control | `0x0100` | ACTION_ACK | Control | `0x0101` |

> 💡 解读：§15.1（p.113）写的是 “0 to 32765 reserved”，与 §18（p.154）的 “0 to 32767” 不一致，以 §18 的“最高位清零”规则为准更自洽。

详细字段布局见 [02_gvcp/03_message_formats.md](../02_gvcp/03_message_formats.md)。

---

## 4. 状态码（Ch19）

状态字 16 位：bit 0 = `severity`（0 信息 / 1 错误），bit 1 = `device_specific_flag`（0 标准 / 1 厂商自定义），bit 2–3 保留，bit 4–15 = `status_value`（§19, p.156）。

| 名称 | 值 | 含义（简述） |
|---|---|---|
| GEV_STATUS_SUCCESS | `0x0000` | 成功 |
| GEV_STATUS_PACKET_RESEND | `0x0100` | 仅用于重传的 GVSP 包（比 SUCCESS 更利于接收端统计重传） |
| GEV_STATUS_NOT_IMPLEMENTED | `0x8001` | 命令不支持 |
| GEV_STATUS_INVALID_PARAMETER | `0x8002` | 参数非法/越界 |
| GEV_STATUS_INVALID_ADDRESS | `0x8003` | 访问不存在的地址 |
| GEV_STATUS_WRITE_PROTECT | `0x8004` | 写只读寄存器 |
| GEV_STATUS_BAD_ALIGNMENT | `0x8005` | 地址或长度未对齐 |
| GEV_STATUS_ACCESS_DENIED | `0x8006` | 当前权限/状态下不可访问 |
| GEV_STATUS_BUSY | `0x8007` | 资源暂不可用，可稍后重试 |
| （Deprecated） | `0x8008`–`0x800B` | 自 2.0 起不再使用 |
| GEV_STATUS_PACKET_UNAVAILABLE | `0x800C` | 请求重传的包已不可用 |
| GEV_STATUS_DATA_OVERRUN | `0x800D` | 发送端内部存储溢出 |
| GEV_STATUS_INVALID_HEADER | `0x800E` | 报文头字段不合规范 |
| （Deprecated） | `0x800F` | 自 2.0 起不再使用 |
| GEV_STATUS_PACKET_NOT_YET_AVAILABLE | `0x8010` | 请求的包尚未采集（如线阵慢触发） |
| GEV_STATUS_PACKET_AND_PREV_REMOVED_FROM_MEMORY | `0x8011` | 该包及之前的包均已丢弃，勿再请求 |
| GEV_STATUS_PACKET_REMOVED_FROM_MEMORY | `0x8012` | 该包已丢弃（更早的包未必） |
| GEV_STATUS_NO_REF_TIME | `0x8013` | 未同步到主时钟（定时 Action 无法调度） |
| GEV_STATUS_PACKET_TEMPORARILY_UNAVAILABLE | `0x8014` | 带宽原因暂时不能重传，稍后再请求 |
| GEV_STATUS_OVERFLOW | `0x8015` | 队列/数据溢出（如定时 Action 队列满） |
| GEV_STATUS_ACTION_LATE | `0x8016` | 定时 Action 的时间已过 |
| GEV_STATUS_LEADER_TRAILER_OVERFLOW | `0x8017` | leader/trailer 包长不足以容纳信息 |
| GEV_STATUS_ERROR | `0x8FFF` | 通用错误（无更具体状态码时至少返回它，[R-240cd]） |

完整解释（含附录 B）见 [02_gvcp/04_command_values_status_events.md](../02_gvcp/04_command_values_status_events.md)。

---

## 5. 常用 Bootstrap 寄存器（Ch36）

地址 `0x0000–0x9FFF` 保留给标准 bootstrap 寄存器，`0xA000` 起为厂商自定义空间（§36, p.247, [R-422cd]）。完整列表见 [reference/bootstrap_register_map.md](../reference/bootstrap_register_map.md)。

| 地址 | 名称 | M/O | 访问 | 长度 | 用途速记 |
|---|---|---|---|---|---|
| `0x0000` | Version | M | R | 4 | 规范版本（3.0 设备返回 `0x00030000`，§36.1, p.269） |
| `0x0004` | Device Mode | M | R | 4 | 设备类别、字符集等 |
| `0x0008` / `0x000C` | Device MAC Address High / Low（网口 #0） | M | R | 4+4 | MAC 地址 |
| `0x0010` / `0x0014` | Network Interface Capability / Configuration（#0） | M | R / R/W | 4 | 支持/启用的 IP 配置方式（Persistent/DHCP/LLA）与 PAUSE |
| `0x0024` / `0x0034` / `0x0044` | Current IP / Subnet Mask / Default Gateway（#0） | M | R | 4 | 当前 IP 参数 |
| `0x0048` / `0x0068` / `0x0088` | Manufacturer Name / Model Name / Device Version | M | R | 32 | 字符串 |
| `0x00A8` | Manufacturer Info | M | R | 48 | 字符串 |
| `0x00D8` / `0x00E8` | Serial Number / User-defined Name | O | R / R/W | 16 | 字符串 |
| `0x0200` / `0x0400` | First URL / Second URL | M | R | 512 | XML 描述文件位置（如 `local:...;地址;长度`） |
| `0x0600` | Number of Network Interfaces | M | R | 4 | 网口数 |
| `0x064C` / `0x065C` / `0x066C` | Persistent IP / Subnet / Gateway（#0） | O | R/W | 4 | 静态 IP 设置 |
| `0x0900` / `0x0904` | Number of Message / Stream Channels | M | R | 4 | 通道数量 |
| `0x092C` | Streaming Capability | M<sup>[1]</sup> | R | 4 | GVSP/GVRSP 能力 |
| `0x0934` | GVCP Capability | M | R | 4 | 可选命令/功能支持位图 |
| `0x0938` | Heartbeat Timeout | M | R/W | 4 | 心跳超时（ms，默认 3000） |
| `0x093C` / `0x0940` | Timestamp Tick Frequency High / Low | O | R | 4+4 | 时间戳频率（64 位） |
| `0x0944` | Timestamp Control | O | W | 4 | 锁存/复位时间戳 |
| `0x0948` / `0x094C` | Timestamp Value High / Low | M | R | 4+4 | 锁存的时间戳 |
| `0x0950` | Discovery ACK Delay | O | R/(W) | 4 | 发现应答随机延时上限 |
| `0x0954` | GVCP Configuration | O | R/W | 4 | 可选功能开关（bit 31 关闭心跳等） |
| `0x0958` | Pending Timeout | M | R | 4 | PENDING_ACK 相关超时 |
| `0x0960` | Streaming Configuration | M | R/W | 4 | 选择 GVSP / GVRSP 流协议 |
| `0x0A00` | **CCP**（Control Channel Privilege） | M | R/W | 4 | 申请/释放控制权 |
| `0x0A24` | CCMPS（Control Channel Max Packet Size） | O | R/W | 4 | GVCP 最大包长 |
| `0x0B00` / `0x0B10` | MCP / MCDA | O | R/W | 4 | 消息通道端口 / 目的 IP |
| `0x0B14` / `0x0B18` | MCTT / MCRC | O | R/W | 4 | 消息通道超时 / 重试次数 |
| `0x0D00 + 0x40·x` | **SCPx**（GVSP 流通道端口） | M<sup>[1][3]</sup>（x=0） | R/W | 4 | host_port≠0 打开、=0 关闭 |
| `0x0D04 + 0x40·x` | **SCPSx**（包长） | M<sup>[1][3]</sup>（x=0） | R/W | 4 | 包长 / 测试包 / DF 位 |
| `0x0D08 + 0x40·x` | SCPDx（包间延时） | M<sup>[2][3]</sup>（x=0） | R/W | 4 | 发包间隔（时间戳计数单位） |
| `0x0D18 + 0x40·x` | **SCDAx**（目的 IP） | M<sup>[1][3]</sup>（x=0） | R/W | 4 | 流数据目的地址 |
| `0x0D3C + 0x40·x` | SCEBAx | M<sup>[1]</sup>（x=0） | R | 4 | 扩展 bootstrap 区基址（GenDC/GVRSP 寄存器在 `SCEBAx + 偏移`） |
| `0x9000` | Manifest Table | O | R | 512 | 多个 XML 文件清单 |
| `0x9800 + 0x10·x` / `0x9804 + 0x10·x` | Action Group Key x / Mask x | O | R/W | 4 | Action 命令分组匹配 |

脚注（Table 36-1, p.269）：[1] 发送/接收/收发设备强制，外设不适用；[2] 仅发送端设备强制；[3] 支持 GVSP 时强制；[4] 支持 GVRSP 时强制。

---

## 6. 关键位定义

### 6.1 CCP（`0x0A00`，§36.46, p.312–314）

| spec 位 | LSB0 掩码 | 名称 | 说明 |
|---|---|---|---|
| 0–15 | `0xFFFF0000` | control_switchover_key | 仅在支持主应用切换且 CSE 已置位时，由**其他**应用写入用于抢占；读回恒为 0 |
| 16–19 | `0x0000F000` | connection_count | 设备维护的连接计数（若支持），每次打开控制通道递增，首次连接后不为 0；写入忽略 |
| 20–28 | — | reserved | 读为 0 |
| 29 | `0x00000004` | control_switchover_enable (CSE) | 允许其他应用（凭 key）接管 |
| 30 | `0x00000002` | control_access (CA) | 申请控制权限（他人仍可监视/读） |
| 31 | `0x00000001` | exclusive_access (EA) | 申请独占权限（他人不能控制也不能监视）；优先级高于 CA |

访问模式（Table 36-3）：EA=1 → Exclusive；EA=0, CA=1, CSE=0 → Control；EA=0, CA=1, CSE=1 → Control with switchover；EA=0, CA=0 → Open access。
释放控制权：主应用向 CCP 写 0（[R-071ca]）。

### 6.2 SCPx（`0x0D00 + 0x40·x`，§36.57, p.322–324）

| spec 位 | LSB0 掩码 | 名称 | 说明 |
|---|---|---|---|
| 0 | `0x80000000` | direction (D) | 只读：0 = 发送端，1 = 接收端 |
| 14–15 | `0x00030000` | network_interface_index (NII) | 使用的网口 0–3（⚠️ §3.2, p.36 正文写作 “bit 12 to 15”，与本寄存器定义的 bit 14–15 不一致，以 §36.57 为准） |
| 16–31 | `0x0000FFFF` | host_port | 目的 UDP 端口；**写非 0 打开流通道，写 0 关闭**；打开时 block_id64 复位为 1 |

要点：配置 SCDAx/SCCFGx/SCPSx 前须先让 host_port = 0（包长协商除外，[R-561ca]）；正常出流期间 SCPx 只允许写 0，否则返回 GEV_STATUS_BUSY（[R-586cd]）；GVRSP 模式下访问 SCPx 返回 GEV_STATUS_ACCESS_DENIED（[R-671cd]）。

### 6.3 SCPSx（`0x0D04 + 0x40·x`，§36.58, p.324–326）

| spec 位 | LSB0 掩码 | 名称 | 说明 |
|---|---|---|---|
| 0 | `0x80000000` | fire_test_packet (F) | 置 1 发一个测试包（DF 置位、大小 = packet_size）；自清零 |
| 1 | `0x40000000` | do_not_fragment (D) | 拷贝到每个流包 IP 头的 DF 位 |
| 2 | `0x20000000` | deprecated | 读为 0（2.0 及以前为 pixel_endianness） |
| 16–31 | `0x0000FFFF` | packet_size | 流包总长（**含 IP 20 + UDP 8 + GVSP 头**）；不支持的值向下取整（小于最小值则向上取整），应读回确认 |

包长协商（§11.7, p.84–85）：打开流通道后、正式出流前，用“写 packet_size → 读回取整值 → 带 F 位写入发测试包”反复二分查找（[R-567s]）。

---

## 7. GVSP 报文头（Ch24）

3.0 中 GVSP 头部**固定使用扩展 ID（EI = 1）**：64 位 `block_id64` + 32 位 `packet_id32`，头长 **20 字节**；16 位 block_id 的旧格式请查 GigE Vision 2.0（§24.2, p.173–174）。

```text
 0                              15 16                             31
+---------------------------------+--------------------------------+
|             status              |              flag              |
+--+------+-------------+---------+--------------------------------+
|EI| rsv  |packet_format|             reserved (24 bits)           |
+--+------+-------------+------------------------------------------+
|                     block_id64 (high 32 bits)                     |
+-------------------------------------------------------------------+
|                     block_id64 (low 32 bits)                      |
+-------------------------------------------------------------------+
|                           packet_id32                             |
+-------------------------------------------------------------------+
 EI = bit 32（第 2 个 32 位字的 bit 0）必须为 1；rsv 3 位；packet_format 4 位
```

| packet_format | 含义 |
|---|---|
| 1 | DATA_LEADER_FORMAT（packet_id32 = 0） |
| 2 | DATA_TRAILER_FORMAT |
| 3 | Generic 数据负载 |
| 5 | H.264 数据负载 |
| 7 | Multi-part 数据负载 |
| 8 | GenDC 数据负载 |

flag 字段（Table 24-1, p.175）：bit 0–7 厂商自定义；bit 8–12 保留；**bit 13** GEV_FLAG_RESEND_RANGE_ERROR；**bit 14** GEV_FLAG_PREVIOUS_BLOCK_DROPPED；**bit 15** GEV_FLAG_PACKET_RESEND（该包是重传结果）。

> 💡 解读：Generic 负载包每包有效数据 = SCPSx.packet_size − 20（IP）− 8（UDP）− 20（GVSP 头）= packet_size − 48。例如 packet_size = 1500 时每包 1452 字节；9000（巨帧）时 8952 字节。

详见 [03_gvsp/01_transport_and_data_block.md](../03_gvsp/01_transport_and_data_block.md)。

---

## 8. GVSP 负载类型（Ch25）

3.0 规范正文只重新定义了 Image、Chunk data、GenDC；JPEG/JPEG 2000/H.264/Multi-part 仍然有效且具规范性，但其定义只在 GigE Vision 2.2 中（§25, p.186–187）。payload_type 的 bit 1（即 `0x4000`）= 扩展 Chunk 模式，bit 0（`0x8000`）= 厂商自定义。

| 负载类型 | 值 | 扩展 Chunk 模式值 | 备注 |
|---|---|---|---|
| Image | `0x0001` | `0x4001` | 未压缩图像；扩展 chunk 时 Image 须为第一个 chunk |
| Chunk data | `0x0004` | — | GenICam 规定的带标签数据块 |
| JPEG | `0x0006` | `0x4006` | 定义见 GEV 2.2 |
| JPEG 2000 | `0x0007` | `0x4007` | 定义见 GEV 2.2 |
| H.264 | `0x0008` | `0x4008` | 定义见 GEV 2.2 |
| Multi-part | `0x000A` | `0x400A` | 定义见 GEV 2.2 |
| GenDC | `0x000B` | 不允许 | chunk 数据应放在 GenDC 容器内 |
| Device-specific | `≥ 0x8000` | — | 厂商自定义 |

详见 [03_gvsp/02_payload_types_and_pixel_layouts.md](../03_gvsp/02_payload_types_and_pixel_layouts.md)。

---

## 9. 最小操作序列速记

| 目标 | 操作（寄存器/报文） | 出处 |
|---|---|---|
| 找设备 | 广播 DISCOVERY_CMD 到 UDP 3956 → 收 DISCOVERY_ACK | §5.1, p.48–49 |
| 读 XML | READMEM `0x0200`（First URL）→ 解析 `local:<文件名>.<xml/zip>;<地址hex>;<长度hex>` → 分块 READMEM | §14.1.1, p.95–96 |
| 取得控制权 | WRITEREG CCP(`0x0A00`) = `0x00000002`（control）或 `0x00000001`（exclusive）→ 期望 GEV_STATUS_SUCCESS，否则 GEV_STATUS_ACCESS_DENIED | §10.3, p.73 |
| 保活 | 周期性 READREG CCP（任何来自主应用的有效 GVCP 命令都能重置心跳，PACKETRESEND/DISCOVERY/ACTION/FORCEIP 除外） | §10.5, p.75（[R-075cd]） |
| 打开 GVSP 流 | SCPx=0 → 写 SCDAx（主机 IP）/SCPDx 等 → 写 SCPx.host_port≠0 → （可选）SCPSx 包长协商 → GenICam `AcquisitionStart` | §11.5–11.7, p.83–85；§36.57, p.322 |
| 打开消息通道 | 写 MCDA（主机 IP）→ 写 MCP.host_port≠0；应用收到 EVENT/EVENTDATA 后用来包的源端口回 ACK | §12.2, p.90（[CR-099ca] [CR-100ca]） |
| 关闭 | `AcquisitionStop` → SCPx 写 0（[R-091ca]）→ MCP 写 0（[CR-106ca]）→ CCP 写 0（[R-071ca]） | §10.4, §11.8, §12.4 |

完整流程与时序图见 [04_end_to_end_walkthrough.md](04_end_to_end_walkthrough.md)。
