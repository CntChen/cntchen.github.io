---
title: Ubuntu16.10 迁移到 SSD
date: 2017-02-11 01:57:06
tags: Linux OS
---

# Ubuntu16.10 迁移到 SSD

## 背景
2016年双十一入手了一块500G的 SSD（Solid State Drive，固态硬盘），打算安装到自己的笔记本上。笔记本的 HDD（Hard Disk Drive，机械硬盘）已经跑了 Ubuntu16.10 + Win10 双系统。光驱位的硬盘支架也装好了，一直虚位以待。工作忙一直拖到了2017年。

公司的 PC 机器也是 Ubuntu16.10，并且安装的软件比较齐全，所以计划将 PC 的 Ubuntu16.10 迁移到 SSD 上，然后在笔记本上运行。

## 开工准备
* Ubuntu16.10，PC 上安装，各项迁移步骤运行的环境并作为要迁移到 SSD 的系统。
* [Gparted Partition Editor]，图形化的分区工具。
* 外接硬盘盒，通过 USB 线将 SSD 连接到 PC 上。

## 基础知识
该章节是计算机启动和系统加载的一些概念，有助于加深对迁移原理的理解，注重实践的话可以直接跳过。

总结不一定准确，仅作为个人理解。干货可以看这篇文章：[uefi-boot-how-does-that-actually-work-then][uefi-boot-how-does-that-actually-work-then]

### BIOS vs UEFI
BIOS（Basic Input/Output System）和 UEFI（Unified Extensible Firmware Interface ）**是不同的计算机启动固件（Fireware）**，需要硬件（通常为主板）支持，相互代替的，其中 UEFI 是比较新的方式。

* BIOS
经典的启动固件，会调用磁盘的 [MBR][MBR]，然后由 MBR 中的 loader 继续加载操作系统。
* UEFI
UEFI 用来代替 BIOS，并克服 BIOS 的缺点，大多数的 UEFI 固件会提供兼容 BIOS 的启动方式。

* 区别
可以看这篇文章：[UEFI是什么？与BIOS的区别在哪里?][UEFI是什么？与BIOS的区别在哪里]

### MBR vs GPT
MBR 与 [GPT][GPT] 用于存储硬盘的分区信息，**是不同的硬盘分区表类型**。

* MBR
**MBR 表示 MBR 分区表**，MBR 分区表在硬盘开头处存放了特殊的启动分区，称为 MBR（Master Boot Record，**主启动记录**），包含 Boot Loader 和硬盘逻辑分区。MBR 支持最大约2T的硬盘，最多能划分4个主分区，更多分区需要使用拓展分区实现。
*（`MBR`在行文中可以表示 `MBR 分区表`和`主启动记录`两个意思，注意甄别。）*

* GPT
GPT 表示 GUID（Globally Unique Identifier） 分区表，是 UEFI 规范的一部分，用于替换 MBR 的分区方式。GPT 没有分区数和分区大小限制。

* 区别
可以看这篇文章：[What’s the Difference Between GPT and MBR When Partitioning a Drive][What’s the Difference Between GPT and MBR When Partitioning a Drive]

### File System
[File System][File System]（文件系统）是存储媒介中文件存储的组织方式。
不同的文件系统类型有不同的速度，灵活性，安全性和占用空间。不同操作系统只支持特定的文件系统类型。
常见的文件系统类型有 FAT16，FAT32，NTFS，EXT3，EXT4，HFS 等。

### 磁盘发展史
Wikipedia 上有许多关于磁盘的资料，在磁盘分区上，我**猜测**的发展脉络是这样的：
1. 磁盘跟内存一样直接物理寻址去访问数据；
2. 为了方便，建立数据 Index，有了 File System；
3. 需要多个分区，搞出了 Partition Tabel。

