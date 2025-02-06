## Part I : User management

### 1. Existing users

#### Affichage des utilisateurs existants:
```bash
cat /etc/passwd
user1:x:1001:1001::/home/user1:/bin/bash
```
#### Affichage des groupes existants:
```bash
cat /etc/group
```
#### Affichage des groupes de l'utilisateur:
```bash
groups user1
user1 : user1
```
#### Affichage des processus en cours lancés par root:
```bash
ps -u root
```
#### Affichage des processus en cours de user1:
```bash
ps -u user1
    PID TTY          TIME CMD
    869 tty1     00:00:00 bash
    925 ?        00:00:00 systemd
    928 ?        00:00:00 (sd-pam)
    935 ?        00:00:00 sshd
    936 pts/0    00:00:00 bash
   1022 pts/0    00:00:00 ps
```
#### Affichage du hash du mot de passe de root:
```bash
sudo cat /etc/shadow | grep root
root:$6$.8fzl//9C0M819BS$Sw1mrG49Md8cyNUn0Ai0vlthhzuSZpJ/XVfersVmgXDSBrTVchneIWHYHnT3mC/NutmPS03TneWAHihO0NXrj1::0:99999:7:::
```
#### Affichage du hash du mot de passe de user1:
```bash
sudo cat /etc/shadow | grep user1
user1:$6$TzFMCM9WOYSNhm6V$4Zqay15mONKKAP5hXlw5/7ilIKY2O.NOXnbn.QJv1PAIyWXF2BS2X6zstnkQ0X399lQCvTbjaoF7wZGHlsM9C1:20122:0:99999:7:::
```

#### Affichage du chemin du shell par défaut de mon user1:
```bash
[user1@efrei-xmg4agau1 ~]$ echo $SHELL
/bin/bash
```
#### Affichage du chemin du repertoire de mon user1:
```bash
[user1@efrei-xmg4agau1 ~]$ echo $HOME
/home/user1
```
#### Avec la commande "sudo cat /etc/sudoers" on peut voir que les utilisateurs dans le groupe wheel sont des utilisateurs sudo.
```bash
[user1@efrei-xmg4agau1 ~]$ sudo cat /etc/sudoers
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
```

### 2. User creation and configuration

#### Création de l'utilisateur meow:
```bash
sudo useradd -M -g admins -s /sbin/nologin meow
```
#### Configuration des permissions sudoers on autorise uniquement ls, cat, less et more en tant que l'utilisateur user1:
```bash
meow ALL=(user1) NOPASSWD: /bin/ls, /bin/cat, /usr/bin/less, /usr/bin/more
[user1@efrei-xmg4agau1 ~]$ sudo su - meow -s /bin/bash
[meow@efrei-xmg4agau1 user1]$ sudo -u user1 ls
```

#### Dans la config de sudoers on autorise uniquement la commande yum en root pour les membres du groupe admins:
```bash
%admins ALL=(root) NOPASSWD: /usr/bin/yum

[user1@efrei-xmg4agau1 ~]$ sudo su - meow -s /bin/bash
[meow@efrei-xmg4agau1 user1]$ yum
usage: yum [options] COMMAND
```

#### Dans la config de sudoers on autorise uniquement la commande yum en root pour les membres du groupe admins:
```bash
%admins ALL=(root) NOPASSWD: /usr/bin/yum

[user1@efrei-xmg4agau1 ~]$ sudo su - meow -s /bin/bash
[meow@efrei-xmg4agau1 user1]$ yum
usage: yum [options] COMMAND
```
#### Dans la config de sudoers on autorise notre utilisateur "user1" a entrer des commandes sans mot de passe:
```bash
user1 ALL=(root) NOPASSWD: ALL
```
### 3. Hackers gonna hack

#### Les commandes autorisé dans la config shudoers : less et more permettent des élévations de priviléges:
```bash
[user1@efrei-xmg4agau1 ~]$ sudo su -s /bin/bash - meow
[meow@efrei-xmg4agau1 user1]$ sudo -u user1 less /var/log/dnf.log
#Dans les fichiers avec less lancer un terminal
!/bin/bash
#On a alors accès a notre user1 qui à les droits root
[user1@efrei-xmg4agau1 ~]$
```
#### La solution est donc de les supprimer dans la config sudoers:
```bash
meow ALL=(user1) NOPASSWD: /bin/ls, /bin/cat
```
---

