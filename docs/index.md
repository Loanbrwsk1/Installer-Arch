# Arch Installation

## Attention

## Veuillez bien lire les commentaires dans le code !

## Mettre en français

```bash
loadkeys fr-latin1 # On est en QWERTY donc on écrira : loqdkeys fr)lqtin&
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
