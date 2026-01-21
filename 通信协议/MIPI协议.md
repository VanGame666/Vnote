# MIPI DSI 驱动原理浅析

## 一、核心思想

**核心思想：**MIPI DSI驱动屏幕的本质是CPU/GPU生成的图像数据，通过一系列的硬件模块和软件协议栈，最终以高速串行差分信号的形式传输到屏幕的驱动IC上，驱动IC在控制液晶屏幕/OLED像素点。

**核心脉络流程:**

应用层/显示服务 -> 显示框架 (DRM/KMS)->RK3588 VOP/DSI驱动 -> MIPI DSI驱动引擎（硬件引擎）->MIPI D-PHY(物理层) ->屏幕驱动IC（TCON）->屏幕面板

## 二、参考文档说明

**参考文档：**开发板光盘A盘-基础资料\06、参考资料\MIPI DSI协议文档

 

| 文件名                                                       | 概略说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 《MIPI Alliance Standard for Display Bus Interface（V2.0）.pdf》 | **作用**：定义整个显示总线接口的**系统级架构和要求**。<br /> **说明：**  是 DSI、DPI、DBI 等显示接口的“上层指导标准”。 不涉及具体协议细节，而是定义接口目标、系统模型、功耗管理、兼容性等。 适合系统架构师阅读。 |
| 《 MIPI Alliance Standard for Display Pixel Interface（DPI-2）.pdf》 | **作用**：定义**并行像素接口（DPI）** 的电气和时序规范。 <br />**说明：**  DPI 是传统并行 RGB 接口，用于连接处理器与显示屏（如 TFT）。 DPI-2 是其升级版，支持更高分辨率和时钟频率。 与 DSI 不同，DPI 是并行、无压缩、无包结构的“裸送像素”接口。 用于老式或低成本显示屏连接。 |
| 《MIPI DSI Specification_v01-00-00.pdf》《MIPI DSI Specification_v01-00-0.pdf》《MIPI DSI Specification_v01-10-00.pdf》《MIPI DSI Specification_v01-20-00.pdf》《MIPI DSI Specification_v1-2.pdf》《MIPI DSI Specification_v1-3.pdf》 | **作用**：定义**串行显示接口协议**，用于处理器 → 显示屏的数据传输。<br /> **说明：** <br />v01-xx 是早期草案或初始版本，功能较基础。<br />v1.2 → 增加了更多像素格式、命令模式增强、ECC 校验等。<br />v1.3 → 支持更高带宽、动态刷新率（如 Adaptive Sync）、低功耗增强、多屏控制等。<br /> DSI 支持两种模式： <br /> **Video Mode**：持续传输像素流（类似 HDMI）<br /> **Command Mode**：按需发送图像块 + 命令（适合 OLED） 是目前手机、平板、车载屏最主流的接口协议。 |
| 《MIPI_CSI-2_Specification_Brief.pdf》                       | **作用**：CSI-2 协议的**快速概览文档**。 <br />**说明**：适合初学者快速了解 CSI-2 的数据包结构、通道、同步机制等。 |
| 《MIPI_CSI-2_specification_v1-3.pdf》<br />《mipi_CSI-2_specification_v2-1-2018.pdf》 | **作用**：定义**摄像头传感器 → 处理器** 的高速串行图像传输协议。 **说明：**  v1.3 → 基础功能：支持 RAW8/10/12、YUV、RGB、多通道、ECC、CRC。 v2.1 → 新增功能：更高带宽、虚拟通道扩展、更灵活的包头、支持压缩、多摄像头同步等。 与 DSI 共享 D-PHY/C-PHY 物理层。 广泛用于手机摄像头、安防、车载环视等。 |
| 《MIPI D-PHY_protocol_fundamentals.pdf》                     | 作用：D-PHY 的基础原理介绍（非完整规范）。<br/>说明：讲解 HS（高速）模式、LP（低功耗）模式、时钟嵌入、状态机等，适合入门学习。 |
| 《MIPI_D-PHY_Specification_v1-0.pdf》《MIPI_D-PHY_Specification_v1-1.pdf》《MIPI_D-PHY_Specification_v1-2.pdf》《mipi_D-PHY_specification_v2-1-er01.pdf》 | **作用**：定义**物理层电气特性、时序、信号结构、状态转换**。<br /> 说明：  v1.0 → 基础版本，最大速率 1 Gbps/lane v1.2 → 支持 1.5 Gbps，优化抖动和功耗 v2.1 → 支持 2.5+ Gbps，增强抗噪、支持更长走线、更灵活的时钟方案 是 DSI 和 CSI-2 的底层传输基础。 硬件工程师、PCB 设计师、FPGA 开发者必备。 |
| 《MIPI_D-PHY_Conformance_Test_Suite.pdf》                    | **作用**：提供**一致性测试用例和方法**，用于验证芯片/模组是否符合 D-PHY 标准。 <br />**说明：**  包含测试项：眼图、抖动、上升/下降时间、LP-HS 切换时序等。 用于实验室认证或芯片出厂前测试。 |
| 《Specification for Display Command Set (DCS).pdf》          | **作用**：定义**通用显示控制命令集**，供 DSI Command Mode 使用。 说明：  包含标准命令如：  `0x29` → Display On `0x11` → Sleep Out `0x3A` → 设置像素格式 `0x2A/2B` → 设置行列地址 屏幕驱动开发必备，所有 MIPI 屏幕都支持部分 DCS 命令。 与厂商私有命令（如初始化序列）配合使用。 |
| 《**MIPI__Protocol_Introduction**.pdf》                      | **作用**：MIPI 协议体系的**入门级 PPT 或培训材料**。 说明：  图文并茂，介绍 DSI、CSI、D-PHY、C-PHY 等基本概念。 适合新人、项目经理、跨部门沟通使用。 |
|                                                              |                                                              |
|                                                              |                                                              |

