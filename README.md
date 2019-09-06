# Install Arch Linux

## Boot from live USB

### setup ntp
  `timedatectl set-ntp true`

## Create and format partitions
### Check hdd partitions with `fdisk` or just use `cfdisk`(user friendly)
  `fdisk l`
### create partitions with for GPT partition. remember to set UEFI boot from bios config
  `gdisk /dev/sda`
### create `/efi` partition.
  `/dev/sda1`
### create `swap` partition.
  `/dev/sda2`
### create `/root` partition.
  `/dev/sda3`
### create `/home` partition.
```bash
/dev/sda4
### format efi(550M), swap(ramsize) /root(30G), /home(rest)  partitions
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
mkswap /dev/sda2
swapon /dev/sda2
mkfs.fat -F32 /dev/sda1
```
### mount partitions
```bash
  `mkdir /mnt/home`
  `mkdir /mnt/efi`
  `mnt /dev/sda3 /mnt`
  `mnt /dev/sda4 /mnt/home`
  `mnt /dev/sda1 /mnt/efi`
```


## Install arch linux
### Get fastest mirrors or find `Server = http://mirror.xeonbd.com/archlinux/$repo/os/$arch` in `/etc/pacman.d/mirrorlist` and push it to the beginning
  `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup`
  `curl -s "https://www.archlinux.org/mirrorlist/?protocol=https&use_mirror_status=on" | sed -e `s/^#Server/Server/` -e `/#/d` | rankmirrors -n 5 - > /etc/pacman.d/mirrorlist`

### Install OS
  `pacstrap /mnt base base-devel`
### chroot
  `arch-chroot /mnt`
### set timezone
  `ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime`
### set hardware clock to generate /etc/adjtime. This command assumes the hardware clock is set to UTC
  `hwclock --systohc`
### Uncomment en_US ISO-8859-1 UTF-8 and other needed locales in /etc/locale.gen, and generate them with
  `locale-gen`
### Set  in `etc/locale.conf` file
  `LANG=en_US.UTF-8`
  `LANG=en_GB.UTF-8`
### Set hostname and hosts
```bash
#/etc/hostname
hostname
#/etc/hosts
127.0.0.1 localhost
::1		localhost
127.0.1.1 myhostname.localdomain myhostname
```
### dns config in `/etc/resolv.conf`
  `nameserver 8.8.8.8`
  `nameserver 8.8.4.4`
### set root password
  `passwd`

### Generate fstab
  `genfstab -U /mnt >> /mnt/etc/fstab`

### Install and configure GRUb
```bash
pacman -Sy grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/mnt/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Exit, unmount partitions and reboot
```bash
exit
umount -R /mnt
reboot
```


## Configure system user

### create system user
```bash
pacman -Sy sudo`
useradd --create-home username`
usermod -aG wheel username`
passwd username` # set password
```
### add user to
  `visudo` # comment out wheel group config
### login to system user
  `su username`


### install required softwares
  `sudo pacman -Sy terminator consolas-font i3-wm dmenu i3status i3lock conky xorg xorg-xinit xorg-server xorg-xev unzip
  openssh python-pip alsa-utils htop git zsh pcmanfm firefox flashplugin pepper-flash keepass docker`

### configure i3wm in `~/.xinitrc`
  `#!/bin/bash
  exec i3`
  `chmod 744 ~/.xinitrc`
  `touch ~/.Xauthority`

### install yaourt to install aur packages
`mkdir ~/tmp `
`cd mkdir`
`git clone https://aur.archlinux.org/yay.git`
`cd yay`
`makepkg -si`

### generate ssh
  `ssh-keygen -t ed25519`
### add public key got github
### configure git
  `git config --global user.email "email"`
  `git config --global user.name "Moniruzzaman Monir"`


### add ssh key to github.com
### Get Dotfiles
  `pip install dotfiles`
  `git clone git@github.com:dostokhan/dotfiles.git ~/Dotfiles`
  `dotfiles sync --force`

### reboot and login to window manager
### Install spf13 vim config
  `sh <(curl https://j.mp/spf13-vim3 -L)`
### Install oh-my-zsh
  `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

### Install Other softwares
`pacman -Sy docker nvm transmission`
`yay -Sy google-chrome-stable postman consolas-font cloudcross grive`
`npm install --global yarn`

### install python36
`yay -Sy `
### install docker-compose
`sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
`



## Laptop Extras
### Wireless Connection during installation before exiting chroot or `install and use connman`
```bash
pacman -Sy iw wpa_supplicant
iw dev # get name of wireless interface
ip link set wlo1 up
ip link show wlo1
iw dev wlo1 scan
wpa_passphrase my_essid my_passphrase > /etc/wpa_supplicant/my_essid.conf
wpa_supplicant -c /etc/wpa_supplicant/my_essid.conf -i wlo1 # test connection
wpa_supplicant -B -c /etc/wpa_supplicant/my_essid.conf -i wlo1 # run in background
dhclient wlo1` # get ip
```
NOTE: MAY NEED TO USE RFKILL. `rfkill unblock wifi`
#### put following in file `/etc/systemd/system/rfkill-unblock-wifi.service`
```conf
[Unit]
Description=RFKill-Unblock WiFi Devices

[Service]
Type=oneshot
ExecStart=/usr/sbin/rfkill unblock wifi
ExecStop=
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
`sudo systemctl enable rfkill-unblock-wifi`
`sudo systemctl start rfkill-unblock-wifi`

### for touchpads
`pacman -Sy xf86-input-libinput xorg-xinput libinput`
`xinput list`
#### put the following in `vim /usr/share/X11/xorg.conf.d/30-touchpad.conf`
```conf
Section "InputClass"
    Identifier "devname"
    Driver "libinput"
    Option "Tapping" "on"
    Option "ClickMethod" "clickfinger"
EndSection
```
### for screen brightness
`pacman -Sy light redshift pulseaudio-alsa pulseaudio-bluetooth bluez-utils`
`light -A 10` # increase brightness by 10%. -U for substracting

