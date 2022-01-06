# 0. Préparation de la machine

## Interfaces réseaux des deux machines

* Accéder au fichier de configuration de la carte enp0s8 ```ifcfg-enp0s8``` en prenant le chemin cd ```/etc/sysconfig/network-scripts/```
#### Machine 1

```
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=10.101.1.11
NETMASK=255.255.255.0
GATEWAY=10.10.1.254
DNS1=1.1.1.1
```
```
machine 2
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=10.101.1.12
NETMASK=255.255.255.0
GATEWAY=10.10.1.254
DNS1=1.1.1.1
```

* Redémarrer l'interface :

```
sudo nmcli con reload
sudo nmcli con up enp0s8
```

```
[targa@node1 ~]$ ip a
[...]
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:93:d0:7d brd ff:ff:ff:ff:ff:ff
    inet 10.101.1.11/24 brd 10.101.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe93:d07d/64 scope link
       valid_lft forever preferred_lft forever
[targa@node2 network-scripts]$ ip a
[...]
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4a:cf:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.101.1.12/24 brd 10.101.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4a:cff7/64 scope link
       valid_lft forever preferred_lft forever
```


* Test de connexion des deux machines

```
[targa@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=19.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=20.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=20.2 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=20.3 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 19.820/20.110/20.322/0.183 ms
[targa@node2 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=19.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=21.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=17.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=20.3 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 17.831/19.683/21.048/1.206 ms
[targa@node2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b2:4f:a6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86308sec preferred_lft 86308sec
    inet6 fe80::a00:27ff:feb2:4fa6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4a:cf:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.10.1.12/24 brd 10.10.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4a:cff7/64 scope link
       valid_lft forever preferred_lft forever
```       
* Les deux machines sont sur la même carte host-only, permettant ainsi de communiquer dans le même réseau.

### Configuration des cartes enp0s8 des deux machines

#### Fichier de configuration de la première machine