## 三、MIPI DSI概述

### 1、MIPI DSI 协议综述

DSI 全称是Display Serial Interface，是主控和显示模组之间的串行连接接口，下图展示了主控和屏幕之间的连接方式：

![1757450026189](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757450026189.png)

MIPI DSI 接口分为数据线和时钟线，均为差分信号。数据线可选择 1/2/3/4 lanes，时钟线有一对，最多 10 根线。MIPI DSI 以串行的方式发送指令和数据给屏幕，也可以读取屏幕中的信息。很明显，如果屏幕的分辨率和帧率越高，需要的带宽就越大，就需要更多的数据线来传输图像数据，RK3588 默认使用 4 lanes 来驱动 MIPI 屏幕，当然了，对于小尺寸的屏幕，也可以使用 2 lanes 来驱动。对于 MIPI DSI 接口而言，最常用的就是 2 lanes 和 4 lanes。

### 2、MIPI DSI 分层

![1757450146567](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757450146567.png)

从图 22.2-2 可以看出，MIPI DSI 一共有四层，从上往下依次为：应用层、协议层、通道管理层、物理层

#### （1）应用层

​	应用层处理更高层次的编码，将要显示的数据打包进数据流中，下层会处理并发送应用层的数据流。发送端将命令以及数据编码正 MIPI DSI 规格的格式，接收端则将接收到的数据还原为原始的数据。

#### （2）协议层

​	写一层主要是打包数据，在原始的数据包上ECC和校验和等东西。应用层传递下来的数据会打包成两种格式的数据：长数据包和短数据包，关于长短数据包后面会有详细讲解。发送端将原始数据打包好，添加包头和包尾，然后将打包好的数据发送给下层。接收端介绍到下层传来的数据包以后执行相反的操作，去除包头和包围，然后使用 ECC 进行校验接收到的数据，如果没问题就将解包后的原始数据交给应用层。

#### （3）链路层

