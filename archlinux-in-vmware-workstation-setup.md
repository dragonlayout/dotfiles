
# Arch Linux Installation guide

## Pre-installation

- Update the system clock
    
    ```bash
    timedatectl status
    timedatectl set-ntp true
    ```
    
- Partition the disk
    
    ```bash
    fdisk -l
    fdisk /dev/sda
    fdisk command: 
    g -> gpt
    d -> delete a partition
    n -> new a paritiion
    t -> partition type
    w -> save
    
    types:
    EFI system: +512M
    Linux swap: +4G
    Linux system
    ```
    
- Format the partition
    
    ```bash
    mkfs.fat -F32 /dev/sda1
    mkfs.ext4 /dev/sda3
    mkswap /dev/sda2
    ```
    
- Mount the file systems
    
    ```bash
    mount /dev/sda3 /mnt
    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot
    swapon /dev/sda2
    ```
    
## Installation

1. Select the mirrors

    ```bash
    vim /etc/pacman.d/mirrorlist
    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
    ```

2. Install essential packages
    
    ```bash
    pacstrap /mnt base linux-lts linux-firmware neovim networkmanager
    ```
    
## Configure the system

1. Fstab
    
    The fstab file can be used to define how disk partitions, various other block devices, or remote file systems should be mounted into the file system.
    
    ```bash
    genfstab -U /mnt >> /mnt/etc/fstab
    ```
    
2. Chroot
    
    ```bash
    arch-chroot /mnt
    ```
    