![](https://i.imgur.com/HsiB90I.png)

#### Fichier de configuration de la deuxième machine

![](https://i.imgur.com/cyrN9xn.png)


### Changement de nom pour les deux machines


* Nous devons éditer le fichier ```hostname``` pour ajouter un nom personnalisé
#### Première machine
![](https://i.imgur.com/K0EzkXn.png)


#### Deuxième machine
![](https://i.imgur.com/ptQx1gE.png)


### Vérification du serveur DNS avec la commande dig

##### Première machine

```
[targa@node1 etc]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
[...]

;; ANSWER SECTION:
ynov.com.               10765   IN      A       92.243.16.143

[...]
;; SERVER: 192.168.1.1#53(192.168.1.1)
```
```
[targa@node2 etc]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
[...]
;; ANSWER SECTION:
ynov.com.               10703   IN      A       92.243.16.143

[...]
;; SERVER: 192.168.1.1#53(192.168.1.1)

```

### Ping des machines avec leurs noms

* Modifier le fichier ```hosts``` pour ajouter le nom de la machine que l'on veut ping.

#### Première machine
![](https://i.imgur.com/pNFv0O6.png)


#### Deuxième machine
![](https://i.imgur.com/47IQ6c1.png)


#### Ping première machine vers deuxième machine

```
[targa@node1 etc]$ [targa@node1 etc]$ ping node2.tp1.b2
PING node2.tp1.b2 (10.10.1.12) 56(84) bytes of data.
64 bytes from node2.tp1.b2 (10.10.1.12): icmp_seq=1 ttl=64 time=0.493 ms
64 bytes from node2.tp1.b2 (10.10.1.12): icmp_seq=2 ttl=64 time=0.408 ms
64 bytes from node2.tp1.b2 (10.10.1.12): icmp_seq=3 ttl=64 time=0.453 ms
64 bytes from node2.tp1.b2 (10.10.1.12): icmp_seq=4 ttl=64 time=0.309 ms
^C
--- node2.tp1.b2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3059ms
rtt min/avg/max/mdev = 0.309/0.415/0.493/0.072 ms
Ping deuxième machine vers première machine
```
```
[targa@node2 etc]$ [targa@node2 etc]$ ping node1.tp1.b2
PING node1.tp1.b2 (10.10.1.11) 56(84) bytes of data.
64 bytes from node1.tp1.b2 (10.10.1.11): icmp_seq=1 ttl=64 time=0.385 ms
64 bytes from node1.tp1.b2 (10.10.1.11): icmp_seq=2 ttl=64 time=0.363 ms
64 bytes from node1.tp1.b2 (10.10.1.11): icmp_seq=3 ttl=64 time=0.367 ms
64 bytes from node1.tp1.b2 (10.10.1.11): icmp_seq=4 ttl=64 time=0.396 ms
^C
--- node1.tp1.b2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3089ms
rtt min/avg/max/mdev = 0.363/0.377/0.396/0.027 ms
```

#### Ajout d'une paire de clé ssh sur les deux machines
* Désactivation de SELinux

```
[toto@node1 ~]$ sudo setenforce 0
[toto@node2 ~]$ sudo setenforce 0
```

* modifier le fichier ```config ```dans ```/etc/selinux/config``` sur les deux machines en remplaçant ```enforcing``` par ```permissive```.
![](https://i.imgur.com/KIKuIMi.png)


### Firewall
* Lister toutes les règles actives actuellement ( sur les deux machines)

```
[targa@node1 ~]$ sudo firewall-cmd --list-all
[sudo] password for targa:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
* On remarque qu'aucune restriction de port est appliqué , donc le port 22 est libre d'accès .
# I. Utilisateurs
## 1. Création et configuration

### Création d'un utilisateur admin sur les deux machines
* Créer un utilisateur à l'aide de la commande : ```sudo useradd NOM_UTILISATEUR``` et ```sudo passwd NOM_UTILISATEUR``` pour ajouter un mot de passe au nouvel utilisateur

### Modification du fichier de config sudoers
* Accéder au fichier de config :

```
[targa@node1 ~]$ sudo visudo
```
* Modifier wheel par le nom du groupe (admins)
```
%admins ALL=(ALL)       ALL
```
* Ajout de l'utilisateur dans le groupe admins

```
[targa@node1 bin]$ sudo usermod -aG admins toto
```
### Preuve que les utilisateurs ont des permissions admins
#### Première machine

```
[toto@node1 ~]$ sudo dnf update
[sudo] password for toto:
Last metadata expiration check: 0:01:16 ago on Tue 21 Sep 2021 11:09:38 PM CEST.
Dependencies resolved.
==========================================================================================================
 Package                           Arch        Version                               Repository      Size
==========================================================================================================
Installing:
 kernel                            x86_64      4.18.0-305.19.1.el8_4                 baseos         5.9 M
 kernel-core                       x86_64      4.18.0-305.19.1.el8_4                 baseos          36 M
 kernel-modules                    x86_64      4.18.0-305.19.1.el8_4                 baseos          28 M
Upgrading:
```
#### Deuxième machine

```
[toto@node2 ~]$ sudo dnf update
Rocky Linux 8 - AppStream                                                 5.2 kB/s | 4.8 kB     00:00
Rocky Linux 8 - AppStream                                                 2.4 MB/s | 8.7 MB     00:03
Roc<<<<<<<<<<<ky Linux 8 - BaseOS                                                    8.7 kB/s | 4.3 kB     00:00
Rocky Linux 8 - BaseOS                                                    3.0 MB/s | 7.4 MB     00:02
Rocky Linux 8 - Extras                                                    6.8 kB/s | 3.1 kB     00:00
Rocky Linux 8 - Extras                                                     17 kB/s |  11 kB     00:00
Dependencies resolved.
==========================================================================================================
 Package                           Arch        Version                               Repository      Size
==========================================================================================================
Installing:
 kernel                            x86_64      4.18.0-305.19.1.el8_4                 baseos         5.9 M
 kernel-core                       x86_64      4.18.0-305.19.1.el8_4                 baseos          36 M
 kernel-modules                    x86_64      4.18.0-305.19.1.el8_4                 baseos          28 M
Upgrading:
 NetworkManager                    x86_64      1:1.30.0-10.el8_4                     baseos         2.6 M
```

### 2. SSH
* Génération de la clé sur le pc locale avec la commande :``` ssh-keygen -t rsa -b 4096```

* Création un dossier ```.ssh ```sur les deux machines dans ```/home/toto/ ```:

```
mkdir .ssh
```
* Création d'un fichier, puis y coller la clé qu'on a généré 

```
vi authorized_keys 
```


* Définir les permissions sur le dossier .ssh et le fichier authorized_keys :
```
chmod 600 /home/toto/.ssh/authorized_keys
chmod 700 /home/toto/.ssh
```

#### Test sur les deux machines

```
C:\Users\lucas>ssh toto@10.101.1.11
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Sep 22 18:04:41 2021 from 10.101.1.1

```
```
[toto@node1 ~]$ cd /home
C:\Users\lucas>ssh toto@10.101.1.12
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Sep 22 18:06:48 2021 from 10.101.1.1
```
* La connexion s'effectue sans demande du mot de passe de l'utilisateur.
# II. Partitionnement
## 1. Préparation de la VM
* Création et ajout de deux disques de 3 Go à la machine
* ![](https://i.imgur.com/mXm54Go.png)

* 2. Partitionnement

*Lister les disques à ajouter

```
[toto@node1 ~]$ [toto@node1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0    8G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0    7G  0 part
 ├─rl-root 253:0    0  6.2G  0 lvm  /
 └─rl-swap 253:1    0  820M  0 lvm  [SWAP]
sdb           8:16   0    3G  0 disk
sdc           8:32   0    3G  0 disk
sr0          11:0    1 1024M  0 rom
```
* ```sdb``` et ```sdc``` correspondent aux disques crées précédemment.

* Ajout des disques en tant que PV ( Physical Volume) ainsi qu'une vérification.

```
[toto@node1 ~]$ sudo pvcreate /dev/sd
sda   sda1  sda2  sdb   sdc
[toto@node1 ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[toto@node1 ~]$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[toto@node1 ~]$ sudo pvs
  PV         VG Fmt  Attr PSize  PFree
  /dev/sda2  rl lvm2 a--  <7.00g    0
  /dev/sdb      lvm2 ---   3.00g 3.00g
  /dev/sdc      lvm2 ---   3.00g 3
```
* Création d'un VG (Volume Group) et l'ajout d'un autre PV (sdc)

```

[toto@node1 ~]$ sudo vgcreate data /dev/sdb
  Volume group "data" successfully created
[toto@node1 ~]$ sudo vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  data   1   0   0 wz--n- <3.00g <3.00g
  rl     1   2   0 wz--n- <7.00g     0

[toto@node1 ~]$ sudo vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  data   1   0   0 wz--n- <3.00g <3.00g
  rl     1   2   0 wz--n- <7.00g     0
[toto@node1 ~]$ sudo vgextend data /dev/sdc
  Volume group "data" successfully extended
[toto@node1 ~]$ sudo vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  data   2   0   0 wz--n-  5.99g 5.99g
  rl     1   2   0 wz--n- <7.00g    0
```

* On peut confirmer l'ajout du second PV par l'augmentation du volume de stockage.

* Création de trois LG ( Logical Volume ) ainsi qu'une vérification.

```
[toto@node1 ~]$ sudo lvcreate -L 1G data -n LG_1
  Logical volume "LG_1" created.
[toto@node1 ~]$ sudo lvcreate -L 1G data -n LG_2
  Logical volume "LG_2" created.
[toto@node1 ~]$ sudo lvcreate -L 1G data -n LG_3
  Logical volume "LG_3" created.
[toto@node1 ~]$ sudo lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LG_1 data -wi-a-----   1.00g
  LG_2 data -wi-a-----   1.00g
  LG_3 data -wi-a-----   1.00g
  root rl   -wi-ao----  <6.20g
  swap rl   -wi-ao---- 820.00m
```
```
[toto@node1 ~]$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/data/LG_1
  LV Name                LG_1
  VG Name                data
  LV UUID                cAKXqt-aYRn-D0wD-1AFa-e4wo-mCaO-miO9Vd
  LV Write Access        read/write
  LV Creation host, time node1.tp1.b2, 2021-09-22 22:56:16 +0200
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2

  --- Logical volume ---
  LV Path                /dev/data/LG_2
  LV Name                LG_2
  VG Name                data
  LV UUID                zzRbtV-AIID-qQDt-CQyF-XTqB-Fzgt-nYkxkf
  LV Write Access        read/write
  LV Creation host, time node1.tp1.b2, 2021-09-22 22:56:19 +0200
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3

  --- Logical volume ---
  LV Path                /dev/data/LG_3
  LV Name                LG_3
  VG Name                data
  LV UUID                4PU0qy-XdVF-5Zl7-vge7-x6lG-dbkA-g2AqEZ
  LV Write Access        read/write
  LV Creation host, time node1.tp1.b2, 2021-09-22 22:56:21 +0200
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:4
  [...]
```

* Formatage des partitions LG .

```

[toto@node1 ~]$ mkfs -t ext4 /dev/data/LG_1
mke2fs 1.45.6 (20-Mar-2020)
Could not open /dev/data/LG_1: Permission denied
[toto@node1 ~]$ sudo mkfs -t ext4 /dev/data/LG_1
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: ed7b3e18-0194-4ed0-ba30-384186d5084c
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[toto@node1 ~]$ sudo mkfs -t ext4 /dev/data/LG_2
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: 58e3a86f-bb69-4932-904c-931837ccddb1
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[toto@node1 ~]$ sudo mkfs -t ext4 /dev/data/LG_3
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: a6001b38-59c5-4cab-8494-fce9b34b37ab
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

* Montage des partitions LG et vérification.

```
[toto@node1 ~]$ sudo mkdir /mnt/part1
[sudo] password for toto:
[toto@node1 ~]$ sudo mkdir /mnt/part2
[toto@node1 ~]$ sudo mkdir /mnt/part3
[toto@node1 ~]$ sudo mount /dev/data/LG_1 /mnt/part1
[toto@node1 ~]$ sudo mount /dev/data/LG_2 /mnt/part2
[toto@node1 ~]$ sudo mount /dev/data/LG_3 /mnt/part3
```
```
[toto@node1 ~]$ df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rl-root    6.2G  1.9G  4.4G  31% /
/dev/sda1             1014M  182M  833M  18% /boot
tmpfs                  286M     0  286M   0% /run/user/2001
/dev/mapper/data-LG_1  976M  2.6M  907M   1% /mnt/part1
/dev/mapper/data-LG_2  976M  2.6M  907M   1% /mnt/part2
/dev/mapper/data-LG_3  976M  2.6M  907M   1% /mnt/part3
```
```
[toto@node1 ~]$ mount
[...]
/dev/mapper/data-LG_1 on /mnt/part1 type ext4 (rw,relatime,seclabel)
/dev/mapper/data-LG_2 on /mnt/part2 type ext4 (rw,relatime,seclabel)
/dev/mapper/data-LG_3 on /mnt/part3 type ext4 (rw,relatime,seclabel)
```
* Montage automatique des LG au boot de la machine

```
[toto@node1 ~]$ sudo vi /etc/fstab

[...]
/dev/data/LG_1 /mnt/part1 ext4 defaults 0 0
/dev/data/LG_2 /mnt/part2 ext4 defaults 0 0
/dev/data/LG_3 /mnt/part3 ext4 defaults 0 0
```

* Vérification si après un démarrage les partitions sont montées automatiquement.
```
C:\Users\lucas>ssh toto@10.101.1.11
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Sep 22 23:22:00 2021
[toto@node1 ~]$ df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.4G     0  1.4G   0% /dev
tmpfs                  1.4G     0  1.4G   0% /dev/shm
tmpfs                  1.4G  8.6M  1.4G   1% /run
tmpfs                  1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/rl-root    6.2G  1.9G  4.4G  31% /
/dev/sda1             1014M  182M  833M  18% /boot
/dev/mapper/data-LG_1  976M  2.6M  907M   1% /mnt/part1
/dev/mapper/data-LG_3  976M  2.6M  907M   1% /mnt/part3
/dev/mapper/data-LG_2  976M  2.6M  907M   1% /mnt/part2
tmpfs                  286M     0  286M   0% /run/user/2001
```
* Les trois partitions sont bien montées.
# III. Gestion de services
## 1. Interaction avec un service existant
* Vérification si le ```firewalld``` est bien démarré et activé

```
[toto@node1 ~]$ systemctl is-active firewalld
active

[toto@node1 ~]$ systemctl is-enabled firewalld
enabled
```

## 2. Création de service
### A. Unité simpliste

* Créer un fichier web.service dans ```/etc/systemd/system``` et insérer le contenu suivant.


```
[Unit]
Description=Very simple web service

[Service]
ExecStart=/bin/python3 -m http.server 8888

[Install]
WantedBy=multi-user.target
```

* Autoriser les connexions sur le port ```8888```

```
[toto@node1 system]$ sudo firewall-cmd --add-port=8888/tcp --permanent
success
```
* Demander à ```systemd``` de relire les fichiers de configuration :
```
[toto@node1 system]$ sudo systemctl daemon-reload
On utilise les commandes suivantes pour interagir avec le serveur :
sudo systemctl start web 
```
* Pour lancer le serveur sudo systemctl enable webpour activer le serveursudo systemctl status web pour vérifier le statut du serveur.
```
[toto@node1 system]$ sudo systemctl start web
[toto@node1 system]$ sudo systemctl enable web
[toto@node1 system]$ sudo systemctl status web
● web.service - Very simple web service
   Loaded: loaded (/etc/systemd/system/web.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-09-22 23:58:26 CEST; 9s ago
 Main PID: 27440 (python3)
    Tasks: 1 (limit: 18054)
   Memory: 9.8M
   CGroup: /system.slice/web.service
           └─27440 /bin/python3 -m http.server 8888

Sep 22 23:58:26 node1.tp1.b2 systemd[1]: Started Very simple web service.
```

### B. Modification de l'unité

- Création d'un utilisateur web
```
[toto@node1 ~]$ sudo useradd web
```

* Modification du fichier ```web.service``` en ajoutant les lignes suivantes dans la section ```[Service]``` .

```
User=web
WorkingDirectory=/srv/web_dir
```

* Création d'un fichier dans le dossier ```web_dir``` crée précédemment dans ```/srv```

* Se connecter au serveur en ```localhost``` ou avec ```curl``` avec l'ip de la machine ```10.101.1.11:8888``` et le port ```8888``` :

http://10.101.1.11:8888/
![](https://i.imgur.com/bG5GaOl.png)