### 小结
* BIOS/UEFI 跟 MBR/GPT 是不同层级的，BIOS/UEFI 是 Fireware，MBR/GPT 是分区表。
* **推荐的使用方式**： BIOS + MBR 或 UEFI + GPT：
>If you want to do a ‘BIOS compatibility’ type installation, you probably want to install to an MBR formatted disk.
If you want to do a UEFI native installation, you probably want to install to a GPT formatted disk.
* 理论上来说是可以组合使用的：
>Of course, to make life complicated, many firmwares can boot BIOS-style from a GPT formatted disk.
UEFI firmwares are in fact technically required to be able to boot UEFI-style from an MBR formatted disk.
* Windows 通常会要求 UEFI 的启动方式使用 GPT，不然不给继续安装。

## SSD 分区
### 硬盘状态
使用外接硬盘盒，将 SSD 连接到 PC 机上，先查看硬盘状态：
```js
$ sudo fdisk -l
Disk /dev/sda: 465.8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xb2708ce0

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048    411647    409600   200M  7 HPFS/NTFS/exFAT
/dev/sda2          411648 210126847 209715200   100G  7 HPFS/NTFS/exFAT
/dev/sda3       210128894 913704959 703576066 335.5G  f W95 Ext'd (LBA)
/dev/sda5       210128896 703989759 493860864 235.5G 83 Linux
/dev/sda6       703991808 704966655    974848   476M 83 Linux
/dev/sda7       704968704 764067839  59099136  28.2G 83 Linux
/dev/sda8       764069888 771973119   7903232   3.8G 82 Linux swap / Solaris

Partition 3 does not start on physical sector boundary.

Disk /dev/sdb: 489.1 GiB, 525112713216 bytes, 1025610768 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 33553920 bytes
```
其中`/dev/sda`为 PC 上的硬盘，装有 Ubuntu16.10 + Win7；`/dev/sdb`为 SSD，当前 SSD 为空盘。

### 分区策略
* 笔记本是 2011 年的机器，主板启动引导好像不支持 UEFI，是用 BIOS。
* 考虑 SSD 的拓展性，**分区表选择 GPL**，选用的**引导方式为 BIOS + GPT**。
* 此时安装 GRUB 引导对分区划分有要求，具体参考接下文的*《GRUB 引导》*章节。
* 先上分区结果：
![](/img/Ubuntu16.10_迁移到_SSD/partition-result.png)
*（注：前文出现的`/dev/sdb1`，`/dev/sdf1`和后面可能出现的`/dev/sd#1`都为同一个分区，因为多次插拔了 SSD ，所以标识一直按字母序递增）*

 * 不建立`/swap`分区了，因为 [Ubuntu17.04也要移除 swap 分区][Ubuntu17.04也要移除 swap 分区]。
 * `/dev/sdf1`分区，建立 GRUB 引导所需分区，大小为 1M，分区文件类型为`unformatted`，分区 flag 为`bios_grub`。
 * `/dev/sdf2`分区，Linux `/boot`分区，大小 1G。
 * `/dev/sdf3`分区，Linux `/`分区，大小 50G。
 * `/dev/sdf4`分区，Linux `/home`分区，大小 300G。

### 分区操作
分区操作在 Gparted 软件中完成，命令行`fdisk`和`parted`也可以操作，但是我不熟悉。

* 建立分区表
SSD 是一个空磁盘，此时并没有分区表，所以要先建立分区表。分区表的格式选用 GPT：
 1. 打开 Gparted，点击 `Device` --> `Created Partition Table`。
 2. 选择`partition tabel type`为`gpt`，然后点击`Apply`。

 ![](/img/Ubuntu16.10_迁移到_SSD/create-gpt.png)

* 建立 GRUB 所需分区
 1. 分区大小为1M，分区类型为`unformatted`。
![](/img/Ubuntu16.10_迁移到_SSD/bios-grub-partition.png)

 2. 在新建的分区上点击右键，选择`managerFlags`，然后选中`bios_grub`选项。
![](/img/Ubuntu16.10_迁移到_SSD/set-bios-grub-flag.png)

