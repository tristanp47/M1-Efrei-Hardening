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

### B. Use it

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

- faites `ls` sur un dossier qui contient des trucs (pas un dossier vide)
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo sysdig proc.name=ls > lk.txt
  ```
- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ grep write lk.txt
  2818 18:31:35.510900262 0 ls (26050) > write fd=1(<f>/dev/tty1) size=15
  2819 18:31:35.510913927 0 ls (26050) < write res=15 data=bj.txt  ok.txt.
  ```
  
🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

- faites `cat` sur un fichier qui contient des trucs
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo sysdig proc.name=cat > la.txt
  ```
- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
  ```bash
  [user1@efrei-xmg4agau1 ~]$ grep open la.txt
  15013 18:37:07.019034474 0 cat (26072) < openat fd=3(<f>/home/user1/test/ok.txt) dirfd=-100(AT_FDCWD) name=ok.txt(/home/user1/test/ok.txt) flags=1(O_RDONLY) mode=0 dev=FD00 ino=13450078
  ```
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
  ```bash
  [user1@efrei-xmg4agau1 ~]$ grep write la.txt
  15030 18:37:07.020949277 0 cat (26072) > write fd=1(<f>/dev/tty1) size=3
  15031 18:37:07.021347218 0 cat (26072) < write res=3 data=ok.
  ```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

- ça va bourriner sec, vu que vous êtes connectés en SSH étou
- juste pour vous éduquer un peu + à ce que fait le kernel à chaque seconde qui passe
- donner la commande pour ça, pas besoin de me mettre le résultat :d
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo sysdig user.name=user1
  ```

🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

- `sysdig` permet d'enregistrer ce qu'il capture dans un fichier pour analyse ultérieure
- l'extension c'est `.scap` par convention
- **capturez les *syscalls*  effectués par un `curl example.org`**

> `sysdig` est un outil moderne qui sert de base à toute la suite d'outils de la boîte du même nom. On pense par exemple à Falco qui permet de tracer, monitorer, lever des alertes sur des *syscalls* , au sein d'un cluster Kubernetes.
