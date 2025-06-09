---
date: '2025-05-31T20:47:46+02:00'
draft: false
title: 'Pull Request Validation Intelligence Artificielle'
cover:
    image: "https://github.com/KitanoB/RikikiBlog/blob/main/content/posts/post-1/pria.png"       
    alt: "Double Check AI"
    caption: ""
    relative: true           
---

<img src="https://github.com/KitanoB/RikikiBlog/blob/main/content/posts/post-1/pria.png" alt="drawing" width="200" height="200" />

## Comment je valide mes PR… avec l’aide de l’IA

Développer en solo, c’est grisant : on avance vite, on prend toutes les décisions, on apprend sur le tas… Mais c’est aussi risqué : pas de relecteur pour repérer les bugs, pas de collègue pour challenger les choix techniques, pas de “deuxième paire d’yeux” pour attraper les oublis.

Sur RikikiLink, je me suis retrouvé face à ce dilemme. J’ai envie d’aller vite, mais je veux aussi un code robuste, cohérent, et des choix techniques solides. Ma solution ? Faire appel à deux reviewers… un peu particuliers : **Gemini** et **ClaudeAI**.

---

### 🤖 Pourquoi utiliser l’IA comme reviewer ?

Personne n’est expert sur tout. Et même avec de l’expérience, on passe tous à côté de détails : une faille de sécurité, une API mal documentée, un pattern mal utilisé…  
En soumettant mes Pull Requests à Gemini et ClaudeAI, je bénéficie d’un regard extérieur, objectif, et (presque) infaillible sur mon code.

**Ce que ça m’apporte :**
- **Détection des oublis** : gestion d’erreurs, edge cases, conventions de code.
- **Conseils d’expert** : bonnes pratiques, alternatives techniques, suggestions de refactoring.
- **Vélocité** : des retours quasi-instantanés, 24h/24, sur tous les sujets.

---

### 🛠️ Mon process de validation

1. **J’écris mes commits normalement** (feature, fix, refacto…).
2. **Avant de merger**, je copie le diff ou les fichiers modifiés dans Gemini et ClaudeAI, en leur précisant le contexte et le type de review souhaitée (sécurité, performance, clarté…).
3. **Je compare leurs retours** : parfois ils se recoupent, parfois ils divergent – c’est là que c’est intéressant.
4. **J’intègre les suggestions pertinentes** : corrections, documentation, simplification du code…
5. **Je merge la PR**… avec l’impression d’avoir eu deux experts à mes côtés.

---

### 🧑‍💻 Ce que ça change (vraiment)

- **Plus de confiance** : même en solo, je sais que mes choix ont été challengés.
- **Moins d’angles morts** : l’IA repère souvent des détails que j’aurais ignorés.
- **Un apprentissage continu** : chaque review est l’occasion d’apprendre de nouveaux patterns ou de revoir mes habitudes.

---

### 🚧 Limites et conseils

Bien sûr, l’IA n’est pas parfaite :  
- Elle peut manquer de contexte métier.
- Parfois, elle propose des solutions “overkill” pour un projet simple.
- Il faut garder un esprit critique et ne pas tout appliquer aveuglément.

**Mon conseil :** utilisez l’IA comme un reviewer exigeant, mais gardez la main sur les décisions finales. C’est un outil, pas un remplaçant.

---

### 🚀 En conclusion

Utiliser Gemini et ClaudeAI comme reviewers, c’est un peu comme avoir une équipe d’experts à portée de main, même quand on code en solo. Cela me permet d’aller plus vite, d’être plus rigoureux, et surtout de progresser à chaque PR.
Je vous encourage à essayer ! Que ce soit pour des revues de code, des suggestions d’architecture, ou même des conseils sur les tests, l’IA peut vraiment faire la différence.
Partagez vos retours, je suis curieux de découvrir d’autres workflows !
