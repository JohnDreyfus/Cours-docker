# Introduction aux réseaux Docker

## Contexte et problématique

Lorsqu'on déploie plusieurs conteneurs, ils doivent souvent communiquer entre eux. Docker fournit un système de réseau qui permet :

- L'isolation réseau entre conteneurs
- La communication contrôlée entre services
- La résolution DNS automatique
- La sécurité par segmentation

## Types de réseaux Docker

### 1. Bridge (par défaut)

**Caractéristiques :**

- Réseau privé interne créé sur l'hôte
- Les conteneurs obtiennent une IP dans la plage du bridge
- Communication possible entre conteneurs du même réseau
- Accès à l'extérieur via NAT

**Cas d'usage :**

- Applications multi-conteneurs sur un même hôte
- Isolation entre différents projets

**Exemple :**

```bash
# Réseau bridge par défaut
docker run -d --name web nginx

# Réseau bridge personnalisé
docker network create mon-reseau
docker run -d --name web --network mon-reseau nginx
```

### 2. Host

**Caractéristiques :**

- Le conteneur partage la stack réseau de l'hôte
- Pas d'isolation réseau
- Performance maximale (pas de NAT)
- Le conteneur utilise directement les ports de l'hôte

**Cas d'usage :**

- Applications nécessitant des performances réseau optimales
- Monitoring système

**Exemple :**

```bash
docker run -d --network host nginx
# Nginx sera accessible directement sur le port 80 de l'hôte
```

### 3. None

**Caractéristiques :**

- Aucune interface réseau
- Isolation totale
- Seule l'interface loopback est disponible

**Cas d'usage :**

- Conteneurs de traitement batch sans besoin réseau
- Tests en isolation complète

**Exemple :**

```bash
docker run -d --network none mon-app
```

## Communication entre conteneurs

### Résolution DNS interne

Docker fournit un serveur DNS intégré :

- Chaque conteneur peut être contacté par son nom
- Valable uniquement sur les réseaux personnalisés (pas le bridge par défaut)
- Le nom du conteneur devient son hostname DNS

**Exemple :**

```bash
# Créer un réseau
docker network create app-network

# Lancer une base de données
docker run -d --name database --network app-network postgres

# Lancer une application qui peut contacter "database"
docker run -d --name backend --network app-network mon-api
# Dans le code : connexion à "database:5432"
```

### Alias réseau

Possibilité de définir des alias pour un conteneur :

```bash
docker run -d --name db --network app-network --network-alias postgres postgres
# Accessible via "db" ou "postgres"
```

## Gestion des réseaux personnalisés

### Créer un réseau

```bash
# Création simple
docker network create mon-reseau

# Avec options
docker network create --driver bridge --subnet 172.20.0.0/16 mon-reseau-custom
```

### Lister les réseaux

```bash
docker network ls
```

### Inspecter un réseau

```bash
docker network inspect mon-reseau
# Affiche : configuration, conteneurs connectés, sous-réseau, etc.
```

### Connecter/déconnecter un conteneur

```bash
# Connecter un conteneur existant
docker network connect mon-reseau mon-conteneur

# Déconnecter
docker network disconnect mon-reseau mon-conteneur
```

### Supprimer un réseau

```bash
docker network rm mon-reseau

# Nettoyer les réseaux non utilisés
docker network prune
```

## Bonnes pratiques

- Créer des réseaux dédiés par application ou projet
- Utiliser des réseaux personnalisés plutôt que le bridge par défaut
- Nommer explicitement les réseaux pour la lisibilité
- Ne pas exposer de ports inutilement
- Utiliser les noms de conteneurs pour la communication interne
- Documenter l'architecture réseau du projet

## Schéma réseau typique

```
┌─────────────────────────────────────┐
│         Réseau: frontend            │
│  ┌──────────┐      ┌─────────────┐  │
│  │   nginx  │──────│  React App  │  │
│  └──────────┘      └─────────────┘  │
└────────┬────────────────────────────┘
         │
┌────────┴────────────────────────────┐
│         Réseau: backend             │
│  ┌──────────┐      ┌─────────────┐  │
│  │   API    │──────│  Database   │  │
│  └──────────┘      └─────────────┘  │
└─────────────────────────────────────┘
```
