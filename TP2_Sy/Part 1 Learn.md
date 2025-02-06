# Part I : Learn

Dans cette partie, je vous fais (re)découvrir quelques commandes usuelles quand on travaille autour des programmes et des processus.

**Au menu : on dissèque des programmes, et on repère les syscalls qu'ils utilisent.**

## Sommaire

- [Part I : Learn](#part-i--learn)
  - [Sommaire](#sommaire)
  - [1. Anatomy of a program](#1-anatomy-of-a-program)
    - [A. `file`](#a-file)
    - [B. `readelf`](#b-readelf)
    - [C. `ldd`](#c-ldd)
  - [2. Syscalls basics](#2-syscalls-basics)
    - [A. Syscall list](#a-syscall-list)
    - [B. `objdump`](#b-objdump)

## 1. Anatomy of a program

**Un programme est un fichier *exécutable*. C'est à dire que :** 

- c'est un simple fichier
- il est composé de plusieurs sections
  - la section `.text` contient les instructions du programme pour le CPU
  - les autres sections contiennent essentiellement des metadonnées
- il peut être compilé...
  - statiquement : tout est dans le programme
  - dynamiquement : le programme pourra faire appel à des librairies du système
- il est marqué comme étant "exécutable"
  - sur Linux, on donne la permission d'exécution avec `chmod`

Dans cette partie, on va voir quelques outils très usuels pour obtenir des infos sur un programme.

### A. `file`

`file` est une commande uqi permet de déterminer le type d'un fichier.

Ceci ne repose pas du tout sur l'extension du fichier. `file` regarde directement les bits qui composent le fichier pour en déterminer le type. Il se concentre sur les premiers octets du fichiers qui contient généralement des métadonnées suffisantes pour déterminer le type.

🌞 **Utiliser `file` pour déterminer le type de :**

- la commande `ls`
- la commande `ip`
- un fichier `.mp3` que vous aurez téléchargé sur le disque de la VM

> Le format des exécutables sous les OS Linux est appelé ELF. ELF est le format qui définit l'ordre des octets dans un programme, le fait qu'il doit être composé de plusieurs sections, comment il doit indiquer les librairies externes dont il a besoin, etc.

### B. `readelf`

`readelf` permet d'obtenir des informations sur un fichier ELF : un exécutable Linux.

De la même façon qu'un fichier texte possède des numéros de ligne quand on l'affiche, si on affiche le contenu d'un programme, chaque ligne est numérotée.

Chaque ligne du programme a donc une adresse, qui est notée en hexadécimal.

`readelf` permet notamment de voir de quelle adresse à quelle adresse se trouve  tell ou telle section.

🌞 **Utiliser `readelf` sur le programme `ls`**

- afficher le *header* du programme
  - il contient toutes les métadonnées principales du programme
  - c'est l'option `readelf -h`
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

