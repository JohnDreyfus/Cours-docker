# Introduction et contexte

## 1. Problématiques du déploiement d'applications

### Le déploiement traditionnel pose plusieurs défis :

**Dépendances**
- Versions différentes des bibliothèques selon les projets
- Conflits entre les dépendances système
- Difficultés à reproduire l'environnement exact

**Environnements**
- "Ça marche sur ma machine" mais pas en production
- Différences entre développement, test et production
- Configuration manuelle chronophage et sujette aux erreurs

**Portabilité**
- Applications liées à un système d'exploitation spécifique
- Migration complexe entre serveurs
- Difficultés de mise à l'échelle

## 2. Qu'est-ce que Docker ?

Docker est une plateforme de conteneurisation qui permet d'empaqueter une application avec toutes ses dépendances dans un conteneur standardisé.

### Conteneurisation vs Virtualisation

**Virtualisation traditionnelle**
- Chaque VM embarque un OS complet
- Hyperviseur nécessaire (VMware, VirtualBox)
- Consommation importante de ressources (RAM, CPU, disque)
- Démarrage lent (plusieurs minutes)

**Conteneurisation Docker**
- Partage du noyau de l'OS hôte
- Isolation au niveau processus
- Léger : quelques Mo vs plusieurs Go
- Démarrage quasi instantané (secondes)
- Plus de conteneurs sur une même machine

**Exemple concret**
```
VM : 3 applications = 3 OS complets (3x 2GB = 6GB)
Docker : 3 applications = 1 OS partagé + 3 conteneurs (2GB + 300MB)
```

## 3. Cas d'usage et avantages

### Cas d'usage principaux

**Développement**
- Environnement identique pour toute l'équipe
- Onboarding rapide des nouveaux développeurs
- Tests sur différentes versions facilités

**Déploiement**
- Build once, run anywhere
- Déploiements reproductibles et automatisables
- Rollback simple en cas de problème

**Microservices**
- Isolation des services
- Scaling indépendant de chaque composant
- Technologies différentes par service

### Avantages clés
- **Cohérence** : même environnement partout
- **Isolation** : chaque conteneur est indépendant
- **Rapidité** : démarrage en quelques secondes
- **Efficacité** : utilisation optimale des ressources
- **Portabilité** : fonctionne sur tout système supportant Docker

## 4. Architecture Docker

Docker repose sur une architecture client-serveur composée de trois éléments principaux.
### Docker Daemon (dockerd)
- Moteur Docker qui tourne en arrière-plan
- Gère les conteneurs, images, réseaux et volumes
- Écoute les requêtes de l'API Docker
- Exécute sur la machine hôte

### Docker Client (docker)
- Interface en ligne de commande
- Communique avec le daemon via l'API REST
- Envoie les commandes : `docker run`, `docker build`, etc.
- Peut communiquer avec des daemons distants

### Docker Registry
- Stocke les images Docker
- Docker Hub : registry public par défaut
- Registries privés possibles (entreprise)
- Commandes principales : `pull` (télécharger), `push` (envoyer)

**Schéma simplifié du flux**
```
Client (docker build) → Daemon → construit l'image → Registry
Client (docker run) → Daemon → pull l'image depuis Registry → lance le conteneur
```

## 5. Installation

### Installation

```bash
https://www.docker.com/products/docker-desktop/
```