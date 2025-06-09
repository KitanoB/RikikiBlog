---
date: '2025-06-09T10:00:00+02:00'
draft: false
title: 'Pourquoi un contrat OpenAPI est essentiel pour votre API (Exemple avec RikikiLink)'         
cover:
    image: "https://github.com/KitanoB/RikikiBlog/blob/main/content/posts/post-4/swagger.png"       
    alt: "open api illustration"
    caption: ""
    relative: true           
---

<img src="https://github.com/KitanoB/RikikiBlog/blob/main/content/posts/post-4/swagger.png" alt="open api illustration" width="200" height="200" />

## Pourquoi un contrat OpenAPI est essentiel pour votre API (Exemple avec RikikiLink)

Dans le développement d’APIs, la clarté et la collaboration sont primordiales. Pourtant, je dois l’avouer : avant de me plonger dans la création de RikikiLink, notre raccourcisseur de liens, je n’avais jamais vraiment pris la mesure de ce que pouvait apporter un contrat OpenAPI. J’ai découvert, parfois à tâtons, à quel point cette spécification pouvait transformer la façon dont on conçoit, documente et fait évoluer une API. Voici donc un retour d’expérience, aussi pratique que sincère, sur l’intégration d’OpenAPI avec Springdoc dans un projet Spring Boot.

### **Pourquoi OpenAPI ? Ce que j’ai compris en chemin**

Au début, OpenAPI me semblait être une couche de documentation un peu « en plus ». Mais très vite, j’ai réalisé que c’était bien plus :  
- **Standardiser et formaliser** la définition des endpoints (routes, verbes HTTP, schémas de données)  
- **Générer automatiquement** une documentation interactive, des clients SDK, et bien plus  
- **Valider et sécuriser** les appels API grâce à des tests contractuels et du linting en CI

J’ai donc commencé à annoter mes contrôleurs Spring Boot avec Swagger/OpenAPI via la bibliothèque springdoc-openapi… sans vraiment savoir jusqu’où ça allait m’emmener.

### **1. Annoter ses contrôleurs : premiers pas (et premières surprises)**

La première étape, c’est d’ajouter les annotations sur les contrôleurs. Au début, je me contentais du minimum, puis, en testant Swagger-UI, j’ai vu à quel point chaque annotation enrichissait la documentation générée. Par exemple :

```java
@PostMapping("/links")
@Operation(
  summary = "Créer un nouveau raccourci",
  description = "Génère un code unique et retourne l'objet Link"
)
@ApiResponses(value = {
  @ApiResponse(responseCode = "201", description = "Lien créé",
      content = @Content(mediaType = "application/json",
        schema = @Schema(implementation = LinkResponse.class))),
  @ApiResponse(responseCode = "400", description = "Requête invalide"),
  @ApiResponse(responseCode = "500", description = "Erreur serveur")
})
public ResponseEntity createLink(
    @Valid @RequestBody LinkCreateRequest request) {
  // ... logique existante ...
}
```

Et pour la redirection :

```java
@GetMapping("/r/{code}")
@Operation(summary = "Rediriger vers l'URL cible pour un code donné")
@ApiResponses({
  @ApiResponse(responseCode = "302", description = "Redirection vers targetUrl"),
  @ApiResponse(responseCode = "404", description = "Lien non trouvé ou inactif")
})
public ResponseEntity redirect(@PathVariable String code) {
  // ... logique existante ...
}
```

À chaque ajout d’annotation, je voyais la documentation Swagger-UI s’enrichir en temps réel. C’est là que j’ai compris l’intérêt de bien décrire chaque endpoint, chaque réponse, chaque schéma.

### **2. Générer le fichier OpenAPI (YAML ou JSON) : la magie opère**

Une fois les annotations en place, j’ai été bluffé : springdoc-openapi génère automatiquement la spécification OpenAPI de l’API.  
Après avoir ajouté la dépendance suivante dans le `pom.xml` :

```xml

  org.springdoc
  springdoc-openapi-starter-webmvc-ui
  2.1.0

```

…il suffit de lancer l’application et d’aller sur :
- `/v3/api-docs` pour la version JSON
- `/v3/api-docs.yaml` pour la version YAML

Une commande toute simple permet de récupérer le fichier :

```bash
curl http://localhost:8080/v3/api-docs.yaml > openapi.yaml
```

C’est ce fichier qui va servir de contrat, de documentation, et même de base pour générer des clients ou des tests.

### **3. Swagger-UI et ReDoc : explorer son API en direct**

Autre découverte : springdoc-openapi ne fait pas que générer le fichier, il propose aussi une interface Swagger-UI prête à l’emploi à l’adresse `/swagger-ui.html`. Tester les endpoints, voir les schémas, tout est interactif !  
Pour une doc plus « légère », ReDoc est aussi disponible (avec la dépendance adaptée), par exemple à `/redoc.html`.  
On peut même personnaliser les chemins dans `application.properties` :

```properties
springdoc.swagger-ui.path=/docs/swagger
springdoc.redoc.path=/docs/redoc
```

### **4. Un extrait concret de la spec générée**

En voyant le YAML généré, j’ai vraiment pris conscience de la puissance d’OpenAPI. Voici un extrait pour nos deux endpoints principaux :

```yaml
openapi: 3.0.3
info:
  title: "RikikiLink API"
  version: "1.0"
paths:
  /links:
    post:
      summary: "Créer un raccourci"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LinkCreateRequest'
      responses:
        '201':
          description: "Lien créé"
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LinkResponse'
        '400':
          description: "Validation Error"
        '500':
          description: "Internal Server Error"
  /r/{code}:
    get:
      summary: "Redirection vers l'URL cible"
      parameters:
        - in: path
          name: code
          required: true
          schema:
            type: string
      responses:
        '302':
          description: "Redirection vers targetUrl"
          headers:
            Location:
              schema:
                type: string
        '404':
          description: "Lien non trouvé ou inactif"
components:
  schemas:
    LinkCreateRequest:
      type: object
      properties:
        targetUrl:
          type: string
          format: uri
      required: [targetUrl]
    LinkResponse:
      type: object
      properties:
        code:
          type: string
        shortUrl:
          type: string
          format: uri
        createdAt:
          type: string
          format: date-time
```

### **Conclusion : une découverte qui change la donne**

Je pensais au départ qu’OpenAPI n’était qu’un outil de documentation. Mais en l’expérimentant, j’ai découvert que c’est un véritable levier :  
- Pour générer des clients dans plusieurs langages et accélérer l’intégration  
- Pour mettre en place des tests contractuels et garantir la cohérence backend  
- Pour offrir une documentation claire, interactive et toujours à jour

Bref, intégrer OpenAPI (et springdoc-openapi) a été une étape clé dans la maturité de RikikiLink. Si vous hésitez encore, je vous encourage à l’essayer… vous découvrirez sûrement, comme moi, des bénéfices insoupçonnés !