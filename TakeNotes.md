# 成长计划

- 开发环境: Git, Cmake, Make, GCC, GDB, OpenOCD
- 编程语言: C, C++, Python, Verilog
- 微控制器: STM32, ESP32, AG32, RA8D1, LPC4000
- 通信协议: OneWire, UART, IIC, SPI, CAN, USB, ETH, BLE, MQTT, ModBus
- 操作系统: FreeRTOS, RT-Thread
- 中间组件: TinyUSB, LVGL, FatFS, Lwip, SimpleFOC, OpenMV

> 1. 良好的编程规范, 面向对象思想编程, 设计模式
> 2. 分层架构, 熟悉CorteM架构
> 3. 熟悉上位机设计QT, 掌握socket网络编程
> 4. 熟悉Jlink, DAP-Link, JTAG, SWD调试

|           |                |
| --------- | -------------- |
| 1.电机FOC | 2.视觉OpenMV   |
| 3.电池BMS | 4.热成像Filter |

软件: MCU-->MPU-->DSP
硬件: SCH-->PCB-->FPGA
架构: 轮询, 中断, 分层, 状态机


|        |          |        |
| ------ | -------- | ------ |
| 1.电子 | 2.计算机 | 3.英语 |
| 4.绘画 | 5.长跑   | 6.吉他 |

给妈妈买洗衣机,给爸爸换手机

---
打开ip_name = analog_ip
执行Upload_LOGIC

```c
void TestDac(DAC_TypeDef *dac)
{
  printf("\nTesting DAC%d:\n", dac == DAC0 ? 0 : 1);

  DAC_Enable(dac);
  for (int i = 0; i < 1024; i += 1) {
    DAC_SetData(dac, 512);
    UTIL_IdleUs(10);
    printf("DAC = %d\r\n",dac->DATA);
  }
  DAC_Disable(dac);
}
```