​	链路层负责如何将数据分配到具体的通道上，MIPI DSI 可以支持 1/2/3/4 Lane，采用几通道取决于你的实际应用，如果带宽需求低，那么 2 Lane 就够了，带宽高的话就要 4 Lane。协议层下来的数据包都是串行的，如果只有 1 Lane 的话，那就直接使用这 1 Lane 将数据串行的发送出去，如果是 2/4 Lane 的话数据该如何发送呢？如图 22.2-3 所示：

![1757450858843](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757450858843.png)

图 22.2-4 中上部分就是整数倍传输，2 条通道一起结束，进入 EoT 模式。下图是非整数倍传输，其中 Lane 1 先传输完，所以 Lane 1 先进入 EoT 模式。同理，3/4 Lane 也一样。在接收端执行相反的操作，将 Lane 上的数据整理打包成串行数据上报给上层，如图22.2-5 所示：

![1757451003208](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757451003208.png)

#### （4）物理层

​	物理层就是最底层了，完成 MIPI DSI 数据在具体电路上的发送与接收，与物理层紧密相关的就是 D-PHY。物理层规定了 MIPI DSI 的整个电气属性，信号传输的时候电压等，关于物理层后面会详细讲解。

### 3、MIPI DSI 物理层和 D-PHY

1. 物理连接：
   - 屏幕通过柔性电路板（FPC）连接到主控板（RK3588开发板）。
   - 连接线包含几对差分信号线（Dp/Dn）构成的lanes（通道），用于高速数据的传输（HS mode）。常见的配置有 1 Lane, 2 Lanes, 4 Lanes，Lane 越多带宽越高。
   - 通常还有一对差分时钟线（CLKp/CLKn）
   - 一条低功耗双向信号线（LPX）,用于低速通信和状态控制（LP Mode）。
   - 一条TE(Tearing Effect)信号线（可选），用于屏幕的帧同步。
   - 电源（VCC）、地线（GND）、复位线（RESET），背光控制（BL_EN,BL_PWM）等。
2. 工作模式：
   - **HS (High-Speed) Mode：**:主模式，用于传输大量像素数据。电压摆幅小 (约 200mV)，差分信号，频率非常高 (可达 1.5Gbps+ per Lane)。**RK3588 D-PHY** 支持高速模式，其性能决定了最大分辨率和刷新率。
   - **LP (Low-Power) Mode：**用于传输控制命令、状态读取、初始化序列、帧同步间隙。电压摆幅大 (约 1.2V)，单端信号，速度慢 (通常 10-100 Mbps)。在 HS 数据传输间隙和屏幕待机时使用。`LPX` 线主要工作在此模式。
3. RK3588 的D-PHY:
   - rk3588内部集成MIPI D-PHY物理层控制器，负责将我们的DSI控制器输出的数字信号转换成符合MIPI D-PHY规范的差分模拟信号（HS mode）或者单端信号（LP Mode）。
   - 它需要精确的时钟源 (`vpll` 或 `cpll` 提供的高速参考时钟) 来驱动 HS 时钟。
   - 驱动需要配置 D-PHY 的参数：Lane数量、HS时钟频率、LP时钟频率、时序参数（如`THS-PREPARE`，`THS-ZERO`, `THS-TRAIL`, `TTA-GO`, `TTA-SURE`, `TWAKEUP` 等），这些参数通常是根据屏幕的规格书和设备树配置的。

## 四、video 和 command 模式

### 1、command 模式

![1757462310419](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757462310419.png)

主控芯片控制显示屏进行显示的模式。

**工作方式：**

1. 控制器向显示面板发送指令（command）和数据（Data）.
2. 指令告诉面板做什么？（例如：设置显示区域、进入睡眠模式、设置像素格式等。）
3. 数据通常是要显示的内容或配置参数。
4. 关键点：显示面板内部集成了GRAM(**Graphics RAM**)或者Frame Buffer。当控制器发送图像数据（如 Write Memory 命令 + 像素数据）时，数据就会被写入显示面板内部的GRAM.
5. 面板的时序控制器（TCON）负责**独立地、持续地**从自己的 GRAM 中读取数据，转换成驱动像素所需的信号，并刷新屏幕。控制器在发送完一帧数据后，可以进入低功耗状态。

