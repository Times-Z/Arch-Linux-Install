## 1/ Load keyboard keymaps FR
```
loadkeys fr
```

## 2/ Disk partitionning
```
cgdisk /dev/nvme0n1
```

  |Part. #    |  Mount Point      |  Size                             | Partition Type          |
  |-----------|-------------------|-----------------------------------|-------------------------|
  | 1         | /                 | 20 Go min                         |  Linux Filesystem       |
  | 2         | /boot/efi         | 128 Mo                            |  EFI System Partition   |
  | 3         |                   | RAM Size                          |  Linux Swap             |
  | 4         | /home             | Depend on Disk Size               |  Linux Filesystem       |

```
mkfs.ext4 /dev/nvme0n1p1
mkfs.fat -F32 /dev/nvme0n1p2
mkfs.ext4 /dev/nvme0n1p4
mkswap /dev/nvme0n1p3
swapon /dev/nvme0n1p3
```

## 3/ Mount disk to iso
```
mount /dev/nvme0n1p1 /mnt
mkdir /mnt/{boot,boot/efi,home}
mount /dev/nvme0n1p2 /mnt/boot/efi
mount /dev/nvme0n1p3 /mnt/home
```

## 4/ Install base soft

```
pacstrap /mnt base base-devel pacman-contrib zip unzip p7zip vim neovim mc alsa-utils syslog-ng mtools dosfstools lsb-release ntfs-3g exfat-utils zsh bash-completion linux-lts
```

Add on this line :
-  `tlp` on laptop
- `amd-ucode` if your cpu is AMD brand
- `intel-ucode` if your cpu is INTEL brand

## 5/ Fstab and grub
```
genfstab -U -p /mnt >> /mnt/etc/fstab
pacstrap /mnt grub os-prober efibootmgr
```

## 6/ Chroooot
```
arch-chroot /mnt
```

## 7/ Locale

Add these lines into /etc/vconsole.conf
```
KEYMAP=fr-latin9
FONT=eurlatgr
```

Change locale into /etc/locale.conf
```
LANG=C.UTF-8
LC_COLLATE=C
```

Generate locale
```
locale-gen
```

Time France
```
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc
```
## 8/ Hostname
```
echo "myhostname" > /etc/hostname
```

## 9/ Make the image
```
mkinitcpio -p linux ou **linux-lts** si vous voulez le noyau lts.
```

## 10/ Install grub
```
mount | grep efivars &> /dev/null || mount -t efivarfs efivarfs /sys/firmware/efi/efivars
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck
mkdir /boot/efi/EFI/boot
cp /boot/efi/EFI/arch_grub/grubx64.efi /boot/efi/EFI/boot/bootx64.efi
grub-mkconfig -o /boot/grub/grub.cfg
```

## 11/ Change root password
```
passwd root
```

## 12/ Install networkmanager
```
pacman -Syy networkmanager
systemctl enable NetworkManager
```

## 13/ Enable pacman multilib
Uncomment this in /etc/pacman.conf 
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```

## 14/ Unmount live CD and reboot
```
exit
umount -R /mnt
reboot
```
## 15/ Install softs
```
pacman -Syy ntp cronie gst-plugins-{base,good,bad,ugly} gst-libav xorg-{server,xinit,apps} xf86-input-libinput xdg-user-dirs
```

## 16/ Install graphic drivers

 | Brand               | OpenSource pilote     |
 |---------------------|-----------------------|
 | AMD                 | xf86-video-amdgpu     |
 | Intel               | xf86-video-intel      |
 | Nvidia              | xf86-video-nouveau    |
```
pacman -S yourdrivers
```

## 17/ Fonts
```
pacman -S ttf-{bitstream-vera,liberation,freefont,dejavu} freetype2
```

## 18/ Install printers 
```
pacman -S cups hplip python-pyqt5 foomatic-{db,db-ppds,db-gutenprint-ppds,db-nonfree,db-nonfree-ppds} gutenprint
```

## 19/ Install chromium
```
pacman -S chromium
```

## 20/ Add user
```
useradd -m -g wheel -c 'User name full' -s /bin/bash user-name
passwd user-name
```

## 21/ Configure sudo
Uncomment the following line into /etc/sudoers
```
%wheel ALL=(ALL:ALL) ALL
```

## 22/ Reboot and install your DE/WM