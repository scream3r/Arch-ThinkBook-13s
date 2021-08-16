# Arch-ThinkBook-13s

### Create partitions:

`cfdisk /dev/nvme0n1`

* efi 1G
* swap 24G
* root All remaining space

### Format partitions:

```
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p3
mkswap /dev/nvme0n1p2
```

### Mount root partition and enable swap:

```
mount /dev/nvme0n1p3 /mnt
swapon /dev/nvme0n1p2
```

### Make basic Arch installation:

`pacstrap -i /mnt base bash-completion linux linux-firmware linux-headers nano sudo vim`

### Generate fstab:

`genfstab -U -p /mnt >> /mnt/etc/fstab`

### Chroot to root partition:

`arch-chroot /mnt /bin/bash`

### Set timezone:

`ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime`

### Generate locales:

* Edit `/etc/locale.gen` file to uncomment preferred locales. For example: `en_US.UTF-8` and `ru_RU.UTF-8`
* Invoke `locale-gen` command
* Set system locale. For example: `echo "LANG=ru_RU.UTF-8" > /etc/locale.conf`

### Set host name:

`echo userpc > /etc/hostname`

### Set hosts:

Edit `/etc/hosts` to set preferred hosts. For example:
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	userpc.localdomain userpc
```

### Install GRUB:

```
pacman -Syu
pacman -S grub efibootmgr
mkdir /boot/efi
mount /dev/nvme0n1p1 /boot/efi
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

### Set root password and create user:

```
passwd
useradd -m user
passwd user
```

Optionally add `<user>` to `wheel` group and allow sudo commands invocation:

```
usermod -a -G wheel user
visudo
```

Find and uncomment `%wheel ALL=(ALL) ALL` string to allow sudo commands invocation

### Install Networkmanager:

`pacman -S networkmanager`

### Install gnome and additional software:

`pacman -S xorg gnome gnome-tweaks dconf-editor`

### Enable systemd services:
```
systemctl enable gdm
systemctl enable NetworkManager
```

### Enable Hibernation:

Edit `/etc/defaults/grub` file and set resume partition. For example:
```
GRUB_CMDLINE_LINUX_DEFAULT="resume=UUID=<UUID_of_partition>"
```
Rebuild GRUB config `grub-mkconfig -o /boot/grub/grub.cfg`

Edit `/etc/mkinitcpio.conf` file and add `resume` to HOOKS. For example:
```
HOOKS="base udev resume autodetect modconf block filesystems keyboard fsck"
```
Regenerate presets `mkinitcpio -P`

Create `/etc/systemd/sleep.conf.d` files to disable `suspend` (in cause of some bugs with S4) and enable `hibernate`. For example:
```
#suspend.conf
[Sleep]
AllowSuspend=no
#SuspendMode=
#SuspendState=mem standby freeze
```
```
#hibernate.conf
[Sleep]
AllowHibernation=yes
HibernateMode=shutdown
HibernateState=disk
```

### Install NotoFonts system-wide:

`pacman -S noto-fonts noto-fonts-cjk`

Create `/etc/fonts/local.conf` file and add aliases:
```XML
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>

	<!-- Override global fonts -->
	<alias>
		<family>sans-serif</family>
		<prefer><family>Noto Sans</family></prefer>
	</alias>
	<alias>
		<family>serif</family>
		<prefer><family>Noto Serif</family></prefer>
	</alias>
	<alias>
		<family>monospace</family>
		<prefer><family>Noto Sans Mono</family></prefer>
	</alias>

</fontconfig>
```