数据传输的特点：

- 通常需要双向通信（控制器发送命令和数据，面板可能返回状态）。
- 接口：CSX(片选)，D/CX(数据/命令选择)，WRX(写选通)，DRX（读选通），D[0:n](数据线)。
- 代表接口：**DBI (Display Bus Interface)**，如 SPI, MCU 8/9/16/18-bit 并口 (Intel 8080 / Moto 6800), MIPI DBI Type A/B/C (基于 DBI 物理层)。

优点：

- 控制器功耗低（发送完数据即可休眠）。
- 对控制器带宽要求相对较低（只需要在图像更新时传输变化部分或整帧）。
- 适合**低分辨率**、**低刷新率**或**静态/缓慢变化**内容的屏幕（如智能手表、低端手机副屏、小尺寸工控屏）。

缺点：

- 面板成本较高（需要集成 GRAM 和更复杂的 TCON）。
- 高分辨率、高刷新率下，写入 GRAM 的带宽可能成为瓶颈。
- 通常难以实现非常高的刷新率（如 >90Hz）和分辨率（如 >FHD）。

### 2 、Video 模式

![1757463468781](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757463468781.png)

控制器充当实时视频流的提供者，面板充当播放器。

工作方式：

1. 控制器持续的、实时地向面板发送像素数据流。
2. 发送过程严格遵循**行同步（HSYNC / Hsync）** 和 **场同步（VSYNC / Vsync）** 信号的时序。
3. 面板内部**没有 GRAM 或 Frame Buffer**（或只有很小的行缓冲 Line Buffer）。它接收到的像素数据**立即**被处理和显示到屏幕上。
4. 控制器必须**连续不断地**提供像素数据流以维持屏幕显示。

数据传输特点：

- 通常是**单向**（控制器到面板）。
- 接口包含：`HSYNC`, `VSYNC`, `DE` (Data Enable, 可选但常用), `D[0:n]` (数据线), `CLK` (像素时钟)。
- 代表接口：**DPI (Display Pixel Interface)**, 也称为 **RGB 接口**。MIPI DPI 是这种模式的一种标准实现。

优点：

- 面板成本较低（省去了 GRAM）。 可以实现**高分辨率**、**高刷新率**（如 4K@120Hz）和**低延迟**。
- 图像质量通常更好（避免 GRAM 读写带来的潜在问题）。

缺点：

- 控制器功耗高（必须持续工作输出像素流）。
- 对控制器带宽要求极高（带宽 = 分辨率 * 刷新率 * 每像素比特数）。
- 需要专用的高速数据传输接口。

## 五、DBI 和 DPI 格式

这是对 **四、 中两种模式所使用的具体物理接口协议/标准** 的细化。

### DBI接口（Display Bus Interface）

![1757507856382](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757507856382.png)

- 传输命令（控制面板行为）。

- 传输数据（写入GRAM的像素数据或配置寄存器值）。

- 可能读取面板的状态。

  

- **物理层变体（常见）：**
  - **SPI（Serial Periphreal interface）：**串行，引脚少（3~4线），速度慢，用于极小屏幕。
  - **MCU并行接口：**
    - **Inter 8080 模式：**CSX(片选)，D/CX(数据/命令)，WRX(写)，WRX(读)，D[0:7]或D[8:15]数据
    - **Motorola 6800模式：**CSX，RS(寄存器选择，类似D/CX),E(使能)，R/WX(读写)，D[0:7]或D[0:15]。E信号有效期间完成读写。
  - **MIPI DBI:**MIPI联盟标准化DBI,可以分为TypeA,B,C（主要的区别在物理引脚定义和时序）。

- **关键点：**DBI实现了Command模式的数据传输机制。

### DPI 接口（Display Pixel Interface）

**定义：**用于**Vedio模式**显示的像素接口标准。它定义了如何将连续的像素数据流以及必要的同步信号传输到GRAM的显示面板上。
![1757510272851](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757510272851.png)