## Part II : Files and permissions

### 1. Listing POSIX permissions

#### Vérification des permissions de fichiers critiques:
```bash
#Permissions du fichier qui contient la liste des utilisateurs
[user1@efrei-xmg4agau1 ~]$ ls -l /etc
-rw-r--r--. 1 root root     1049 Feb  3 12:21 passwd

#Permissions du fichier qui contient les hash des mot de passe
[user1@efrei-xmg4agau1 ~]$ ls -l /etc
----------. 1 root root      990 Feb  3 12:42 shadow

#Permissions du repertoire du contient OpenSSH
[user1@efrei-xmg4agau1 ~]$ sudo ls -l /usr/sbin
-rwxr-xr-x. 1 root root  968792 Apr 18  2024 sshd

#Permissions du repertoire de Root
[user1@efrei-xmg4agau1 ~]$ sudo ls -l /
dr-xr-x---.   3 root root  163 Feb  3 10:49 root

#Permissions du repertoire user1
[user1@efrei-xmg4agau1 ~]$ sudo ls -l /home
drwx------. 2 user1 user1  83 Feb  3 15:11 user1

#Persmissions du repertoire de ls
[user1@efrei-xmg4agau1 ~]$ sudo ls -l /usr/bin
-rwxr-xr-x. 1 root root  140872 Apr 20  2024  ls

#Persmissions du repertoire systemctl
[user1@efrei-xmg4agau1 ~]$ sudo ls -l /usr/bin
-rwxr-xr-x. 1 root root  305680 Apr  8  2024  systemctl
```

#### Recherche des fichiers avec SUID activé:
```bash
[user1@efrei-xmg4agau1 ~]$ find / -type f -perm /6000 -ls 2>/dev/null
  8640211     72 -rwsr-xr-x   1 root     root        73704 Oct 30  2023 /usr/bin/chage
  8640212     80 -rwsr-xr-x   1 root     root        78024 Oct 30  2023 /usr/bin/gpasswd
  8640215     44 -rwsr-xr-x   1 root     root        41744 Oct 30  2023 /usr/bin/newgrp
  8818717     48 -rwsr-xr-x   1 root     root        48600 Apr 20  2024 /usr/bin/mount
  8818723     36 -rwsr-xr-x   1 root     root        36224 Apr 20  2024 /usr/bin/umount
  8834129     56 -rwsr-xr-x   1 root     root        57056 Apr 20  2024 /usr/bin/su
  8834138     24 -rwxr-sr-x   1 root     tty         23912 Apr 20  2024 /usr/bin/write
  8847081     56 -rwsr-xr-x   1 root     root        57240 Apr 16  2024 /usr/bin/crontab
  9095262     32 -rwsr-xr-x   1 root     root        32656 May 15  2022 /usr/bin/passwd
  9095398    184 ---s--x--x   1 root     root       185304 Feb 14  2024 /usr/bin/sudo
 13020238     24 -rwsr-xr-x   1 root     root        23952 Apr 16  2024 /usr/sbin/unix_chkpwd
 13020236     16 -rwsr-xr-x   1 root     root        15608 Apr 16  2024 /usr/sbin/pam_timestamp_check
 13071236     16 -rwsr-xr-x   1 root     root        15440 May  1  2024 /usr/sbin/grub2-set-bootflag
  4629262     16 -rwx--s--x   1 root     utmp        16064 May 16  2022 /usr/libexec/utempter/utempter
  4699690    336 -r-xr-sr-x   1 root     ssh_keys   341248 Apr 18  2024 /usr/libexec/openssh/ssh-keysign
```