3. Time zone
    
    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    # the hardware clock is set to UTC
    hwclock --systohc
    ```
    
4. Localization
    
    ```bash
    # uncomment en_US.UTF-8 UTF-8 and zh_CN.UTF-8 UTF-8 /etc/locale.gen
    locale-gen
    echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
    ```
    
5. Network configuration
    
    ```bash
    echo 'archlinux' >> /etc/hostname
    # add follow lines to /etc/hosts
    127.0.0.1	localhost
    ::1		    localhost
    127.0.0.1	archlinux
    ```
    
6. Root password
    
    ```bash
    passwd
    ```
    
7. Boot loader
    
    ```bash
    
    pacman -S grub efibootmgr 
    grub-install --target=x86_64-efi --efi-directory=/boot
    grub-mkconfig -o /boot/grub/grub.cfg
    exit
    umount -R /mnt
    reboot
    ```
    

## Post installation

- 关闭响铃
    
    ```bash
    rmmod pcspkr
    nvim /etc/modprobe.d/nobeep.conf
    blacklist pcspkr
    ```
    
- 网络配置
    
    ```bash
    # start networkmanager
    systemctl start NetworkManager.service
    systemctl enable NetworkManager.service
    ```
    
- System administration
    
    ```bash
    useradd -m -G wheel  -s /bin/bash chenlong
    passwd chenlong
    pacman -S sudo
    # uncomment %wheel ALL=(ALL) ALL to /etc/sudoers
    # add user to video grouop
    ```
    
- 时间同步
    
    ```bash
    timedatectl set-ntp true
    ```

- ssh

    ```
    sudo pacman -S openssh
    sudo systemctl start sshd
    sudo systemctl enable sshd
    ```

- shell
    
    ```bash
    # onmyzsh
    sudo pacman -S zsh git
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
    # plugin
    # 1. zsh-autosuggestions
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    ```

- Display server
    
    ```bash
    sudo pacman -S xorg-server xorg-apps
    ```

- open-vm-tools

    ```bash
    sudo pacman -S open-vm-tools
    sudo systemctl start vmtoolsd.service vmware-vmblock-fuse.service
    sudo systemctl enable vmtoolsd.service vmware-vmblock-fuse.service

    sudo pacman -S xf86-input-vmmouse xf86-video-vmware mesa
    # 复制粘贴
    vmware-user
    ```

- Display manager
    
    ```bash
    sudo pacman -S xorg-xinit
    cp /etc/X11/xinit/xinitrc ~/.xinitrc
    # exec bspwm

    # ~/.zprofile
    # -- -maxbigreqsize 127 fix alacritty crash https://github.com/alacritty/alacritty/issues/3500
    if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
        exec startx -- -maxbigreqsize 127
    fi
    
    # ~/.Xresources
    Xft.dpi: 144

    ! These might also be useful depending on your monitor and personal preference:
    Xft.autohint: 0
    Xft.lcdfilter:  lcddefault
    Xft.hintstyle:  hintfull
    Xft.hinting: 1
    Xft.antialias: 1
    Xft.rgba: rgb
    ```

- Window managers
    
    ```bash
    # using bspwm & sxhkbd
    sudo pacman -S bspwm sxhkd
    # config files
    install -Dm755 /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/bspwmrc
    install -Dm644 /usr/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd/sxhkdrc
    ```

- User directories

    ```bash
    sudo pacman -S xdg-user-dirs
    xdg-user-dirs-update
    ```

- Terminal

    ```bash
    sudo pacman -S alacritty
    # 'alacritty': unknown terminal type.
    alias ssh='TERM=xterm-256color ssh'
    ```

- Audio

    ```bash
    sudo pacman -S pulseaudio pavucontrol
    sudo nvim /etc/pulse/client.conf 
    # autospawn = yes
    ```

- File manager

    ```bash
    sudo pacman -S thunar thunar-volman gvfs thunar-archive-plugin
    sudo pacman -S file-roller 
    sudo pacman -S p7zip unrar
    # custom actions
    alacritty --working-directory %f
    ```

- 字体
    
    ```bash
    # 中文字体
    sudo pacman -S noto-fonts
    sudo pacman -S noto-fonts-cjk
    sudo pacman -S noto-fonts-emoji
    sudo pacman -S noto-fonts-extra
    # 字形显示不对问题, 修改简体中文优先级
    echo ~/.config/fontconfig/fonts.conf
    <?xml version="1.0"?>
    <!DOCTYPE fontconfig SYSTEM "fonts.dtd">
    <fontconfig>
      <alias>
        <family>sans-serif</family>
        <prefer>
          <family>Noto Sans CJK SC</family>
          <family>Noto Sans CJK TC</family>
          <family>Noto Sans CJK JP</family>
        </prefer>
      </alias>
      <alias>
        <family>monospace</family>
        <prefer>
          <family>Noto Sans Mono CJK SC</family>
          <family>Noto Sans Mono CJK TC</family>
          <family>Noto Sans Mono CJK JP</family>
        </prefer>
      </alias>
    </fontconfig>
    # fc-cache -fv
    # fc-match -s | grep 'Noto Sans CJK'
    
    # nerdfont
    yay -S nerd-fonts-complete
    ```

- yay

    ```bash
    mkdir ~/Softwares
    sudo pacman -S base-devel
    git clone https://aur.archlinux.org/yay.git
    export GOPROXY=https://goproxy.cn
    makepkg -si
    ```

- 浏览器
    
    ```bash
    yay -S google-chrome
    google-chrome-stable --proxy-server="http://localhost:7890"
    # switchyomega: https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
    ```

- Polybar

    ```bash
    yay -S polybar
    ```

- 输入法
    
    ```bash
    # 中文输入法
    sudo pacman -S fcitx5-rime
    sudo pacman -S fcitx5-im
    # /etc/environment
    GTK_IM_MODULE=fcitx
    QT_IM_MODULE=fcitx
    XMODIFIERS=@im=fcitx
    ```
    
- clipboard

    ```bash
    # 未使用
    yay -S rofi-greenclip
    rofi -modi "clipboard:greenclip print" -show clipboard -run-command '{cmd}'
    ```
    
- dunst

    `sudo pacman -S dunst`

    
- 命令行代理
    
    ```bash
    sudo pacman -S proxychains-ng
    /etc/proxychains.conf
    ```
- network utils

    ```bash
    # nc
    sudo pacman -S openbsd-netcat
    # telnet
    sudo pacman -S inetutils
    ```
    
- sdkman
    
    ```bash
    # 未使用
    sudo pacman -S zip unzip
    curl -s "https://get.sdkman.io" | bash
    maven
    cxf
    ```
    
- openjdk
    
    ```bash
    sudo pacman -S jdk8-openjdk
    sudo pacman -S openjdk8-doc openjdk8-src
    # archlinux-java
    ```
    
- jetbrains toolbox
    
    ```bash
    yay -S jetbrains-toolbox
    # Intellij IDEA
    # plugins:
    # 1. IdeaVim
    # 2. GsonFormatPlus
    # 3. MyBatisCodeHelperPro
    # 4. Protocol Buffers
    # 5. MapStruct Support
    # Goland
    # Datagrip
    ```

- neofetch
    
    `sudo pacman -S neofetch`
    
- htop
    
    `sudo pacman -S htop`
    
- Docker
    
    ```bash
    sudo pacman -S docker
    sudo systemctl start/enable docker.service
    # add user to docker user group
    sudo usermod -aG docker chenlong
    sudo mkdir -p /etc/docker
    # 添加阿里云镜像地址
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://ifsn0r8t.mirror.aliyuncs.com"]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

- Rofi
    ```bash
    sudo pacman -S rofi
    ```

- youtube-dl

    ```bash
    sudo pacman -S yt-dlp
    ```

- appimagelauncher

    ```bash
    yay -S appimagelauncher
    ```

- tree

    ```bash
    sudo pacman -S tree
    ```

- wget

    ```bash
    sudo pacman -S wget
    ```
- dotdrop

    ```bash
    # doc: https://dotdrop.readthedocs.io/en/latest/usage/
    yay -S dotdrop
    ```
