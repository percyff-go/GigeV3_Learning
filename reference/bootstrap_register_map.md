# Bootstrap 寄存器总表（Table 36-1 地址索引）

> 自动生成：用 pdfplumber 从规范 Table 36-1: Bootstrap Registers（PDF p.248–269）逐行提取地址、名称、支持级别、访问类型与长度（事实性数据，未收录原文描述）。
> 中文分组讲解与一句话说明见 [05_bootstrap_registers/01_overview_and_register_map.md](../05_bootstrap_registers/01_overview_and_register_map.md)，各寄存器位定义见 [02_device_and_network_registers.md](../05_bootstrap_registers/02_device_and_network_registers.md) 与 [03_channel_registers.md](../05_bootstrap_registers/03_channel_registers.md)。

## 读表说明

- **Support**：`M` = 必须实现（Mandatory），`O` = 可选（Optional），`-` = 保留/不适用。少数行在原规范中印作数字 `0`（如 SCMPC1/SCMBS1/SCEBA1 及 511 号通道），按上下文应为 `O`（可选）——本表保持原样。
- **Type（访问）**：`R` 只读，`W` 只写，`R/W` 读写，`R/(W)` 表示写访问为可选/有条件（以各寄存器章节为准）。
- 所有寄存器 32 位对齐、大端（big-endian）；位编号采用 MSB-first（bit 0 = 最高位）。地址 0x0000–0x9FFF 保留给标准 bootstrap 寄存器，0xA000 起为厂商自定义空间（§36, p.246–247）。
- `SCEBAx + 0x....`：相对于第 x 个流通道的 *Stream Channel Extended Bootstrap Address* 寄存器（SCEBAx，地址 `0x0D3C + 0x40*x`）读出的基地址的偏移。
- Support 列脚注（PDF p.269）：
  1. `[1]` Mandatory only for transmitter, receiver and transceiver devices. Not applicable for peripherals.（仅对发送端/接收端/收发设备强制，外设不适用）
  2. `[2]` Mandatory only for transmitter devices. Not applicable for others.（仅对发送端设备强制）
  3. `[3]` Mandatory when the GVSP protocol is supported.（支持 GVSP 时强制）
  4. `[4]` Mandatory when the GVRSP protocol is supported.（支持 GVRSP 时强制）

共 223 行（含 Reserved 行）。