#### Restreindre l'accès à un fichier personnel:
```bash
#La commande chattr avec +i permet de rendre le fichier de conf de OpenSSH immuable
[user1@efrei-xmg4agau1 /]$ sudo chattr +i /etc/ssh/sshd_config
#Avec la commande lsattr permet de vérifier que le fichier est bien immuable
[user1@efrei-xmg4agau1 /]$ sudo lsattr /etc/ssh/sshd_config
----i----------------- /etc/ssh/sshd_config

#On applique les droits de lecture et écriture sur le fichier 
[user1@efrei-xmg4agau1 bonsoir]$ chmod 600 dont_readme.txt
-rw-------. 1 user1 user1 8 Feb  3 16:03 dont_readme.txt

#Avec notre utilisateur on a bien le droit de lecture sur le fichier
[user1@efrei-xmg4agau1 bonsoir]$ cat dont_readme.txt
bonjour

#On peut également ecrire sur le fichier avec notre user
[user1@efrei-xmg4agau1 bonsoir]$ sudo echo "hello" > dont_readme.txt

#L'utilisateur meow ne peut pas écrire ni lire sur le fichier
[meow@efrei-xmg4agau1 bonsoir]$ sudo -u user1 echo "hello" > dont_readme.txt
-bash: dont_readme.txt: Permission denied

#Root lui a tout les droits sur le fichier
[root@efrei-xmg4agau1 bonsoir]# echo "salut" > dont_readme.txt
[root@efrei-xmg4agau1 bonsoir]# cat dont_readme.txt
salut
```

---

## Part III : Networking

### 1. Listening ports

#### La commande ss nous permet d'afficher la liste des programmes qui écoutent sur TCP et UDP
```bash
[user1@efrei-xmg4agau1 ~]$ sudo ss -tulnp
[sudo] password for user1:
Netid       State        Recv-Q       Send-Q               Local Address:Port               Peer Address:Port       Process
udp         UNCONN       0            0                        127.0.0.1:323                     0.0.0.0:*           users:(("chronyd",pid=654,fd=5))
udp         UNCONN       0            0                            [::1]:323                        [::]:*           users:(("chronyd",pid=654,fd=6))
tcp         LISTEN       0            128                        0.0.0.0:22                      0.0.0.0:*           users:(("sshd",pid=693,fd=3))
tcp         LISTEN       0            128                           [::]:22       
```

### 2. Firewalling
#### Seul pour le port 22 il existe une régle de Pare-feu qui autorise le traffic entrant
```bash
chain filter_IN_public_allow {
                tcp dport 22 accept
                ip6 daddr fe80::/64 udp dport 546 accept
                tcp dport 9090 accept
        }

#Etant donné que rien n'écoute sur le port 323 car il est UNCONN on va le fermer avec la commande:
sudo iptables -A INPUT -p tcp --dport 323 -j DROP
```

---

## Part IV : Storage and partitions

### 1. Existing partitions
```bash
#La commande df nous permet d'afficher les partitions montées
[user1@efrei-xmg4agau1 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                229M     0  229M   0% /dev/shm
tmpfs                 92M  2.4M   90M   3% /run
/dev/mapper/rl-root  6.2G  1.5G  4.8G  24% /
/dev/sda1            960M  225M  736M  24% /boot
tmpfs                 46M     0   46M   0% /run/user/1001

#Cette commande nous permet d'identifier la partition monté sur /
[user1@efrei-xmg4agau1 ~]$ df -h | grep /dev
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                229M     0  229M   0% /dev/shm
/dev/mapper/rl-root  6.2G  1.5G  4.8G  24% /
/dev/sda1            960M  225M  736M  24% /boot
```

### 2. Mount options
```bash
#rw signifie read-write, lecture et écriture.
#relatim ignifie relative atime, temps d’accès relatif.
#seclabel indique que le système de fichiers prend en charge l'étiquetage de sécurité.
#attr2 active une gestion plus efficace des attributs étendus (xattrs).
#inode64 permet l'utilisation de numéros d'inodes sur 64 bits.
#logbufs=8 définit le nombre de buffers utilisés pour la journalisation.
#logbsize=32k Définit la taille des blocs utilisés pour stocker les entrées du journal XFS (ici, 32 Ko).
#noquota Désactive le système de quotas sur le système de fichiers.

[user1@efrei-xmg4agau1 ~]$ findmnt -no OPTIONS /
rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota

#On monter une partition de type tmpfs sur le dossier /tmp avec l'option noexec pour empécher un programme d'être lancé.
[user1@localhost ~]$ sudo mount -t tmpfs -o rw,nosuid,nodev,noexec,relatime,size=2G tmpfs /tmp
[user1@localhost ~]$ mount | grep /tmp
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noexec,relatime,seclabel,size=2097152k,inode64)
#On copie le programme ls
[user1@localhost ~]$ cp /bin/ls /tmp/
#On éléve les permissions
[user1@localhost ~]$ chmod 777 /tmp/ls
#On ne peut donc pas lancer le programme
[user1@localhost ~]$ /tmp/ls
-bash: /tmp/ls: Permission denied
```

