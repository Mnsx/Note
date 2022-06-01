# 操作系统

## 学习资料

* [操作系统（哈工大李治军老师）](https://www.bilibili.com/video/BV1d4411v7u7/?spm_id_from=333.788.recommend_more_video.2)

## 操作系统的由来

1. 从白纸到图灵机
2. 从图灵机到通用图灵机
3. 从通用图灵机到计算机

**取址执行**

![](.\计算机的基本执行流程.jpg)

### X86执行流程

1. X86 PC刚开机时CPU处于实模式
2. 开机时，CS=0xFFFF;IP=0x0000
3. 寻址0xFFFF0（ROM BIOS映射区）
4. 检查RAM，键盘，显示器，软硬磁盘
5. 讲磁盘0磁道0扇区读入0x7c00处
6. 设置CS=0x07c0，IP=0x0000

## 操作系统启动

执行BOOTSEG = 0x07c0跳转到

INITSEG = 0x9000展示图片，打印文字

跳转到SETUPSEG = 0x9020执行启动操作