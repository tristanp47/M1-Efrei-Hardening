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
#0	read	read(2)	sys_read
- écrire dans un fichier stocké sur disque
- lancer un nouveau processus

### B. `objdump`

`objdump` permet de désassembler un programme, c'est à dire d'afficher le code contenu par un exécutable, sous forme de langage assembleur compréhensible par les humains (un peu, beaucoup plus qu'une purée d'octets en tout cas !)

🌞 **Utiliser `objdump`** sur la commande `ls`

- afficher le contenu de la section `.text`
  - je vous laisse trouver la commande sur l'internet :D
- mettez en évidence quelques lignes qui contiennent l'instruction `call`
  - il devrait y en avoir plusieurs
  - chaque `call` est un appel à une fonction, potentiellement importée *via* une librairie
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  - il y en a aucune normalement : `ls` ne contient pas directement de syscalls
  - car il importe la Glibc, qui contient des syscalls, et les appelle avec `call`

🌞 **Utiliser `objdump`** sur la librairie Glibc

- vous avez repéré son chemin exact au point d'avant avec `ldd`
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  - il devrait y en avoir pas mal
  - chaque ligne qui contient l'instruction `syscall` est la dernière d'un bloc de code qui est le syscall lui-même
- trouvez l'instrution `syscall` qui exécute le syscall `open()`

> Pour exécuter un `syscall`, le programme met dans le registre `eax` l'identifiant du syscall (avec l'instruction `mov`) puis exécute l'instruction `syscall`. Vous cherchez donc une instruction `syscall` précédé d'un `mov` qui met l'identifiant de `open()` dans `eax`.

![How it works](./img/syscall_work.jpg)

