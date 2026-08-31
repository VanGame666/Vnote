# MSYS2
> pacman -Sy
> pacman -Su
> pacman -Su
> pacman -S git vim cmake make mingw-w64-x86_64-arm-none-eabi-gcc mingw-w64-x86_64-arm-none-eabi-gdb mingw-w64-x86_64-openocd


- "D:\msys64\msys2_shell.cmd" -here -defterm -mingw64


# 一次性安装嵌入式开发核心套件
```
pacman -S --needed \
mingw-w64-ucrt-x86_64-arm-none-eabi-gcc \  # ARM GCC 交叉编译器
mingw-w64-ucrt-x86_64-arm-none-eabi-gdb \  # ARM GDB 调试器
mingw-w64-ucrt-x86_64-openocd \            # OpenOCD 调试编程工具
mingw-w64-ucrt-x86_64-make \               # Make 构建工具
mingw-w64-ucrt-x86_64-cmake \              # CMake 跨平台构建工具
mingw-w64-ucrt-x86_64-git \                # 版本控制
mingw-w64-ucrt-x86_64-python \             # Python（很多脚本依赖）
base-devel                                  # 基础开发工具集


```



> openocd -f "D:/msys64/mingw64/share/openocd/scripts/interface/jlink.cfg" -c "transport select swd" -f "D:/msys64/mingw64/share/openocd/scripts/target/stm32f4x.cfg"

> gdb-multiarch stm32f407_project.elf