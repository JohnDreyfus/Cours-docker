# 3. Commandes essentielles Docker

## Gestion des conteneurs

### `docker run`

Crée et démarre un conteneur à partir d'une image.

**Syntaxe de base :**

```bash
docker run [OPTIONS] IMAGE [COMMAND]
```

**Options courantes :**

- `-d` : Mode détaché (background)
- `-p` : Mapping de ports (host:container)
- `--name` : Nom du conteneur
- `-e` : Variables d'environnement
- `-v` : Montage de volumes
- `--rm` : Suppression automatique après arrêt
- `-a` : 'attach' afficher la **sortie (stdout/stderr)** du conteneur dans le terminal

**Exemples :**

```bash
# Exécution simple en mode interactif
docker run -it ubuntu bash

# Serveur web en arrière-plan avec port exposé
docker run -d -p 8080:80 --name mon-nginx nginx

# Avec variables d'environnement et volume
docker run -d -e MYSQL_ROOT_PASSWORD=secret -v data:/var/lib/mysql mysql
```

### `docker start`

Démarre un conteneur existant (arrêté).

```bash
docker start mon-nginx
docker start -a mon-nginx  # Avec attachement à la sortie
```

### `docker stop`

Arrête proprement un conteneur en cours d'exécution.

```bash
docker stop mon-nginx
docker stop $(docker ps -q)  # Arrête tous les conteneurs actifs
```

### `docker restart`

Redémarre un conteneur.

```bash
docker restart mon-nginx
```

### `docker rm`

Supprime un ou plusieurs conteneurs arrêtés.

```bash
docker rm mon-nginx
docker rm -f mon-nginx     # Force la suppression (même si actif)
docker rm $(docker ps -aq) # Supprime tous les conteneurs arrêtés
```

## Inspection et surveillance

### `docker ps`

Liste les conteneurs.

```bash
docker ps           # Conteneurs actifs uniquement
docker ps -a        # Tous les conteneurs
docker ps -q        # IDs uniquement
docker ps --filter "status=exited"  # Filtrage par statut
```

**Colonnes importantes :**

- `CONTAINER ID` : Identifiant court
- `IMAGE` : Image source
- `STATUS` : État actuel
- `PORTS` : Ports exposés
- `NAMES` : Nom du conteneur

### `docker logs`

Affiche les logs d'un conteneur.

```bash
docker logs mon-nginx
docker logs -f mon-nginx      # Mode suivi (follow)
docker logs --tail 100 mon-nginx  # 100 dernières lignes
docker logs --since 10m mon-nginx # Logs des 10 dernières minutes
```

### `docker inspect`

Informations détaillées sur un conteneur ou une image (format JSON).

```bash
docker inspect mon-nginx
docker inspect --format '{{.NetworkSettings.IPAddress}}' mon-nginx  # Extraction spécifique
```

**Informations obtenues :**

- Configuration complète
- État du conteneur
- Configuration réseau
- Montages de volumes
- Variables d'environnement

### `docker stats`

Statistiques en temps réel sur les conteneurs.

```bash
docker stats
docker stats mon-nginx  # Stats d'un conteneur spécifique
```

**Métriques affichées :**

- CPU %
- Mémoire utilisée/limite
- I/O réseau
- I/O disque

## Interaction avec les conteneurs

### `docker exec`

Exécute une commande dans un conteneur actif.

```bash
docker exec mon-nginx ls /etc
docker exec -it mon-nginx bash  # Shell interactif
docker exec -u root mon-nginx whoami  # En tant qu'utilisateur spécifique
```

**Cas d'usage :**

- Debugging
- Vérification de configuration
- Maintenance ponctuelle
- Exécution de scripts

### `docker attach`

Attache le terminal au processus principal du conteneur.

```bash
docker attach mon-nginx
```

**Différence avec `exec` :**

- `attach` : Se connecte au processus principal (PID 1)
- `exec` : Lance un nouveau processus dans le conteneur

