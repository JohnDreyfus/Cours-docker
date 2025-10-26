# 5. Créer son Dockerfile

## Introduction au Dockerfile

Un Dockerfile est un fichier texte contenant l'ensemble des instructions nécessaires pour construire une image Docker de manière automatisée et reproductible.

**Avantages :**

- Automatisation complète du processus de création d'images
- Versionnement possible avec Git
- Documentation implicite de l'environnement
- Reproductibilité garantie

## Structure et syntaxe

### Format de base

```dockerfile
# Commentaire
INSTRUCTION arguments
```

**Règles :**

- Une instruction par ligne
- Les commentaires commencent par `#`
- Les instructions sont insensibles à la casse (convention : MAJUSCULES)
- Le fichier doit s'appeler `Dockerfile` (sans extension)

## Instructions principales

### FROM - Image de base

Définit l'image de départ pour la construction.

```dockerfile
FROM ubuntu:22.04
FROM node:18-alpine
FROM python:3.11-slim
```

**Points clés :**

- Toujours la première instruction (sauf `ARG` global)
- Privilégier les images officielles
- Utiliser des tags spécifiques plutôt que `latest`

### RUN - Exécution de commandes

Execute des commandes lors du build pour installer des packages, créer des fichiers, etc.

```dockerfile
# Forme shell
RUN apt-get update && apt-get install -y curl

# Forme exec (recommandée)
RUN ["apt-get", "update"]

# Multiples commandes (best practice)
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Bonnes pratiques :**

- Chaîner les commandes avec `&&` pour réduire les layers
- Nettoyer les caches après installation
- Utiliser `\` pour la lisibilité

### COPY - Copie de fichiers

Copie des fichiers/dossiers depuis le contexte de build vers l'image.

```dockerfile
COPY package.json /app/
COPY src/ /app/src/
COPY . /app/
```

**Caractéristiques :**

- Chemin source : relatif au contexte de build
- Chemin destination : absolu ou relatif au `WORKDIR`
- Préserve les métadonnées des fichiers

### ADD - Copie avancée

Similaire à `COPY` avec fonctionnalités supplémentaires.

```dockerfile
ADD archive.tar.gz /app/
ADD https://example.com/file.zip /tmp/
```

**Différences avec COPY :**

- Décompresse automatiquement les archives tar
- Peut télécharger des fichiers depuis une URL

**Recommandation :** Préférer `COPY` sauf si extraction automatique nécessaire.

### CMD - Commande par défaut

Définit la commande exécutée au démarrage du conteneur.

```dockerfile
# Forme exec (recommandée)
CMD ["nginx", "-g", "daemon off;"]

# Forme shell
CMD npm start

# Paramètres pour ENTRYPOINT
CMD ["--port", "8080"]
```

**Points importants :**

- Une seule instruction `CMD` par Dockerfile (la dernière gagne)
- Peut être surchargée au `docker run`
- Utilisée pour les arguments par défaut si `ENTRYPOINT` est défini

### ENTRYPOINT - Point d'entrée

Définit l'exécutable principal du conteneur.

```dockerfile
ENTRYPOINT ["python", "app.py"]
ENTRYPOINT ["node", "server.js"]
```

**Différence avec CMD :**

- `ENTRYPOINT` : exécutable fixe
- `CMD` : arguments par défaut modifiables

```dockerfile
# Combinaison ENTRYPOINT + CMD
ENTRYPOINT ["python"]
CMD ["app.py"]

# Équivaut à : python app.py
# docker run image script.py → python script.py
```

## Instructions avancées

### WORKDIR - Répertoire de travail

Définit le répertoire courant pour les instructions suivantes.

```dockerfile
WORKDIR /app
COPY . .
RUN npm install
```

**Avantages :**

- Évite les chemins absolus répétitifs
- Crée automatiquement le répertoire s'il n'existe pas
- Définit le répertoire par défaut au démarrage du conteneur

### ENV - Variables d'environnement

Définit des variables d'environnement disponibles au build et au runtime.

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000 \
    DB_HOST=localhost
```

**Utilisation :**

```dockerfile
ENV APP_HOME=/app
WORKDIR $APP_HOME
COPY . $APP_HOME
```

### EXPOSE - Documentation des ports

Documente les ports sur lesquels l'application écoute.

```dockerfile
EXPOSE 80
EXPOSE 8080/tcp
EXPOSE 53/udp
```

**Important :** N'expose pas réellement les ports, c'est uniquement documentaire. Utiliser `-p` au `docker run`.

### USER - Utilisateur d'exécution

Définit l'utilisateur pour les instructions `RUN`, `CMD`, `ENTRYPOINT`.

```dockerfile
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser
```

**Sécurité :** Ne jamais exécuter des applications en tant que root en production.

