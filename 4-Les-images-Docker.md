# 4. Les images Docker

## Le Docker Hub et les registries

### Qu'est-ce qu'un registry Docker ?

Un registry est un service de stockage et de distribution d'images Docker. Il permet de :

- Partager des images entre développeurs
- Versionner les images d'applications
- Automatiser les déploiements

### Docker Hub

Docker Hub est le registry public par défaut :

- Plus de 100 000 images officielles et communautaires
- Gratuit pour les images publiques
- Accessible via `docker pull` sans configuration

**Utilisation :**

```bash
# Rechercher une image
docker search nginx

# Télécharger une image depuis Docker Hub
docker pull nginx

# Télécharger une version spécifique
docker pull nginx:1.25.0
```

### Autres registries

- **Registries privés** : GitHub Container Registry, GitLab Container Registry, AWS ECR
- **Registry auto-hébergé** : installation locale avec `docker run -d -p 5000:5000 registry:2`

**Exemple avec un registry privé :**

```bash
# Tag pour un registry privé
docker tag mon-app:latest registry.exemple.com/mon-app:latest

# Push vers le registry
docker push registry.exemple.com/mon-app:latest
```

## Comprendre les layers et le système de cache

### Architecture en couches (layers)

Une image Docker est composée de plusieurs couches empilées :

- Chaque instruction du Dockerfile crée une nouvelle couche
- Les couches sont en lecture seule
- Les couches sont partagées entre images (économie d'espace)
- Seule la couche supérieure (conteneur) est modifiable

**Exemple :**

```dockerfile
FROM ubuntu:22.04          # Layer 1
RUN apt-get update         # Layer 2
RUN apt-get install -y nginx  # Layer 3
COPY app.conf /etc/nginx/  # Layer 4
```

### Visualiser les layers

```bash
# Afficher l'historique des layers
docker history nginx

# Inspecter les détails d'une image
docker inspect nginx
```

### Système de cache

Docker utilise un cache intelligent lors du build :

- Si une couche n'a pas changé, Docker réutilise la version en cache
- Le cache s'invalide dès qu'une couche change
- Toutes les couches suivantes sont reconstruites

**Exemple de cache invalidé :**

```dockerfile
FROM node:18
COPY package.json .     # Si ce fichier change, cache invalidé ici
RUN npm install         # Cette ligne sera réexécutée
COPY . .                # Et toutes les suivantes aussi
```

**Optimisation avec le cache :**

```dockerfile
FROM node:18
COPY package.json .     # Copier d'abord les dépendances
RUN npm install         # npm install en cache si package.json inchangé
COPY . .                # Changements du code n'invalident pas npm install
```

## Versions et tags d'images

### Comprendre les tags

Un tag identifie une version spécifique d'une image :

- Format : `nom-image:tag`
- Si aucun tag n'est spécifié, Docker utilise `:latest` par défaut
- `:latest` ne signifie pas forcément "la plus récente"

**Exemples de tags courants :**

```bash
# Versions spécifiques
docker pull node:18.17.0
docker pull postgres:15.3

# Versions majeures
docker pull node:18
docker pull python:3.11

# Variantes
docker pull node:18-alpine     # Version légère
docker pull node:18-slim       # Version réduite
docker pull python:3.11-bookworm  # Basée sur Debian Bookworm
```

### Convention de nommage

```
registry/namespace/repository:tag
```

**Exemples :**

```bash
docker.io/library/nginx:latest           # Image officielle
docker.io/moncompte/mon-app:v1.2.3      # Image personnelle
ghcr.io/organisation/projet:develop      # GitHub Container Registry
```

### Gestion des tags locaux

```bash
# Lister les images avec leurs tags
docker images

# Créer un nouveau tag pour une image existante
docker tag nginx:latest mon-nginx:v1

# Supprimer un tag (l'image reste si d'autres tags existent)
docker rmi mon-nginx:v1
```

## Bonnes pratiques de choix d'images de base

### Privilégier les images officielles

Les images officielles sont maintenues par Docker et les éditeurs :

- Mises à jour de sécurité régulières
- Documentation complète
- Optimisation et bonnes pratiques intégrées

**Comment les reconnaître :**

- Badge "Docker Official Image" sur Docker Hub
- Pas de préfixe utilisateur (ex: `nginx` au lieu de `utilisateur/nginx`)

### Choisir le bon tag

#### Éviter `:latest` en production

```dockerfile
# À éviter
FROM node:latest

# Préférer
FROM node:18.17.0
```

Raisons :

- `:latest` peut changer sans préavis
- Impossibilité de reproduire un build identique
- Risques de régression lors des mises à jour

#### Utiliser des versions spécifiques

```dockerfile
# Version complète (recommandé pour la production)
FROM python:3.11.4-slim

# Version majeure.mineure (acceptable pour le développement)
FROM python:3.11-slim
```

### Choisir la variante appropriée

#### Images standard vs images allégées

**Standard :**

```dockerfile
FROM node:18
# Taille : ~900 MB
# Contient : outils de build, bibliothèques complètes
# Usage : développement, build complexes
```

**Slim :**

```dockerfile
FROM node:18-slim
# Taille : ~200 MB
# Contient : runtime minimal, moins d'outils
# Usage : production, applications simples
```

**Alpine :**

```dockerfile
FROM node:18-alpine
# Taille : ~120 MB
# Contient : Alpine Linux (minimaliste)
# Usage : production, optimisation maximale
```

#### Comparaison

|Variante|Taille|Cas d'usage|Avantages|Inconvénients|
|---|---|---|---|---|
|Standard|Grande|Développement|Outils complets, compatibilité|Taille importante|
|Slim|Moyenne|Production|Bon compromis|Moins d'outils|
|Alpine|Petite|Production optimisée|Très léger, rapide|Compatibilité (musl vs glibc)|

### Considérations de sécurité

```bash
# Vérifier les vulnérabilités d'une image
docker scout cves nginx:latest

# Préférer les images récentes
docker pull node:18-bookworm  # Debian 12 (récent)
```

**Principes de sécurité :**

- Utiliser des images maintenues activement
- Mettre à jour régulièrement les images de base
- Scanner les vulnérabilités avant déploiement
- Éviter les images non officielles sans vérification

### Exemples de choix selon le contexte

**Application Node.js en développement :**

```dockerfile
FROM node:18
```

**Application Node.js en production :**

```dockerfile
FROM node:18.17.0-alpine
```

**Application Python avec dépendances système complexes :**

```dockerfile
FROM python:3.11-slim-bookworm
```

**Service simple sans compilation :**

```dockerfile
FROM nginx:1.25.0-alpine
```