---

## Part V : OpenSSH Server

### 1. Basics

#### Changement du port SSH:
```bash
#Avec la commande ps aux on cherche l'identifiant du processus openssh ici 687
[user1@localhost ~]$ ps aux | grep sshd | grep -v grep
root         687  0.0  1.9  16772  9216 ?        Ss   11:55   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 star

#Dans la configuration de sshd on modifie le port pour qu'il soit 2222
sudo nano /etc/ssh/sshd_config
Port 2222

#Puis on relance le service sshd
sudo systemctl restart sshd

#On vérifie que SSH écoute bien maintenant sur le port 2222
[user1@localhost ~]$ sudo ss -tuln | grep 2222
tcp   LISTEN 0      128          0.0.0.0:2222      0.0.0.0:*
tcp   LISTEN 0      128             [::]:2222         [::]:*

#On peut donc maintenant établir une connection
PS C:\Users\trist> ssh -p 2222 user1@10.0.1.8
user1@10.0.1.8's password:
Last login: Thu Feb  6 12:22:22 2025 from 10.0.1.8
```

### 2. Authentication modes
```bash
#Sur mon hote on va générer une clé ssh 
PS C:\Users\trist> ssh-keygen -t rsa -b 4096 -C "test@example.com"

#On va ensuite copier la clé depuis mon hote vers le serveur ssh depuis powershell
PS C:\Users\trist> type $env:USERPROFILE\.ssh\id_rsa.pub | ssh -p 2222 user1@10.0.1.8 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

#Je peux maintenant me connecter à mon hote sans mot de passe
PS C:\Users\trist> ssh -p 2222 user1@10.0.1.8
Last login: Thu Feb  6 12:29:20 2025 from 10.0.1.1
[user1@localhost ~]$

#Dans le fichier de config sshd on ajoute la ligne suivante pour interdire l'authentification par mot de passe:
PasswordAuthentication no

#Un mot de passe ne nous est donc plus demandé pour s'authentifier
PS C:\Users\trist> ssh -p 2222 user1@10.0.1.8
Last login: Thu Feb  6 12:35:36 2025 from 10.0.1.1

#Dans le fichier de config sshd on ajoute la ligne suivante pour interdire l'authentification par root:
PermitRootLogin no

#On ne peut pas s'authentifier avec root
PS C:\Users\trist> ssh -p 2222 root@10.0.1.8
root@10.0.1.8: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

### 3. Further hardening
```bash

#Dans la config SSH "PermitEmptyPasswords no" Désactiver les mots de passe vides.
PermitEmptyPasswords no

#Dans la config SSH on peut Configurer un délai d'inactivité :
ClientAliveInterval 300
ClientAliveCountMax 0

#On peut limiter le nombres d'utilisateurs autorisés
AllowUsers user1 user2

#On peut désactiver le transfert X11 pour réduire les risques de sécurité liés à l'affichage graphique.
X11Forwarding no

#Il est également possible de configurer un message de bannière d'avertissement.
Banner /etc/issue.net
```

### 4. fail2ban
```bash
sudo yum install fail2ban
sudo nano /etc/fail2ban/jail.local
```
```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/secure
maxretry = 7
findtime = 300
```

#### Vérification des bannissements:
```bash
#On installe fail2ban:
sudo yum install fail2ban

#On modifie la config pour qu'elle soit en accord avec ce que l'on veut un maximum de 7 tentatives,
#en cas de multiples tentatives de connexion échouées sur le serveur SSH, l'utilisateur sera banni
#c'est l'adresse IP de la personne qui fait des connexions échouées de façon répétée qui est blacklistée
 
[user1@localhost ~]$ sudo nano /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/secure
maxretry = 7
findtime = 300

#On vérifie que l'adresse IP est bien bannie
[user1@localhost ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	7
|  `- Journal matches:	_SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned:	1
   |- Total banned:	1
   `- Banned IP list:	10.0.1.8

[user1@localhost ~]$ sudo fail2ban-client set sshd unbanip 10.0.1.8
1
[user1@localhost ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	7
|  `- Journal matches:	_SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned:	0
   |- Total banned:	1
   `- Banned IP list:	 
```