* 建立 Linux 系统分区
根据上文*<<分区策略>>*章节，依次建立其他分区，分区的文件格式选择`ext4`。
分区结果：
```js
$ sudo fdisk -l /dev/sdh
Device         Start       End   Sectors  Size Type
/dev/sdh1       2048      4095      2048    1M BIOS boot
/dev/sdh2       4096   2101247   2097152    1G Linux filesystem
/dev/sdh3    2101248 106958847 104857600   50G Linux filesystem
/dev/sdh4  106958848 736104447 629145600  300G Linux filesystem
```

## GRUB 引导
## GRUB 是什么
[GRUB][GRUB]（Grand Unified Boot loader）是硬盘中的软件，引导器（loader）的一种。目前主流版本是 GRUB2，可以看 [GRUB2 中文介绍][GRUB2 中文介绍]。

GRUB 用于从多操作系统的计算机中选择一个系统来启动，或从系统分区中选择特殊的内核配置。
> provides a user the choice to boot one of multiple operating systems installed on a computer or select a specific kernel configuration available on a particular operating system's partitions. -- [GRUB][GRUB]

示例：
![](/img/Ubuntu16.10_迁移到_SSD/grub-loader.jpg)
如图：第一个选项和最后一个选项是选择不同的操作系统；第一个选项和第二个选项是选择不同的内核配置。

### GRUB 位置
其启动代码（boot.img）直接安装在 MBR 中，然后执行 GRUB 内核镜像（core.img），最后从`/boot/grub`中读取配置和其他功能代码。
BIOS 引导方式中，MBR 分区表和 GPT 分区表的 [GRUB 引导文件所放分区不同][BIOS boot partition]：
![](/img/Ubuntu16.10_迁移到_SSD/GNU_GRUB_components.svg)

如图，GRUB 的执行顺序为 `boot.img` --> `core.img` --> `/boot/grub/`。
* 在 MBR 分区表中，`boot.img` 和 `core.img` 都在 MBR 中。MBR 虽然只占用一个扇区(512Byte)，但是其所在的磁道是空闲的，不会用于分区，可以放下 `core.img`。
>Some MBR code loads additional code for a boot manager from the first track of the disk, which it assumes to be "free" space that is not allocated to any disk partition, and executes it.  -- [MBR][MBR]

* 在 GPT 分区表中，MBR 为 [protected MBR][protected MBR]（为兼容 MBR，在硬盘起始位置保留的空间），后面并没有空间放`core.img`，需要建一个专门的分区来放，称为[BIOS boot partition][BIOS boot partition]，该分区的文件类型为`unformatted`，flag 为`BOIS_grub`，该 flag 用于标识`core.img`所要安装到的分区。若果使用 UEFI 引导，GRUB 读取的是 ESP 分区中的数据，不需要 flag 为 `BIOS_grub`的分区。

### 建立 GRUB 引导
使用 [grup-install 的教程][Installing-GRUB-using-grub_002dinstall]来安装 GRUB 到 SSD 盘。
* 挂载 `/boot`
挂载 SSD 的`/boot`为 PC Ubuntu 的`/mnt`，因为我们需要将 GRUB 配置文件放入 SSD 的`/boot/grub`中。
```
$ sudo mount /dev/sdb2 /mnt
```
* 安装 GRUB
执行以下命令：
```
$ sudo grub-install --target=i386-pc --root-directory=/mnt --recheck --debug /dev/sdb
```
如果看到以下输出，应该就是成功了：
```
...
Installation finished. No error reported.
```
此时`/mnt`目录下，应该有一个`./boot/grub`的文件夹：
```
/mnt/boot/grub ⌚ 20:54:33
$ ls
fonts  grubenv  i386-pc  locale
```

