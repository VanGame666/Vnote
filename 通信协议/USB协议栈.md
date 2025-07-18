# USB协议
USB通信距离短,是不对等协议
USB设备按功能分为集线器和功能设备
USB拓展深度最多七层, <mark>2^7 = 127</mark> 个USB设备,每个设备 <mark>2^4 = 16</mark> 个端点
- 六个设备状态: 开机, 缺省, 地址, 配置, 连接, 挂起

- [USB协议解析](https://blog.csdn.net/zhoutaopower/article/details/82083043)

- 四种PID类,每个类有四个PID,共16个PID
    1. 令牌类: OUT, IN, SETUP, SOF
    2. 数据类: DATA0, DATA1, DATA2\*, MDATA\*
    3. 握手类: ACK, NACK, STALL, NYET\*
    4. 特殊类: PRE, ERR\*, SPLIT\*, PING\*

- [USB设备描述结构](https://www.cnblogs.com/chd-zhangbo/p/5249955.html)
    1. 设备描述符: <mark>一个设备仅一个</mark>设备描述符,包含了设备类型、设备遵循协议、厂商ID、产品id、序列号等
    2. 配置描述符: 一个设备<mark>同一时刻</mark>只能有一种配置生效
    3. 接口描述符: 一个interface就代表一个设备
    4. 端点描述符: USB通信的基本物理单位,一个endpiont只能承载一个方向的数据
            
- 四种传输类型: 控制传输, 中断传输, 批量传输, 同步传输
    1. 控制传输: 设置阶段, 数据阶段, 状态阶段(每个阶段包含多个事务)
    2. 中断传输: 
    3. 批量传输: 
    4. 同步传输: 
        
- 事务: 令牌包, 数据包, 握手包(<mark>包是最小传输单元,不可被打断</mark>,事务也不可被打断,传输可被打断)

|  包名  | 起始标志 | 同步标志 |    包PID     |    填充    | 校验 | 结束标志 |
| ----- | -------- | -------- | ----------- | --------- | ---- | -------- |
| 令牌包 | SOP      | SYNC     | IN/OUT      | ADDR,ENDP | CRC  | EOP      |
| 数据包 | SOP      | SYNC     | DATA0/DATA1 | DATA      | CRC  | EOP      |
| 握手包 | SOP      | SYNC     | ACK/NACK    | \         | CRC  | EOP      |

- 低速(D-) < 全速(D+) < 高速

**当一个USB设备被接入USB集线器端口后，USB设备开始被枚举，过程大概如下：**
![](vx_images/458205240430745.png =700x)



![](vx_images/154427243615981.png =700x)
![](vx_images/247807031103204.png =700x)
![](vx_images/543263909402888.png =700x)
```puml
@startmindmap
* USB协议栈体系
** 1. 基础概念
	*** USB版本演进
		**** USB 1.x (12Mbps)
		**** USB 2.0 (480Mbps)
		**** USB 3.x (5-20Gbps)
		**** USB4/Thunderbolt
	*** 拓扑结构
		**** 主机-设备架构
		**** 集线器级联
		**** OTG双角色设备
** 2. 物理层规范
	*** 电气特性
		**** 差分信号(D+/D-)
		**** VBUS供电管理
		**** 电缆类型(TypeA/B/C)
	*** 连接器演变
		**** Standard/Mini/Micro
		**** USB-C特性
		**** 交替模式(Alt Mode)
** 3. 协议层架构
	*** 数据包结构
		**** 令牌包(Token)
		**** 数据包(Data)
		**** 握手包(Handshake)
	*** 传输类型
		**** 控制传输(枚举配置)
		**** 中断传输(实时设备)
		**** 批量传输(大文件)
		**** 同步传输(音视频)
	*** 设备枚举流程
		**** 获取描述符
		**** 设置地址
		**** 配置设备
** 4. 设备类规范
	*** 标准设备类
		**** HID人机接口
		**** MSC大容量存储
		**** Audio音频类
		**** CDC通信设备
	*** 自定义设备
		**** 供应商自定义类
		**** 复合设备设计
		**** 接口关联描述符
** 5. 主机控制器
	*** 主机控制器类型
		**** UHCI/OHCI(USB1.1)
		**** EHCI(USB2.0)
		**** xHCI(USB3.0+)
	*** 驱动栈结构
		**** Windows USBDI
		**** Linux USB Core
		**** 端点0控制管道
** 6. 开发工具链
	*** 硬件调试工具
		**** USB协议分析仪
		**** 逻辑分析仪抓包
		**** Beagle USB嗅探器
	*** 软件开发工具
		**** Wireshark USB插件
		**** USBlyzer分析软件
		**** libusb跨平台库
@endmindmap
```