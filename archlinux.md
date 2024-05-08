# **The installation and the configuration of my archlinux**

### Some preparation before installing

First, you should have one installation media about the archlinux system.  

Then, just do it.

### Start to install

0. Ban reflector
   
   ```
   systemctl stop reflector.service
   ```
   
   The *reflector*  may choose some suitable mirror sources for us, but it's usually uncorrectly.

1. Make sure whether it's in UEFI mode
   
   ```
   ls /sys/fireware/efi/efivars
   ```
   
   If you can output something, you are in.

2. Connect to internet
   
   wireless connection:  
   
   ```
   iwctl
   
   device list
   
   station wlan0 scan
   
   station wlan0 get-networks
   
   station wlan0 connect YOUR-WIRELESS -NAME
   
   exit
   ```
   
   If succeed, you can the command to check out:

```
ping www.baidu.com
```

   annotation (if you have some trobule when you excecute the above commandline):  

   Also check the `ip link`command to see if a wireless was created; the naming of the wireless `network interfaces`starts with the letters `wl`,e.g. `wlan0` or `wlp2s0`.Then bring the interface up with:  

```
ip link set *interface* up
```

   make sure your device is not hard-blocked or softblock.

```
rfkill list all // this commandline may help you have some test.

rfkill unblock wifi
```

3. Update the system clock
   
   ```
   timedatectl set-ntp true
   
   timedatectl status
   ```

4. Section
   
   Use `fdisk` to part it.  
   
   You can use the below example:
   
   ```
   mkfs.ext4 /dev/sdax // home and root directory
   
   mkfs.vfat /dev/sdax // boot section
   ```

5. Mount 
   
   ```
   mount /dev/sdax /mnt
   
   mkdir /mnt/boot
   
   mount /dev/sdax /mnt/boot
   ```

6. The choice of mirror sources
   
   edit the mirror list :  
   
   ```
   vim /etc/pacman.d/mirrorlist
   ```
   
   Reserve the China mirror and delete the others.And add the ustc and tinghua mirror:  

```
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch

Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

7. Install system
   
   Some necessary basic packages:  
   
   ```
   pacstrap /mnt base base-devel linux linux-headers linux-firmware
   ```
   
   Some functionally basic packages:

```
pacstrap /mnt dhcpcd networkmanager vim sudo
```

8. Generate the `fstab` file
   
   ```
   genfstab -U /mnt >> /mnt/etc/fstab
   ```

9. Change root
   
   ```
   arch-chroot /mnt
   ```

10. Local setting
    
    ```
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    
    hwclock --systohc
    ```

11. Set `Locale` to localize
    
    ```
    vim /etc/locale.gen
    ```
    
    delete the `#` before **en_US.UTF-8** and **zh_CN.UTF-8** ,then:
    
    ```
    locale-gen
    
    echo 'LANG=en_US.UTF-8' > /etc/locale.conf
    ```

12. Set hostname
    
    ```
    vim /etc/hostname // just enter `archlinux`
    ```
    
    Then:  
    
    ```
    vim /etc/hosts
    ```
    
    Just enter:  
    
    ```
    127.0.0.1 localhost
    ::1 localhost
    
    127.0.1.1 arch
    ```

13. Passport for root
    
    ```
    passwd root
    ```

14. Intall Micro-code
    
    ```
    pacman -S intel-ucode #Intel
    
    pacman -S amd-ucode #AMD
    ```

15. Install bootstrap program
    
    ```
    pacman -S grub efibootmgr #grub是启动引导器，efibootmgr被 grub 脚本用来将启动项写入 NVRAM。
    
    grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
    ```
    
    Then edit /etc/default/grub
    
    ```
    vim /etc/default/grub
    ```
    
    Find `GRUB_CMDLINE_LINUX_DEFAULT` to modify the value of loglevel from 3 to 5. And in the same row, add `nowatchdog` arguement.
    
    Then generate grub file:
    
    ```
    grub-install --target=x86_64-efi --efi-directory=/efi --removable
    
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

16. Finish installation and add the User
    
    First of all, you should `exit`.  
    
    Then, add common user:  
    
    ```
    useradd -m -G wheel -s /bin/bash sxz #wheel附加组可sudo，以root用户执行命令 -m同时创建用户家目录
    ```
    
    New password for new user:  
    
    ```
    passwd sxz
    ```
    
    edit `sudoers` setting file:  
    
    ```
    EDITOR=vim visudo # 需要以 root 用户运行 visudo 命令
    ```
    
    find the below line before deleting the `#` of it:  
    
    ```
    #%wheel ALL=(ALL:ALL) ALL
    ```

