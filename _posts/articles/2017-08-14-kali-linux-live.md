---
layout: post
title: 用 Kali Linux 创建U盘随身系统
excerpt: "用 Kali 创建可持久化（persistence）的U盘随身系统，支持从 PC 和 Mac 启动，并且持久化的硬盘分区大小不受 4GB 限制"
categories: articles
tags: [Unix-like]
comments: true
share: true
---



之前，想通过 Linux 上的 `hdparm` 命令，擦除降速了的 SanDisk CZ80，然而装在 VirtualBox 里的 Linux 是没法对宿主机的 USB 设备进行 ATA Secure Erase 操作的。于是萌生了制作 Linux Live USB 的想法。

曾经也用过 ubuntu 的 Live CD。既然是 CD 嘛，那整个系统都是只读的，所做的任何修改、安装的任何软件都是重启就丢，即便把 `.iso` 制作成U盘，也还是这样。

也有各种第三方制作工具，可以实现修改持久化，就是所谓 *persistence*。比如 LinuxLive USB，实现持久化的方法，是在U盘的 FAT32 文件系统上创建一个文件，用来保存修改。由于 FAT32 不支持 4GB 以上的大文件，持久化的存储空间最多也就只能是 4GB。现在 USB 3.0 接口的U盘动辄 16GB 起步，4GB 的限制浪费了大部分空间。那能把U盘格式化成 exFAT，突破限制吗？对不起，不行。并且，这些第三方的持久化模式对 Mac 的支持也不好，要么直接不能启动，要么只能启动为非持久模式。

归根结底，像 ubuntu 之类的发行版提供的 Live 模式，只是为了让新用户体验，从设计上就没打算让你装U盘上带着干活。

还有一种办法，就是把系统直接安装在U盘上。这样做对发行版没有多少要求，也不用担心数据关机就丢，但是兼容性差：这台机器上用得好好的，到了另外一台上面可能就出现各种驱动问题。

有没有这么一种发行版，可以从U盘启动，支持持久化，还不挑计算机呢？



