# Installazione Arch Linux Full Disk Encryption (LVM on LUKS) (incluso /boot)

La seguente repo contiene istruzioni minimali per installare Arch Linux con Full Disk Encryption (LVM su LULS) inclusa la partizione /boot.

Per istruzioni dettagliate fare riferimento alla guida ufficiale: [Installation guide (Italiano)](https://wiki.archlinux.org/index.php/Installation_guide_(Italiano))

***
Scaricate una immagine iso da: https://www.archlinux.org/

Create una pennina bootable utilizzando la ISO che avete scaricato:

```shell
dd if=archlinux.img of=/dev/sdX bs=16M && sync # su linux
```

Eseguite il Boot dalla pennina USB appena creata.

<img src="images/1.png" height="200px" />
<img src="images/2.png" height="200px" />
<img src="images/3.png" height="200px" />

Nelle istruzioni che seguono si assume che il boot sia stato eseguito correttamente e che vi troviate nella shell fornita dal setup di Arch Linux.

***

Connettetevi a una rete Wi-Fi utilizzando il comando

```shell
root@archiso ~ # wifi-menu
```
oppure usate un cavo ethernet.

***

Identificate il disco sul quale volete installare Arch Linux utilizzando il comando:

```shell
root@archiso ~ # lsblk
```
In questa guida utilizzeremo il disco

```shell
NAME          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda             8:0    0 232.9G  0 disk
```

La struttura finale che vogliamo ottenere:

```shell
NAME          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda             8:0    0 232.9G  0 disk  
└─sda1          8:1    0 232.9G  0 part  
  └─lvm       254:0    0 232.9G  0 crypt 
    ├─vg-swap 254:1    0     4G  0 lvm   
    ├─vg-root 254:2    0    20G  0 lvm   /
    └─vg-home 254:3    0 208.9G  0 lvm   /home
```

***

Eseguiamo una pulizia del disco per eliminare eventuali dati sensibili:

```shell
shred -vfz --random-source=/dev/urandom -n 1 /dev/sda
```

Ci vorranno un paio di ore (minimo).

***

Iniziamo ora a partizionare il disco:

```shell
root@archiso ~ # parted -s /dev/sda mklabel msdos
root@archiso ~ # parted -s /dev/sda mkpart primary 2048s 100%
root@archiso ~ # cryptsetup -c aes-xts-plain64 -y --use-random --key-size 512 luksFormat /dev/sda1
root@archiso ~ # cryptsetup luksOpen /dev/sda1 lvm
root@archiso ~ # pvcreate /dev/mapper/lvm
root@archiso ~ # vgcreate vg /dev/mapper/lvm
root@archiso ~ # lvcreate -L 4G vg -n swap
root@archiso ~ # lvcreate -L 15G vg -n boot
root@archiso ~ # lvcreate -l +100%FREE vg -n home
root@archiso ~ # mkswap -L swap /dev/mapper/vg-swap
root@archiso ~ # mkfs.ext4 /dev/mapper/vg-boot
root@archiso ~ # mkfs.ext4 /dev/mapper/vg-home
root@archiso ~ # mount /dev/mapper/vg-boot /mnt
root@archiso ~ # mkdir /mnt/home
root@archiso ~ # mount /dev/mapper/vg-home /mnt/home
```
Un partizionamento di questo tipo mantiene separate le partizioni boot e home

***

A questo punto possiamo installare il sistema. Ho aggiunto anche alcuni pacchetti che tornano sicuramente utili al primo avvio del sistema:

```shell
root@archiso ~ # pacstrap -i /mnt base base-devel grub-efi-x86_64 zsh vim git efibootmgr dialog wpa_supplicant
```
Generate l'fstab:

```shell
root@archiso ~ # genfstab -pU /mnt >> /mnt/etc/fstab
```
Aggiungete questa riga al file /mnt/etc/fstab:
```shell
tmpfs /tmp tmpfs defaults,noatime,mode=1777 0 0
```

***

A questo punto, per evitare che ci siamo provlemi con lvm nei passaggi successivi, eseguite questi comandi:

```shell
root@archiso ~ # mkdir /mnt/hostrun
root@archiso ~ # mount --bind /run /mnt/hostrun
root@archiso ~ # arch-chroot /mnt /bin/bash
root@archiso ~ # mount --bind /hostrun/lvm /run/lvm
```

Notate che con ```root@archiso ~ # arch-chroot /mnt /bin/bash``` siamo anche entrati nella shell del nuovo sistema.
