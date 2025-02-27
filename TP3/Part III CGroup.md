# Part III : CGroup

## 1. Explore

Pour rappel : la configuration actuelle des *CGroups* est dispo dans `/sys/fs/cgroup`.

🌞 **Afficher...** :

- la liste des controllers *CGroups* dispos sur le système
  ```bash
  [user1@efrei-xmg4agau1 ~]$ cat /sys/fs/cgroup/cgroup.controllers
  ```
- la quantité de mémoire max que vous êtes autorisés à utiliser dans votre session utilisateur
  - par défaut, sous Rocky, le *controller* memory n'est pas activé : normal si vous ne voyez aucun fichier `memory.max`
  - si c'est le cas, ça veut dire qu'aucune restriction RAM est en place (vous devez justement constater ça)
    ```bash
    [user1@efrei-xmg4agau1 ~]$ cat /sys/fs/cgroup/user.slice/memory.max
    max
    ```
- les noms de tous les *CGroups* créés
  - ce sont tous les sous-dossiers de `/sys/fs/cgroup`
  - il devrait y avoir au moins les slices et scopes de systemd, on en parle plus bas
  ```bash
  [user1@efrei-xmg4agau1 ~]$ ls /sys/fs/cgroup/
  ```


## 2. Do it

🌞 **Créer un nouveau *CGroup*** :

