# 单片机USB实现虚拟U盘SCSI接口协议笔记

## 一、概述

USB Mass Storage Class（MSC，大容量存储类）是实现虚拟U盘的核心协议。该协议使单片机能够被主机（PC）识别为标准USB存储设备，从而通过USB接口进行数据的读写操作。

MSC协议基于Bulk-Only Transport（BOT，仅批量传输）方式，使用SCSI透明命令集（SCSI Transparent Command Set）进行数据交换。BOT是目前最普遍使用的方式，其特点是系统通过默认管道（地址0、端点0）枚举完成后，所有数据/命令/状态均仅通过批量端点进行传输。

**核心组成**：

- **USB设备枚举**：主机获取设备描述符，识别为MSC设备
- **BOT协议**：定义CBW/CSW/Data三段式传输结构
- **SCSI命令集**：主机通过CBW发送SCSI命令，单片机执行并返回CSW状态

**关键参考文档**：
- 《Universal Serial Bus Mass Storage Class Bulk-Only Transport Revision 1.0》
- 《Universal Serial Bus Mass Storage Class Specification Overview》
- 《SCSI Primary Commands - 3 (SPC-3)》
- 《SCSI Block Commands (SBC-2)》（其中TYPE为“M”的命令是SCSI设备必须实现的）


## 二、USB描述符配置

### 2.1 设备描述符

MSC规范没有定义特定于类的描述符，使用标准描述符即可。设备描述符中的关键设置：

- `bDeviceClass`、`bDeviceSubClass`、`bDeviceProtocol`：统一设置为0，表示在接口描述符中指定设备类和子类
- `iSerialNumber`：必须包含序列号字符串描述符。序列号应至少包含12位有效数字（UNICODE编码），用于主机生成全局唯一标识符（GUID）

### 2.2 接口描述符

接口描述符是MSC配置的核心，关键字段如下：

| 字段 | 值 | 说明 |
|------|-----|------|
| `bInterfaceClass` | 0x08 | Mass Storage Class |
| `bInterfaceSubClass` | 0x06 | SCSI Transparent命令集 |
| `bInterfaceProtocol` | 0x50 | Bulk-Only Transport（BOT） |

CBI（Control/Bulk/Interrupt）传输方式仅能用于全速软盘驱动，不得用于高速设备或软盘驱动器以外的设备，不建议在任何新设计中采用。

### 2.3 端点描述符

MSC设备应支持至少三个端点：控制端点（默认）、批量输入端点（Bulk-In）和批量输出端点（Bulk-Out）：

- **Bulk-In端点**：用于向主机发送数据（Data-In）和返回CSW状态包
- **Bulk-Out端点**：用于接收主机发送的CBW命令包和数据（Data-Out）

控制端点由USB协议自带上无需描述，只需添加两个批量端点的描述符。

### 2.4 字符串描述符

设备描述符的`iSerialNumber`字段指向包含序列号的字符串描述符。序列号字符串的格式为12位数字（每个数字对应一个字节），使用UNICODE编码，只能包含字符'0'~'9'和'A'~'F'。


## 三、类相关请求

MSC在控制端点添加了两个类特定请求，用于设备复位和逻辑单元数量查询。

### 3.1 Bulk-Only Mass Storage Reset（0xFF）

此请求用于重置大容量存储设备及其相关接口，为接收下一个CBW做准备。

**请求格式**：
- `bmRequestType`：Class, Interface, Host to device
- `bRequest`：0xFF
- `wValue`：0
- `wIndex`：接口号
- `wLength`：0

收到此请求后，设备应复位Bulk-In和Bulk-Out端点，清除STALL状态，准备接收下一个CBW。

### 3.2 Get Max LUN（0xFE）

用于获取设备支持的逻辑单元（Logical Unit Number）数量。

**请求格式**：
- `bmRequestType`：Class, Interface, device to host
- `bRequest`：0xFE
- `wValue`：0
- `wIndex`：接口号
- `wLength`：1（返回1字节数据）

返回值为设备支持的最大LUN编号。0表示支持1个逻辑单元（LUN0），1表示支持2个逻辑单元（LUN0和LUN1）。


## 四、BOT协议——CBW/CSW详解

BOT协议定义了三种类型的数据在USB主机和设备之间传送：**CBW**（Command Block Wrapper，命令块包）、**CSW**（Command Status Wrapper，状态包）和普通数据。

### 4.1 基本传输流程

BOT传输基于一个简单的状态机，始于空闲状态：

1. **命令阶段**：主机通过Bulk-Out端点发送31字节的CBW包
2. **数据阶段**（可选）：根据CBW中的方向标志进行Data-In或Data-Out传输
3. **状态阶段**：设备通过Bulk-In端点返回13字节的CSW包

