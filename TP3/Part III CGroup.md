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
   ```bash
   [user1@efrei-xmg4agau1 cgroup]$ sudo stress-ng --vm 1 --vm-bytes 100%
   ```
6. constatez que le processus `stress-ng` est tué en boucle dès qu'il remplit la RAM au delà de la limite
   ```bash
   [user1@efrei-xmg4agau1 ~]$ watch -n0.1 "ps -eo cmd,pid,rss | grep stress"
   ```

🌞 **Créer un nouveau sous-*CGroup*** :

- appelez-le `task2`
  ```bash
  [user1@efrei-xmg4agau1 cgroup]$sudo mkdir /sys/fs/cgroup/meow/task1
  ```

🌞 **Appliquer des restrictions CPU** :

- utilisez la mécanique de `cpu.weight` pour définir des priorités différentes à `task1` et `task2`
  ```bash
  [user1@efrei-xmg4agau1 meow]$ echo 100 | sudo tee /sys/fs/cgroup/meow/task1/cpu.weight
  100
  [user1@efrei-xmg4agau1 meow]$ echo 200 | sudo tee /sys/fs/cgroup/meow/task2/cpu.weight
  200
  ```
- utilisez `stress-ng` ou un bon vieux `cat /dev/random` pour lancer un processus CPU-intensive, et pour prouver que la restriction est en place
- pour tester, vous devez :
  - lancer deux shells en même temps
  - ajouter le premier au *CGroup* `task1`
    ```bash
    [user1@efrei-xmg4agau1 ~]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task1/cgroup.procs
    ```
  - ajouter le deuxième au *CGroup* `task2`
    ```bash
    [user1@efrei-xmg4agau1 ~]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task2/cgroup.procs
    ```
  - dans les deux shells, lancer un processus CPU-intensive
    ```bash
    [user1@efrei-xmg4agau1 ~]$ stress-ng --cpu 1
    stress-ng: info:  [24431] defaulting to a 1 day, 0 secs run per stressor
    [user1@efrei-xmg4agau1 ~]$ stress-ng --cpu 1
    stress-ng: info:  [24426] defaulting to a 1 day, 0 secs run per stressor
    ```
  - constatez avec un `htop` par exemple que les deux processus ne se répartissent pas équitablement la puissance du CPU
    ```bash
    24427 user1     20   0   37476   5820   3584 R  65.8   1.2   2:06.29 stress-ng-cpu
    24432 user1     20   0   37476   5820   3584 R  33.2   1.2   0:54.61 stress-ng-cpu
    ```

## 3. systemd

### A. One-shot

🌞 **Lancer un serveur Web Python sous forme de service temporaire**

- avec la commande `python -m http.server <PORT>`
- n'oubliez pas d'ouvrir ce port dans le firewall pour tester
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo firewall-cmd --permanent --add-port=8888/tcp
  [user1@efrei-xmg4agau1 ~]$ sudo firewall-cmd --reload
  ```
- il faudra le lancer avec la commande `systemd-run` pour en faire un service temporaire
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo systemd-run -u meow_test /bin/bash -c "python3 -m http.server 8888 & sleep 9999"
  Running as unit: meow_test.service
  ```
- affichez le `status` du service pour prouver qu'il run
  ```bash
  [user1@efrei-xmg4agau1 ~]$ systemctl status meow_test
  ● meow_test.service - /bin/bash -c python3 -m http.server 8888 & sleep 9999
       Loaded: loaded (/run/systemd/transient/meow_test.service; transient)
    Transient: yes
       Active: active (running) since Thu 2025-02-27 11:54:11 CET; 6s ago
     Main PID: 24716 (bash)
        Tasks: 3 (limit: 2664)
       Memory: 9.7M
          CPU: 62ms
       CGroup: /system.slice/meow_test.service
               ├─24716 /bin/bash -c "python3 -m http.server 8888 & sleep 9999"
               ├─24717 python3 -m http.server 8888
               └─24718 sleep 9999
  ```

🌞 **Appliquer à la volée des restrictions**

- avec l'option `-p` de `systemd-run` vous pouvez préciser des paramètres pour le service
- utilisez le paramètre `MemoryMax` pour mettre en place une limite à 234M
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo systemd-run -u meow_test -p MemoryMax=234M /bin/bash -c "python3 -m http.server 8888 & sleep 9999"
  ```

🌞 **Restrictions *CGroup* ?**

- prouvez que la restriction à 234M de `systemd-run` est mise en place avec les *CGroups* Linux
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo grep -nri $(( 234 * 1024 * 1024 )) /sys/fs/cgroup 2>/dev/null
  /sys/fs/cgroup/system.slice/meow_test.service/memory.max:1:245366784
  ```

### B. Real service

🌞 **Créez un service `web.service`**

- habitués nan ? Faut créer un fichier dans `/etc/systemd/system/`
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo nano /etc/systemd/system/web.service
  [Unit]
  Description=Serveur Web Python
  After=network.target
  
  [Service]
  ExecStart=/usr/bin/python3 -m http.server 9999
  WorkingDirectory=/home/user1
  Restart=always
  User=user1
  
  [Install]
  WantedBy=multi-user.target
  ```
- n'oubliez pas de `sudo systemctl daemon-reload` à chaque modification
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo systemctl daemon-reload
  ```
- ce service doit lancer un serveur web python sur le port 9999/tcp
  ```bash
  [user1@efrei-xmg4agau1 ~]$ sudo systemctl status web.service
  ● web.service - Serveur Web Python
       Loaded: loaded (/etc/systemd/system/web.service; enabled; preset: disabled)
       Active: active (running) since Thu 2025-02-27 12:05:08 CET; 1min 13s ago
     Main PID: 24832 (python3)
        Tasks: 1 (limit: 2664)
       Memory: 9.1M
          CPU: 73ms
       CGroup: /system.slice/web.service
               └─24832 /usr/bin/python3 -m http.server 9999
  ```

🌞 **Restriction *CGroup***

- modifier le fichier `web.service` pour inclure des limitations mises en place par des CGroups :
  - mémoire max : 321M
  - limitation d'écriture disque : 1M
  - limitation de lecture disque : 1M
  - limitation CPU : 50% d'utilisation
    ```bash
    MemoryMax=321M
    IOReadBandwidthLimit=1M
    IOWriteBandwidthLimit=1M
    CPUQuota=50%
    ```

🌞 **Prouvez que ces restrictions ont été mises en place avec les *CGroups***

- en explorant le dossier `/sys/` toujours !
  ```bash
  #Mémoire max : 321M
  [user1@efrei-xmg4agau1 ~]$ cat /sys/fs/cgroup/system.slice/web.service/memory.max
  336592896

  #Limitation du CPU
  [user1@efrei-xmg4agau1 ~]$ cat /sys/fs/cgroup/system.slice/web.service/cpu.max
  50000 100000

  #Limitation de lecture et d'écriture d'un disque
  [user1@efrei-xmg4agau1 ~]$ sudo cat /sys/fs/cgroup/system.slice/web.service/io.max
  ```
