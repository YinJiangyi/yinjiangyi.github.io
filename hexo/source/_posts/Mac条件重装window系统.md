---
title: Mac条件重装window系统
typora-root-url: ../../source
date: 2021-06-19 13:56:17
tags:
catagories:
sticky:
---

主要分为两步，制作系统盘+重装系统

### 制作系统盘

#### 下载windows 10 镜像

下载地址：https://www.microsoft.com/zh-cn/software-download/windows10ISO

Tip：网页下载速度很慢，复制链接在迅雷下载10分钟搞定（当然还是要看网络环境怎么样）

#### Mac电脑上制作系统盘

手上除了工作本，只有一个mac本，网上的系统盘制作工具不适用。具体参考https://www.jianshu.com/p/153a1a4fd587

- 查看当前硬盘列表

  ```
  diskutil list
  
  /dev/disk0 (internal, physical):
     #:                       TYPE NAME                    SIZE       IDENTIFIER
     0:      GUID_partition_scheme                        *251.0 GB   disk0
     1:                        EFI EFI                     314.6 MB   disk0s1
     2:                 Apple_APFS Container disk1         250.7 GB   disk0s2
  
  /dev/disk1 (synthesized):
     #:                       TYPE NAME                    SIZE       IDENTIFIER
     0:      APFS Container Scheme -                      +250.7 GB   disk1
                                   Physical Store disk0s2
     1:                APFS Volume Macintosh HD - 数据     206.5 GB   disk1s1
     2:                APFS Volume Preboot                 292.2 MB   disk1s2
     3:                APFS Volume Recovery                613.9 MB   disk1s3
     4:                APFS Volume VM                      3.2 GB     disk1s4
     5:                APFS Volume Macintosh HD            24.0 GB    disk1s5
     6:              APFS Snapshot com.apple.os.update-... 24.0 GB    disk1s5s1
  
  /dev/disk2 (external, physical):
     #:                       TYPE NAME                    SIZE       IDENTIFIER
     0:     FDisk_partition_scheme                        *15.5 GB    disk2
     1:             Windows_FAT_32 18811339060             14.8 GB    disk2s1
  ```

  我的优盘是最后一个disk2

- 重命名&&格式化u盘

  ```
  diskutil eraseDisk ExFAT "WINDOWS10" MBR disk2
  ```

### 重装系统

#### 进入bios

开机过程按F1，出现文字时按提示点enter键，可以看到进入bios模式的提示按F1

---

**失败**，mac系统下制作的系统盘，在安装的过程中，无法正常启动，报错未解决

> 安装windows时找不到任何设备驱动程序。请确保安装媒体包含的驱动程序正确



