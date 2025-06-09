---
date: '2025-05-31T20:01:02+02:00'
draft: false
title: 'Docker Compose Explications'
cover:
    image: "dockercompose.png"       
    alt: "docker compose illustration"
    caption: ""
    relative: true           
---

{{< figure src="dockercompose.png" alt="docker compose illustration" width="400" >}}


## Dans les coulisses du docker-compose de RikikiLink

J'aime la photographie, la video, même si je suis toujours mauvais après dès années. Le problème c'est qu'après chaque road trip, je me retrouve avec des Goctets de photos et vidéos à envoyer à mes amis. Entre les liens de drive que j'aime pas trop partager, les services en ligne de partager de fichier lourds qui expire tous les 3 jours, c’est vite le casse-tête : impossible de mémoriser ces liens, et je perds vite le fil de qui a récupéré quoi. C’est de là qu’est née l’idée de RikikiLink : un raccourcisseur d’URL événementiel, auto-hébergé, pensé pour simplifier la vie… et garder le contrôle sur le partage.

On va un peu plonger dans ce que j'ai imaginé pour cette application, à travers son fichier `docker-compose.yml`. On va voir ensemble comment Docker, les microservices, et quelques choix techniques me permettront de déployer un service robuste et moderne… en moins de 100 lignes de configuration.

---

### 📦 Pourquoi Docker ?

Avant, installer une stack complète (base de données, cache, outils d’admin, services backend…) c’était la galère : conflits de versions, dépendances manquantes, “ça marche chez moi mais pas chez toi”… Avec Docker, tout ça disparaît. En un fichier, je peux lancer toute l’architecture, sur mon laptop ou dans le cloud, et être sûr que tout le monde a le même environnement.

**Les atouts pour RikikiLink :**

- **Portabilité** : le même setup, partout.
- **Isolation** : chaque service dans sa bulle, fini les conflits.
- **Reproductibilité** : un environnement stable, toujours identique.

---

### 🏗️ Architecture du docker-compose

Voici la structure de base de RikikiLink :

```yaml
version: "3.9"

services:
  postgres:
  redis:
  pgadmin:
  link-service:
  stats-service:

networks:
  rikikilink-net:

volumes:
  pgdata:
```

Passons en revue chaque service, et pourquoi il est là.

---

### 🗄️ PostgreSQL : le coffre-fort des liens

```yaml
postgres:
  image: postgres:15
  ...
  volumes:
    - pgdata:/var/lib/postgresql/data
```

C’est ici que tout est stocké : les URLs raccourcies, les clics, les stats. J’ai choisi PostgreSQL parce qu’il est fiable, robuste, et qu’il gère très bien les recherches et l’indexation. Idéal pour retrouver en un clin d’œil qui a cliqué sur quoi.

---

### ⚡ Redis : la mémoire ultra-rapide

```yaml
redis:
  image: redis:7-alpine
```

Redis, c’est le post-it hyperactif de l’application. Il sert à :

- transmettre les événements de clics entre les services,
- accélérer la résolution des URLs grâce au cache.

Pourquoi Redis ? Parce qu’il est imbattable en rapidité et qu’il gère nativement les files d’événements.

---

### 👨‍💻 pgAdmin : le cockpit du développeur

```yaml
pgadmin:
  image: dpage/pgadmin4
  ...
```

Pour explorer les données, écrire des requêtes SQL ou surveiller la structure des tables, pgAdmin est mon allié du quotidien. Un clic, et j’ai une vue claire sur ce qui se passe dans la base.

---

### 🚀 Link Service : le chef d’orchestre des redirections

```yaml
link-service:
  build:
    context: ../../link-service
  ...
```

C’est ce service qui génère les shortcodes (en base62 pour la V0, en V1 ils seront customisables je l'espère), gère les redirections HTTP 302, et publie chaque clic dans Redis. Techniquement, il tourne sur Spring Boot 3 avec WebFlux pour maximiser la réactivité, et s’appuie sur PostgreSQL et Redis.

---

### 📊 Stats Service : l’œil sur les usages

```yaml
stats-service:
  build:
    context: ../../stat-service
  ...
```

Ce microservice consomme les clics depuis Redis, agrège les stats par lien, jour, device ou pays, et expose tout ça via une API REST ou SSE. Pratique pour savoir qui a récupéré les photos… et arrêter le partage au bon moment.

---

### 🔒 Isolation réseau et persistance

Tous ces services communiquent sur un réseau Docker dédié (`rikikilink-net`), ce qui isole l’application du reste de la machine. Les données PostgreSQL sont sauvegardées dans un volume (`pgdata`), pour ne rien perdre même en cas de redémarrage.

---

### 🛠️ Dockerfile : la recette de chaque service

Chaque microservice a son propre Dockerfile, qui permet de compiler l’appli, construire une image isolée, et déployer partout, sans surprise.

---

### ✅ Conclusion

En moins de 100 lignes de configuration, RikikiLink propose :

- 2 bases de données isolées,
- 2 microservices Spring,
- Un frontend Angular,
- Des analytics temps réel,
- Un environnement prêt pour le cloud ou Kubernetes.

C’est la puissance de Docker et du DevOps moderne : un setup portable, isolé, testable, prêt pour la CI/CD… et qui me simplifie la vie, que je sois à Paris, Tokyo ou au fond d’un van !

Je pense que j'ai oublié beaucoup de choses, mais c'est déjà un bon aperçu de ce que j'ai imaginé pour RikikiLink. Si vous avez des questions, des suggestions ou des idées d'amélioration, n'hésitez pas à me contacter !


