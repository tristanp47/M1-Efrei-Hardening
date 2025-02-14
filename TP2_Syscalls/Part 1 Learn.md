# Part I : Learn

## 1. Anatomy of a program

### A. `file`

🌞 **Utiliser `file` pour déterminer le type de :**

- la commande `ls`
```bash
[user1@efrei-xmg4agau1 ~]$ file /usr/bin/ls
/usr/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=fe37adecca22a782c4fb274ae601f220cc1fbb4d, for GNU/Linux 3.2.0, stripped
```
- la commande `ip`
```bash
[user1@efrei-xmg4agau1 ~]$ file /usr/sbin/ip
/usr/sbin/ip: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=77a2f5899f0529f27d87bb29c6b84c535739e1c7, for GNU/Linux 3.2.0, stripped
```
- un fichier `.mp3` que vous aurez téléchargé sur le disque de la VM
```bash
[user1@efrei-xmg4agau1 ~]$ file SoundHelix-Song-1.mp3
SoundHelix-Song-1.mp3: Audio file with ID3 version 2.3.0, contains:MPEG ADTS, layer III, v1, 192 kbps, 44.1 kHz, Stereo
```

### B. `readelf`

🌞 **Utiliser `readelf` sur le programme `ls`**

- afficher le *header* du programme
  - il contient toutes les métadonnées principales du programme
  - c'est l'option `readelf -h`
```bash
[user1@efrei-xmg4agau1 ~]$ readelf -h /usr/bin/ls
ELF Header:
```
- afficher la liste des sections du programme
  - c'est l'option `readelf -S`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ readelf -S /usr/bin/ls
  There are 30 section headers, starting at offset 0x21ec8:
  ```
- déterminer à quel adresse commence le code du programme
  - pour rappel, le code est dans la section `.text`
  - vous devriez voir cette adresse dans la sortie de `readelf -S`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ readelf -S /usr/bin/ls | grep '.text'
  [15] .text             PROGBITS         0000000000004d50  00004d50
  ```

### C. `ldd`

🌞 **Utiliser `ldd` sur le programme `ls`**

- afficher la liste des librairies que va utiliser `ls` pendant son fonctionnement
```bash
[user1@efrei-xmg4agau1 ~]$ ldd /usr/bin/ls
        linux-vdso.so.1 (0x00007ffc50bb7000)
        libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f05589ee000)
        libcap.so.2 => /lib64/libcap.so.2 (0x00007f05589e4000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f0558600000)
        libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007f0558948000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f0558a45000)
```
- déterminer, parmi ces librairies, laquelle est la Glibc
```bash
libc.so.6 => /lib64/libc.so.6 (0x00007f0558600000)
```
## 2. Syscalls basics

### A. Syscall list

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

- lire un fichier stocké sur disque
  ```bash
  0	read read(2)	sys_read
  ```
- écrire dans un fichier stocké sur disque
  ```bash
  1	write	write(2) sys_write
  ```
- lancer un nouveau processus
  ```bash
  57	fork fork(2) sys_fork
  ```
  
### B. `objdump`

🌞 **Utiliser `objdump`** sur la commande `ls`

- afficher le contenu de la section `.text`
  ```bash
  objdump -d -j .text /bin/ls
  ```
- mettez en évidence quelques lignes qui contiennent l'instruction `call`
  ```bash
  16ef9:       e8 82 da fe ff          callq  4980 <memset@plt>
  16f3e:       e8 bd e1 ff ff          callq  15100 <_obstack_memory_used@@Base+0x47f0>
  16fb3:       e8 f8 d8 fe ff          callq  48b0 <__stack_chk_fail@plt>
  ```
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
```bash
Il n'y a pas d'instructions syscall directement dans ls car il utilise la Glibc pour effectuer les appels système.
```

🌞 **Utiliser `objdump`** sur la librairie Glibc

- vous avez repéré son chemin exact au point d'avant avec `ldd`
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ objdump -d -j .text /lib64/libc.so.6 | grep syscall
  000000000003f4d0 <__GI___syscall_clock_gettime>:
   3f4d9:       0f 05                   syscall
  0000000000108730 <time_syscall>:
  108739:       0f 05                   syscall
  ```
- trouvez l'instrution `syscall` qui exécute le syscall `close()`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ objdump -d /lib64/libc.so.6 | grep 'syscall' -B2 | grep '$0x3' -A3
  167a02:       b8 03 00 00 00          mov    $0x3,%eax
  167a07:       0f 05                   syscall
  ```

