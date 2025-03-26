# Install
---
## WiFi
---
Connect to wifi with iwctl.  Verify that address is given once exited.
```
# iwctl

device list
station wlan0 connect <SSID>
exit

# ip a
```

## SSH
I SSH from another machine because it's easier from an existing setup, plus I have copy and paste when needed.
```
# systemctl enable sshd
```
Set a bullshit root password so you can SSH in
```
# passwd
```

## Disk setup
Find your disk
```
# lsblk
```
Setup 3 partitions with gdisk.
```
# gdisk /dev/sda
```
Type __p__ to see the existing partitions.  Type __d__ to delete a partition.  Clear them all off.  Start from scratch.
```
d
3
d
2
d
```
Use __n__ to create a new partition.  First question is what number partition.  Second number is where to begin the partition.  Leave it default.  Third is where to send the sector.  Using __+__ before the number will make sizing a lot easier.  Finally the partition code.
|use  |partition|first sector|last sector|code|
|-----|---------|------------|-----------|----|
|boot |1        |default     |+4G        |ef00|
|swap |2        |default     |+32G       |8200|
|root |3        |default     |default    |8300|
Swap size is whatever size your RAM is, usually.
## Partitions
FAT32 on boot.
```
# mkfs.fat -F32 /dev/sda1
```
Swap
```
# mkswap /dev/sda2
```
Root
```
# mkfs.ext4 /dev/sda3
```
## Mounts
```
# swapon /dev/sda2
# swapon -a
# mount /dev/sda3 /mnt
# mkdir /mnt/boot
# mount /dev/sda1 /mnt/boot
```
Don't __mkdir /mnt/boot__ before mounting __/dev/sda3__ oyay -S otf-font-awesome noto-fonts powerline-fonts noto-fonts-cjk geist-font ttf-hack-nerdr it won't be inside the partition.
## Install Arch
```
# pacstrap -K /mnt base base-devel linux linux-firmware
```
Load file table
```
# genfstab -U -p /mnt > /mnt/etc/fstab
```
chroot into your box for setup
```
# arch-chroot /mnt /bin/bash
```
# Setup
## Basics
I use neovim as my editor.  I use zsh for my shell.  Swap them out how you wish.
```
# pacman -S neovim git zsh
```
Personally I still type __vim__ for my editor, and rather than setting up an alias, I create a softlink.
```
# ln -s /usr/bin/nvim /usr/bin/vim
```
## Bootloader
I use systemd-boot instead of GRUB (I use systemd for everything).  It comes installed, but needs setting up if you're using it.
```
# bootctl install
# vim /boot/loader/loader.conf
```
If you're using vi/vim/nvim, hit the __i__ key to enter insert mode.
```
default arch.conf
timeout 2
console-mode max
editor no
```
Press __esc__ then type __:wq__ to save and quit.
```
# cat /etc/fstab
```
Copy the UUID of your root partition into your clipboard.
```
# vim /boot/loader/entries/arch.conf
```
```
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=UUID=xxxxxxxxxxxx rw
```
Verify there are no issues.
```
# bootctl
```
Save it
```
# bootctl update
```
## Generate initial ramdisk
```
# mkinitcpio -p linux
```
## System Configuration
### Timezone
```
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```
### NTP
```
# vim /etc/systemd/timesyncd.conf
```
Add line, save and exit.
```
NTP=time.google.com
```
Enable systemd's timesyncd
```
# systemctl enable systemd-timesyncd
```
### Locale
```
# vim /etc/locale.gen
```
Uncomment the following line, save and exit
```
en_US.UTF-8 UTF-8
```
Generate
```
# locale-gen
```
Not all programs use __locale.gen__ so here's the backup
```
vim /etc/locale.conf
```
Add line, save and exit
```
LANG=en_US.UTF-8
```
### hostname
```
vim /etc/hostname
```
### User
Set real root password.  The one before was the root password for your live disk
```
# passwd
```
Add your user, setting the shell that you installed earlier.
```
# useradd -m -G wheel -s /bin/zsh user
```
Password
```
# passwd user
```
Add wheel group to sudoers
```
# EDITOR=nvim visudo
```
Type `/wheel<ENTER>` to search.  Type __^xx__ to delete the comment and whitespace at the start of the line.  __:wq__ to save and exit.
## Network
I use systemd-networkd
```
# vim /etc/systemd/network/20-wired.network
```
```
[Match]
Name=enp1s0

[Link]
RequiredForOnline=routable

[DHCPv4]
RouteMetric=100

[Network]
DHCP=yes
UseDNS=yes
```
```
# vim /etc/systemd/network/25-wireless.network
```
```
[Match]
Name=wlp0s20f3

[Link]
RequiredForOnline=routable

[DHCPv4]
RouteMetric=600

[Network]
DHCP=yes
IgnoreCarrierLoss=3s
UseDNS=yes
```
Enable systemd-networkd at boot
```
# systemctl enable systemd-networkd
```
Add DNS resolvers manually (for now)
```
# vim /etc/resolv.conf
```
```
nameserver 10.1.25.206
nameserver 10.1.25.216
```
Wireless network still needs __wpasupplicant__ etc.  The route metric allows both to be connected simultaneously and prefer wired.
```
# pacman -S wpa_supplicant
```
## Intel Microcode
```
# pacman -S intel-ucode
```
## Switch to your user
```
# su user
```
## Hyprland
```
$ yay -S uwsm hyprland hyprlock hypridle hyprshot swaync waybar rofi pipewire wireplumber pipewire-alsa
```
## Oh My ZSH
```
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
## Fonts
```
$ yay -S otf-font-awesome noto-fonts powerline-fonts noto-fonts-cjk geist-font ttf-hack-nerd
```
## SSH, File Manager, Teams, Spotify, Global Protect, Remmina for RDP
```
yay -S openssh thunar teams-for-linux spotify globalprotect-openconnect remmina freerdp
```
