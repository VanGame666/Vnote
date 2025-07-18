# FatFS文件系统

![](vx_images/518130938251138.png =700x)
- mbr_pt: 16字节,MBR（Master Boot Record）中的分区表（Partition Table),是磁盘分区的核心数据结构，用于定义磁盘的分区布局
- MBR: 主引导记录区,<mark>磁盘的第一个物理扇区(LBA 0)</mark>,位于整个硬盘的0磁道0柱面1扇区。不过，在总共512字节的主引导扇区中，MBR只占用了其中的446个字节，另外的64个字节交给了DPT（Disk Partition Table硬盘分区表），最后两个字节“55，AA”是分区的结束标志。这个整体构成了硬盘的主引导扇区。
- MBR:
![](vx_images/541718418470645.png =700x)

- MBR->DPT:
![](vx_images/12184174769039.png =700x)

- DBR:分区引导表,<mark>每个分区的第一个扇区</mark>(如分区起始 LBA + 0)
![](vx_images/379039336990512.png =700x)


---

### 固态硬盘结构
![](vx_images/599381766445492.png =700x)

### 机械硬盘结构
![](vx_images/458456215598244.png =700x)

