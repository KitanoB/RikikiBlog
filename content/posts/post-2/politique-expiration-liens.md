---
date: '2025-05-31T20:14:03+02:00'
draft: false
title: "Politique d'expiration des liens pour RikikiLink"
---

## Pourquoi une expiration simple des liens ? Retour sur un choix pragmatique

Quand on construit un raccourcisseur de liens, une question revient vite : doit-on laisser les liens vivre éternellement ? Beaucoup de services gardent leurs liens actifs sans limite, mais ce n’est pas sans risques : sécurité, confidentialité, pollution de la base… et puis, qui veut vraiment gérer des redirections vieilles de plusieurs années ?

Pour RikikiLink, j’ai choisi une approche radicalement simple, adaptée à un MVP développé en 60 heures : chaque lien expire automatiquement 10 jours après sa création. Pas de planificateur, pas de cron, pas de complexité cachée. Juste une règle claire, facile à expliquer et à maintenir.

---

### 🎯 Les objectifs

- **Éviter les redirections éternelles** : un lien qui traîne trop longtemps peut devenir une faille ou une gêne.
- **Rester simple** : pas de tâches planifiées, pas de TTL avancé, pas de batch de nettoyage.
- **Rendre l’expiration visible et testable** : l’API et le frontend affichent clairement qu’un lien a expiré.

---

### 🛠️ Comment ça marche ?

À chaque accès, le service vérifie la date de création du lien :  
- Si le lien a plus de 10 jours, il est désactivé en base et la redirection est bloquée avec une erreur HTTP 410 Gone.
- Sinon, la redirection s’effectue normalement.

Ce mécanisme s’inspire des meilleures pratiques du secteur, où la vérification de l’expiration se fait à la volée, sans nécessiter de nettoyage périodique complexe.

**Exemple d’implémentation :**

```java
if (link.getCreatedAt().isBefore(Instant.now().minus(10, ChronoUnit.DAYS))) {
    link.setActive(false);
    linkRepository.save(link);
    throw new LinkExpiredException(); // HTTP 410 Gone
}
```

---

### ❌ Pourquoi ne pas faire plus compliqué ?

J’ai écarté :
- **Le TTL Redis** : rapide, mais pas persistant, et cela aurait cassé le modèle centré sur PostgreSQL.
- **Un champ `expiresAt` + job planifié** : surdimensionné pour un MVP mono-utilisateur, et source de bugs potentiels.
- **Un garbage collector périodique** : inutile tant que la volumétrie reste faible.

Dans la littérature, beaucoup de services proposent des liens courts à expiration configurable, mais cela implique souvent une gestion plus lourde (cron, batch, etc.)[1]. Pour un MVP, la simplicité prime.

---

### 📢 Expérience utilisateur et transparence

L’utilisateur est informé dès qu’un lien a expiré :  
- L’API retourne un statut explicite (HTTP 410 Gone).
- Le frontend peut afficher “lien expiré” ou “désactivé”.

C’est une bonne pratique recommandée pour éviter toute frustration et garantir une expérience claire.

---

### 🚀 Et pour la suite ?

Si RikikiLink évolue, il sera facile d’ajouter :
- Un champ `expiresAt` configurable,
- Un job de nettoyage pour les usages à grande échelle,
- La possibilité pour l’utilisateur de choisir la durée de vie de ses liens.

---

**En résumé** :  
Ce choix d’expiration automatique à chaque accès permet de garder le code simple, fiable et transparent, tout en protégeant la confidentialité et la sécurité des utilisateurs. C’est une base solide pour un MVP, et une fondation saine pour de futures évolutions !

---
