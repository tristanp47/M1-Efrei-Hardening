# Part IV : Linux Namespaces

## 1. Explore

🌞 **Utiliser /proc**

- déterminer les *namespaces* de votre `bash` actuel
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo ls -l /proc/$$/ns
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 cgroup -> 'cgroup:[4026531835]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 ipc -> 'ipc:[4026531839]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 mnt -> 'mnt:[4026531841]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 net -> 'net:[4026531840]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 pid -> 'pid:[4026531836]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 pid_for_children -> 'pid:[4026531836]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 time -> 'time:[4026531834]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 time_for_children -> 'time:[4026531834]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 user -> 'user:[4026531837]'
  lrwxrwxrwx. 1 user1 user1 0 Feb 27 13:03 uts -> 'uts:[4026531838]'
  ```
- déterminer les *namespaces* du processuis qui porte l'identifiant PID 1
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo ls -l /proc/1/ns
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 cgroup -> 'cgroup:[4026531835]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 ipc -> 'ipc:[4026531839]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 mnt -> 'mnt:[4026531841]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 net -> 'net:[4026531840]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 pid -> 'pid:[4026531836]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 pid_for_children -> 'pid:[4026531836]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 time -> 'time:[4026531834]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 time_for_children -> 'time:[4026531834]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 user -> 'user:[4026531837]'
  lrwxrwxrwx. 1 root root 0 Feb 27 13:08 uts -> 'uts:[4026531838]'
  ```
- ils devraient être identiques
  ```bash
  oui
  ```

🌞 **Lister tous les *namespaces* en cours d'utilisation**

- avec une simple commande `lsns`

## 2. Create

### A. net

🌞 **Créer un nouveau *namespace* `network`**

- avec une commande `unshare`
- lancez un `bash` à l'intérieur

> `unshare` est aussi le nom de l'appel système (syscall) qui permet de demander au kernel de créer un *namespace*. Ainsi, cette commande, c'est vraiment juste appeler ce syscall depuis la ligne de commande.

🌞 **Prouvez que votre nouveau *namespace* est bien là**

- déjà un `ip a` devrait montrer aucune des cartes réseau depuis l'intérieur du *namespace*
- et on peut `lsns` pour voir ce nouveau *namespace*
- et on peut voir avec un `ls -al` dans `/proc` que ce nouveau terminal est dans un autre *namespace* `network`

### B. pid

🌞 **Créer un nouveau *namespace* `pid`**

- avec une commande `unshare`
- lancez un `bash` à l'intérieur

🌞 **Prouvez que votre nouveau *namespace* est bien là**

- déjà un `ps -ef` devrait pas montrer grand chose
- et on peut `lsns` pour voir ce nouveau *namespace*
- et on peut voir avec un `ls -al` dans `/proc` que ce nouveau terminal est dans un autre *namespace* `pid`


## 3. AND MY CONTAINERS

Oèoè les conteneurs les conteneurs, ils arrivent ils arrivent.

Un outil comme Docker repose **complètement** sur les *namespaces* et les *CGroups* pour lancer des processus de façon isolée sur la machine.

> Maintenant vous comprenez pourquoi j'me vénère quand on me dit que c'est des ptites VMs (genre non, mais alors pas du tout). Aussi pourquoi le conteneur est "isolé" du reste du système. Aussiiiiiiii pourquoi Docker est une techno fondamentalement Linux : Linux *namespaces* + Linux *CGroups* bois&gyals.

### A. Quick install

🌞 **Installer Docker sur la machine**

- suivez les instructions de la doc officielle spécifique pour notre OS !
- une fois installé, ajoutez votre user au groupe docker : `sudo usermod -aG docker $(whoami)`
- et démarrez le service docker ! `sudo systemctl enable --now docker`

### B. A simple container

🌞 **Lancer un simple conteneur qui sleep**

- utilisez la commande suivante :

```bash
docker run -d debian sleep 9999
```

🌞 **Avez les commandes de votre choix, avec le plus de détails possible, prouvez-que :**

- ce processus `sleep` est bien lancé sur votre machine, par votre utilisateur courant
- ce processus `sleep` est isolé à l'aide de *namespaces*
- un *CGroup* a été attribué à ce conteneur
- tout nouveau processus "dans" le conteneur est lui aussi isolé (voir ci-dessous pour lancer d'autres process dans le conteneur)

```bash
# pour obtenir un shell dans un conteneur existant
docker exec -it <conteneur> bash
# par exemple, si le conteneur est nommé "toto"
docker exec -it toto bash

# depuis là, vous pouvez lancez des nouveaux programmes
apt update -y
## procps contient la commande ps
## iproute2 contient la commande ip
## iputils-ping contient la commande ping
apt install -h procps iproute2 iputils-ping
```

### C. CGroup

Et les CGroups dans tout ça ?

🌞 **Lancer un conteneur restreint**

- avec des options du `docker run`
- limiter l'accès RAM de ce conteneur à 456M

🌞 **CGroup ?**

- prouvez que cette limite a été mise en place avec une conf CGroup
- un *CGroup* est automatiquement créé à chaque fois que vous lancez un conteneur (un `scope` systemd) !

![not afraid](./img/nowask.png)
