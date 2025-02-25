# Part I : A bit of exploration

## 1. /proc

🌞 **Afficher...** :

- l'état complet de la mémoire (RAM)
  ```bash
  [user1@efrei-xmg4agau1 ~]$ cat /proc/meminfo
  ```
- le nombre de coeurs que votre CPU a (uniquement ce nombre)
  ```bash
  [user1@efrei-xmg4agau1 ~]$ cat /proc/cpuinfo | grep -c processor
  1
  ```
- le nombre de processus lancés (uniquement ce nombre)
  ```bash
  [user1@efrei-xmg4agau1 ~]$ ls /proc | grep -E '^[0-9]+$' | wc -l
  101
  ```
- la ligne de commande utilisée pour lancer le kernel actuel
  ```bash
  [user1@efrei-xmg4agau1 ~]$ cat /proc/cmdline
  BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-503.22.1.el9_5.x86_64 root=/dev/mapper/rl-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap
  ```
- la liste des connexions TCP actuelles (même si c'est un peu imbuvable avec nos p'tits yeux)
  ```bash
  cat /proc/net/tcp
  ```
- la valeur actuelle de la *swappiness* (cf le tip ci-dessous)
  ```bash
  [user1@efrei-xmg4agau1 ~]$ cat /proc/sys/vm/swappiness
  60
  ```

## 2. /sys

🌞 **Afficher...** :

- la liste des périphériques de types bloc reconnus par l'OS (genre les disques durs par exemple koa)
  ```bash
  ls /sys/block/
  ```
- la liste des modules kernel qui sont actuellements en cours d'utilisation
  ```bash
  ls /sys/module/
  ```
- la liste des cartes réseau
  ```bash
  [user1@efrei-xmg4agau1 ~]$ ls /sys/class/net/
  enp0s3  enp0s8  lo
  ```