* 修复`/grub`位置
查看下 PC Ubuntu 的`/boot`,`/grub`是直接放置在`/boot`下的:
```
/boot/grub ⌚ 13:30:33
$ ls
fonts  gfxblacklist.txt  grub.cfg  grubenv  i386-pc  locale  unicode.pf2
```
而`grub-install /dev/sdb`安装的 GRUB 是`/mnt/boot/grub`，其中`/mnt`是 SSD `/dev/sdb2`分区，从 SSD 启动 Ubuntu 的话，`/dev/sdb2`会挂载为`/boot`，此时 GRUB 的位置是`/boot/boot/grub`。而当`grub-install /dev/dsa`安装 GRUB 到 PC Ubuntu 启动磁盘时，生成的`/grub`是在`/boot/grub`。`grub-install`的处理逻辑应该是先判断`/boot`路径是否存在，没有就新建。
所以，**要将`/mnt/boot/grub`移动到`/mnt/grub`**：
```
$ sudo mv /mnt/boot/grub /mnt/grub
```

### GRUB 引导修复类型
启动电脑后，当 GRUB 无法按照`boot.img` --> `core.img` --> `/boot/grub/`顺序执行时，会看到命令行界面，等待用户输入命令。此时可以通过输入 GRUB 内置的命令来修复 GRUB 引导。

`boot.img`是写在 MBR 中的，如果不能执行，直接跟 GRUB 引导方式说再见了，所以执行`boot.img`一般没问题。`boot.img`不能识别任何文件系统，`core.img`的位置是硬编码进`boot.img`的，所以执行`boot.img`一般没问题。因此，常见的引导问题集中在`/boot/grub/`，主要有两种，对应有两种引导修复模式：
* GRUB Rescue 模式
GRUB Rescue 模式是 GRUB 无法找到`/boot`分区，也就无法找到`/boot/grub/`。修复方法可以参考：[grub rescue 模式下修复][grub rescue模式下修复]。

* GRUB Normal 模式
GRUB Normal 模式是 GRUB 无法找到 GRUB 菜单`grub.cfg`，无法选择合适的内核或系统来启动。修复方法可以参考：[Boot GNU/Linux from GRUB][Boot GNU/Linux from GRUB]。

## 数据复制
该步骤是把 PC 硬盘中几个 Linux 分区的数据拷贝到 SSD 上对应的分区。
（*注意：PC Ubuntu 和 SSD Ubuntu 都有`/`、`/boot`、`/home`分区，阅读下文时注意辨别，我有时并没有写得很清晰。*）

### 操作方式
操作的套路是先将 SSD 的分区使用`mount`命令挂载为 PC 的`/mnt`，使用`cp`命令复制数据，再用`umount`命令移出这个分区；对下一个分区做同样操作。
* 挂载和移出操作
```
// 挂载
$ sudo mount /dev/sdb2 /mnt
// 移出
$ sudo umount /mnt
```
* 复制操作
使用`cp`指令要加`-r`，`-f`，`-a`参数，`-r`表示递归复制，`-f`表示强制覆盖，`-a`表示保留原文件的属性（mode，ownership，tiemstamps等）
```
$ sudo cp -rf -a source destination
```

### 复制`/boot`分区
SSD Ubuntu 的`/boot`从 PC Ubuntu 上看为`/dev/sdb2`，将`/dev/sdb2`挂载为 PC Ubuntu 的`/mnt`。安装 GRUB 之后，`/mnt`已经有`/grub`这个文件夹和默认的`lost+found`文件夹。
使用`cp`将 PC 的`/boot`中其他文件复制到`/mnt`。结果类似：
```js
/mnt/ ⌚ 13:56:06
$ ls | sort
abi-4.8.0-36-generic
config-4.8.0-36-generic
grub
initrd.img-4.8.0-36-generic
lost+found
memtest86+.bin
memtest86+.elf
memtest86+_multiboot.bin
System.map-4.8.0-36-generic
vmlinuz-4.8.0-36-generic
```

### 复制`/`分区
SSD Ubuntu 的`/`分区（根目录）比较特殊：一些子目录挂载了其他分区，并存在“伪目录”，[不同子目录有特定的用途][linux-directory-structure]。

