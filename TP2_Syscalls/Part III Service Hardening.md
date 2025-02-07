# Part III : Service Hardening

## 1. Install NGINX

```bash
[user1@efrei-xmg4agau1 ~]$ curl 10.0.1.9 -v
*   Trying 10.0.1.9:80...
* Connected to 10.0.1.9 (10.0.1.9) port 80 (#0)
< HTTP/1.1 200 OK
< Server: nginx/1.20.1
```

## 2. NGINX Tracing

🌞 **Tracer l'exécution du programme NGINX**

- lancer NGINX à la main, et utilisez `strace` ou `sysdig` pour voir tous les appels systèmes qu'il effectue
- visitez la page web d'accueil pendant que vous tracez l'exécution, pour voir les *syscalls*  nécessaires lors d'un fonctionnement normal
- dans le compte-rendu, listez tous les *syscalls*  passés par NGINX
  ```bash
  [user1@efrei-xmg4agau1 ~]$ ps aux | grep nginx
  nginx       3717  0.0  1.1  15532  5560 ?        S    14:59   0:00 nginx: worker process
  [user1@efrei-xmg4agau1 ~]$ sudo strace -o /home/user1/nginx_trace.log -p nginx.service
  [user1@efrei-xmg4agau1 ~]$ cat nginx_trace.log
  epoll_wait(9, ...) , accept4(6, ...) , epoll_ctl(9, ...) , recvfrom(3, ...) , newfstatat(...) , openat(...) , writev(3, ...) , write(5, ...) , close(12) , setsockopt()
  ```

## 3. NGINX Hardening

🌞 **HARDEN**

- modifier le fichier `nginx.service` pour inclure un filtrage des *syscalls*
- principe du moindre privilège : vous n'autorisez que le strict nécessaire
- vous me remettez le fichier `nginx.service` modifié dans le compte-rendu naturellement !
