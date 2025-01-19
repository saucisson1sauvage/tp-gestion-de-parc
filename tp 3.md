### Script autoconfig.sh

```bash
#!/bin/bash

echo "Demarage du script"


#check user valide

whoami="$(whoami)"

if [ $whoami != root ]; then
  echo "le script doit être démaré en tant que root"
fi


#check SELinux
echo "gestion de SELinux "
sestatus="$(sestatus | cut -d ' ' -f 14-14 | tr -d '\n')"
if [ $sestatus != permissive ]; then
	echo "changement du mode de sestatus en permisisve"
	setenforce 0
fi

#check conf SELinux

sestatus2="$(sestatus | cut -d ' ' -f 21-21 | tr -d '\n')"
if [ $sestatus2 != permissive ]; then
        echo "changement de la configuration de sestatus en permisisve"
        sed -i 's/SELINUX=/#SELINUX=/g' /etc/selinux/config
	echo 'SELINUX=permissive' >>  /etc/selinux/config
fi

#check firewall actif ou pa

echo "gestion du firewall"
firewallstat="$(firewall-cmd --list-all | cut -d ' ' -f 2-2)"
if [ $firewallstat != "(active)" ]; then
	exit
fi

#check si y a des trucs sur port 22

p22="$(ss -lnpt | grep sshd | grep ':22' | wc -l)"

#si oui

if [ $p22 != 0 ]; then

	#genere
	randport="$(shuf -i 1025-65534 -n 1)"

	#change default
	echo "Port $randport" >> /etc/ssh/sshd_config		

	#change firewall
	firewall-cmd --permanent --add-port=$randport/tcp && firewall-cmd --permanent --remove-port=22/tcp

	firewall-cmd --reload
	systemctl restart sshd
	echo "port changé sur  $randport"
else
	echo "affichage du message de succès . . ."
fi

#def nom machine

#check nom
echo "gestion du nom de la machine"
if [ $# == 1 ]; then
	name="$(hostnamectl | grep "Tran" | cut -d ' ' -f 3-3)"
fi

#check et changement if nom is localhost

if [ "$name" = "localhost" ]; then

	echo "nous avons détécté que la machine s'appelait localhost"
	hostnamectl set-hostname $1
	echo "changement du nom pour $1"

else
	
	echo "cette machine est deja nomée"
	echo "voulez vous tout de meme renomer votre machine en $1?" 
	echo "y : yes, n : no"
	oenn="$(read)"
	if [ "$oenn" = "y" ]; then

		hostnamectl set-hostname $1	
	fi
fi

#verif appartient au group wheel

echo "gestion du groupe de l'utilisateur"

grp="$(cat /etc/group | grep "wheel" | cut -d ':' -f 4-4)"



if [ $grp != $SUDO_USER ]; then

	usermod -aG wheel $SUDO_USER
	echo "vous etes desormais dans le group wheel"
else
	echo "vous etes deja dans le group wheel"
fi

echo "configuration terminée."
```

###  Exécution du script autoconfig.sh développé à la partie I

```
[me@localhost ~]$ sudo /opt/autoconfig.sh music.tp3.b1
Demarage du script
gestion de SELinux 
gestion du firewall
success
success
success
port changé sur  65037
gestion du nom de la machine
nous avons détécté que la machine s'appelait localhost
changement du nom pour music.tp3.b1
gestion du groupe de l'utilisateur
vous etes desormais dans le group wheel
configuration terminée.
```

###  Création d'un dossier où on hébergera les fichiers de musique

```
[me@B001-04 ~]$ sudo mkdir /srv/music

```

###  Déposez quelques fichiers son là dedans

```
me@music music]$ ls
'Roblox Death Sound - OOF ｜ Sound Effect HD - HomeMadeSoundEffects.mp3'   tomatoAnxiety.mp3

```

### Installer le paquet jellyfin

```
sudo dnf install jellyfin
```

###  Lancer le service jellyfin

```
sudo service jellyfin start
```

### Afficher la liste des ports TCP en écoute

```
[me@music ~]$ sudo ss -tulpn | grep jellyfin | grep LISTEN
tcp   LISTEN 0      512          0.0.0.0:8096       0.0.0.0:*    users:(("jellyfin",pid=2905,fd=310))
```

###  Ouvrir le port derrière lequel Jellyfin écoute

```
[me@music ~]$ sudo firewall-cmd --permanent --add-port=8096/tcp
success
[me@music ~]$ sudo firewall-cmd --reload
success
```

###  Visitez l'interface Web !
```
curl -I http://10.3.1.11:8096
HTTP/1.1 302 Found
Date: Wed, 15 Jan 2025 10:44:41 GMT
Server: Kestrel
Location: /web/index.html
```

###  Dérouler le script autoconfig.sh développé à la partie I
```
[me@localhost ~]$ sudo /opt/autoconfig.sh music.tp3.b1
Demarage du script
gestion de SELinux 
gestion du firewall
success
success
success
port changé sur 12769
gestion du nom de la machine
nous avons détécté que la machine s'appelait localhost
changement du nom pour monitoring.tp3.b1
gestion du groupe de l'utilisateur
vous etes desormais dans le group wheel
configuration terminée.
```

###  Installer Netdata
```
curl https://get.netdata.cloud/kickstart.sh > /tmp/netdata-kickstart.sh && sh /tmp/netdata-kickstart.sh --no-updates --stable-channel --disable-telemetry
```