核心功能：

- 传输连续的像素数据流（D[0:n]）。
- 提供精确的时序控制信号（HSYNC，VSYNC,CLK）。
- 提供数据有效信号（DE）来简化设计（代替或辅助HSYNC/VSYNC）。

物理特点：

- 并行传输数据（数据线带宽常见16，18，24bit）。
- 专用的同步信号线（HSYNC，VSYNC）。
- 像素时钟信号（CLK）,频率很高（Mhz或Ghz量级）。
- 数据有效信号（DE），高电平期间数据线上的像素有效。

- 关键点：DPI实现了Video模式的数据传输机制。它常被称之为RGB接口，因为数据线直接传输RGB分量（或经过便编码的数据值）。

总结：

- `DBI` 是 `Command` 模式的**运输工具**（运指令和货物/像素数据到仓库/GRAM）。
- `DPI` 是 `Video` 模式的**运输工具**（实时运送连续的像素流到工厂流水线/面板）。
- 它们服务于不同的显示架构（有无 GRAM）和更新策略（间断 vs 连续）。

## 六、Video模式下三种传输方式

### 1、 Non-Burst Mode with Sync Pulses (带同步脉冲的非突发模式)

**工作原理：**

1. 使用专用得HSYNC 和 VSYNC信号线。
2. VSYNC脉冲表示一帧得开始。
3. 在每一帧内，HSYNC表示一行得开始。
4. DE(data enable)信号通常是**高电平有效**，表示数据线上的像素数据有效。
5. 像素数据 (`D[0:n]`) 在 `DE` 为高、`CLK` 有效沿时传输。
6. 在行同步 (`HSYNC`) 和场同步 (`VSYNC`) 期间，`DE` 为低，数据线无效（或传输消隐数据）。

**时序图得关键区域（一行内）：**

- **HSYNC Pulse Width:** HSYNC 信号保持有效的持续时间。
- **Back Porch:** HSYNC 结束到 DE 开始（有效像素开始）之间的时间。
- **Active Video:** DE 为高的时间，有效像素数据被传输。
- **Front Porch:** DE 结束（有效像素结束）到下一个 HSYNC 开始之间的时间。

**特点：**

- 最传统最直观得RGB接口模式。
- 同步信号独立而且明确。
- 需要比较多得物理引脚（HSYNC，VSYNC，DE，CLK，D[0:n]）。
- 时序要求严格，但设计相对比较直接。

### 2、Non-Burst Mode with Sync Events (带同步事件的非突发模式)

**工作原理：**

1. 主要是依赖DE信号来指示有效像素区域。
2. HSYNC和VSYNC被弱化和整合：
   - 方式一（弱化）： HSYNC/VSYNC 信号仍然存在，但只在消隐期(DE为低时)的特定位置产生一个非常短(通常1个时钟周期)的脉冲来标记行/帧的边界，它们不再是持续的脉冲称为 Sync Event.
   - 方式二（整合）：方式二(整合): 有时 HSYNC/VSYNC 信息被编码成特殊的数据包，在消隐期通过数据线传输(需要控制器和面板支持特定协议)。
3. 有效像素传输方式与带同步脉冲得非突发模式相同（DE高 +CLK ）。

**特点：**

- 减少了对HSYNC/VSYNC得需求。
- 需要控制面板和控制器都支持Sync Event或数据包形式得同步信息得识别处理。
- DE信号变得至关重要。

### 3、Burst Mode (突发模式)

**工作原理：**

1.    仍然是使用HSYNC、VSYNC，DE，CLK等信号（功能同非突发模式）。
2. 关键区别在像素数据的传输组织：
   - 在非突发模式中，每一个像素都是在一个时钟周期（CLK）内传输。
   - 在突发模式中，多个像素被打包成一个数据包（Burst）。
   - 在一个时钟周期内，传输整个数据包（包含多个像素得数据）。
   - 数据线的有线带宽被成倍的利用。例如：如果数据线是24-bit(RGB888),在一个时钟周期传输2个像素的突发模式下，相当于数据速率翻倍（每个CLK传输48bit 的数据）。
