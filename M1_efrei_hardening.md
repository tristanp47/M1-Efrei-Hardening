# TP1 Hardening

## Compte root:
```
root
dbc
```

## Compte user1:
```
user1
user2025
```

---

## Part I : User management

### 1. Existing users

#### Affichage des utilisateurs existants:
```bash
cat /etc/passwd
```
#### Affichage des groupes existants:
```bash
cat /etc/group
```
#### Affichage des groupes de l'utilisateur:
```bash
groups user1
```
#### Affichage des processus en cours lancés par root:
```bash
ps -u root
```
#### Affichage des processus en cours de user1:
```bash
ps -u user1
```
#### Affichage du hash du mot de passe de root:
```bash
sudo cat /etc/shadow | grep root
```
#### Affichage du hash du mot de passe de user1:
```bash
sudo cat /etc/shadow | grep user1
```

### 2. User creation and configuration

#### Création de l'utilisateur meow:
```bash
sudo useradd -M -g admins -s /sbin/nologin meow
```
#### Configuration des permissions sudoers:
```bash
meow ALL=(user1) NOPASSWD: /bin/ls, /bin/cat, /usr/bin/less, /usr/bin/more
%admins ALL=(root) NOPASSWD: /usr/bin/yum
```

---

## Part II : Files and permissions

### 1. Listing POSIX permissions

#### Vérification des permissions de fichiers critiques:
```bash
ls -l /etc/passwd
ls -l /etc/shadow
ls -l /usr/sbin/sshd
ls -l /home
```

#### Recherche des fichiers avec SUID activé:
```bash
find / -type f -perm /6000 -ls 2>/dev/null
```

#### Application de l'attribut immuable sur la configuration SSH:
```bash
sudo chattr +i /etc/ssh/sshd_config
lsattr /etc/ssh/sshd_config
```

---

## Part III : Networking

### 1. Listening ports
```bash
sudo ss -tulnp
```

### 2. Firewalling
```bash
sudo iptables -A INPUT -p tcp --dport 323 -j DROP
```

---

## Part IV : Storage and partitions

### 1. Existing partitions
```bash
df -h
```

### 2. Mount options
```bash
findmnt -no OPTIONS /
```

---

## Part V : OpenSSH Server

### 1. Basics

#### Changement du port SSH:
```bash
sudo nano /etc/ssh/sshd_config
Port 2222
sudo systemctl restart sshd
```

#### Vérification:
```bash
sudo ss -tuln | grep 2222
```

### 2. Authentication modes
```bash
sudo nano /etc/ssh/sshd_config
PasswordAuthentication no
PermitRootLogin no
```

### 3. Further hardening
```bash
PermitEmptyPasswords no
ClientAliveInterval 300
ClientAliveCountMax 0
AllowUsers user1 user2
X11Forwarding no
Banner /etc/issue.net
```

### 4. fail2ban
```bash
sudo yum install fail2ban
sudo nano /etc/fail2ban/jail.local
```
```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/secure
maxretry = 7
findtime = 300
```

#### Vérification des bannissements:
```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```
