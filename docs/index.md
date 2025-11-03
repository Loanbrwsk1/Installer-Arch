hide:
    - navigation

# Arch Installation

## Attention

## Veuillez bien lire les commentaires dans le code !

## Mettre en français

```bash
loadkeys fr-latin1 #On est en QWERTY donc on écrira : loqdkeys fr)lqtin&
timedatectl
```

## Carte Wi-Fi

```bash
iwctl
device list

device <name> set-property Powered on       # Si off
adapter <adapter> set-property Powered on   # Si off

station <name> scan
station <name> get-networks
station <name> connect SSID

station <name> connect-hidden SSID          # Si caché
```

## Partionnement non chiffré

Vérifier si EFI

```bash
cat /sys/firmware/efi/fw_platform_size
```

### Si EFI

```bash
cfdisk
```

MBR

- /dev/sda1 [EFI system] (/boot)
- /dev/sda2 [Linux filesystem] (/)
- /dev/sda3 [Linux filesystem] (/home)

```bash
mkfs.ext4 /dev/sda2

mkfs.ext4 /dev/sda3

mkfs.fat -F 32 /dev/sda1

mount /dev/sda2 /mnt

mount --mkdir /dev/sda3 /mnt/home

mount --mkdir /dev/sda1 /mnt/boot
```

### Si BIOS

```bash
cfdisk
```

MBR

- /dev/sda1 [BIOS boot]
- /dev/sda2 [Linux filesystem] (/)
- /dev/sda3 [Linux filesystem] (/home)

```bash
mkfs.ext4 /dev/sda2

mkfs.ext4 /dev/sda3

mount /dev/sda2 /mnt

mount --mkdir /dev/sda3 /mnt/home
```

## Partitionnement chiffré

Vérifier si EFI

```bash
cat /sys/firmware/efi/fw_platform_size
```

### Si EFI

```bash
cfdisk
```

MBR

- /dev/sda1 [EFI system] (/boot)
- /dev/sda2 [Linux filesystem] (/)
- /dev/sda3 [Linux filesystem] (/home)

```bash
cryptsetup --verify-passphrase luksFormat /dev/sda2
cryptsetup --verify-passphrase luksFormat /dev/sda3
cryptsetup luksOpen /dev/sda2 root
cryptsetup luksOpen /dev/sda3 home
mkfs.ext4 /dev/mapper/root
mkfs.ext4 /dev/mapper/home
mkfs.fat -F 32 /dev/sda1
mount /dev/mapper/root /mnt
mount --mkdir /dev/mapper/home /mnt/home
mount --mkdir /dev/sda1 /mnt/boot
```

### Si BIOS

```bash
cfdisk
```

MBR

- /dev/sda1 [BIOS boot]
- /dev/sda2 [Linux filesystem] (/)
- /dev/sda3 [Linux filesystem] (/home)

```bash
cryptsetup --verify-passphrase luksFormat /dev/sda2
cryptsetup --verify-passphrase luksFormat /dev/sda3
cryptsetup luksOpen /dev/sda2 root
cryptsetup luksOpen /dev/sda3 home
mkfs.ext4 /dev/mapper/root
mkfs.ext4 /dev/mapper/home
mount /dev/mapper/root /mnt
mount --mkdir /dev/mapper/home /mnt/home
```

## Installation

```bash
pacstrap -K /mnt base linux linux-firmware grub net-tools sudo glibc nano networkmanager network-manager-applet efibootmgr
```

Ajouter le paquet ```efibootmgr``` pour EFI

Ajouter le paquet ```gnome``` pour installer l'environnement GNOME

Ajouter les paquets ```xorg plasma kde-applications plasma-wayland-session sddm``` pour installer l'environnement KDE

Ajouter les paquets ```hyprland hyprpaper xdg-desktop-portal-hyprland wayland wlroots waybar wofi kitty sddm``` pour installer l'environnement Hyprland

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
nano /etc/locale.gen
locale-gen
nano /etc/locale.conf    #On écrit : LANG=fr_FR.UTF-8
nano /etc/vconsole.conf  #On écrit  :KEYMAP=fr-latin1
nano /etc/hostanme : arch

mkinitcpio -P

passwd
useradd -G wheel -m <username>
passwd <username>
```

## Installation du GRUB

### Si EFI

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Si BIOS

```bash
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finaliser

```bash
exit
umount -R /mnt
reboot
```

## Post installation

```bash
login : root

nano /etc/sudoers #Décommenter %wheel ALL=(ALL:ALL) ALL

systemctl enable --now bluetooth # Si vous avez du Bluetooth
systemctl enable --now NetworkManager.service
```

### Pour GNOME

```bash
systemctl enable gdm.service
systemctl start gdm.service
```

### Pour KDE et Hyprland

```bash
systemctl enable gdm.service
systemctl start gdm.service
```