### ARG - Arguments de build

Définit des variables utilisables uniquement pendant le build.

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

ARG BUILD_DATE
RUN echo "Build date: ${BUILD_DATE}"
```

**Utilisation au build :**

```bash
docker build --build-arg NODE_VERSION=20 --build-arg BUILD_DATE=$(date) .
```

**Différence avec ENV :**

- `ARG` : disponible uniquement au build
- `ENV` : disponible au build et au runtime

## Build d'une image

### Commande docker build

```bash
# Syntaxe de base
docker build -t mon-image:tag .

# Avec contexte spécifique
docker build -t mon-image:1.0 ./mon-projet

# Avec Dockerfile personnalisé
docker build -t mon-image -f Dockerfile.prod .

# Avec build args
docker build --build-arg VERSION=1.2.3 -t mon-image .

# Sans cache
docker build --no-cache -t mon-image .
```

**Options courantes :**

- `-t` : tag de l'image
- `-f` : spécifier un Dockerfile alternatif
- `--build-arg` : passer des arguments de build
- `--no-cache` : forcer la reconstruction complète

### Contexte de build

Le contexte est l'ensemble des fichiers envoyés au daemon Docker.

```bash
docker build -t app .
# Le point "." désigne le contexte (répertoire courant)
```

**Optimisation avec .dockerignore :**

```
node_modules/
.git/
*.log
.env
tests/
```

## Optimisation du build

### Ordre des layers

Docker met en cache chaque instruction. Une modification invalide le cache de cette instruction et de toutes les suivantes.

**Mauvaise pratique :**

```dockerfile
FROM node:18-alpine
COPY . /app
WORKDIR /app
RUN npm install
```

**Bonne pratique :**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
```

**Principe :** Placer les instructions qui changent rarement en premier.

### Stratégies d'optimisation

**1. Minimiser le nombre de layers**

```dockerfile
# Moins efficace (3 layers)
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# Plus efficace (1 layer)
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*
```

**2. Exploiter le cache**

```dockerfile
# Les dépendances changent rarement
COPY package.json package-lock.json ./
RUN npm install

# Le code source change souvent
COPY . .
```

**3. Multi-stage builds** (aperçu)

```dockerfile
# Stage de build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage de production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

## TP Pratique : Application Node.js

### Objectif

Créer une image Docker pour une application Node.js simple.

### Structure du projet

```
mon-app/
├── Dockerfile
├── .dockerignore
├── package.json
├── app.js
└── public/
    └── index.html
```

### Fichier app.js

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.static('public'));

app.get('/api/health', (req, res) => {
    res.json({ status: 'OK', timestamp: new Date() });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Fichier package.json

```json
{
  "name": "mon-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.0"
  },
  "scripts": {
    "start": "node app.js"
  }
}
```

### Dockerfile

```dockerfile
# Image de base
FROM node:18-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install --production

# Copier le code source
COPY . .

# Créer un utilisateur non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Exposer le port
EXPOSE 3000

# Variable d'environnement
ENV NODE_ENV=production

# Commande de démarrage
CMD ["npm", "start"]
```

### Fichier .dockerignore

```
node_modules/
npm-debug.log
.git/
.env
*.md
.DS_Store
```

### Construction et exécution

```bash
# Build de l'image
docker build -t mon-app:1.0 .

# Vérifier l'image créée
docker images mon-app

# Lancer le conteneur
docker run -d -p 3000:3000 --name mon-app mon-app:1.0

# Vérifier les logs
docker logs mon-app

# Tester l'application
curl http://localhost:3000/api/health

# Arrêter et supprimer
docker stop mon-app
docker rm mon-app
```

### Exercice supplémentaire

Modifier le Dockerfile pour :

1. Utiliser une variable `ARG` pour la version de Node.js
2. Ajouter une variable d'environnement `APP_VERSION`
3. Créer le répertoire `/app/logs` avec les bonnes permissions
4. Ajouter un `HEALTHCHECK`

**Solution :**

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

RUN mkdir -p logs && \
    addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3000

ENV NODE_ENV=production \
    APP_VERSION=1.0.0

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/api/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["npm", "start"]
```

## Résumé des bonnes pratiques

- Utiliser des images de base officielles et légères (alpine)
- Ordonner les instructions du moins changeant au plus changeant
- Chaîner les commandes `RUN` pour réduire les layers
- Copier d'abord les fichiers de dépendances avant le code source
- Utiliser `.dockerignore` pour exclure les fichiers inutiles
- Créer et utiliser un utilisateur non-root
- Spécifier des tags précis plutôt que `latest`
- Nettoyer les caches et fichiers temporaires
- Documenter avec `EXPOSE` et des commentaires
- Privilégier `COPY` à `ADD` sauf besoin spécifique