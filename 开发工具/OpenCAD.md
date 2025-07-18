# OpenCAD

## 1. 基础概念
- **定义**：OpenOCD（Open On-Chip Debugger）是一个开源的调试工具，支持JTAG/SWD接口的芯片调试和编程。
- **核心功能**：
  - 芯片调试（GDB集成）
  - Flash编程（烧录固件）
  - 硬件边界扫描（JTAG）
- **硬件接口**：
  - JTAG（联合测试行动组）
  - SWD（串行线调试）
  - cJTAG（压缩JTAG）
- **相关工具**：
  - GDB（调试器）
  - ST-Link/J-Link（调试探针）

## 2. 安装与配置
- **安装方法**：
  - Linux: `sudo apt-get install openocd`
  - Windows: 从官网下载预编译包
  - macOS: `brew install openocd`
- **配置文件**：
  - 调试探针配置（`interface/`目录，如`stlink.cfg`）
  - 目标芯片配置（`target/`目录，如`stm32f4x.cfg`）
- **启动命令**：
  ```bash
  openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
  ```

```puml
@startuml
skinparam backgroundColor #EEEBDC

start
:安装OpenOCD;
:编写配置文件
（interface/stlink.cfg
target/stm32f4x.cfg）;
:启动OpenOCD服务器
**命令**：openocd -f <配置路径>;
split
  -[#blue]->
  :通过GDB连接
  **命令**：target remote :3333;
  if (连接成功?) then (是)
    :执行调试操作
    (单步执行/断点/寄存器查看);
    :或执行固件烧录
    **命令**：flash write_image...;
    if (烧录验证通过?) then (是)
      :复位并运行程序
      **命令**：reset run;
      -[#green]->
      stop
    else (否)
      #red:报错：烧录失败;
      -[#red]-> 
      :检查Flash配置或硬件连接;
      detach
    endif
  else (否)
    #red:报错：连接失败;
    -[#red]->
    :检查配置文件或硬件接口;
    detach
  endif
split again
  -[#orange]->
  :直接调用Telnet/HTTP
  (端口4444/6080);
  :发送裸机命令
  (reset/halt/memory读写);
  stop
end split
@enduml

```