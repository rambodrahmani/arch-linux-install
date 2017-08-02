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

Identificate il disco sul quale volete installare Arch Linux utilizzando il comando lsblk:

```shell
lsblk
```