对于无数据传输的命令（如TEST UNIT READY），数据阶段被跳过，设备在收到CBW后直接返回CSW。

### 4.2 CBW（Command Block Wrapper）结构

CBW是从USB主机发送到设备的命令包，固定31字节。注意：所有多字节字段均使用**小端格式**（Little Endian）。

| 偏移 | 长度 | 字段名 | 说明 |
|------|------|--------|------|
| 0-3 | 4 | dCBWSignature | 固定值0x43425355，标识这是一个CBW |
| 4-7 | 4 | dCBWTag | 主机发送的命令块标签，设备须在CSW中原样回传 |
| 8-11 | 4 | dCBWDataTransferLength | 数据阶段要传输的字节数（0表示无数据阶段） |
| 12 | 1 | bmCBWFlags | Bit7=方向（0=Data-Out，1=Data-In）；Bits6-0保留为0 |
| 13 | 1 | bCBWLUN | 逻辑单元号（多数设备设为0） |
| 14 | 1 | bCBWCBLength | CBWCB有效长度（1~16字节） |
| 15-30 | 16 | CBWCB | SCSI命令块（Command Block），存放SCSI命令 |

### 4.3 CSW（Command Status Wrapper）结构

CSW是设备返回给主机的状态包，固定13字节。

| 偏移 | 长度 | 字段名 | 说明 |
|------|------|--------|------|
| 0-3 | 4 | dCSWSignature | 固定值0x53425355，标识这是一个CSW |
| 4-7 | 4 | dCSWTag | 原样回传CBW中的dCBWTag值 |
| 8-11 | 4 | dCSWDataResidue | CBW中dCBWDataTransferLength与实际传输数据的差值 |
| 12 | 1 | bCSWStatus | 命令执行状态（0=成功，1=失败，2=相位错误） |

**状态值说明**：
- 0x00（Good）：命令执行成功
- 0x01（Fail）：命令执行失败，需返回CHECK CONDITION，主机将发送REQUEST SENSE获取详细信息
- 0x02（Phase Error）：相位错误，命令阶段或数据阶段发生了协议违规，设备进入错误恢复状态


## 五、必备SCSI命令详解

SCSI命令以操作码（Operation Code）标识，嵌入在CBW的CBWCB字段中。以下为虚拟U盘必须实现的命令。

### 5.1 INQUIRY（0x12）

**功能**：请求设备将自身参数信息（制造商ID、产品ID、版本等）发送给主机。主机通常在设备通电或复位后使用此命令询问设备配置。

**CBWCB格式**（12字节）：
- Byte 0：0x12（操作码）
- Byte 1：EVPD=0，LUN编号
- Byte 2：Page Code=0
- Byte 3：Reserved
- Byte 4：Allocation Length（请求返回的最大字节数，通常为36）
- Byte 5：Control（一般0）

**返回数据格式**（36字节）：
| 偏移 | 长度 | 字段 |
|------|------|------|
| 0 | 1 | Peripheral Device Type（00h=直接访问设备） |
| 1 | 1 | RMB（Bit7=1表示可移动介质） |
| 2 | 1 | ISO/ECMA（固定0） |
| 3 | 1 | Response Data Format（固定0x01或0x02） |
| 4 | 1 | Additional Length（后续数据长度，设为0x1F即31） |
| 5-7 | 3 | Reserved |
| 8-15 | 8 | Vendor Identification（ASCII，8字符） |
| 16-31 | 16 | Product Identification（ASCII，16字符） |
| 32-35 | 4 | Product Revision Level（ASCII，4字符） |

Peripheral Device Type：00h表示直接访问设备（如U盘），1Fh表示无连接设备。RMB（Removable Media Bit）设为1表示介质可移除。返回的全部36字节必须填充完整，未定义的字节设为0x20（空格）。

### 5.2 READ CAPACITY（0x25）

**功能**：请求获取当前存储介质的容量信息。

**CBWCB格式**（10字节）：
- Byte 0：0x25
- Byte 1：LUN
- Byte 2-5：Logical Block Address（设为0）
- Byte 6-8：Reserved
- Byte 8：PMI（设为1）
- Byte 9：Control（0）

**返回数据格式**（8字节）：
| 偏移 | 内容 |
|------|------|
| 0-3 | Last Logical Block Address（最后一个LBA的地址，即扇区总数-1） |
| 4-7 | Block Length In Bytes（每个扇区的字节数，通常为512） |

