# Arch Installation

## Mettre en français

D'abord, pour plus de simplicité d'écriture, on va charger le clavier français AZERTY

```bash
loadkeys fr # On est en QWERTY donc on écrira : loqdkeys fr
```

## SSH

Pour plus de simplicité pour copier coller les commandes dans ce wiki et si vous en avez la possibilité, je vous conseille de mettre en place le SSH. Pour ce faire, changer le mot de passe root par un mot de passe simple comme ```r```, ce mot de passe ne sera pas définitif et ne sert que pour l'authentification au live ISO et en aucun cas sur votre futur système.

```bash
passwd
```

Ensuite démarrez le service sshd

```bash
systemctl start sshd
```

Récupérez son IP

```bash
ip a
```

Enfin, avec un autre ordinateur relié à celui-ci en réseau, connectez-vous :

```bash
ssh root@<ip-de-la-machine>
```

Entrez le mot de passe entré à l'instant et vous êtes connecté.

## Wi-Fi

Si vous installez Arch depuis un PC qui n'est pas relié en réseau via un câble, il faut initialiser le Wi-Fi

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

## Vérifier EFI ou BIOS

Pour savoir ce qui tourne entre l'EFI ou le BIOS exécutez :

```bash
cat /sys/firmware/efi/fw_platform_size
```

Si cette commande retourne une erreur, alors vous êtes en BIOS sinon vous êtes en EFI.

## Commencer l'installation

[**Installer en BIOS**](BIOS)

[**Installer en EFI**](EFI)