- appelez-le `meow`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo mkdir /sys/fs/cgroup/meow
  ```
- activez les controllers `cpu` `cpuset` et `memory` s'ils ne le sont pas déjà
  ```bash
  [user1@efrei-xmg4agau1 meow]$ echo "+cpu +cpuset +memory" | sudo tee /sys/fs/cgroup/cgroup.subtree_control
  +cpu +cpuset +memory

  [user1@efrei-xmg4agau1 meow]$ cat cgroup.controllers
  cpuset cpu memory pids
  ```

🌞 **Créer un nouveau sous-CGroup** :

- appelez-le `task1`
  ```bash
  [user1@efrei-xmg4agau1 cgroup]$ sudo mkdir /sys/fs/cgroup/meow/task1
  ```
- on parle de créer le dossier `/sys/fs/cgroup/meow/task1/`
  ```bash
  [user1@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/meow/task1/cgroup.controllers  
  cpuset cpu memory
  ```
- prouvez que les controllers activés sur `meow` ont bien été hérités
  ```bash
  [user1@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/meow/task1/cgroup.controllers  
  cpuset cpu memory
  ```

🌞 **Mettez en place une limitation RAM**

- définissez une limite de 150M de RAM pour ce CGroup `task1`
  ```bash
  [user1@efrei-xmg4agau1 task1]$ echo "150M" | sudo tee /sys/fs/cgroup/meow/task1/memory.max
  150M
  [user1@efrei-xmg4agau1 task1]$ cat /sys/fs/cgroup/meow/task1/memory.max
  157286400
  ```

🌞 **Prouvez que la limite est effective**

1. utilisez la commande `stress-ng` pour remplir la mémoire de la machine
   ```bash
   [user1@efrei-xmg4agau1 ~]$ stress-ng --vm 4 --vm-bytes 100% --vm-keep --timeout 30s
   ```
3. constatez que la RAM est pleine
   ```bash
   [user1@efrei-xmg4agau1 ~]$ free -m
                  total        used        free      shared  buff/cache   available
   Mem:             456         391           9          17          85          65
   Swap:            819          14         805
   ```
4. ajoutez votre shell `bash` actuel au *CGroup* `task1`
   ```bash
   [user1@efrei-xmg4agau1 cgroup]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task1/cgroup.procs
   817
   [user1@efrei-xmg4agau1 cgroup]$ cat /sys/fs/cgroup/meow/task1/cgroup.procs
   817
   18495
   ```
5. utilisez de nouveau `stress-ng`
6. constatez que le processus `stress-ng` est tué en boucle dès qu'il remplit la RAM au delà de la limite

> On rappelle que tout processus lancé par un processus existant se retrouvera par défaut dans le même *CGroup* que son parent. C'est pour ça que vous ajoutez votre shell `bash` au *CGroup* : tout ce que vous exécuterez depuis ce `bash` sera exécuté dans le même *CGroup* que lui. Ha et le truc qui tue votre processus quand il prendre trop de RAM, c'est le [**OOM-killer**](https://en.wikipedia.org/wiki/Out_of_memory).

🌞 **Créer un nouveau sous-*CGroup*** :

- appelez-le `task2`

🌞 **Appliquer des restrictions CPU** :

- utilisez la mécanique de `cpu.weight` pour définir des priorités différentes à `task1` et `task2`
- utilisez `stress-ng` ou un bon vieux `cat /dev/random` pour lancer un processus CPU-intensive, et pour prouver que la restriction est en place
- pour tester, vous devez :
  - lancer deux shells en même temps
  - ajouter le premier au *CGroup* `task1`
  - ajouter le deuxième au *CGroup* `task2`
  - dans les deux shells, lancer un processus CPU-intensive
  - constatez avec un `htop` par exemple que les deux processus ne se répartissent pas équitablement la puissance du CPU

## 3. systemd

Par défaut, sous Linux, y'a systemd (ouais encore lui). Il utilise pas mal les *CGroup* nativement. Il fait naturellement deux trucs :

- **il met les `service` dans des `slice`s**
  - créer un `slice` systemd, c'est juste créer un cgroup (dont le nom se terminera par `.slice`)
  - tous les `service`s sont forcément dans un `slice`
- **il met des programmes lancés à l'extérieur (genre des trucs qui sont pas des `services`) dans des `scope`**
  - c'est pareil : un `scope` c'est juste un CGroup créé automatiquement par systemd, dont le nom se termine par `.scope`
  - ça permet à systemd de gérer certains processus même si c'est pas lui qui les lance
  - par exemple quand t'ouvres une session SSH n_n

Bref, l'un des rôles principaux de systemd c'est lancer et gérer des processus (services). Il est donc tout naturel qu'il utilise les *CGroups* Linux pour mener à bien ce job.

### A. One-shot

Y'a une commande rigolote et parfois pratique qui permet de jouer avec tout ça. Une commande qui permet de créer un service système temporaire à la volée en une seule ligne de commande : `systemd-run`.

> Pour plusieurs tests dans le TP, on se servira du serveur Web embarqué par Python. Il se lance en une seule commande, fait des trucs sur le système (serveur web, donc il lit des fichiers, fait des connexions réseau), c'est parfait pour faire des tests ! Pour le lancer : `python -m http.server 8888`.

🌞 **Lancer un serveur Web Python sous forme de service temporaire**

- avec la commande `python -m http.server <PORT>`
- n'oubliez pas d'ouvrir ce port dans le firewall pour tester
- il faudra le lancer avec la commande `systemd-run` pour en faire un service temporaire
- affichez le `status` du service pour prouver qu'il run

```bash
# lancer un sleep sous la forme d'un service nommé meow_test.service
sudo systemd-run -u meow_test sleep 9999
```

🌞 **Appliquer à la volée des restrictions**

- avec l'option `-p` de `systemd-run` vous pouvez préciser des paramètres pour le service
- utilisez le paramètre `MemoryMax` pour mettre en place une limite à 234M

> En vrai, `systemd-run` est un tool vraiment pratique pour limiter l'accès aux ressources d'un process qu'on lance oneshot.

🌞 **Restrictions *CGroup* ?**

- prouvez que la restriction à 234M de `systemd-run` est mise en place avec les *CGroups* Linux

> Les montants de RAM chelous c'pour vous permettre de faire des `grep` pour trouver facilement. Attention par contre, les restrictions automatiques appliquées par systemd sont exprimées en octets (pas KB ni MB) donc faut faire une ptite multiplication pour le trouver facilement. En l'occurence, un `grep -nri $(( 234 * 1024 * 1024 ))` depuis `/sys/fs/cgroup` fera le taff ;)

### B. Real service

🌞 **Créez un service `web.service`**

- habitués nan ? Faut créer un fichier dans `/etc/systemd/system/`
- n'oubliez pas de `sudo systemctl daemon-reload` à chaque modification
- ce service doit lancer un serveur web python sur le port 9999/tcp

🌞 **Restriction *CGroup***

- modifier le fichier `web.service` pour inclure des limitations mises en place par des CGroups :
  - mémoire max : 321M
  - limitation d'écriture disque : 1M
  - limitation de lecture disque : 1M
  - limitation CPU : 50% d'utilisation

🌞 **Prouvez que ces restrictions ont été mises en place avec les *CGroups***

- en explorant le dossier `/sys/` toujours !
