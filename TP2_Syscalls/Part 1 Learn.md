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
- déterminer à quel adresse commence le code du programme
  - pour rappel, le code est dans la section `.text`
  - vous devriez voir cette adresse dans la sortie de `readelf -S`

### C. `ldd`

`ldd` est un outil qui permet de manipuler le *dynamic linker* de Linux. Le *dynamic linker* c'est un programme qui s'occupe de trouver les librairies nécessaires quand un autre programme se lance.

**On peut utiliser `ldd` notamment pour visualiser de quelle librairie a besoin un programme donné.**

🌞 **Utiliser `ldd` sur le programme `ls`**

- afficher la liste des librairies que va utiliser `ls` pendant son fonctionnement
- déterminer, parmi ces librairies, laquelle est la Glibc

> La Glibc est une des librairies les plus importantes au sein d'un système Linux, car elle contient notamment tout le nécessaire pour passer des *syscalls* élémentaires. Si un programme souhaite lire ou écrire dans un fichier par exemple, il aura besoin d'inclure la Glibc.

## 2. Syscalls basics

### A. Syscall list

> Vous pourrez trouver une [liste des syscalls Linux sur un système x86_64 iciiii](https://filippo.io/linux-syscall-table/).

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

- lire un fichier stocké sur disque
- écrire dans un fichier stocké sur disque
- lancer un nouveau processus

> Pour la suite du TP, gardez-vous sous le coude les réponses apportées à cette question. Juste après vous allez regarder le langage machine contenu dans des exécutables à la recherche de l'appel à un *syscall*. Il faudra le repérer grâce à son identifiant !

![Fork exec](./img/forkexec.png)

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