3. 需要更高的时钟频率或带宽的数据总线来驱动突发传输。
4. 控制器内部和面板内部需要有打包和解包逻辑。

**特点：**

- 显著提高数据的传输效率，降低了接口对吴丽萍率的要求（或允许在相同频率下传输更高分辨率/刷新率）。
- 有助于减少电磁干扰（EMI）,因为数据线状态变化的平均频率可能降低（虽然它的瞬时速率更高）。
- 增加了控制器和面板设计的复杂性（需要打包/解包逻辑）。
- 需要跟为严格的时序控制和信号完整性设计。

对三个模式进行总结：

| 传输方式                     | 核心特点                                                     | 优点                                  | 缺点                               | 适用场景                        |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------- | ---------------------------------- | ------------------------------- |
| **Non-Burst w/ Sync Pulses** | 有专用的HSYNC，VSYNC持续脉冲，DE指示有效数据                 | 直观，时序清晰，设计简单              | 引脚多                             | 传统应用，引脚资源不紧张        |
| **Non-Burst w/ Sync Events** | 依赖DE，同步信号被弱化为短脉冲（Sync Event）或通过数据包传输 | 减少或省去HSYNC/VSYNC引脚，简化布线   | 需支持 Sync Event/数据包协议       | 需要减少引脚数的应用            |
| **Burst Mode**               | 可以将多个像素打包，一个CLK传输一个包，同步信号同非突发模式  | 高效利用带宽，降低EMI，支持更高的性能 | 设计复杂 (打包/解包)，时序要求更严 | 高分辨率、高刷新率、低 EMI 需求 |

## 七、数据传输协议-MIPI DSI的协议层

1、数据包化：MIPI DSI的核心是把要传输的数据（像素数据或命令）封装成数据包（packet）。

- 数据类型：
  - **vedio Mode:**发送连续的像素数据流（RGB/YUV）。屏幕驱动I通常有一个内置的缓冲区（FIFO）。**RK3588 VOP** 生成的像素流通过 DSI 控制器打包成 Video Mode 数据包发送。
  - **Command Mode:** 发送显示命令和少量数据 (如初始化参数、窗口设置、Gamma 校正)。屏幕驱动 IC 解析命令并执行。命令本身也遵循 DCS (Display Command Set) 或 MCS (Manufacturer Command Set) 规范。
- 包结构（简化）：
  - **Data Identifier (DI):**包含 `VC` (Virtual Channel ID, 用于多屏幕/分区) 和 `DT` (Data Type, 指明包类型，如 `0x39` 代表长包写命令 `Long Write`, `0x05` 代表短包写命令 `Short Write`, `0x15` 代表短包读命令 `Short Read`, `0x3C` 代表长包像素数据包 `Long Write (Pixel Stream)` 等)。
  - **Packet Header (长包特有):**包含WC（Word Count），指明包内有效数据的字节数（WC * 1 bit）.
  - **Packet Payload:** 实际数据内容 (像素 RGB 值、命令码、命令参数)。
  - **Packet Footer(长包特有)：**包含ECC（ERROR Correction Code）用于校验头部。
  - **CRC (长包特有):** 对整个包 (Header+Payload) 的循环冗余校验。
- 短包（short packet）：用于发送命令、帧同步信号（VSYNC、HSYNC）或TE响应。只有DI和可选的2字节数据（命令参数）。
- 长包（long Packet）：用于发送像素数据，初始化序列数据或长命令参数。

2、RK3588de DSI Host Controller：

![1757536504361](C:\Users\宏\AppData\Roaming\Typora\typora-user-images\1757536504361.png)

