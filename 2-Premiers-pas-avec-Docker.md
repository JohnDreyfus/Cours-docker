# 2. Premiers pas avec Docker

## La commande `docker run` - Premier conteneur

### Syntaxe de base
```bash
docker run [OPTIONS] IMAGE [COMMAND]
```

### Premier exemple concret
```bash
docker run hello-world
```

**Ce qui se passe :**
- Docker cherche l'image `hello-world` localement
- Si absente, téléchargement depuis Docker Hub
- Création d'un conteneur à partir de l'image
- Exécution du conteneur
- Affichage du message de confirmation
- Arrêt automatique du conteneur

### Exemple avec nginx
```bash
docker run nginx
```

**Observations :**
- Le terminal reste bloqué (processus au premier plan)
- Logs nginx visibles en temps réel
- `CTRL+C` pour arrêter

### Options importantes
```bash
docker run -d nginx                    # Détaché (background)
docker run -d --name mon-nginx nginx   # Nommer le conteneur
docker run -d -p 8080:80 nginx         # Mapper les ports
docker run -it ubuntu bash             # Mode interactif avec terminal
```

**Détail des options :**
- `-d` : mode détaché (daemon)
- `--name` : attribuer un nom personnalisé
- `-p HOST:CONTAINER` : mapping de ports
- `-it` : mode interactif avec pseudo-TTY
- `--rm` : suppression automatique après arrêt

---

## Comprendre les images et les conteneurs

### Les images
**Définition :** Template en lecture seule contenant tout le nécessaire pour exécuter une application.

**Composition :**
- Système de fichiers
- Dépendances et bibliothèques
- Code applicatif
- Configuration par défaut
- Métadonnées

**Caractéristiques :**
- Immuables (non modifiables)
- Organisées en couches (layers)
- Versionnées via tags
- Stockées localement ou dans des registries

### Les conteneurs

**Définition :** Instance exécutable d'une image.

**Relation image/conteneur :**
- Une image = un modèle
- Un conteneur = une instance en cours d'exécution
- Plusieurs conteneurs peuvent provenir de la même image

**Exemple d'analogie :**
```
Image       → Classe (POO)
Conteneur   → Instance de classe
```

### Lister les éléments
```bash
docker images              # Toutes les images locales
docker ps                  # Conteneurs actifs
docker ps -a               # Tous les conteneurs (actifs + arrêtés)
```

---

## Cycle de vie d'un conteneur

### États d'un conteneur
```
Created → Running → Paused → Stopped → Deleted
```

### Transitions entre états

**Created → Running :**
```bash
docker create nginx           # Créer sans démarrer
docker start <container_id>   # Démarrer
# OU directement
docker run nginx              # Créer + démarrer
```

**Running → Stopped :**
```bash
docker stop <container_id>    # Arrêt gracieux (SIGTERM puis SIGKILL)
docker kill <container_id>    # Arrêt forcé (SIGKILL immédiat)
```

**Stopped → Running :**
```bash
docker start <container_id>
docker restart <container_id>  # Stop + start
```

**Running → Paused :**
```bash
docker pause <container_id>
docker unpause <container_id>
```

**Suppression :**
```bash
docker rm <container_id>       # Supprimer un conteneur arrêté
docker rm -f <container_id>    # Forcer la suppression
```

### Diagramme du cycle
```
[Image] --docker run--> [Created] --auto--> [Running]
                                              |
                                        docker stop
                                              |
                                              v
                                          [Stopped]
                                              |
                                         docker rm
                                              |
                                              v
                                          [Deleted]
```

---

## TP pratique : Premiers conteneurs

### Exercice 1 : Lancer nginx
```bash
# Lancer nginx en arrière-plan sur le port 8080
docker run -d -p 8080:80 --name web-server nginx

# Vérifier que le conteneur tourne
docker ps

# Tester dans le navigateur
# http://localhost:8080

# Observer les logs
docker logs web-server

# Suivre les logs en temps réel
docker logs -f web-server
```

### Exercice 2 : Ubuntu interactif
```bash
# Lancer un conteneur Ubuntu avec un shell interactif
docker run -it --name mon-ubuntu ubuntu bash

# Une fois dans le conteneur :
cat /etc/os-release
ls /
apt update
apt install curl -y
curl --version
exit

# Le conteneur s'arrête après exit
docker ps -a

# Redémarrer et se reconnecter
docker start mon-ubuntu
docker attach mon-ubuntu
```

### Exercice 3 : Application Node.js
```bash
# Lancer un conteneur Node.js
docker run -it --name node-app node:18 node

# Dans le REPL Node :
console.log("Hello from Docker");
process.version
exit

# Lancer un script directement
docker run --rm node:18 node -e "console.log('Hello Docker')"
```

### Exercice 4 : Manipulation du cycle de vie
```bash
# Créer plusieurs conteneurs nginx
docker run -d --name web1 -p 8081:80 nginx
docker run -d --name web2 -p 8082:80 nginx
docker run -d --name web3 -p 8083:80 nginx

# Lister tous les conteneurs
docker ps

# Arrêter web2
docker stop web2

# Vérifier l'état
docker ps -a

# Redémarrer web2
docker start web2

# Supprimer web3 (nécessite un arrêt d'abord)
docker stop web3
docker rm web3

# Ou en une commande
docker rm -f web1 web2
```

### Exercice 5 : Inspection et nettoyage
```bash
# Informations détaillées sur un conteneur
docker inspect web-server

# Statistiques en temps réel
docker stats web-server

# Nettoyer tous les conteneurs arrêtés
docker container prune

# Nettoyer images non utilisées
docker image prune
```
