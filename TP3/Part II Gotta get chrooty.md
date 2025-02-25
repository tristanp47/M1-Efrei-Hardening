# Part II. Gotta get chrooty

## 1. Play manually

🌞 **Créez le dossier `/srv/get_chrooted/`**
```bash
[user1@efrei-xmg4agau1 /]$ sudo mkdir /srv/get_chrooted/
```
🌞 **Essayez de `chroot` à l'intérieur en lançant un shell**
```bash
[user1@efrei-xmg4agau1 ~]$ sudo cp /bin/bash /srv/get_chrooted/bin/

[user1@efrei-xmg4agau1 ~]$ sudo mkdir -p /srv/get_chrooted/bin/

[user1@efrei-xmg4agau1 ~]$ sudo mkdir /srv/get_chrooted/lib64/
[user1@efrei-xmg4agau1 ~]$ sudo cp /lib64/libtinfo.so.6 /srv/get_chrooted/lib64/
[user1@efrei-xmg4agau1 ~]$ sudo cp /lib64/ld-linux-x86-64.so.2 /srv/get_chrooted/lib64/

[user1@efrei-xmg4agau1 ~]$ sudo chroot /srv/get_chrooted/ /bin/bash
bash-5.1#

[user1@efrei-xmg4agau1 ~]$ sudo cp /bin/ls /srv/get_chrooted/bin/

[user1@efrei-xmg4agau1 ~]$ sudo cp /lib64/libselinux.so.1 /srv/get_chrooted/lib64/
[user1@efrei-xmg4agau1 ~]$ sudo cp /lib64/libcap.so.2 /srv/get_chrooted/lib64/
[user1@efrei-xmg4agau1 ~]$ sudo cp /lib64/libpcre2-8.so.0 /srv/get_chrooted/lib64/

[user1@efrei-xmg4agau1 ~]$ sudo chroot /srv/get_chrooted/ /bin/bash
bash-5.1# ls
bin  lib64
```

## 2. SSH old friend

🌞 **Créez un user `imsad`**

🌞 **Modifier la configuration du serveur SSH**

- uniquement quand le user `imsad` se connecte en SSH :
  - il est `chroot`é dans `/srv/get_chrooted/`
  - son shell doit fonctionner normalement
  - il se connecte avec une clé SSH
  - il ne doit pas pouvoir remarquer qu'il est `chroot`é dans un dossier particulier
- dans le compte-rendu :
  - montrez une connexion SSH fonctionnelle sur le user `imsad`
  - prouvez que le `bash` ouvert est bien `chroot`é
