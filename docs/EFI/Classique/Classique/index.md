hide:
    -navigation

# EFI - Classique - Classique

## Attention

## Veuillez bien lire les commentaires dans le code !

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

Ici, nous allons faire 3 partitions :

- EFI
- /
- /home

```bash
cfdisk
```

On choisit le 1ère option : GPT

- /dev/sda1 [EFI System Partition] (/boot/efi ; 521M)
- /dev/sda2 [Linux filesystem] (/ ; 50G)
- /dev/sda3 [Linux filesystem] (/home ; reste)

On formate les partitions avec un système de fichier

```bash
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
```

On monte les partitions

```bash
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot/efi
mount --mkdir /dev/sda3 /mnt/home
```

## Installation

On installe le système de base

```bash
pacstrap -K /mnt base linux linux-firmware grub net-tools sudo glibc vim networkmanager network-manager-applet efibootmgr
```

Ensuite, on choisit un environnement de bureau (si vous en voulez un) :

Si vous souhaitez GNOME, ajoutez ceci à la commande précédente : ```gnome gnome-tweaks```

Si vous souhaitez KDE, ajoutez ceci à la commande précédente : ```ark dolphin kate konsole plasma-meta plasma-workspace```

Si vous souhaitez Hyprland, ajoutez ceci à la commande précédente : ```dolphin dunst grim hyprland kitty polkit-kde-agent qt5-wayland qt6-wayland slurp uwsm wofi xdg-desktop-portal-hyprland```

Enfin, rajoutez les paquets dont vous avez besoin.

Si l'installation échoue et qu'il y a une erreur, exécutez :

```bash
pacman-key --init
pacman-key --populate archlinux

pacman -Sy archlinux-keyring

pacman-key --populate archlinux
```

Et réessayez la commande

Après cela on génère le fstab et on chroot

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

On ajoute manuellement le fuseau horaire

```bash
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
```

On synchronise horloge matérielle et logicielle

```bash
hwclock --systohc
```

On charge la langue française

```bash
echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
```

On change la langue du système

```bash
echo "LANG=fr_FR.UTF-8" > /etc/locale.conf
```

On met le clavier en français

```bash
echo "KEYMAP=fr
XKBLAYOUT=fr" > /etc/vconsole.conf
```

On choisit un nom de machine

```bash
echo "arch" > /etc/hostname
```

Enfin, on change le mot de passe root, puis on crée notre utilisateur qui aura les droits sudo

```bash
passwd

useradd -G wheel -m <nom d'utilisateur>
passwd <nom d'utilisateur>

visudo # Décommenter %wheel ALL=(ALL:ALL) ALL
```

## Configuration du GRUB

On installe le bootloader GRUB

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finaliser

On active les différents services

```bash
systemctl enable bluetooth # Si vous avez du Bluetooth
systemctl enable NetworkManager.service
```

### Pour GNOME

```bash
systemctl enable gdm.service
```

### Pour KDE et Hyprland

```bash
systemctl enable sddm.service
```

## Quitter et redémarrer

```bash
exit
umount -R /mnt
reboot
```

## Fin

Le système devrait désormais être complètement opérationnel !
