# Part IV : Linux Namespaces

> Le terme *namespace* est utilisé dans plein plein de contextes différents en informatique. Ici, on parle des *namespaces* du noyau Linux (qui n'a rien à votre avec les *namespaces* Java ou Kubernetes ou autres.)

➜ **Les Linux namespaces permettent d'isoler les processus les uns des autres, en leur proposant une vue limitée et restreinte de l'OS.**

Ici on parle pas de l'accès aux ressources (matérielles) de la machine comme avec les CGroups, mais plutôt de l'accès aux features de l'OS.

➜ **Les namespaces isolent notamment les processus en terme de** (liste non-exhaustive) :

- **PID** : les processus voient qu'une partie des autres processus qui s'exécutent
- **network** : les processus ne voient pas toutes les cartes réseau du système
- **user** : les processus n'ont pas les même users (pas les même fichiers `/etc/passwd` et `/etc/shadow` par exemple)
- **mount** : les processus ne voient pas les même partitions et points de montage que les autres

> On peut choisir d'isoler un processus uniquement en terme de network ou uniquement en terme de PID, ou tout à la fois, y'a pas de contraintes à ce niveau là.

![container](./img/container.png)

## Sommaire

- [Part IV : Linux Namespaces](#part-iv--linux-namespaces)
  - [Sommaire](#sommaire)
  - [1. Explore](#1-explore)
  - [2. Create](#2-create)
    - [A. net](#a-net)
    - [B. pid](#b-pid)
  - [3. AND MY CONTAINERS](#3-and-my-containers)
    - [A. Quick install](#a-quick-install)
    - [B. A simple container](#b-a-simple-container)
    - [C. CGroup](#c-cgroup)

## 1. Explore

🌞 **Utiliser /proc**

- déterminer les *namespaces* de votre `bash` actuel
- déterminer les *namespaces* du processuis qui porte l'identifiant PID 1
- ils devraient être identiques

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
