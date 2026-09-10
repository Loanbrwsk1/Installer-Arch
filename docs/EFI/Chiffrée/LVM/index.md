hide:
    -navigation

# EFI - Chiffrée - LVM

## Partitionnement du disque

Ici, nous allons faire 3 partitions :

- EFI
- /boot
- LVM

```bash
cfdisk
```

On choisit le 1ère option : GPT

- /dev/sda1 [EFI System Partition] (/boot/efi ; 512M)
- /dev/sda2 [Linux filesystem] (/boot ; 1G)
- /dev/sda3 [Linux filesystem] (reste)

On chiffre la partition

```bash
cryptsetup luksFormat /dev/sda3
cryptsetup open /dev/sda3 cryptlvm
```

On crée le LVM

```bash
pvcreate /dev/mapper/cryptlvm
vgcreate vg0 /dev/mapper/cryptlvm

lvcreate -L 40G vg0 -n root # pour la partition root
lvcreate -l 100%FREE vg0 -n home # pour la partition /home
```

On formate les partitions avec un système de fichier

```bash
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/vg0/root
mkfs.ext4 /dev/vg0/home
```

On monte les systèmes de fichiers

```bash
mount /dev/vg0/root /mnt
mount --mkdir /dev/vg0/home /mnt/home
mount --mkdir /dev/sda2 /mnt/boot
mount --mkdir /dev/sda1 /mnt/boot/efi
```

## Installation

On installe le système de base

```bash
pacstrap -K /mnt base linux linux-firmware grub net-tools sudo glibc vim networkmanager network-manager-applet efibootmgr cryptsetup lvm2
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

## Configuration

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

Editer le fichier ```/etc/mkinitcpio.conf```. Il faut que la ligne suivante soit **exactement** comme suit :

```bash
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```

Puis on regénère l'initramfs

```bash
mkinitcpio -P
```

On change le mot de passe root, puis on crée notre utilisateur qui aura les droits sudo

```bash
passwd

useradd -G wheel -m <nom d'utilisateur>
passwd <nom d'utilisateur>

visudo # Décommenter %wheel ALL=(ALL:ALL) ALL
```

### Dévérouillage automatique

Au démarrage, il faudra dévérouiller la partition à la main avec le même mot de passe défini précédemment.
Pour éviter cela, Il est possible de faire en sorte que le système la dévérouille toute seule.

On génère une clé aléatoire pour notre keyfile

```bash
dd bs=512 count=4 if=/dev/random of=/etc/cryptsetup-keys.d/cryptlvm.key iflag=fullblock # Généré par Claude Sonnet 4.6
chmod 600 /etc/cryptsetup-keys.d/cryptlvm.key
```

On ajoute notre keyfile sur la. Cette commande vous demandera un mot de passe, évidemment entrez le même mot de passe que précédemment pour la partition.

```bash
cryptsetup luksAddKey /dev/sda4 /etc/cryptsetup-keys.d/cryptlvm.key
```

On récupère l'UUID de la partition, ici /dev/sda3 :

```bash
blkid /dev/sda3
```

On édite le fichier ```/etc/crypttab```.

```bash
cryptlvm  UUID=<UUID-de-sda4>  /etc/cryptsetup-keys.d/cryptlvm.key    luks
```

Editer le fichier ```/etc/mkinitcpio.conf``` et modifiez cette ligne pour qu'elle contienne le chemin du keyfile

```bash
FILES=(/etc/cryptsetup-keys.d/cryptlvm.key)
```

Puis on regénère l'initramfs

```bash
mkinitcpio -P
```

### Configuration du GRUB

On récupère l'UUID de la partition root, ici /dev/sda3

```bash
blkid /dev/sda3
```

On édite ```/etc/default/grub```

```bash
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID-de-sda3>:cryptlvm root=/dev/vg0/root"
```

Et on décomment le ligne ```GRUB_ENABLE_CRYPTODISK=y```

On installe le bootloader GRUB

```bash
grub-install --target=x86_64-efi
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finaliser

On active les différents services

```bash
systemctl enable bluetooth # Si vous avez du Bluetooth
systemctl enable NetworkManager.service
```

Si vous avez choisi GNOME commen environnement de bureau, démarrez ce service :

```bash
systemctl enable gdm.service
```

Sinon pour KDE ou Hyprland, démarrez celui-ci :

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
