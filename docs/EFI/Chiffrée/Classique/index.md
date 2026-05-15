hide:
    -navigation

# EFI - Chiffrée - Classique

## Attention

## Veuillez bien lire les commentaires dans le code !

## Mettre en français

```bash
timedatectl set-timezone Europe/Paris
```

## Carte Wi-Fi

Si vous avez une carte Wi-Fi, connectez-vous à votre réseau :

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

## Partitionnement du disque

Ici, nous allons faire 4 partitions :

- EFI
- /boot
- /
- /home

```bash
cfdisk
```

On choisit le 1ère option : GPT

- /dev/sda1 [EFI System Partition] (/boot/efi ; 521M)
- /dev/sda2 [Linux filesystem] (/boot ; 1G)
- /dev/sda2 [Linux filesystem] (/ ; 50G)
- /dev/sda3 [Linux filesystem] (/home ; reste)

```bash
cryptsetup luksFormat /dev/sda3
cryptsetup open /dev/sda3 cryptroot
cryptsetup luksFormat /dev/sda4
cryptsetup open /dev/sda3 crypthome

mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/mapper/cryptroot
mkfs.ext4 /dev/mapper/crypthome

mount /dev/mapper/cryptroot /mnt
mount --mkdir /dev/sda2 /mnt/boot
mount --mkdir /dev/sda1 /mnt/boot/efi
mount --mkdir /dev/mapper/crypthome /mnt/home
```

## Installation

On installe le système de base

```bash
pacstrap -K /mnt base linux linux-firmware grub net-tools sudo glibc vim networkmanager network-manager-applet cryptsetup efibootmgr
```

Ensuite, on choisit un environnement de bureau (si vous en voulez un) :

Si vous souhaitez GNOME, ajoutez ceci à la commande précédente : ```gnome gnome-tweaks```

Si vous souhaitez KDE, ajoutez ceci à la commande précédente : ```ark dolphin kate konsole plasma-meta plasma-workspace```

Si vous souhaitez Hyprland, ajoutez ceci à la commande précédente : ```dolphin dunst grim hyprland kitty polkit-kde-agent qt5-wayland qt6-wayland slurp uwsm wofi xdg-desktop-portal-hyprland```

Si l'installation échoue et qu'il y a une erreur, exécutez :

```bash
pacman-key --init
pacman-key --populate archlinux

pacman -Sy archlinux-keyring

pacman-key --populate archlinux
```

Et réessayez la commande

Après cela :

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=fr_FR.UTF-8" > /etc/locale.conf
echo "KEYMAP=fr-latin1" > /etc/vconsole.conf
echo "arch" > /etc/hostname
```

Editer le fichier ```/etc/mkinitcpio.conf```. Il faut qu la ligne suivante soit **exactement** comme suit :

```bash
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```

Puis on regénère l'initramsfs

```bash
mkinitcpio -P
```

Enfin, on change le mot de passe root, puis on crée notre utilisateur qui aura les droits sudo

```bash
passwd

useradd -G wheel -m <nom d'utilisateur>
passwd <nom d'utilisateur>

visudo # Décommenter %wheel ALL=(ALL:ALL) ALL
```

## Configuration du GRUB

On récupère l'UUID de la partition root, ici /dev/sda3

```bash
blkid /dev/sda3
```

On édite ```/etc/default/grub```

```bash
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID-de-sda3>:cryptroot root=/dev/mapper/cryptroot"
```

Et on décomment le ligne ```GRUB_ENABLE_CRYPTODISK=y```

On installe le bootloader GRUB

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finaliser

On active les différents services

```bash
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
systemctl enable sddm.service
systemctl start sddm.service
```

## Quitter et redémarrer

```bash
exit
umount -R /mnt
reboot
```

## Fin

Le système devrait désormais être complètement opérationnel !