- RK3588 内部集成 MIPI DSI Host Controller (通常称为 `DSI Tx` 或 `DSI` 模块)。
- 核心功能：
  - 接收来自 **VOP (Video Output Processor)** 的并行像素数据流 (`AXI` 或 `VP` 总线)。
  - 根据配置 (`Video Mode` 或 `Command Mode`)，将数据/命令 **打包** 成符合 MIPI DSI 协议的数据包。
  - 与 **D-PHY** 交互，将打包好的数据流发送到物理层。
  - 管理传输状态机 (HS/LP 切换，时序控制)。
  - 处理屏幕驱动 IC 的应答 (ACK/NACK) 和读取状态 (通过 LP Mode)。

- 驱动配置：Linux驱动（）需要配置DSI控制器：
  - 工作模式 (`mode_flags`: `MIPI_DSI_MODE_VIDEO`, `MIPI_DSI_MODE_CMD` 等)。
  - Lane 数量 (`lanes`)。
  - 格式 (`format`: `MIPI_DSI_FMT_RGB888`, `RGB565` 等)。
  - **屏幕时序参数 (关键！):** 从设备树 `panel-timing` 节点获取或硬编码，包括：
    - `clock-frequency`: DSI 链路总像素时钟 (单位 Hz, `hactive + hfront-porch + hsync-len + hback-porch`) * (`vactive + vfront-porch + vsync-len + vback-porch`) * `refresh_rate` / `lanes` * `bpp` / `8` 计算得出。**驱动会根据此值配置 VOP 和 DSI 的时钟分频器。**
    - `hactive`, `hfront-porch`, `hsync-len`, `hback-porch`
    - `vactive`, `vfront-porch`, `vsync-len`, `vback-porch`
    - `hsync-active`, `vsync-active`, `de-active`: 同步信号极性。

## 八、显示框架DRM

**核心内容：**

​	`rockchip_drm_drv.c`时RockChip SoC DRM驱动的主控驱动（Master Driver）或者系统级驱动（system Driver）。它不直接操作具体显示硬件（如 VOP、DP、HDMI等），而是扮演一个“指挥官”和“协调者”的角色，负责：

1. 负责初始化并管理整个DRM子系统。
2. 负责动态绑定和解绑定各个硬件子模块（通过Component框架）。
3. 提供统一的用户空间接口（DRM API）。
4. 管理共享资源（如IOMMU、GEM内存池）。
5. 实现通用的、垮子模块的功能（如EDID解析、VRR支持、调试接口）。

**整体的架构定位**：

Rockchip 的DRM架构遵循Linux kernel 的Component框架和**DRM/KMS (Kernel Mode Setting) 框架**。

- **DRM/KMS 框架**: 提供了标准的用户空间 API（如 `ioctl`）、帧缓冲、模式设置、原子提交等。
- **Component 框架**:允许将复杂的设备（Display Subsystem）拆分成多个独立的、可重用的硬件驱动模块（如VOP、DP、HDMI等）。这些模块可以独立开发、编译并加载，最终有一个主控制器驱动（即rockchip_drm_drv.c）将他们“粘合”在一起，形成一个完整的功能齐备的显示系统。

因此，`rockchip_drm_drv.c` 就是这个“粘合剂”，是整个 Rockchip 显示子系统的 **入口点和中枢神经**。

## 九、RK3588 VOP/DSI驱动

**核心内容：**

​	rockchip_drm_vop2.c是 Rockchip SoC **VOP (Video Output Processor) 2.0** 硬件模块的 **具体硬件驱动** **(Hardware Driver)**。他不是整个显示系统的主控，而是负责直接控制和管理VOP2硬件寄存器，实现像素数据读取、处理（缩放、颜色空间转换、aplha混合）、输出特定接口（如DSI、HDMI、LVDS、DP等）的核心模块。

**它整个框架中扮演着“执行者”的角色：**

1. 接受指令：从上层的rockchip_drm_drv.c接受源自提交请求。
2. 配置硬件：解析这些请求，并精确设置VOP2内部复杂的寄存，包括窗口位置、格式、缩放、混合、时序、DSC、输出接口等。
3. 管理资源：直接与DSI、HDMI、DP等物理接口的控制器通信（通过寄存器或共享内存），将处理后的像素数据发送出去。
4. 处理中断：响应 VOP2产生各种硬件中断（垂直同步、帧完成、错误等）。

