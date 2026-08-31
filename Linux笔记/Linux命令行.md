# Linux命令行

- [SSH连接Linux开发板](https://geek-blogs.com/blog/ssh-windows-to-linux/)

 - 命令行删除快捷键
```
# 快捷键：Ctrl + U
# 效果：删除光标前的所有内容
# 示例：输入 "ls -la /home/user" 后按 Ctrl+U，全部删除

# 快捷键：Ctrl + K
# 效果：删除光标后的所有内容
# 示例：输入 "ls -la /home/user"，光标在 "ls" 后，按 Ctrl+K 删除 "-la /home/user"

# 快捷键：Ctrl + C
# 效果：取消当前行，重新开始新行
# 示例：输入错误命令后按 Ctrl+C，直接清空并换行

# 快捷键：Ctrl + L
# 效果：清空屏幕但保留当前输入的命令
# 示例：输入长命令时按 Ctrl+L，屏幕清空但命令还在

# 快捷键：Ctrl + Y
# 效果：粘贴最近删除的内容
# 示例：按 Ctrl+U 删除整行后，按 Ctrl+Y 恢复
```





  - APT
```
apt-mark showmanual
sudo apt purge --auto-remove pkg
```


- SSH连接远程Linux系统
```
ssh -p [user@host] [command]
```

- 远程复制文件到Linux系统
```
scp -r [~/my_project] [user@host:/opt/]
```

- 查看Linux硬件和系统资源
```
df -h
lscpu
cat /proc/cpuinfo
```

- 解压和压缩命令
```
tar -cf backup.tar.gz /home/user/data
tar -xf backup.tar.gz 

zip -r backup.zip /home/user/data
unzip backup.zip
```

- 复制粘贴命令
```
cp file.zip /home/debian/bin
mv file.zip /home/debian/bin
```

- 挂载U盘
```
wsl下额外命令:
usbipd list
usb bind --busid <id>
usbipd attach --wsl --busid <id>

Linux下命令:
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
```

- 网站下载内容
```
wget https://example.com/file.zip
curl -O https://example.com/file.zip
```

---

```
eMMC/SD卡 = NAND闪存芯片 + FTL控制器 + 接口电路
          ┌─────────────────────────────────┐
          │  eMMC/SD卡内部                  │
          │  ┌─────────┐ ┌──────────────┐  │
NAND芯片 → │  │  FTL   │ │  接口控制器  │  │ → 标准块设备接口
          │  │ 控制器  │ │ (MMC/SD协议) │  │    (512B扇区)
          │  └─────────┘ └──────────────┘  │
          │  坏块管理 磨损均衡 ECC校验      │
          └─────────────────────────────────┘
```

---

```
原始NAND = 裸NAND闪存芯片 + 简单接口
         ┌─────────────────┐
         │  原始NAND芯片    │
         │  ┌───────────┐  │
主机 →    │  │ NAND阵列  │  │ → 原始NAND接口
         │  └───────────┘  │    (页/块操作)
         │  无管理功能      │
         └─────────────────┘
         
```