所以复制`/`分区是有选择性的，不区分子目录进行复制，可能会提示“权限问题”、“无法访问”等错误。
* 不需要复制的目录
 * `/boot`,`/home`，`/mnt`挂载了其他分区
 * `/media` `/cdrom` 挂载可移除的媒体（cdrom 等）
 * `/swap`交换分区（不需要交换分区了）

* 需要复制的目录
 主要参考: [Linux操作系统备份之二][Linux操作系统备份之二]
 * `/bin` 系统可执行文件
 * `/etc` 系统核心配置文件
 * `/opt` 用户程序文件
 * `/root` root用户主目录
 * `/sbin` 系统可执行文件
 * `/usr` 程序安装目录
 * `/var` 系统运行目录

* 需要手动创建的目录
 在`/mnt`中需要给 SSD 的`/`创建几个**空目录**。
 * `/dev` 主要存放与设备（包括外设）有关的文件
 * `/proc`  正在运行的内核信息映射
 * `/sys` 硬件设备的驱动程序信息

 这几个目录是 Linux 内核启动后由内核来挂载并存放信息的，不能从运行中的 PC Ubuntu 复制过去，但是需要建立空目录，不然内核启动后会报类似错误：
```
mount: mount point /dev does not exist
```
创建命令：
```
$ sudo mkdir dev proc sys
```

* 操作策略
 * **每个目录单独执行复制命令，出错了好处理**。

### 复制`/home`分区
挂载 SSD Ubuntu`/home`到 PC Ubuntu `/mnt`，然后**全盘复制**：
```
$ sudo mount /dev/sdb4 /mnt
$ sudo cp -rf -a /home/* /mnt
```

### 挂载`/home`和`/boot`分区
SSD Ubuntu 的`/home`和`/boot`需要挂载到`/`，挂载方法为：修改`/ect/fstab`。
* 挂载`/dev/sda3`为 PC Ubuntu `/mnt`
* 使用`blkid`查看 SSD 各分区的 UUID
```
$ sudo blkid
...
/dev/sdb3 UUID="a5eb2b0c-2104-4afe-aa78-93396d3e0986" TYPE="ext4" PARTUUID="b2708ce0-07"
...
```
* 修改 SSD Ubuntu 的`fstab`文件
```
$ sudo vim /mnt/etc/fatab
```
`fstab`文件大概是这样子的：
```js
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda3 during installation
UUID=a5eb2b0c-2104-4afe-aa78-93396d3e0986 /               ext4    errors=remount-ro 0       1
#
# /boot was on /dev/sda2 during installation
UUID=8cba10c6-dff2-4300-a630-ab0e7a4782af /boot           ext4    defaults        0       2
#
# /home was on /dev/sda4 during installation
UUID=298ba5ad-d306-4b4a-aaa8-54312590dec6 /home           ext4    defaults        0       2
```

## GRUB 引导修复
将 SSD 通过 USB 插入到笔记本，开机，选择从 USB 启动。此时应该会是看到类似下图的画面。
![](/img/Ubuntu16.10_迁移到_SSD/GRUB-Prompt-crop.png)

说明已经进入到 GRUB 引导程序中，但是没有 GRUB 启动选项，无法继续引导了。距离成功仅剩一步：修复 GRUB 引导。

### 指定内核启动
* 指定`/boot`分区和`/grub`位置*（好像不需要这步，GRUB Rescue 才需要）*
```
// grub> root=hd0,gpt2
// grub> prefix=(hd0,gpt2)/grub
grub> set root=hd0,gpt2
grub> set prefix=(hd0,gpt2)/grub
```

* 设置启动的 Linux 内核
```
grub> linux /vmlinuz-4.8.0-36-generic ro root=/dev/sda2
```

* 设置虚拟内存
```
grub> initrd /initrd.img-4.8.0-36-generic
```

* 启动 SSD Ubuntu
```
grub> boot
```
到这一步应该可以启动 SSD 的 Ubuntu，但是下次重新开机，又需要手动指定内核才能启动，通过在 SSD Ubuntu 中重建 GRUB 引导可以解决该问题。