**运行流程分析：**

​	首先驱动遵守LInux DRM/KMS和Component框架的典型模式，核心是 **vop2_bind()** 和 **vop2_unbind()** 函数，它们由 `rockchip_drm_drv.c` 中的 `component_master_add_with_match()` 调用。

**整个框架的作用：**

| 作用                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| 1、硬件抽象层（HAL）  | 提供了一个统一的、基于 DRM/KMS 的 API，向上屏蔽了 VOP2 硬件的复杂性。用户空间只需通过标准的`DRM_IOCTL_MODE_ATOMIC`即可控制显示。 |
| 2、硬件寄存控制器     | 直接读写 VOP2 的数百个寄存器，精确地控制像素数据流的路径、格式、缩放、混合、时序和输出。 |
| 3、资源协调           | 管理 VOP2 内部的复杂资源：多个 Win、Mixer、DSC、电源域、时钟。它确保这些资源被正确、高效地分配和使用。 |
| 4、与物理接口的桥接器 | 它不直接驱动 DSI PHY 或 HDMI PHY，但它**是 VOP2 内部逻辑与外部物理接口之间的唯一桥梁**。它通过配置`VOP_CTRL_SET`和`VOP_GRF_SET`等宏，**开启/关闭 DSI/HDMI/DP 控制器的使能位，并将 VOP2 的输出端口路由到正确的物理接口**。它还负责设置接口的时钟分频、信号极性等。**它告诉 VOP2：“把处理好的画面，通过 DSI0 发送给你的屏幕”。** |
| 5、实现高级功能       | 实现了现代显示技术，如 DSC (压缩)、HDR (高动态范围)、Vivid HDR、ACM (自适应色彩管理)、多区域显示、可变刷新率等。 |
| 6、中断处理           | 响应硬件事件，保证了显示的实时性和稳定性。                   |

## 十、MIPI中得VOP是如何驱动DSI和D-PHY进行工作

`vop2_crtc_atomic_enable（）`是Rockchip Vop2驱动得核心，它在用户空间请求一个新显示模式（切换分辨率、刷新率或连接/断开显示器）时被调用。

**核心作用：**

为指定得Video port(VP)配置并启动整个显示流水线，使其按照新得`drm_display_mode` 输出正确的图像。本质上你可以理解为将处理好得像素数据通过指定得物理接口（DSI，HDMI，DP）发送出去。

PSR(Panel self Refresh)模式：GPU停止发送数据，仅当画面更新时才唤醒，面板内部缓存自行刷新画面功耗降低30~70%（待机界面、文字、图标）

## 十一、HDMI的驱动调试过程

使用rk3588的HDMI 2.1 Tx输出（最高8K）,内部结构：

【video Source】——》【VOP（video Output Processor）】——》【HDMI Controller(DW HDMI IP)】——》【HDMI PHY】——》【HDMI Connector】

常见问题与规避方法

1、无显示输出（黑屏）

如果是自己的手搓的设备树配置节点（AI）：

- HDMI 时钟配置是否正确，受否使能，频率是否正确
- PHY 未正确初始化（电压和阻抗）
- EDID读取失败（DDC I2C通信失败）
- 显示模式是不是支持（如4K@60fps 是不支持YUV420）
- HPD未触发，系统认为无显示器

2、显示花屏、闪屏、颜色异常

可能的原因：

- 时钟相位/采样点偏移（需要调整VOP或HDMI采样相位）
- TMDS差分信号线阻抗不匹配或者未做等长
- 电压摆幅/预加重设置不当（长线或4K场景）
- 色彩空间转换错误（RGB/YUV/RYYB）

3、无音频输出

- I2S或SPDIF未路由到HDMI
- ALSA未正确配置HDMI声卡
- 音频时钟未同步

