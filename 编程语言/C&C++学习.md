# C++学习笔记
---
QT打包步骤:
- 环境变量设置
- CMD内输入    `windeployqt <工程名>.exe`

# 源码阅读心得技巧
---
1. 先阅读`main.c`和`main.h`
2. 其次阅读所用源文件的`头文件`,头文件提供对外的实现`API接口`
3. 了解软件运行的`启动流程`,跟随启动流程抽丝剥茧层层深入

# C/C++学习笔记
`typedef void (* TaskFunction_t)( void * arg );`重定义函数指针为<TaskFunction_t>
