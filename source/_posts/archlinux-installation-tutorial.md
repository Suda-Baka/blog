---
title: Arch Linux 安装教程（以 BIOS/MBR 为例）
date: 2025-04-11 18:03:40
categories: 
- 技术
tags:
- Arch Linux
- 教程
---

# 1 Arch Linux 的安装

**注意：本教程以 BIOS/MBR 引导模式为例，UEFI 引导模式请参考 [Arch Linux UEFI 安装教程](https://linux.do/t/topic/8990)。**

**注意：本教程以 Arch Linux 2025.04.01 版本为例，其他版本请自行调整。**

## 1.1 前置工作

```bash
$ ls /sys/firmware/efi/efivars
# 验证引导模式（如果目录不存在，即为 Legacy BIOS 引导模式；反之，请使用以 UEFI 为例的教程
$ iwctl
```

运行 `iwctl` 进行联网（如果是有线网络，可直接跳到 `ping archlinux.org` 这一步）。

```bash
[iwctl]# device list
# 列出 WiFi 设备（一般为 wlan0，这里以 wlan0 为例）

[iwctl]# station wlan0 scan
# 扫描网络

[iwctl]# station wlan0 get-networks
# 列出可用网络

[iwctl]# station wlan0 connect XXX
# 连接到XXX（XXX改成你的 WiFi 名称）

[iwctl]# exit
# 退出 iwctl

$ ping archlinux.org
# 检查网络连接（如果不停的有输出内容，即为联网成功，按 `Ctrl+C` 退出）

$ reflector --country China --save /etc/pacman.d/mirrorlist
# 换源（注意大小写）

$ systemctl stop reflector
# 关闭 reflector 服务

$ vim /etc/pacman.d/mirrorlist
# 编辑 /etc/pacman.d/mirrorlist 文件，保留需要的源【一般推荐使用中科大源（USTC）或清华（TUNA）源】

# 如果不会使用 vim ，记住这三个就行了：
# 按键盘上面的 `I` 或者 `Insert` 键进入编辑模式
# 按 `ESC` 退出编辑模式
# 输入 `:wq` （冒号别漏了）保存并退出

$ timedatectl set-ntp true  
# 同步时间

$ timedatectl status 
# 检查服务状态

$ pacman -Syy 
# 同步数据
```


## 1.2 分区挂载

{% note warning %}
**警告：此过程必须慎重（尤其是对于双硬盘/多硬盘等存有大量或高价值数据者），严重者可能会丢失所有数据！**
{% endnote %}

**此处使用 SATA 硬盘为例**

**如果你使用的是 NVMe 硬盘，请将 `/dev/sda` 替换为 `/dev/nvme0n1` 或者其他对应的设备名称。**

**第一种方法**：使用 cfdisk （方便快捷，推荐新手使用）

```bash
$ cfdisk /dev/sda
```

**第二种方法**：使用 fdisk

```bash
$ fdisk /dev/sda
# 使用 fdisk 对 sda 进行相关操作   

# 步骤如下：
Command (m for help): o    
# 输入 `o` 新建 MBR 分区表

Command (m for help): n
# 输入 `n` 创建新分区

Select (default p): p  
# 这里按 `Enter` 键创建主分区（如果想创建逻辑扩展分区请输入 `e` ）

Partition number (1-4, default 1):  
First sector (2048-X, default 2048):    
# 这两步按 `Enter` 键

Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-X, default X): +10G   
# 输入+10G   

Command (m for help): t
# 输入 `t` 更改分区类型
 
Hex code or alias (type L to list all): 82
# 输入 82 ，创建 swap 分区

Command (m for help): n
# 输入 `n` 创建新分区，然后一直按 `Enter` 键，把剩下的空间全部分配给/分区

Command (m for help): w
# 输入 `w` 写入
```


进入 cfdisk 程序，并且对第一块硬盘进行编辑操作（得看你这里是第几块硬盘安装系统，如果是第二块就 sdb ，第三块就 sdc ...以此类推）。
进入以后你会看到你的分区信息。 

操作方法（按键盘上下键选择分区，左右键选择功能）：
- 如果你想删除分区，请将光标移到 `delete` 并按下回车
- 如果你想创建分区则移到列表下的 free space 按 `new`  
- 如果是调节你想要的分区的大小，请按 `resize` 
- 更改分区类型按 `type` 

请准备两个空闲分区来安装 Arch Linux 。

**准备一个任意大小（建议 20GB 以上）的分区**（假设为 **sda1** ，请根据实际情况判断），和一个**等于你的 RAM （运行内存） 容量的 3/4 或全部的分区**（假设为 **sda2** ，请根据实际情况判断）。

将 sda1 的分区类型改为 `83 Linux ` ， sda2 的分区类型改为 `82 Linux Swap/Solaris` 

更改完毕后按左右键选择 write 按回车， `输入yes` ，再按一下。

**至此， arch 的分区工作就完成了。**


## 1.3 安装 Arch linux

```bash
$ pacstrap /mnt linux linux-firmware linux-headers base base-devel vim bash-completion iwd dhcpcd networkmanager
# 安装 linux & linux-firmware & linux-headers & base & base-devel & vim & bash- completion & iwd & dhcpcd & networkmanager

$ genfstab -U /mnt >> /mnt/etc/fstab
# 生成 /mnt/etc/fstab 文件（注意大小写）

$ cat /mnt/etc/fstab 
# 查看 /mnt/etc/fstab 文件是否正确（如果不正确，请重新分区、挂载、 pacstrap ） 

$ arch-chroot /mnt 
# 进入目标系统 

$ pacman -Syy 
# 同步数据 

$ pacman -S grub amd-ucode intel-ucode 
# 安装 grub & amd-ucode 或 intel-ucode （ AMD 的 CPU 安装 amd-ucode ， intel 的 CPU 安装 intel- ucode ） 

$ lsblk 
# 查看硬盘名称 

$ grub-install /dev/sda 
# 将 grub 写入 sda 

$ grub-mkconfig -o /boot/grub/grub.cfg 
# 生成 /boot/grub/grub.cfg 文件 

$ systemctl enable iwd dhcpcd 
# 开机自启 iwd 服务和 dhcpcd 服务 

$ passwd root 
# 设置 root 密码
```


## 1.4 重启并进入下一步工作

```bash
$ exit
# 退出目标系统

$ umount /mnt
# 卸载 /mnt 目录（这里的意思是取消挂载，不是卸载软件的卸载！）

$ reboot
# 重启并登陆 root （彻底关机那一刻请立即拔出安装 U 盘，如果觉得手慢就输入 poweroff ）
```


**至此，Archlinux的基本安装就已经结束了。但是还没完，因为还要配置、本地化和安装桌面环境等操作。**

# 2 Arch Linux 的配置

```bash
$ iwctl
# 运行 iwctl （使用方法请参考第一阶段的联网部分；台式机可跳过）

$ ping archlinux.org   
# 检查网络连接（如果不停的有输出内容，即为联网成功；按 `Ctrl+C` 终止输出）

$ vim /etc/hostname
# 创建 /etc/hostname 文件，加入以下内容：
arch
# 将主机名设置为 arch

$ vim /etc/hosts

# 编辑 /etc/hosts 文件，在末尾加入以下内容：    
127.0.0.1 localhost    
::1 localhost   
127.0.1.1 arch.localdomain arch   
# 配置 hosts 文件，映射 IP 地址和主机名   

$ timedatectl set-timezone Asia/Shanghai && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && hwclock --systohc   
# 设置时区为上海（注意大小写）

$ timedatectl set-ntp true
# 同步时间

$ timedatectl status
# 检查服务状态

$ vim /etc/skel/.bashrc
# 编辑 /etc/skel/.bashrc 文件，在开头加入以下内容：
export EDITOR='vim'
# 设置 vim 为默认文本编辑器（注意大小写）

$ cp -a /etc/skel/. ~/
# 复制 /etc/skel 目录下的文件到主目录

$ reboot
# 重启并登陆 root

$ useradd --create-home arch
# 添加普通用户，用户名为 arch

$ passwd arch
# 设置 arch 密码

$ usermod -aG adm,wheel,storage arch
# 将 arch 添加到相应的组中

$ id arch
# 查看用户组是否添加到相应的组中

$ visudo
# 设置用户权限，删除 `%wheel ALL=(ALL:ALL) ALL` 前面的 `#`（取消对 wheel 组的限制，允许 arch 用户使用 sudo 命令）

$ reboot
# 重启并登陆 root

$ vim /etc/locale.gen
# 编辑 /etc/locale.gen 文件，删除 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 前面的 `#` （开启语言支持）

$ locale-gen
# 生成语言

$ vim /etc/locale.conf
# 创建/etc/locale.conf文件，加入以下内容：
LANG=en_US.UTF-8
# 设置语言为en_US.UTF-8（注意大小写）

$ reboot
# 重启并登陆

$ vim /etc/pacman.conf
# 编辑 /etc/pacman.conf 文件，删除 `[multilib]` 区域的所有 `#` （开启 32 位支持）并在末尾加入以下内容：
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
# 添加 archlinuxcn 源（一般推荐使用中科大源，除了可以添加 archlinuxcn 源外，还可以添加 arch4edu 源、blackarch 源以及各种私有源，后面会提到，注意大小写）

$ pacman -Syy
# 同步数据

$ pacman -S archlinuxcn-keyring
# 安装 archlinuxcn-keyring 

$ rm -rf /etc/pacman.d/gnupg && pacman-key --init && pacman-key --populate archlinux && pacman-key --populate archlinuxcn
# 生成新的密钥环并重新签署密钥（安装 archlinuxcn-keyring 不报错时可跳过）

$ pacman -Syy
# 再次同步数据

$ pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau xf86-video- ati
# 安装 AMD 核显相关驱动

$ pacman -S mesa lib32-mesa xf86-video-intel vulkan-intel lib32-vulkan-intel
# 安装 intel 核显相关驱动

$ pacman -S alsa-utils alsa-plugins alsa-oss alsa-firmware sof-firmware alsa-ucm- conf pulseaudio pulseaudio-alsa pulseaudio-bluetooth
# 安装声音相关驱动

$ systemctl enable bluetooth
# 开机自启 bluetooth 服务

$ reboot   
# 重启并登陆root
```


# 3 安装 KDE 桌面环境

```bash
$ pacman -Syy
# 同步数据

$ pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-emoji-blob noto-fonts-extra adobe-source-han-sans-otc-fonts adobe- source-han-serif-otc-fonts wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy- zenhei ttf-arphic-extra ttf-arphic-ukai ttf-arphic-uming
# 安装字体（请根据需要自行补充，这里只安装常用的包）

$ pacman -S plasma plasma-meta konsole dolphin kate ark gwenview vlc firefox firefox-i18n-zh-cn packagekit-qt5
# 安装 KDE 桌面及软件（这里只安装最必要的包，如果想完整使用 KDE 的各种功能请根据对应提示安装需要的包）

$ systemctl enable sddm
# 开机自启 sddm 服务

$ systemctl disable iwd
# 取消自启 iwd 服务

$ systemctl enable NetworkManager
# 开机自启 NetworkManager 服务（注意大小写）

$ reboot 
# 重启
```


# 4 KDE 的中文化

System Settings（系统设置）>> Regional Settings（区域设置）>> Language（语言）>>Add language（添加语言），找到简体中文后点 Add（添加）。

**添加简体中文后，将其移到最上面，并删除其他多余语言，否则会出现汉化不全的情况。上述操作完成后，点击Apply（应用）。**

System Settings（系统设置）>>Regional Settings（区域设置）>>Formats（格式）>>Region（区域），选择简体中文（中国）。

上述操作完成后，点击Apply（应用）。

```bash
$ reboot  
# 重启
```


# 5 AUR helper 的安装

```bash
$ sudo pacman -S yay
# 安装 yay（它在某些时候可以替代 pacman 来安装软件）

$ sudo pacman -S yay
# 安装 yay（它在某些时候可以替代 pacman 来安装软件）
```

---

# 6 添加其他软件源

```bash
$ sudo pacman-key --recv-keys 7931B6D628C8D3BA && sudo pacman-key --finger 7931B6D628C8D3BA && sudo pacman-key --lsign-key 7931B6D628C8D3BA
# 导入 arch4edu 源的 GPG key

$ sudo vim /etc/pacman.conf
# 编辑 /etc/pacman.conf 文件，在末尾加入以下内容：
[arch4edu] Server = https://mirrors.tuna.tsinghua.edu.cn/arch4edu/$arch
# 添加 arch4edu 源

$ sudo pacman -Syy
# 同步数据

$ sudo pacman -S arch4edu-keyring
# 安装 arch4edu-keyring

$ sudo rm -rf /etc/pacman.d/gnupg && sudo pacman-key --init && sudo pacman-key --populate archlinux && sudo pacman-key --populate archlinuxcn && sudo pacman-key --populate arch4edu
# 生成新的密钥环并重新签署密钥

$ sudo pacman -Syy
# 再次同步数据
```

# 7 更新系统

```bash
$ sudo pacman -Syu
# 更新系统

$ yay
# 更新系统及 AUR 软件
```

{% note warning %}
在更新时请先查看 ArchLinux 官网的新闻公告，看是否需要升级时人为干预，请勿无脑更新。
{% endnote %}


# 8 END

想再看一遍本教程吗？输入 `sudo rm -rf /*` ，你会回来的。