示例：总容量为1GB（约2,097,152个扇区，512字节/扇区）：
- Last LBA = 0x001FFFFF（2,097,151）
- Block Length = 0x00000200（512）

返回的Block Length必须与底层存储介质的物理扇区大小一致。如非512对齐，主机的文件系统可能无法正常格式化。


### 5.3 REQUEST SENSE（0x03）

**功能**：当命令执行失败时，主机发送此命令获取详细的错误信息（Sense Key、Additional Sense Code等）。

**CBWCB格式**（6字节）：
- Byte 0：0x03
- Byte 1：LUN
- Byte 2：Reserved
- Byte 3：Reserved
- Byte 4：Allocation Length（请求返回字节数，通常18）
- Byte 5：Control

**返回数据格式**（18字节）：
| 偏移 | 字段 |
|------|------|
| 0 | Response Code（0x70或0x71） |
| 1 | Valid（Bit7）/ 保留 |
| 2 | Sense Key |
| 3-6 | Information |
| 7 | Additional Sense Length（18-7=11） |
| 8-11 | Command Specific Information |
| 12 | Additional Sense Code（ASC） |
| 13 | Additional Sense Code Qualifier（ASCQ） |
| 14-17 | Reserved |

**常用Sense Key**：
- 0x00：NO SENSE
- 0x02：NOT READY
- 0x03：MEDIUM ERROR
- 0x05：ILLEGAL REQUEST
- 0x06：UNIT ATTENTION

### 5.4 MODE SENSE(6)（0x1A）

**功能**：获取设备的模式参数，包括写保护状态、介质类型等信息。

**CBWCB格式**（6字节）：
- Byte 0：0x1A
- Byte 1：LUN + DBD
- Byte 2：Page Code
- Byte 3：Subpage Code
- Byte 4：Allocation Length（请求返回字节数）
- Byte 5：Control

**推荐实现**：
至少支持返回0x3F（返回所有页）模式参数。写保护位（WP）必须根据实际硬件支持设置（支持写操作设为0，只读设为1）。若误启WP=1（实际可写），主机格式化操作会被阻止。



### 5.5 TEST UNIT READY（0x00）

**功能**：检查设备是否准备就绪。这是一个无数据传输的命令，通过CSW返回状态来判断是否就绪。

**CBWCB格式**（6字节）：
- Byte 0：0x00
- Byte 1：LUN
- Byte 2-4：Reserved（0）
- Byte 5：Control（0）

**处理逻辑**：
- 设备就绪 → 返回CSW状态=0x00（Good）
- 设备未就绪 → 返回CSW状态=0x01（Fail），sense key设为NOT READY

主机使用此命令轮询设备状态，直到设备准备就绪，而无需为返回数据分配空间。特别适用于检查介质状态变化（如设备热插拔）。


### 5.6 READ(10)（0x28）

**功能**：从指定LBA地址读取指定数量的扇区数据，通过Bulk-In端点发送给主机。

**CBWCB格式**（10字节）：
- Byte 0：0x28
- Byte 1：LUN + DPO/FUA标志
- Byte 2-5：Logical Block Address（起始扇区地址）
- Byte 6：Reserved
- Byte 7-8：Transfer Length（传输扇区数）
- Byte 9：Control（0）

**数据处理**：
1. 解析CBWCB中的LBA（地址）和Transfer Length（扇区数）
2. 从存储介质读取对应扇区数据（每个扇区512字节）
3. 通过Bulk-In端点分批次传输，每次发送最大包长（64字节/512字节）
4. 传输完成后返回CSW状态

DPO和FUA位通常设为0，FUA=1时要求直接从介质读取（绕过缓存）。

**返回CSW时的Residue计算**：
- 实际传输字节数 = Transfer Length × 512
- dCSWDataResidue = dCBWDataTransferLength - 实际传输字节数
- 正常传输完成后Residue应为0

### 5.7 WRITE(10)（0x2A）

**功能**：从主机接收指定扇区的数据并写入存储介质。

**CBWCB格式**（10字节）：
- Byte 0：0x2A
- Byte 1：LUN + DPO/FUA标志
- Byte 2-5：Logical Block Address（目标扇区地址）
- Byte 6：Reserved
- Byte 7-8：Transfer Length（写入扇区数）
- Byte 9：Control（0）

**数据处理**：
1. 解析CBWCB中的LBA和扇区数
2. 通过Bulk-Out端点接收数据（每个扇区512字节）
3. 将数据写入存储介质
4. 写入完成后返回CSW状态

注意：WRITE(10)命令对应bmCBWFlags的方向位为0（Data-Out），传输方向是从主机到设备。若主机请求的扇区数超出实际存储范围，应返回CHECK CONDITION状态。