### Desktop and Common application

1. Ensure the network environment:
   
   ```
   sudo systemctl stop iwd
   sudo systemctl disable iwd
   sudo systemctl enable --now dhcpcd 
   sudo systemctl enable --now NetworkManager
   ```
   
   Then test it:  

```
ping www.baidu.com

pacman -Syyu // upgrade the packages
```

2. Intall the desktop: KDE Plasma
   
   ```
   pacman -S plasma-meta konsole dolphin
   ```
   
   set greeter sddm:  

```
systemctl enable sddm
```

3. Enable the 32-bit support library
   
   ```
   vim /etc/pacman.conf
   ```
   
   delete the `#` before the two of the `[multilib]` 
   
   update it:  

```
pacman -Syyu
```

4. Basic packages
   
   ```
   sudo pacman -S sof-firmware alsa-firmware alsa-ucm-conf #一些可能需要的声音固件
   
   sudo pacman -S ntfs-3g #识别NTFS格式的硬盘
   
   sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei #安装几个开源中文字体 一般装上文泉驿就能解决大多wine应用中文方块的问题
   
   sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra #安装谷歌开源字体及表情
   
   sudo pacman -S firefox chromium #安装常用的火狐、谷歌浏览器 sudo pacman -S ark #与dolphin同用右键解压
   
   sudo pacman -S p7zip unrar unarchiver lzop lrzip #安装ark可选依赖 sudo pacman -S packagekit-qt5 packagekit appstream-qt appstream #确保Discover(软件中心）可用 需重启
   
   sudo pacman -S gwenview #图片查看器
   
   sudo pacman -S git wget kate bind #一些工具
   ```

5. Set DNS for your privacy and secruity
   
   ```
   vim /etc/resolv.conf
   ```
   
   then, enter below contents:  

```
nameserver 8.8.8.8
nameserver 2001:4860:4860::8888
nameserver 8.8.4.4
nameserver 2001:4860:4860::8844

sudo chattr +i /etc/resolv.conf
```

6. Intall `yay`
   
   ```
   sudo pacman -S yay
   ```

7. Install the input method:  
   
   ```
   sudo pacman -S fcitx5-im #基础包组
   
   sudo pacman -S fcitx5-chinese-addons #官方中文输入引擎
   
   sudo pacman -S fcitx5-pinyin-zhwiki #中文维基百科词库
   
   sudo pacman -S fcitx5-material-color #主题
   ```
   
   set environment variables( edit the file ) : `EDITOR=vim sudoedit /etc/environment` : 

```
GTK_IM_MODULE=fcitx

QT_IM_MODULE=fcitx

XMODIFIERS=@im=fcitx

SDL_IM_MODULE=fcitx
```

   打开 *系统设置* > *区域设置* > _输入法_，先点击`运行Fcitx`即可，拼音为默认添加项。

### Magic college

1. v2ray and v2ray
   
   ```
   yay -S v2ray v2raya
   
   sudo systemctl enable --now v2ray
   
   sudo systemctl enable --now v2raya
   ```

### Some practical packages and changes

1. /etc/pacman.conf
   
   ```
   [archlinuxcn]
   Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
   ```
   
   ```
   sudo pacman -Sy archlinuxcn-keyring
   ```

2. zsh
   
   ```
   yay -S zsh oh-my-zsh-git zsh-syntax-highlighting zsh-autosuggestions zsh-completions
   chsh -s /bin/zsh
   sudo ln -s /usr/share/zsh/plugins/zsh-syntax-highlighting /usr/share/oh-my-zsh/custom/plugins/
   sudo ln -s /usr/share/zsh/plugins/zsh-autosuggestions /usr/share/oh-my-zsh/custom/plugins/
   ```

3. Octopi

4. Coding
   
   ```
   yay -S  visual-studio-code-bin  go  gdb  docker   python-pip cargo meson  cmake gtk4
   ```

5. Other 
   
   ```
   yay -S google-chrome marktext
   ```