| 地址 Address | 名称 Name | Support | Type | 长度(B) | PDF 页 |
|---|---|---|---|---|---|
| `0x0000` | Version | M | R | 4 | 248 |
| `0x0004` | Device Mode | M | R | 4 | 248 |
| `0x0008` | Device MAC Address - High (Network interface #0) | M | R | 4 | 248 |
| `0x000C` | Device MAC Address - Low (Network interface #0) | M | R | 4 | 248 |
| `0x0010` | Network Interface Capability (Network interface #0) | M | R | 4 | 248 |
| `0x0014` | Network Interface Configuration (Network interface #0) | M | R/W | 4 | 249 |
| `0x0018` | Reserved | - | - | - | 249 |
| `0x0024` | Current IP Address (Network interface #0) | M | R | 4 | 249 |
| `0x0028` | Reserved | - | - | - | 249 |
| `0x0034` | Current Subnet Mask (Network interface #0) | M | R | 4 | 249 |
| `0x0038` | Reserved | - | - | - | 249 |
| `0x0044` | Current Default Gateway (Network interface #0) | M | R | 4 | 249 |
| `0x0048` | Manufacturer Name | M | R | 32 | 249 |
| `0x0068` | Model Name | M | R | 32 | 249 |
| `0x0088` | Device Version | M | R | 32 | 250 |
| `0x00A8` | Manufacturer Info | M | R | 48 | 250 |
| `0x00D8` | Serial Number | O | R | 16 | 250 |
| `0x00E8` | User-defined Name | O | R/W | 16 | 251 |
| `0x00F8` | Reserved | - | - | - | 251 |
| `0x0200` | First URL | M | R | 512 | 251 |
| `0x0400` | Second URL | M | R | 512 | 251 |
| `0x0600` | Number of Network Interfaces | M | R | 4 | 251 |
| `0x0604` | Reserved | - | - | - | 251 |
| `0x064C` | Persistent IP Address (Network interface #0) | O | R/W | 4 | 251 |
| `0x0650` | Reserved | - | - | - | 251 |
| `0x065C` | Persistent Subnet Mask (Network interface #0) | O | R/W | 4 | 251 |
| `0x0660` | Reserved | - | - | - | 251 |
| `0x066C` | Persistent Default Gateway (Network interface #0) | O | R/W | 4 | 252 |
| `0x0670` | Link Speed (Network interface #0) | O | R | 4 | 252 |
| `0x0674` | Reserved | - | - | - | 252 |
| `0x0680` | Device MAC Address - High (Network interface #1) | O | R | 4 | 252 |
| `0x0684` | Device MAC Address - Low (Network interface #1) | O | R | 4 | 252 |
| `0x0688` | Network Interface Capability (Network interface #1) | O | R | 4 | 252 |
| `0x068C` | Network Interface Configuration (Network interface #1) | O | R/W | 4 | 252 |
| `0x0690` | Reserved | - | - | - | 252 |
| `0x069C` | Current IP Address (Network interface #1) | O | R | 4 | 252 |
| `0x06A0` | Reserved | - | - | - | 252 |
| `0x06AC` | Current Subnet Mask (Network interface #1) | O | R | 4 | 252 |
| `0x06B0` | Reserved | - | - | - | 253 |
| `0x06BC` | Current Default Gateway (Network interface #1) | O | R | 4 | 253 |
| `0x06C0` | Reserved | - | - | - | 253 |
| `0x06CC` | Persistent IP Address (Network interface #1) | O | R/W | 4 | 253 |
| `0x06D0` | Reserved | - | - | - | 253 |
| `0x06DC` | Persistent Subnet Mask (Network interface #1) | O | R/W | 4 | 253 |
| `0x06E0` | Reserved | - | - | - | 253 |
| `0x06EC` | Persistent Default Gateway (Network interface #1) | O | R/W | 4 | 253 |
| `0x06F0` | Link Speed (Network interface #1) | O | R | 4 | 253 |
| `0x06F4` | Reserved | - | - | - | 253 |
| `0x0700` | Device MAC Address - High (Network interface #2) | O | R | 4 | 253 |
| `0x0704` | Device MAC Address - Low (Network interface #2) | O | R | 4 | 253 |
| `0x0708` | Network Interface Capability (Network interface #2) | O | R | 4 | 253 |
| `0x070C` | Network Interface Configuration (Network interface #2) | O | R/W | 4 | 254 |
| `0x0710` | Reserved | - | - | - | 254 |
| `0x071C` | Current IP Address (Network interface #2) | O | R | 4 | 254 |
| `0x0720` | Reserved | - | - | - | 254 |
| `0x072C` | Current Subnet Mask (Network interface #2) | O | R | 4 | 254 |
| `0x0730` | Reserved | - | - | - | 254 |
| `0x073C` | Current Default Gateway (Network interface #2) | O | R | 4 | 254 |
| `0x0740` | Reserved | - | - | - | 254 |
| `0x074C` | Persistent IP Address (Network interface #2) | O | R/W | 4 | 254 |
| `0x0750` | Reserved | - | - | - | 254 |
| `0x075C` | Persistent Subnet Mask (Network interface #2) | O | R/W | 4 | 254 |
| `0x0760` | Reserved | - | - | - | 254 |
| `0x076C` | Persistent Default Gateway (Network interface #2) | O | R/W | 4 | 254 |
| `0x0770` | Link Speed (Network interface #2) | O | R | 4 | 254 |
| `0x0774` | Reserved | - | - | - | 255 |
| `0x0780` | Device MAC Address - High (Network interface #3) | O | R | 4 | 255 |
| `0x0784` | Device MAC Address - Low (Network interface #3) | O | R | 4 | 255 |
| `0x0788` | Network Interface Capability (Network interface #3) | O | R | 4 | 255 |
| `0x078C` | Network Interface Configuration (Network interface #3) | O | R/W | 4 | 255 |
| `0x0790` | Reserved | - | - | - | 255 |
| `0x079C` | Current IP Address (Network interface #3) | O | R | 4 | 255 |
| `0x07A0` | Reserved | - | - | - | 255 |
| `0x07AC` | Current Subnet Mask (Network interface #3) | O | R | 4 | 255 |
| `0x07B0` | Reserved | - | - | - | 255 |
| `0x07BC` | Current Default Gateway (Network interface #3) | O | R | 4 | 255 |
| `0x07C0` | Reserved | - | - | - | 255 |
| `0x07CC` | Persistent IP Address (Network interface #3) | O | R/W | 4 | 256 |
| `0x07D0` | Reserved | - | - | - | 256 |
| `0x07DC` | Persistent Subnet Mask (Network interface #3) | O | R/W | 4 | 256 |
| `0x07E0` | Reserved | - | - | - | 256 |
| `0x07EC` | Persistent Default Gateway (Network interface #3) | O | R/W | 4 | 256 |
| `0x07F0` | Link Speed (Network interface #3) | O | R | 4 | 256 |
| `0x07F4` | Reserved | - | - | - | 256 |
| `0x0900` | Number of Message Channels | M | R | 4 | 256 |
| `0x0904` | Number of Stream Channels | M | R | 4 | 256 |
| `0x0908` | Number of Action Signals | O | R | 4 | 256 |
| `0x090C` | Action Device Key | O | W | 4 | 256 |
| `0x0910` | Number of Active Links | M | R | 4 | 256 |
| `0x0914` | Reserved | - | - | - | 256 |
| `0x092C` | Streaming Capability | M<sup>[1]</sup> | R | 4 | 257 |
| `0x0930` | Message Channel Capability | M | R | 4 | 257 |
| `0x0934` | GVCP Capability | M | R | 4 | 257 |
| `0x0938` | Heartbeat Timeout | M | R/W | 4 | 257 |
| `0x093C` | Timestamp Tick Frequency - High | O | R | 4 | 257 |
| `0x0940` | Timestamp Tick Frequency - Low | O | R | 4 | 257 |
| `0x0944` | Timestamp Control | O | W | 4 | 257 |
| `0x0948` | Timestamp Value - High | M | R | 4 | 257 |
| `0x094C` | Timestamp Value - Low | M | R | 4 | 257 |
| `0x0950` | Discovery ACK Delay | O | R/(W) | 4 | 257 |
| `0x0954` | GVCP Configuration | O | R/W | 4 | 258 |
| `0x0958` | Pending Timeout | M | R | 4 | 258 |
| `0x095C` | Control Switchover Key | O | W | 4 | 258 |
| `0x0960` | Streaming Configuration | M | R/W | 4 | 258 |
| `0x0964` | Physical Link Configuration Capability | M | R | 4 | 258 |
| `0x0968` | Physical Link Configuration | M | R/(W) | 4 | 258 |
| `0x096C` | IEEE 1588 Status | O | R | 4 | 258 |
| `0x0970` | Scheduled Action Command Queue Size | O | R | 4 | 258 |
| `0x0974` | IEEE 1588 Extended Capabilities | O | R | 4 | 258 |
| `0x0978` | IEEE 1588 Supported Profiles | O | R | 4 | 258 |
| `0x097C` | IEEE 1588 Selected Profile | O | R/W | 4 | 258 |
| `0x0980` | Reserved | - | - | - | 258 |
| `0x0A00` | Control Channel Privilege (CCP) | M | R/W | 4 | 258 |
| `0x0A04` | Primary Application Port | O | R | 4 | 258 |
| `0x0A08` | Reserved | - | - | - | 259 |
| `0x0A14` | Primary Application IP Address | O | R | 4 | 259 |
| `0x0A18` | Reserved | - | - | - | 259 |
| `0x0A24` | Control Channel Max Packet Size | O | R/W | 4 | 259 |
| `0x0A28` | Control Channel Test Region Address | O | R/W | 4 | 259 |
| `0x0A2C` | Reserved | - | - | - | 259 |
| `0x0B00` | Message Channel Port (MCP) | O | R/W | 4 | 259 |
| `0x0B04` | Reserved | - | - | - | 259 |
| `0x0B10` | Message Channel Destination Address (MCDA) | O | R/W | 4 | 259 |
| `0x0B14` | Message Channel Transmission Timeout (MCTT) | O | R/W | 4 | 259 |
| `0x0B18` | Message Channel Retry Count (MCRC) | O | R/W | 4 | 259 |
| `0x0B1C` | Message Channel Source Port (MCSP) | O | R | 4 | 259 |
| `0x0B20` | Message Channel Configuration (MCCFG) | O | R/W | 4 | 259 |
| `0x0B24` | Reserved | - | - | - | 259 |
| `0x0D00` | GVSP Stream Channel Port 0 (SCP0) | M<sup>[1]</sup><sup>[3]</sup> | R/W | 4 | 259 |
| `0x0D04` | GVSP Stream Channel Packet Size 0 (SCPS0) | M<sup>[1]</sup><sup>[3]</sup> | R/W | 4 | 260 |
| `0x0D08` | GVSP Stream Channel Packet Delay 0 (SCPD0) | M<sup>[2]</sup><sup>[3]</sup> | R/W | 4 | 260 |
| `0x0D0C` | Reserved | - | - | - | 260 |
| `0x0D18` | GVSP Stream Channel Destination Address 0 (SCDA0) | M<sup>[1]</sup><sup>[3]</sup> | R/W | 4 | 260 |
| `0x0D1C` | GVSP Stream Channel Source Port 0 (SCSP0) | O | R | 4 | 260 |
| `0x0D20` | GVSP Stream Channel Capability 0 (SCC0) | O | R | 4 | 260 |
| `0x0D24` | GVSP Stream Channel Configuration 0 (SCCFG0) | O | R/W | 4 | 260 |
| `0x0D28` | Deprecated | - | - | 8 | 260 |
| `0x0D30` | GVSP Stream Channel Max Packet Count (SCMPC0) | M<sup>[1]</sup><sup>[3]</sup> | R | 4 | 260 |
| `0x0D34` | GVSP Stream Channel Max Block Size (SCMBS0) - High | M<sup>[1]</sup><sup>[3]</sup> | R | 4 | 260 |
| `0x0D38` | GVSP Stream Channel Max Block Size (SCMBS0) - Low | M<sup>[1]</sup><sup>[3]</sup> | R | 4 | 260 |
| `0x0D3C` | Stream Channel Extended Bootstrap Address (SCEBA0) | M<sup>[1]</sup> | R | 4 | 261 |
| `SCEBA0 + 0x0000` | Stream Channel GenDC Descriptor Address (SCGDA0) | O | R | 4 | 261 |
| `SCEBA0 + 0x0004` | Stream Channel GenDC Descriptor Size (SCGDS0) | O | R | 4 | 261 |
| `SCEBA0 + 0x0008` | Stream Channel GenDC Flow Mapping Table Address (SCGFTA0) | O | R | 4 | 261 |
| `SCEBA0 + 0x000C` | Stream Channel GenDC Flow Mapping Table Size (SCGFTS0) | O | R | 4 | 261 |
| `SCEBA0 + 0x0010` | Reserved | O | R | 32 | 261 |
| `SCEBA0 + 0x0030` | GVRSP Stream Channel Control (RSCCTL0) | M<sup>[2]</sup><sup>[4]</sup> | R/W | 4 | 261 |
| `SCEBA0 + 0x0034` | GVRSP Stream Channel Buffer Layout Table Address (RSCBLTA0) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 261 |
| `SCEBA0 + 0x0038` | GVRSP Stream Channel Buffer Layout Table Size (RSCBLTS0) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 261 |
| `SCEBA0 + 0x003C` | GVRSP Stream Channel Credits Counter Maximum Supported Value (RSCCMAX0) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 262 |
| `SCEBA0 + 0x0040` | GVRSP Stream Channel Credits Counter Current Value (RSCCCNT0) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 262 |
| `SCEBA0 + 0x0044` | GVRSP Stream Channel Upstream Capability (RSCUCAP0) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 262 |
| `SCEBA0 + 0x0048` | GVRSP Stream Channel Upstream Maximum Supported Message Size (RSCUMAX0) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 262 |
| `SCEBA0 + 0x004C` | GVRSP Stream Channel QP Connection Key (RSCCKEY0) | M<sup>[2]</sup><sup>[4]</sup> | W | 4 | 262 |
| `0x0D40` | GVSP Stream Channel Port 1 (SCP1) | O | R/W | 4 | 262 |
| `0x0D44` | GVSP Stream Channel Packet Size 1 (SCPS1) | O | R/W | 4 | 262 |
| `0x0D48` | GVSP Stream Channel Packet Delay 1 (SCPD1) | O | R/W | 4 | 262 |
| `0x0D4C` | Reserved | - | - | - | 262 |
| `0x0D58` | GVSP Stream Channel Destination Address 1 (SCDA1) | O | R/W | 4 | 263 |
| `0x0D5C` | GVSP Stream Channel Source Port 1 (SCSP1) | O | R | 4 | 263 |
| `0x0D60` | GVSP Stream Channel Capability 1 (SCC1) | O | R | 4 | 263 |
| `0x0D64` | GVSP Stream Channel Configuration 1 (SCCFG1) | O | R/W | 4 | 263 |
| `0x0D68` | Reserved | - | - | - | 263 |
| `0x0D6C` | Reserved | - | - | - | 263 |
| `0x0D70` | GVSP Stream Channel Max Packet Count (SCMPC1) | 0 | R | 4 | 263 |
| `0x0D74` | GVSP Stream Channel Max Block Size (SCMBS1) - High | 0 | R | 4 | 263 |
| `0x0D78` | GVSP Stream Channel Max Block Size (SCMBS1) - Low | 0 | R | 4 | 263 |
| `0x0D7C` | Stream Channel Extended Bootstrap Address (SCEBA1) | 0 | R | 4 | 263 |
| `SCEBA1 + 0x0000` | Stream Channel GenDC Descriptor Address (SCGDA1) | O | R | 4 | 263 |
| `SCEBA1 + 0x0004` | Stream Channel GenDC Descriptor Size (SCGDS1) | O | R | 4 | 263 |
| `SCEBA1 + 0x0008` | Stream Channel GenDC Flow Mapping Table Address (SCGFTA1) | O | R | 4 | 264 |
| `SCEBA1 + 0x000C` | Stream Channel GenDC Flow Mapping Table Size (SCGFTS1) | O | R | 4 | 264 |
| `SCEBA1 + 0x0010` | Reserved | O | R | 32 | 264 |
| `SCEBA1 + 0x0030` | GVRSP Stream Channel Control (RSCCTL1) | M<sup>[2]</sup><sup>[4]</sup> | R/W | 4 | 264 |
| `SCEBA1 + 0x0034` | GVRSP Stream Channel Buffer Layout Table Address (RSCBLTA1) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 264 |
| `SCEBA1 + 0x0038` | GVRSP Stream Channel Buffer Layout Table Size (RSCBLTS1) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 264 |
| `SCEBA1 + 0x003C` | GVRSP Stream Channel Credits Counter Maximum Supported Value (RSCCMAX1) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 264 |
| `SCEBA1 + 0x0040` | GVRSP Stream Channel Credits Counter Current Value (RSCCCNT1) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 264 |
| `SCEBA1 + 0x0044` | GVRSP Stream Channel Upstream Capability (RSCUCAP1) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 264 |
| `SCEBA1 + 0x0048` | GVRSP Stream Channel Upstream Maximum Supported Message Size (RSCUMAX1) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 265 |
| `SCEBA1 + 0x004C` | GVRSP Stream Channel QP Connection Key (RSCCKEY1) | M<sup>[2]</sup><sup>[4]</sup> | W | 4 | 265 |
| `0x0D80` | Other stream channels registers | O | - | - | 265 |
| `0x8CC0` | GVSP Stream Channel Port 511 (SCP511) | O | R/W | 4 | 265 |
| `0x8CC4` | GVSP Stream Channel Packet Size 511 (SCPS511) | O | R/W | 4 | 265 |
| `0x8CC8` | GVSP Stream Channel Packet Delay 511 (SCPD511) | O | R/W | 4 | 265 |
| `0x8CCC` | Reserved | - | - | - | 265 |
| `0x8CD8` | GVSP Stream Channel Destination Address 511 (SCDA511) | O | R/W | 4 | 265 |
| `0x8CDC` | GVSP Stream Channel Source Port 511 (SCSP511) | O | R | 4 | 265 |
| `0x8CE0` | GVSP Stream Channel Capability 511 (SCC511) | O | R | 4 | 265 |
| `0x8CE4` | GVSP Stream Channel Configuration 511 (SCCFG511) | O | R/W | 4 | 266 |
| `SCEBA511 + 0x0010` | Reserved | O | R | 32 | 266 |
| `SCEBA511 + 0x0030` | GVRSP Stream Channel Control (RSCCTL511) | M<sup>[2]</sup><sup>[4]</sup> | R/W | 4 | 266 |
| `SCEBA511 + 0x0034` | GVRSP Stream Channel Buffer Layout Table Address (RSCBLTA511) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 266 |
| `SCEBA511 + 0x0038` | GVRSP Stream Channel Buffer Layout Table Size (RSCBLTS511) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 266 |
| `SCEBA511 + 0x003C` | GVRSP Stream Channel Credits Counter Maximum Supported Value (RSCCMAX511) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 266 |
| `SCEBA511 + 0x0040` | GVRSP Stream Channel Credits Counter Current Value (RSCCCNT511) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 266 |
| `SCEBA511 + 0x0044` | GVRSP Stream Channel Upstream Capability (RSCUCAP511) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 266 |
| `SCEBA511 + 0x0048` | GVRSP Stream Channel Upstream Maximum Supported Message Size (RSCUMAX511) | M<sup>[2]</sup><sup>[4]</sup> | R | 4 | 267 |
| `SCEBA511 + 0x004C` | GVRSP Stream Channel QP Connection Key (RSCCKEY511) | M<sup>[2]</sup><sup>[4]</sup> | W | 4 | 267 |
| `0x8CE8` | Reserved | - | - | - | 267 |
| `0x8CEC` | Reserved | - | - | - | 267 |
| `0x8CF0` | GVSP Stream Channel Max Packet Count (SCMPC511) | 0 | R | 4 | 267 |
| `0x8CF4` | GVSP Stream Channel Max Block Size (SCMBS511) - High | 0 | R | 4 | 267 |
| `0x8CF8` | GVSP Stream Channel Max Block Size (SCMBS511) - Low | 0 | R | 4 | 267 |
| `0x8CFC` | Stream Channel Extended Bootstrap Address (SCEBA511) | 0 | R | 4 | 267 |
| `SCEBA511 + 0x0000` | Stream Channel GenDC Descriptor Address (SCGDA511) | O | R | 4 | 267 |
| `SCEBA511 + 0x0004` | Stream Channel GenDC Descriptor Size (SCGDS511) | O | R | 4 | 267 |
| `SCEBA511 + 0x0008` | Stream Channel GenDC Flow Mapping Table Address (SCGFTA511) | O | R | 4 | 268 |
| `SCEBA511 + 0x000C` | Stream Channel GenDC Flow Mapping Table Size (SCGFTS511) | O | R | 4 | 268 |
| `0x8CF0` | Reserved | - | - | - | 268 |
| `0x9000` | Manifest Table | O | R | 512 | 268 |
| `0x9200` | Reserved | - | - | - | 268 |
| `0x9800` | Action Group Key 0 | O | R/W | 4 | 268 |
| `0x9804` | Action Group Mask 0 | O | R/W | 4 | 268 |
| `0x9808` | Reserved | - | - | - | 268 |
| `0x9810` | Action Group Key 1 | O | R/W | 4 | 268 |
| `0x9814` | Action Group Mask 1 | O | R/W | 4 | 268 |
| `0x9818` | Reserved | - | - | - | 268 |
| `0x9820` | Other action signals registers | O | R/W | - | 268 |
| `0x9FF0` | Action Group Key 127 | O | R/W | 4 | 268 |
| `0x9FF4` | Action Group Mask 127 | O | R/W | 4 | 268 |
| `0x9FF8` | Reserved | - | - | - | 268 |
| `0xA000` | Start of manufacturer-specific register space | - | - | - | 269 |
