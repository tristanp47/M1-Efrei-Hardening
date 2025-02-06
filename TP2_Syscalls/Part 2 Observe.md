# Part II : Observe

## 1. strace

Si on veut tracer un processus avec `strace`, c'est comme ça :

```bash
# pour tracer l'exécution d'un echo par exemple
$ strace echo yo
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `ls`**

- faites `ls` sur un dossier qui contient des trucs
- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ strace -o lm.txt ls /usr/bin/test
  /usr/bin/test
  [user1@efrei-xmg4agau1 ~]$ grep write lm.txt
  write(1, "/usr/bin/test\n", 14)         = 14
  ```
  
🌞 **Utiliser `strace` pour tracer l'exécution de la commande `cat`**

- faites `cat` sur un fichier qui contient des trucs
  ```bash
  strace -o lam.txt cat ~/test/ok.txt
  ```
- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
  ```bash
  [user1@efrei-xmg4agau1 ~]$ grep open lam.txt
  openat(AT_FDCWD, "/home/user1/test/ok.txt", O_RDONLY) = 3
  ```
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
  ```bash
  [user1@efrei-xmg4agau1 ~]$ grep write lam.txt
  write(1, "ok\n", 3)                     = 3
  ```

🌞 **Utiliser `strace` pour tracer l'exécution de `curl example.org`**

- vous devez utiliser une option de `strace`
- elle affiche juste un tableau qui liste tous les *syscalls*  appelés par la commande tracée, et combien de fois ils ont été appelé
  ```bash
  [user1@efrei-xmg4agau1 ~]$ strace -c curl example.org
  % time     seconds  usecs/call     calls    errors syscall
  ------ ----------- ----------- --------- --------- ----------------
   20.16    0.000943          36        26           poll
   18.23    0.000853           6       141           mmap
   10.69    0.000500         250         2           socket
   10.58    0.000495           8        60        14 openat
    8.19    0.000383          54         7           write
    5.45    0.000255           3        77           rt_sigaction
    4.90    0.000229         114         2           socketpair
    4.72    0.000221           4        54           close
    4.45    0.000208           5        37           mprotect
    2.37    0.000111           3        36           read
    2.33    0.000109           2        46           fstat
    1.69    0.000079           3        24           futex
    1.30    0.000061          61         1         1 connect
    0.75    0.000035          35         1           sendto
    0.53    0.000025          25         1           recvfrom
    0.43    0.000020          20         1           set_tid_address
    0.43    0.000020          20         1           clone3
    0.36    0.000017           4         4           setsockopt
    0.34    0.000016           8         2           statfs
    0.30    0.000014           2         6           fcntl
    0.21    0.000010           2         4           pread64
    0.19    0.000009           9         1           munmap
    0.19    0.000009           2         4           brk
    0.17    0.000008           8         1           pipe
    0.15    0.000007           3         2           ioctl
    0.13    0.000006           2         3           rt_sigprocmask
    0.13    0.000006           3         2         1 access
    0.11    0.000005           5         1           getsockopt
    0.09    0.000004           4         1           getsockname
    0.09    0.000004           4         1           sysinfo
    0.06    0.000003           3         1           getpeername
    0.06    0.000003           1         2         1 arch_prctl
    0.06    0.000003           3         1           prlimit64
    0.06    0.000003           3         1           getrandom
    0.04    0.000002           2         1           set_robust_list
    0.04    0.000002           2         1           rseq
    0.00    0.000000           0         1           execve
    0.00    0.000000           0         2           getdents64
    0.00    0.000000           0         2           newfstatat
  ------ ----------- ----------- --------- --------- ----------------
  100.00    0.004678           8       561        17 total
  ```

## 2. sysdig

### A. Intro

`sysdig` est un outil qui permet de faire pleiiin de trucs, et notamment tracer les *syscalls*  que le kernel reçoit.

Si on le lance sans préciser, il affichera TOUS les *syscalls*  que reçoit votre kernel.

On peut ajouter des filtres, pour ne voir que les *syscalls*  qui nous intéressent.

Par exemple :

```bash
# si on veut tracer les *syscalls*  effectués par le programme echo
sysdig proc.name=echo
```

> Il existe des tonnes et des tonnes de champs utilisables pour les filtres, on peut consulter la liste avec `sysdig -l`.

Ensuite on le laisse tourner, et si un *syscall* est appelé et que ça matche notre filtre, il s'affichera !

Pour installer sysdig, utilisez les commandes suivantes (instructions pour Rocky Linux 9) :

```bash
# mettons complètement à jour l'OS d'abord si nécessaire
sudo dnf update -y 

# redémarrer pour charger la nouvelle version du kernel si besoin (c'est automatique, juste lance un reboot)
sudo reboot

# installer sysdig et ses dépendances
sudo dnf install -y epel-release
sudo dnf install -y dkms gcc kernel-devel make perl kernel-headers
curl -SLO https://github.com/draios/sysdig/releases/download/0.39.0/sysdig-0.39.0-x86_64.rpm
sudo rpm -ivh sysdig-0.39.0-x86_64.rpm
```

### B. Use it

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

- faites `ls` sur un dossier qui contient des trucs (pas un dossier vide)
- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`

> Vous pouvez isoler à la main les lignes intéressantes : copier/coller de la commande, et des seule(s) ligne(s) que je vous demande de repérer.

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

- faites `cat` sur un fichier qui contient des trucs
- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

- ça va bourriner sec, vu que vous êtes connectés en SSH étou
- juste pour vous éduquer un peu + à ce que fait le kernel à chaque seconde qui passe
- donner la commande pour ça, pas besoin de me mettre le résultat :d

![Too much](./img/doge-strace.jpg)

🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

- `sysdig` permet d'enregistrer ce qu'il capture dans un fichier pour analyse ultérieure
- l'extension c'est `.scap` par convention
- **capturez les *syscalls*  effectués par un `curl example.org`**

> `sysdig` est un outil moderne qui sert de base à toute la suite d'outils de la boîte du même nom. On pense par exemple à Falco qui permet de tracer, monitorer, lever des alertes sur des *syscalls* , au sein d'un cluster Kubernetes.

## 3. Bonus : Stratoshark

Un tout nouveau tool bien stylé : [Stratoshark](https://wiki.wireshark.org/Stratoshark). L'interface de Wireshark (et ses fonctionnalités de fou) mais pour visualiser des captures de *syscalls*  (et autres).

Vous prenez pas trop la tête avec ça, mais si vous voulez vous amuser avec une interface stylée, il est là !

Vous pouvez exporter une capture `sysdig` avec `sysdig -w meo.scap proc.name=echo` par exemple, et la lire dans Stratoshark. 
