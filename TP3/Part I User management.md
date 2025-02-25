# Part I : User management

**Hum, cette partie est censée être envoyée vite fait bien fait ! Prouvez-le moi :D**

## 1. Existing users

🌞 **Déterminer l'existant :**

- lister tous les utilisateurs créés sur la machine
- lister tous les groupes d'utilisateur
- déterminer la liste des groupes dans lesquels se trouvent votre utilisateur

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par `root`**

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par votre utilisateur**

🌞 **Déterminer le hash du mot de passe de `root`**

🌞 **Déterminer le hash du mot de passe de votre utilisateur**

🌞 **Déterminer la fonction de hachage qui a été utilisée**

🌞 **Déterminer, pour l'utilisateur `root`** :

- son shell par défaut
- le chemin vers son répertoire personnel

🌞 **Déterminer, pour votre utilisateur** :

- son shell par défaut
- le chemin vers son répertoire personnel

🌞 **Afficher la ligne de configuration du fichier `sudoers` qui permet à votre utilisateur d'utiliser `sudo`**

![sudo](./img/sudo.svg)


## 2. User creation and configuration

🌞 **Créer un utilisateur :**

- doit s'appeler `meow`
- ne doit appartenir QUE à un groupe nommé `admins`
- ne doit pas avoir de répertoire personnel utilisable
- ne doit pas avoir un shell utilisable

> Il s'agit donc ici d'un utilisateur avec lequel on pourra pas se connecter à la machine (ni en console, ni en SSH).

🌞 **Configuration `sudoers`**

- ajouter une configuration `sudoers` pour que l'utilisateur `meow` puisse exécuter seulement et uniquement les commandes `ls`, `cat`, `less` et `more` en tant que votre utilisateur
- ajouter une configuration `sudoers` pour que les membres du groupe `admins` puisse exécuter seulement et uniquement la commande `apt` en tant que `root`
- ajouter une configuration `sudoers` pour que votre utilisateur puisse exécuter n'importe quel commande en tant `root`, sans avoir besoin de saisir un mot de passe
- prouvez que ces 3 configurations ont pris effet (vous devez vous authentifier avec le bon utilisateur, et faire une commande `sudo` qui doit fonctioner correctement)

> Pour chaque point précédent, c'est une seule ligne de configuration à ajouter dans le fichier `sudoers` de la machine.

## 3. Hackers gonna hack

🌞 **Déjà une configuration faible ?**

- l'utilisateur `meow` est en réalité complètement `root` sur la machine hein là. Prouvez-le.
- proposez une configuration similaire, sans présenter cette faiblesse de configuration
  - vous pouvez ajouter de la configuration
  - ou supprimer de la configuration
  - du moment qu'on garde des fonctionnalités à peu près équivalentes !


