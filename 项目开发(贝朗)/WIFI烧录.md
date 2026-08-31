## 一、工具:
- 软件: 
1. usblogview(确认端口号)
![](vx_images/500513338341501.png =400x)

2. flash_download_tool(下载工具)
![](vx_images/428741779922051.png =400x)


- 硬件:
1. 串口工具(下载工具)
![](vx_images/451622193608903.png =400x)

2. 公对母杜邦线
![](vx_images/119224601233612.png =400x)

3. 电路板
![](vx_images/465014519096917.png =400x)


# 二、流程


1. 杜邦线对齐串口工具插入电路板,<mark>线序为对应颜色的杜邦线对齐插入</mark>
![](vx_images/421382665212495.png =400x)
2. 打开usblogview, 串口工具插入电脑, 会提示什么设备插入电脑,寻找记录对应的端口号
![](vx_images/543724148633607.png =600x)

4. 用typec口给设备电路板供电,<mark>注意一定是用电脑接typec给设备供电,并且是先插入烧录工具最后在给设备供电</mark>

5. 打开flash_download_tool下载工具,选择ESP32C5, OK确定
![](vx_images/334911960866956.png)

6. 选择需要烧录的固件
![](vx_images/88644237858441.png)

![](vx_images/386734999160180.png)

7. 选择第3步usblogview确定的端口号
![](vx_images/331783774109633.png)

8. 电机START开始下载,然后等待下载完成, 结束, 如下
![](vx_images/131404144852468.png)