|        |                  ubuntu                  |                  ubuntu                  |   ubuntu    |                   Kali                   |
| -----: | :--------------------------------------: | :--------------------------------------: | :---------: | :--------------------------------------: |
|   制作工具 | [LinuxLive USB](http://www.linuxliveusb.com) | [UNetbootin](https://unetbootin.github.io) |    `dd`     | [Win32 Disk Imager](https://launchpad.net/win32-image-writer)/`dd` |
|   制作平台 |                 Windows                  |           Windows/Linux/macOS            | Linux/macOS |           Windows/Linux/macOS            |
|    持久化 |                   仅限PC                   |                   仅限PC                   |      无      |                  PC/Mac                  |
| 4GB 限制 |                    有                     |                    无                     |      —      |                    无                     |



## Kali Linux

[Kali Linux](https://www.kali.org) 是一个基于 Debian 的、用于数字取证和渗透测试的发行版。其用户的使用场景往往是，绕过目标机原操作系统，直接从安装了 Kali 的U盘或光盘启动，事了拂衣去，深藏身与名。Live USB系统是 Kali 用户很常用的一种方式，发行版设计的时候就考虑到了持久化等问题。官方文档上直接提供了[制作U盘系统](https://docs.kali.org/downloading/kali-linux-live-usb-install)并[创建持久化分区](https://docs.kali.org/downloading/kali-linux-live-usb-persistence)的教程，不用再去搜索第三方工具了。

## 制作U盘系统

下载并**验证**得到的 `.iso` 文件。官方特别强调，应当从官方来源下载，并且必须进行 SHA256 验证。考虑到速度问题，我是从[科大源](https://mirrors.ustc.edu.cn/kali-images/)下载的（好像并不是官方来源？），但是下载后按照[文档](https://docs.kali.org/introduction/download-official-kali-linux-images)要求进行了验证。在下载文件里植入恶意内容的事情已经发生过很多次了，Kali 作为一款取证和渗透用的发行版，如此要求并不是杞人忧天。

Windows 上还需要第三方工具 Win32 Disk Imager，Linux/macOS 只要 `dd` 命令即可。

当然，首先你得认准了要写入的设备才行。在 Windows 上，好像并不是件难事，毕竟谁也不会把 ISO 文件写到 C 盘去。Linux 和 macOS 上就困难一些，`sda`、`disk2` 之类的设备太多，一不留神真的错把主机硬盘给覆写了，哭都来不及。建议先把计算机上能拔掉的 USB 设备全都拔掉，也包括制作系统的U盘，然后 Linux 用 `fdisk -l` 、macOS 用 `diskutil list`，列出所有存储设备；插入U盘，再列出设备。两相对照，新增的 `/dev/sdd` 或者 `/dev/disk2` 之类，就是你要写的U盘了。

如果是 macOS，还需要把刚刚插入的U盘卸载：

```shell
diskutil unmountDisk /dev/diskN
```

为了防止有人直接复制命令，我用 `diskN` 来指代设备， `diskN` 不是正常的设备名，万一执行了也会因为找不到设备而退出。使用时应该把 `N` 替换成U盘设备实际对应的数字。

然后就是写入。`cd` 到 ISO 文件所在目录，

```bash
dd if=kali-linux-2017.1-amd64.iso of=/dev/diskN bs=1m
```

可能会需要 root 权限。

其中 `bs` 是一次读写的字节数上限。Linux 建议设置为 `bs=512k`。如果 macOS 出现 *Invalid number '1m'* 的提示，那你可能用的是 GNU 版本的 `dd`， 改成 `bs=1M` 即可。

macOS 上可以把设备名称从 `/dev/diskN` 改为 `/dev/rdiskN` （其中 *r* 代表 *raw*），可以显著提高写入速度。

然后就是漫长的、无响应的阻塞和等待。直到出现类似下面的提示时，U盘系统就制作完成了。

```shell
2911+1 records in
2911+1 records out
3053371392 bytes transferred in 2151.132182 secs (1419425 bytes/sec)
```



### 持久化

但是这时的 Live 系统还不支持持久化。

Kali 的持久化是直接在U盘上开辟分区的，大小仅受U盘容量限制，还支持加密。持久化操作应当在 Linux 上进行，其他系统不方便建立 **ext3** 文件系统。我是在 ubuntu Live 上操作的，似乎直接在刚才建立的 Kali Live 上操作也可以。

为了便于说明，我们作如下假设：

* 你拥有系统的 **root** 权限，
* U盘的设备名为 `/dev/sdb`，
* U盘容量为 **8GB**。

先用 `fdisk -l` 检查查看一下U盘，你应该能看到 2 个分区，`/dev/sdb1` 和 `/dev/sdb2`，我们将要创建的是 `/dev/sdb3`。

然后用 `parted` 划分分区：

```shell
end=7gb
read start _ < <(du -bcm kali-linux-2016.2-amd64.iso | tail -1); echo $start
parted /dev/sdb mkpart primary $start $end
```

Shell 变量 `start` 和 `end` 代表分区的起始点。因为U盘容量只有 8GB，我们把分区的 `end` 设为 7GB；`start` 是通过 ISO 文件的大小算出来的，所以这时的工作目录下需要有一份和制作U盘系统一致的 ISO 文件。如果 ISO 文件的大小是 3GB，新分区的大小大约就是 `end` - `start` = 4GB。如果U盘容量更大，`end` 也可以设得大些。

`parted` 可能会提示，无法使用你指定的 `start`，接受它建议的数值就好；还会提示分区没有放在 optimal location，忽略即可。`gparted` 是 `parted` 的图形界面版本，也可以使用。

第三步，创建 ext3 文件系统并命名为 *persistence*。

```shell
mkfs.ext3 -L persistence /dev/sdb3
e2label /dev/sdb3 persistence
```

第四步，挂载新的 `/dev/sdb3`，创建配置文件。

```shell
mkdir -p /mnt/my_usb
mount /dev/sdb3 /mnt/my_usb
echo "/ union" > /mnt/my_usb/persistence.conf
umount /dev/sdb3
```

完成！

从制作完成的U盘启动，选择进入 *Live USB Persistence* ，创建的修改就都会被保存在U盘上了。



## 参考资料：

* [Making a Kali Bootable USB Drive](https://docs.kali.org/downloading/kali-linux-live-usb-install)
* [Kali Linux Live USB Persistence](https://docs.kali.org/downloading/kali-linux-live-usb-persistence)