### 重建 GRUB 引导
从 SSD 开启 Ubuntu 成功后，执行以下命令：
```
$ sudo update-grub
$ sudo grub-install /dev/dsa
```
以上命令更新了 GRUB 可引导的系统/内核列表：`/boot/grub/grub.cf`，并重新安装了 GRUB。可以参考：[Grub2/Installing][Grub2/Installing]。

笔记本下次开机，就能看到类似画面：
![](/img/Ubuntu16.10_迁移到_SSD/grub-loader.jpg)

### 完成
将 SSD 放入笔记本内置硬盘位，将旧的 HDD 放到光驱位置，开机，完成！（撒花）！

## 结语
总共花了三天时间搞定这个事情，整理出文章花了N天，查看了很多资料，对计算机开机引导，硬盘分区和 GRUB 算是比较了解了。
现在笔记本有了 SSD + HDD，下一步可能会实践双硬盘的数据备份。
最后放上 HDD 凌乱的分区图，纪念这几年装机折腾的日子。**折腾中总有收获。**
![](/img/Ubuntu16.10_迁移到_SSD/old-disk-partition.png)

## Refenrences
* UEFI是什么？与BIOS的区别在哪里
> http://www.ihacksoft.com/uefi.html
[UEFI是什么？与BIOS的区别在哪里]:http://www.ihacksoft.com/uefi.html

* Linux：系统启动引导过程
> http://zhaodedong.leanote.com/post/Linux%EF%BC%9A%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E5%BC%95%E5%AF%BC%E8%BF%87%E7%A8%8B

* What’s the Difference Between GPT and MBR When Partitioning a Drive
>http://www.howtogeek.com/193669/whats-the-difference-between-gpt-and-mbr-when-partitioning-a-drive/
[What’s the Difference Between GPT and MBR When Partitioning a Drive]:http://www.howtogeek.com/193669/whats-the-difference-between-gpt-and-mbr-when-partitioning-a-drive/

* 6 Stages of Linux Boot Process (Startup Sequence)
> http://www.thegeekstuff.com/2011/02/Linux-boot-process/

[uefi-boot-how-does-that-actually-work-then]:https://www.happyassassin.net/2014/01/25/uefi-boot-how-does-that-actually-work-then/

* GRUB2 中文介绍
> https://my.oschina.net/guol/blog/37373
[GRUB2 中文介绍]:https://my.oschina.net/guol/blog/37373

[Slient-links]:Slient-links
[MBR]:https://en.wikipedia.org/wiki/Master_boot_record
[GPT]:https://en.wikipedia.org/wiki/GUID_Partition_Table
[Gparted Partition Editor]:http://gparted.sourceforge.net/
[File System]:https://en.wikipedia.org/wiki/File_system
[GRUB]:https://en.wikipedia.org/wiki/GNU_GRUB
[Ubuntu17.04也要移除 swap 分区]:http://blog.surgut.co.uk/2016/12/swapfiles-by-default-in-ubuntu.html
[BIOS boot partition]:https://en.wikipedia.org/wiki/BIOS_boot_partition
[protected MBR]:https://en.wikipedia.org/wiki/GUID_Partition_Table#Protective_MBR_.28LBA_0.29
[Installing-GRUB-using-grub_002dinstall]:http://www.gnu.org/software/grub/manual/html_node/Installing-GRUB-using-grub_002dinstall.html
[Linux操作系统备份之二]:http://www.cnblogs.com/xred/p/3898678.html
[linux-directory-structure]:https://slashmedia.wordpress.com/2007/12/23/linux-directory-structure/
[grub rescue模式下修复]:https://www.douban.com/note/66041888/
[Boot GNU/Linux from GRUB]:https://www.gnu.org/software/grub/manual/html_node/GNU_002fLinux.html
[Grub2/Installing]:https://help.ubuntu.com/community/Grub2/Installing

# END