###  Ajouter un check TCP

```
jobs:
 - name: jtebaisqz
   host: 10.3.1.11
   ports:
    - 8096
```

### Ajout d'une alerte Discord

https://discord.com/channels/1329051604028096533/1329051604028096536


###  Partitionner le disque dur

```[me@backup dev]$ lsblk
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                8:0    0   20G  0 disk 
├─sda1             8:1    0    1G  0 part /boot
└─sda2             8:2    0   19G  0 part 
  ├─rl_vbox-root 253:0    0   17G  0 lvm  /
  └─rl_vbox-swap 253:1    0    2G  0 lvm  [SWAP]
sdb                8:16   0    5G  0 disk 
sr0               11:0    1 1024M  0 rom  
[me@backup dev]$ sudo pvcreate /dev/sdb
[sudo] password for me: 
  Physical volume "/dev/sdb" successfully created.


[me@backup dev]$ sudo pvs
  PV         VG      Fmt  Attr PSize   PFree
  /dev/sda2  rl_vme@backup dev]$ sudo vgcreate backupstor /dev/sdb
 
 
 Volume group "backupstor" successfully createdbox lvm2 a--  <19.00g    0 
  /dev/sdb           lvm2 ---    5.00g 5.00g

[me@backup dev]$ sudo lvcreate -l 100%FREE backupstor -n backup
  Logical volume "backup" created.

```

###  Formater la partition créée

```
[me@backup dev]$ sudo mkfs -t ext4 /dev/backupstor/backup
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1309696 4k blocks and 327680 inodes
Filesystem UUID: d22dcfc1-0a54-4948-9ba4-09058f7e93d7
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
``` 

###  Monter la partition

```

[me@backup dev]$ sudo mkdir /mnt/backup
[me@backup dev]$ sudo mount /dev/backupstor/backup /mnt/backup
```


###  Configurer un montage automatique de la partition

```
[me@backup ~]$ sudo mount -av
/                        : ignored

...

mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
/mnt/backup              : successfully mounted
```


###  Installer et configurer un service NFS

```
[me@backup ~]$ sudo firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client mountd nfs rpc-bind ssh

```

### Déterminer sur quel port écoute le service NFS

```
[me@backup backupstor]$ ss -tuln | grep nfs
[me@backup backupstor]$ rpcinfo -p | grep nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl


...
 ```

 ###  Installer les outils NFS



 ```
 [me@music ~]$ sudo dnf install -y nfs-utils
Installing:
 nfs-utils              x86_64       1:2.5.4-27.el9  
```

###  Essayer d'accéder au dossier partagé

```
me@music ~]$ df -h | grep music_backup
df: /nfs/backup: Stale file handle
10.3.1.13:/mnt/backup     4.9G     0  4.6G   0% /mnt/music_backup
[me@music ~]$ sudo mount -t nfs 10.3.1.13:/mnt/backup /mnt/music_backup
[me@music ~]$ ls /mnt/music_backup
general.test  lost+found


```

###  Configurer un montage automatique

```
[me@music ~]$ sudo umount /mnt/music_backup
[me@music ~]$ sudo mount -a
```


###  Script backup.sh#!/bin/bash

```bash

#!/bin/bash

#compression
echo "comp"
date=$(date +"%y%m%d_%H%M%S")
nom_archive=$"music_${date}.tar.gz"
tar -czf $nom_archive /srv/music

#deplacement
echo "dep"
mv $nom_archive /mnt/music_backup

#chech erreur

if [ $? == 0 ]; then
    
	echo "fichier sauvgardé avec succes"
else

	echo "erreur lors de la sauvgarde"
fi
```
###  Créer un nouveau service backup.service

```bash

[Unit]
Description=backup mouzik

[Service]
Type=oneshot
ExecStart=/opt/savebackup.sh

[Install]
WantedBy=multi-user.target

```

### Indiquer au système qu'on a ajouté un nouveau service

```
[me@music opt]$ sudo systemctl daemon-reload
```

###  Utiliser et tester le nouveau service

```
[me@music opt]$ systemctl status backup
○ backup.service - backup mouzik
     Loaded: loaded (/etc/systemd/system/backup.service; disabled; preset: disab>
[me@music opt]$ sudo systemctl start backup.service
```

###  Faire un test et prouvez que ça a fonctionné


```
Jan 19 13:49:30 music.tp3.b1 systemd[1]: Finished backup_mouzik.
░░ Subject: A start job for unit backup.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░ 
░░ A start job for unit backup.service has finished successfully.
░░ 
░░ The job identifier is 14585.
```
```
[me@music opt]$ ls /mnt/music_backup/
general.test  lost+found  music_250119_132833.tar.gz  music_250119_134930.tar.gz
[me@backup backupstor]$ ls /mnt/backup/
general.test  lost+found  music_250119_132833.tar.gz  music_250119_134930.tar.gz
```

###  Configurer un lancement automatique du service à intervalles réguliers

```bash
[Unit]
Description=timer_backup_mouzik

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```
```
[me@music opt]$ sudo systemctl list-timers | grep backup
Sun 2025-01-19 15:00:00 CET 58min left Sun 2025-01-19 14:00:01 CET 59s ago   backup.timer                 backup.service
```