**Note :** `Ctrl+C` dans un conteneur attaché arrête le conteneur. Utiliser `Ctrl+P` puis `Ctrl+Q` pour détacher sans arrêter.

## Gestion des images

### `docker pull`

Télécharge une image depuis un registry.

```bash
docker pull nginx
docker pull nginx:1.25        # Version spécifique
docker pull nginx:1.25-alpine # Variante alpine
```

### `docker images`

Liste les images locales.

```bash
docker images
docker images -a        # Inclut les images intermédiaires
docker images -q        # IDs uniquement
docker images --filter "dangling=true"  # Images non taguées
```

**Colonnes :**

- `REPOSITORY` : Nom de l'image
- `TAG` : Version/variante
- `IMAGE ID` : Identifiant
- `CREATED` : Date de création
- `SIZE` : Taille sur le disque

### `docker rmi`

Supprime une ou plusieurs images.

```bash
docker rmi nginx
docker rmi -f nginx     # Force la suppression
docker rmi $(docker images -q)  # Supprime toutes les images
```

**Note :** Impossible de supprimer une image utilisée par un conteneur (même arrêté).

### `docker tag`

Crée un alias pour une image.

```bash
docker tag nginx:latest mon-registry.com/nginx:v1
docker tag nginx:latest nginx:backup
```

**Utilité :**

- Versioning
- Préparation au push vers un registry
- Organisation des images

## TP pratique : Manipulation de plusieurs conteneurs

### Exercice 1 : Déploiement multi-conteneurs

```bash
# Démarrer un serveur web
docker run -d --name web nginx

# Démarrer une base de données
docker run -d --name db -e POSTGRES_PASSWORD=secret postgres

# Démarrer une application Node.js
docker run -d --name api -p 3000:3000 node:18 node -e "require('http').createServer((req,res) => res.end('Hello')).listen(3000)"

# Vérifier l'état
docker ps
```

### Exercice 2 : Inspection et debugging

```bash
# Consulter les logs
docker logs web
docker logs -f api

# Vérifier les ressources
docker stats

# Accéder au shell d'un conteneur
docker exec -it db psql -U postgres

# Inspecter la configuration réseau
docker inspect web --format '{{.NetworkSettings.IPAddress}}'
```

### Exercice 3 : Cycle de vie complet

```bash
# Arrêter les conteneurs
docker stop web db api

# Redémarrer un conteneur spécifique
docker start web

# Supprimer un conteneur
docker rm api

# Nettoyage complet
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

### Exercice 4 : Gestion des images

```bash
# Télécharger différentes versions
docker pull python:3.11
docker pull python:3.11-slim
docker pull python:3.11-alpine

# Comparer les tailles
docker images python

# Taguer une image
docker tag python:3.11-alpine python:prod

# Nettoyer les images non utilisées
docker rmi python:3.11 python:3.11-slim
```

## Commandes utiles supplémentaires

### Nettoyage

```bash
docker system prune        # Supprime les ressources inutilisées
docker system prune -a     # Inclut les images non utilisées
docker volume prune        # Nettoie les volumes orphelins
docker network prune       # Nettoie les réseaux non utilisés
```

### Copie de fichiers

```bash
docker cp local/file.txt mon-nginx:/etc/nginx/
docker cp mon-nginx:/var/log/nginx/access.log ./logs/
```

### Export/Import

```bash
docker export mon-nginx > nginx-backup.tar
docker import nginx-backup.tar mon-nginx:backup
```

## Points clés à retenir

- `docker run` crée et démarre, `docker start` redémarre un conteneur existant
- `docker ps` pour surveiller, `docker logs` pour débugger
- `docker exec` pour interagir sans perturber le processus principal
- `docker inspect` fournit toutes les informations techniques
- Toujours nettoyer les ressources inutilisées avec `prune`
- Les IDs peuvent être abrégés (3-4 premiers caractères suffisent)
- L'option `-f` force généralement une opération
- L'option `-a` affiche généralement "tout" (all)