hide:
    -navigation

# BIOS - Chiffrée - Classique

## Partitionnement du disque

Ici, nous allons faire 3 partitions :

- BIOS Boot
- /
- /home

```bash
cfdisk
```

On choisit le 1ère option : GPT

- /dev/sda1 [BIOS Boot] (1G)
- /dev/sda2 [Linux filesystem] (/ ; 50G)
- /dev/sda3 [Linux filesystem] (/home)

On chiffre les partitions

```bash
cryptsetup luksFormat /dev/sda2
cryptsetup open /dev/sda2 cryptroot
cryptsetup luksFormat /dev/sda3
cryptsetup open /dev/sda3 crypthome
```

On formate les partitions avec un système de fichier

```bash
mkfs.ext4 /dev/mapper/cryptroot
mkfs.ext4 /dev/mapper/crypthome
```

On monte les parititons

```bash
mount /dev/mapper/cryptroot /mnt
mount --mkdir /dev/mapper/crypthome /mnt/home
```

## Installation

On installe le système de base

```bash
pacstrap -K /mnt base linux linux-firmware grub net-tools sudo glibc vim networkmanager network-manager-applet cryptsetup
```

Ensuite, on choisit un environnement de bureau (si vous en voulez un) :

Si vous souhaitez GNOME, ajoutez ceci à la commande précédente : ```gnome gnome-tweaks```

Si vous souhaitez KDE, ajoutez ceci à la commande précédente : ```ark dolphin kate konsole plasma-meta plasma-workspace```

Si vous souhaitez Hyprland, ajoutez ceci à la commande précédente : ```dolphin dunst grim hyprland kitty polkit-kde-agent qt5-wayland qt6-wayland slurp uwsm wofi xdg-desktop-portal-hyprland```

Enfin, installez les paquets dont vous avez besoin.

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

Ici, étant donné que nous avons deux paritions chiffrées, au démarrage il faudra dévérouiller les 2 partitions individuellement avec le même mot de passe défini précédemment.
Pour éviter cela, je choisis de devoir entrer le mot de passe de la partition root mais que la partition home se déchiffre toute seul, à vous de choisir si vous voulez que votre système se déchiffre tout seul ou non, tout sera expliqué.

On génère une clé aléatoire pour notre keyfile

```bash
dd bs=512 count=4 if=/dev/random of=/etc/cryptsetup-keys.d/crypt.key iflag=fullblock # Généré par Claude Sonnet 4.6
chmod 600 /etc/cryptsetup-keys.d/crypt.key
```

On ajoute notre keyfile sur la ou les partitions que l'on souhaite dévérouiller automatiquement. Si vous souhaitez déchiffrer également cryptroot, la commande à l'identique en modifiant la parition pour qu'elle corresponde à la parition root (ici /dev/sda3) suffira. Cette commande vous demandera un mot de passe, évidemment entrez le même mot de passe que précédemment pour la partition.

```bash
cryptsetup luksAddKey /dev/sda4 /etc/cryptsetup-keys.d/crypt.key
```

On récupère l'UUID des partitions root et home, ici /dev/sda3 et /dev/sda4 :

```bash
blkid /dev/sda3
blkid /dev/sda4
```

On édite le fichier ```/etc/crypttab```. Si vous souhaitez tout déchiffrer, il suffit de remplacer ```none``` par le même ligne du dessous qui correspond au fichier

```bash
cryptroot  UUID=<UUID-de-sda3>  none                                    luks
crypthome  UUID=<UUID-de-sda4>  /etc/cryptsetup-keys.d/crypt.key    luks
```

Editer le fichier ```/etc/mkinitcpio.conf``` et modifiez cette ligne pour qu'elle contienne le chemin du keyfile

```bash
FILES=(/etc/cryptsetup-keys.d/crypt.key)
```

Puis on regénère l'initramfs

```bash
mkinitcpio -P
```

### Configuration du GRUB

Ici, étant donné que nous avons deux paritions chiffrées, au démarrage il faudra dévérouiller les 2 partitions individuellement avec le même mot de passe défini précédemment.
Pour éviter cela, je choisis de devoir entrer le mot de passe de la partition root mais que la partition home se déchiffre toute seul, à vous de choisir si vous voulez que votre système se déchiffre tout seul ou non, tout sera expliqué.

On génère une clé aléatoire pour notre keyfile

```bash
dd bs=512 count=4 if=/dev/random of=/etc/cryptsetup-keys.d/crypthome.key iflag=fullblock
chmod 600 /etc/cryptsetup-keys.d/crypthome.key
```

On ajoute notre keyfile sur la ou les partitions que l'on souhaite dévérouiller automatiquement. Si vous souhaitez déchiffrer également cryptroot, la commande à l'identique en modifiant la parition pour qu'elle corresponde à la parition root (ici /dev/sda3) suffira. Cette commande vous demandera un mot de passe, évidemment entrez le même mot de passe que précédemment.

```bash
cryptsetup luksAddKey /dev/sda4 /etc/cryptsetup-keys.d/crypthome.key
```

On récupère l'UUID des partitions root et home, ici /dev/sda3 et /dev/sda4 :

```bash
blkid /dev/sda3
blkid /dev/sda4
```

On édite le fichier ```/etc/crypttab```. Si vous souhaitez tout déchiffrer, il suffit de remplacer ```none``` par le même ligne du dessous qui correspond au fichier

```bash
cryptroot  UUID=<UUID-de-sda3>  none                                    luks
crypthome  UUID=<UUID-de-sda4>  /etc/cryptsetup-keys.d/crypthome.key    luks
```

On édite ```/etc/default/grub```

```bash
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID-de-sda3>:cryptroot root=/dev/mapper/cryptroot"
```

Et on décomment le ligne ```GRUB_ENABLE_CRYPTODISK=y```

On modifie aussi ```/etc/mkinitcpio.conf```

```bash
FILES=(/etc/cryptsetup-keys.d/crypthome.key)
```

Et on régénère

```bash
mkinitcpio -P
```

On installe le bootloader GRUB

```bash
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finaliser

On active les différents services

```bash
systemctl enable --now bluetooth # Si vous avez du Bluetooth
systemctl enable --now NetworkManager.service
```

Si vous avez choisi GNOME commen environnement de bureau, démarrez ce service

```bash
systemctl enable gdm.service
```

Sinon pour KDE ou Hyprland, démarrez clui-ci

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
