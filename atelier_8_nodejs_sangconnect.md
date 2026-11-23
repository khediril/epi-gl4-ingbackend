# Atelier 8 — Documentation d'une API REST avec Swagger et OpenAPI

**Projet fil rouge :** SangConnect — API de gestion des dons de sang  
**Durée :** 3 heures  
**Technologies :** Node.js, Express.js, Swagger, OpenAPI, swagger-jsdoc, swagger-ui-express  
**Prérequis :** Ateliers 1 à 11 terminés. Les étudiants disposent d'une API Express fonctionnelle avec des routes, des contrôleurs, des validations Zod et une authentification JWT.

---

## 1. Contexte et objectifs

Une API REST peut fonctionner correctement sans être facile à utiliser par les autres développeurs.

Lorsqu'une équipe développe une API, elle doit fournir une documentation permettant de comprendre :

- Les fonctionnalités proposées.
- Les endpoints disponibles.
- Les méthodes HTTP utilisées.
- Les paramètres attendus.
- Les données à envoyer dans les requêtes.
- Les réponses retournées par le serveur.
- Les erreurs possibles.
- Les mécanismes d'authentification et d'autorisation.

Dans cet atelier, nous allons utiliser **Swagger et OpenAPI** pour documenter l'API SangConnect.

### Objectifs pédagogiques

À la fin de cet atelier, vous serez capables de :

1. Comprendre le rôle de la documentation d'une API REST.
2. Comprendre la différence entre Swagger et OpenAPI.
3. Installer et configurer swagger-jsdoc et swagger-ui-express.
4. Décrire les endpoints d'une API avec des annotations OpenAPI.
5. Documenter les paramètres, les requêtes et les réponses HTTP.
6. Décrire les schémas de données avec OpenAPI.
7. Documenter l'authentification JWT.
8. Tester les endpoints directement depuis Swagger UI.
9. Générer une documentation cohérente et exploitable de SangConnect.

### Résultat final attendu

À la fin de l'atelier, la documentation de l'API sera accessible à travers une URL :

```text
http://localhost:3000/api-docs
```

Elle permettra à un développeur de découvrir et de tester les fonctionnalités de SangConnect sans consulter directement le code source.

---

## 2. Découvrir Swagger et OpenAPI

### Question 1 — Comprendre le rôle de Swagger

Avant de commencer, consultez les ressources suivantes :

- OpenAPI Specification : https://spec.openapis.org/oas/latest.html
- Swagger : https://swagger.io/
- Swagger UI : https://swagger.io/tools/swagger-ui/

**Travail demandé :**

Répondez aux questions suivantes :

1. Pourquoi une API doit-elle être documentée ?
2. Quelle est la différence entre Swagger et OpenAPI ?
3. Quel est le rôle de Swagger UI ?
4. Quelles informations doit-on retrouver dans la documentation d'un endpoint ?
5. Quel est l'intérêt de pouvoir tester une API directement depuis sa documentation ?

**Éléments de correction :**

- **OpenAPI** est une spécification standard permettant de décrire une API HTTP dans un format structuré.
- **Swagger** désigne un ensemble d'outils qui facilitent la création, la visualisation et l'exploitation de descriptions OpenAPI.
- **Swagger UI** génère une interface web interactive à partir d'une spécification OpenAPI.
- La documentation décrit les endpoints, les paramètres, les schémas, les réponses et les mécanismes de sécurité.

### Question 2 — Identifier les informations à documenter

On considère les fonctionnalités suivantes de SangConnect :

| Fonctionnalité | Méthode HTTP | Endpoint |
|---|---|---|
| Lister les donneurs | GET | `/api/donors` |
| Consulter un donneur | GET | `/api/donors/{id}` |
| Créer un donneur | POST | `/api/donors` |
| Modifier un donneur | PUT | `/api/donors/{id}` |
| Supprimer un donneur | DELETE | `/api/donors/{id}` |
| Se connecter | POST | `/api/auth/login` |
| Consulter son profil | GET | `/api/users/me` |

**Travail demandé :**

Pour chaque endpoint, identifier :

1. Son objectif.
2. Les paramètres éventuels.
3. Le contenu de la requête.
4. Les réponses possibles.
5. La nécessité ou non d'une authentification.

**Remarque :** cette liste sert de support pédagogique. Adaptez les chemins, les méthodes et les règles d'accès à ceux réellement implémentés dans votre API SangConnect.

---

## 3. Installer les outils Swagger

Nous allons utiliser deux bibliothèques :

- `swagger-jsdoc` : génère une spécification OpenAPI à partir des commentaires présents dans les fichiers JavaScript.
- `swagger-ui-express` : expose une interface Swagger UI dans l'application Express.

### Question 3 — Installer les dépendances

**Travail demandé :**

À la racine du projet SangConnect :

1. Installer les deux bibliothèques.
2. Vérifier leur présence dans `package.json`.
3. Identifier leur rôle respectif.

**Solution :**

```bash
npm install swagger-jsdoc swagger-ui-express
```

Vérifier les dépendances :

```bash
npm list swagger-jsdoc swagger-ui-express
```

Documentation :

- swagger-jsdoc : https://github.com/Surnet/swagger-jsdoc
- swagger-ui-express : https://github.com/scottie1984/swagger-ui-express

---

## 4. Configurer Swagger dans Express

Nous allons créer une configuration centralisée de la documentation.

### Question 4 — Créer la configuration OpenAPI

**Travail demandé :**

1. Créer un dossier `src/config`.
2. Créer un fichier `swagger.js`.
3. Importer `swagger-jsdoc`.
4. Définir les informations générales de l'API SangConnect.
5. Définir sa version OpenAPI.
6. Déclarer le serveur de développement.
7. Indiquer les fichiers contenant les annotations à analyser.

**Solution :**

Créer `src/config/swagger.js` :

```javascript
import swaggerJsdoc from "swagger-jsdoc";

const options = {
    definition: {
        openapi: "3.0.3",

        info: {
            title: "SangConnect API",
            version: "1.0.0",
            description: `
                API REST de gestion des dons de sang.

                Cette API permet de gérer les donneurs,
                les utilisateurs, les centres de collecte
                et les opérations liées aux dons de sang.
            `
        },

        servers: [
            {
                url: "http://localhost:3000",
                description: "Serveur de développement"
            }
        ],

        tags: [
            {
                name: "Donors",
                description: "Gestion des donneurs"
            },
            {
                name: "Authentication",
                description: "Authentification des utilisateurs"
            },
            {
                name: "Users",
                description: "Gestion des utilisateurs"
            }
        ]
    },

    apis: ["./src/routes/*.js"]
};

const swaggerSpec = swaggerJsdoc(options);

export default swaggerSpec;
```

**Explication :**

| Propriété | Rôle |
|---|---|
| `openapi` | Version de la spécification OpenAPI |
| `info` | Informations générales sur l'API |
| `servers` | Adresses des serveurs |
| `tags` | Organisation des endpoints |
| `apis` | Fichiers contenant les annotations OpenAPI |

Le chemin indiqué dans `apis` doit correspondre à l'organisation réelle du projet et au répertoire depuis lequel l'application est lancée.

---

### Question 5 — Ajouter Swagger UI à Express

**Travail demandé :**

1. Importer `swagger-ui-express`.
2. Importer la spécification générée.
3. Ajouter une route `/api-docs`.
4. Démarrer l'application.
5. Accéder à la documentation depuis le navigateur.

**Solution :**

Dans `src/app.js`, ajouter :

```javascript
import swaggerUi from "swagger-ui-express";
import swaggerSpec from "./config/swagger.js";

// Configuration des routes et middlewares existants...

app.use(
    "/api-docs",
    swaggerUi.serve,
    swaggerUi.setup(swaggerSpec)
);
```

Démarrer l'application :

```bash
npm run dev
```

Ouvrir le navigateur :

```text
http://localhost:3000/api-docs
```

<text>Résultat attendu :</text>

L'interface Swagger UI apparaît avec le titre SangConnect API et les informations générales de la documentation.

<text>À retenir :</text>

À ce stade, Swagger UI est installé, mais aucun endpoint n'est encore documenté. Nous allons maintenant décrire les routes de notre API.

---

## 5. Documenter un premier endpoint GET

Nous allons commencer par la consultation de la liste des donneurs.

On suppose que SangConnect possède une route :

```http
GET /api/donors
```

### Question 6 — Documenter la liste des donneurs

**Travail demandé :**

1. Ouvrir le fichier contenant les routes des donneurs.
2. Ajouter une annotation OpenAPI au-dessus de la route GET.
3. Définir un résumé et une description.
4. Associer la route à la catégorie `Donors`.
5. Décrire la réponse HTTP 200.
6. Décrire le contenu JSON retourné.

**Solution :**

Exemple dans `src/routes/donor.routes.js` :

```javascript
/**
 * @openapi
 * /api/donors:
 *   get:
 *     summary: Lister les donneurs
 *     description: Retourne la liste des donneurs enregistrés.
 *     tags:
 *       - Donors
 *     responses:
 *       200:
 *         description: Liste des donneurs récupérée avec succès.
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Donor'
 */
router.get("/", donorController.getAll);
```

**Explication :**

- `@openapi` indique le début d'une annotation.
- `/api/donors` correspond au chemin de l'endpoint.
- `get` correspond à la méthode HTTP.
- `summary` décrit brièvement la fonctionnalité.
- `tags` classe l'endpoint.
- `responses` décrit les réponses possibles.
- `$ref` permet de réutiliser un schéma défini ailleurs.

**Important :** le chemin OpenAPI doit correspondre au chemin HTTP complet. Si le routeur est monté avec `app.use("/api/donors", donorRoutes)`, l'annotation doit utiliser `/api/donors`, même si la route Express interne est simplement `router.get("/")`.

---

## 6. Définir les schémas de données

Les schémas permettent de décrire précisément la structure des objets JSON échangés avec l'API.

Au lieu de répéter les mêmes propriétés dans chaque endpoint, nous allons définir des schémas réutilisables.

### Question 7 — Décrire un donneur

On suppose qu'un donneur possède les propriétés suivantes :

- `id`
- `firstName`
- `lastName`
- `email`
- `bloodGroup`
- `phone`

**Travail demandé :**

1. Ajouter une section `components` dans la configuration OpenAPI.
2. Définir un schéma `Donor`.
3. Définir le type de chaque propriété.
4. Identifier les propriétés obligatoires.
5. Ajouter une description aux propriétés.

**Solution :**

Dans `src/config/swagger.js`, compléter la propriété `definition` :

```javascript
components: {
    schemas: {

        Donor: {
            type: "object",

            properties: {
                id: {
                    type: "integer",
                    example: 1
                },

                firstName: {
                    type: "string",
                    example: "Ahmed"
                },

                lastName: {
                    type: "string",
                    example: "Ben Ali"
                },

                email: {
                    type: "string",
                    format: "email",
                    example: "ahmed@example.com"
                },

                bloodGroup: {
                    type: "string",
                    example: "A+"
                },

                phone: {
                    type: "string",
                    example: "+21612345678"
                }
            },

            required: [
                "firstName",
                "lastName",
                "email",
                "bloodGroup"
            ]
        }

    }
}
```

Les propriétés et les types sont des exemples pédagogiques : vérifiez leur correspondance avec le modèle réellement utilisé dans SangConnect.

### Question 8 — Créer un schéma spécifique à la création

Lorsqu'un client crée un donneur, il ne doit pas envoyer l'identifiant `id`, puisque celui-ci est généralement généré par la base de données.

**Travail demandé :**

1. Créer un schéma `CreateDonor`.
2. Définir uniquement les propriétés attendues lors de la création.
3. Réutiliser les propriétés du schéma `Donor` lorsque cela est pertinent.
4. Définir les propriétés obligatoires.

**Solution :**

Ajouter dans `components.schemas` :

```javascript
CreateDonor: {
    type: "object",

    properties: {
        firstName: {
            type: "string",
            example: "Ahmed"
        },

        lastName: {
            type: "string",
            example: "Ben Ali"
        },

        email: {
            type: "string",
            format: "email",
            example: "ahmed@example.com"
        },

        bloodGroup: {
            type: "string",
            example: "A+"
        },

        phone: {
            type: "string",
            example: "+21612345678"
        }
    },

    required: [
        "firstName",
        "lastName",
        "email",
        "bloodGroup"
    ]
}
```

**Question de réflexion :**

Pourquoi est-il préférable de distinguer le schéma d'un donneur complet du schéma utilisé pour créer un donneur ?

---

## 7. Documenter une requête POST

Nous allons maintenant documenter la création d'un donneur.

### Question 9 — Décrire le endpoint POST

On suppose que l'API expose :

```http
POST /api/donors
```

**Travail demandé :**

1. Décrire la fonctionnalité.
2. Ajouter le tag `Donors`.
3. Décrire le corps de la requête avec `requestBody`.
4. Utiliser le schéma `CreateDonor`.
5. Documenter les réponses 201, 400 et 409.

**Solution :**

Dans le fichier des routes des donneurs :

```javascript
/**
 * @openapi
 * /api/donors:
 *   post:
 *     summary: Créer un donneur
 *     description: Enregistre un nouveau donneur dans SangConnect.
 *     tags:
 *       - Donors
 *
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/CreateDonor'
 *
 *     responses:
 *       201:
 *         description: Donneur créé avec succès.
 *
 *       400:
 *         description: Données invalides.
 *
 *       409:
 *         description: Un donneur avec cet email existe déjà.
 */
router.post("/", donorController.create);
```

**Explication :**

`requestBody` décrit les données envoyées dans le corps de la requête.

`required: true` signifie que le corps de la requête est obligatoire.

`content` précise le format utilisé.

`application/json` indique que le client doit envoyer du JSON.

**Remarque :** les codes HTTP doivent correspondre au comportement réel de votre contrôleur et de votre middleware de gestion des erreurs. Ne documentez pas un code 409 si l'API ne le retourne pas.

---

## 8. Documenter les paramètres d'URL

Les routes REST utilisent fréquemment des paramètres pour identifier une ressource.

Exemple :

```http
GET /api/donors/12
```

Le nombre `12` représente l'identifiant du donneur.

### Question 10 — Documenter la consultation d'un donneur

**Travail demandé :**

1. Documenter `GET /api/donors/{id}`.
2. Déclarer le paramètre `id`.
3. Indiquer son emplacement dans l'URL.
4. Définir son type.
5. Documenter les réponses 200 et 404.

**Solution :**

```javascript
/**
 * @openapi
 * /api/donors/{id}:
 *   get:
 *     summary: Consulter un donneur
 *     tags:
 *       - Donors
 *
 *     parameters:
 *       - name: id
 *         in: path
 *         required: true
 *         description: Identifiant du donneur.
 *         schema:
 *           type: integer
 *         example: 12
 *
 *     responses:
 *       200:
 *         description: Donneur trouvé.
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Donor'
 *
 *       404:
 *         description: Donneur introuvable.
 */
router.get("/:id", donorController.getById);
```

**À retenir :**

La propriété `in` permet de préciser où se trouve le paramètre :

| Valeur | Emplacement |
|---|---|
| `path` | Dans le chemin URL |
| `query` | Dans la chaîne de requête |
| `header` | Dans les en-têtes HTTP |
| `cookie` | Dans les cookies |

---

## 9. Documenter les paramètres de recherche

Une API peut proposer des filtres pour faciliter la recherche.

Exemple :

```http
GET /api/donors?bloodGroup=A+&page=1&limit=10
```

### Question 11 — Documenter les filtres

**Travail demandé :**

Documenter les paramètres suivants :

- `bloodGroup` : groupe sanguin recherché.
- `page` : numéro de page.
- `limit` : nombre de résultats par page.

Définir leur emplacement, leur type et leur caractère obligatoire ou facultatif.

**Solution :**

Exemple d'annotation :

```javascript
/**
 * @openapi
 * /api/donors:
 *   get:
 *     summary: Lister et filtrer les donneurs
 *     tags:
 *       - Donors
 *
 *     parameters:
 *       - name: bloodGroup
 *         in: query
 *         required: false
 *         description: Filtrer par groupe sanguin.
 *         schema:
 *           type: string
 *         example: A+
 *
 *       - name: page
 *         in: query
 *         required: false
 *         description: Numéro de la page.
 *         schema:
 *           type: integer
 *           default: 1
 *
 *       - name: limit
 *         in: query
 *         required: false
 *         description: Nombre de résultats par page.
 *         schema:
 *           type: integer
 *           default: 10
 *
 *     responses:
 *       200:
 *         description: Résultats de la recherche.
 */
```

**Attention :** cette annotation suppose que les filtres et la pagination sont effectivement implémentés dans le contrôleur. La documentation doit décrire le fonctionnement réel de l'API.

---

## 10. Documenter l'authentification JWT

Dans l'atelier 8, nous avons mis en place l'authentification avec JWT.

Un utilisateur authentifié peut accéder à certaines routes protégées.

Il faut donc expliquer à Swagger comment utiliser un token Bearer.

### Question 12 — Déclarer le mécanisme de sécurité

**Travail demandé :**

1. Ajouter une section `securitySchemes`.
2. Définir un mécanisme HTTP de type Bearer.
3. Préciser que le token utilise JWT.
4. Déclarer une règle de sécurité réutilisable.

**Solution :**

Dans `src/config/swagger.js`, compléter `components` :

```javascript
components: {
    securitySchemes: {

        bearerAuth: {
            type: "http",
            scheme: "bearer",
            bearerFormat: "JWT",
            description: "Saisir le token JWT obtenu après authentification."
        }

    }
}
```

Ajouter ensuite la règle de sécurité globale dans `definition` :

```javascript
security: [
    {
        bearerAuth: []
    }
]
```

**Explication :**

- `type: http` indique un mécanisme d'authentification HTTP.
- `scheme: bearer` indique que le token est transmis avec le mécanisme Bearer.
- `bearerFormat: JWT` précise le format du token.

Dans Swagger UI, un bouton `Authorize` apparaît. Il permet de saisir un token JWT.

### Question 13 — Protéger uniquement les endpoints concernés

Toutes les routes de SangConnect ne nécessitent pas forcément une authentification.

Par exemple :

- La connexion est publique.
- La création d'un compte peut être publique.
- La consultation du profil nécessite une authentification.
- La consultation de certaines données administratives nécessite un rôle particulier.

**Travail demandé :**

1. Retirer la sécurité globale si elle ne correspond pas à toutes les routes.
2. Ajouter `security` uniquement aux endpoints protégés.
3. Documenter les réponses 401 et 403.

**Solution :**

Exemple pour consulter le profil :

```javascript
/**
 * @openapi
 * /api/users/me:
 *   get:
 *     summary: Consulter son profil
 *     tags:
 *       - Users
 *
 *     security:
 *       - bearerAuth: []
 *
 *     responses:
 *       200:
 *         description: Profil récupéré avec succès.
 *
 *       401:
 *         description: Authentification requise ou token invalide.
 *
 *       403:
 *         description: Accès interdit.
 */
```

**Important :**

La déclaration OpenAPI ne sécurise pas réellement l'endpoint. Elle décrit les mécanismes de sécurité attendus par l'API.

La protection effective doit toujours être assurée par les middlewares Express et les contrôles de rôles déjà implémentés.

---

## 11. Documenter la connexion

### Question 14 — Documenter le endpoint de connexion

On suppose que l'API possède :

```http
POST /api/auth/login
```

Le client transmet une adresse e-mail et un mot de passe.

**Travail demandé :**

1. Décrire le corps de la requête.
2. Créer un schéma `LoginRequest`.
3. Documenter la réponse en cas de connexion réussie.
4. Documenter les erreurs 400 et 401.
5. Décrire la réponse contenant le token si votre implémentation retourne un token dans le JSON.

**Solution :**

Dans `components.schemas` :

```javascript
LoginRequest: {
    type: "object",
    properties: {
        email: {
            type: "string",
            format: "email",
            example: "admin@sangconnect.tn"
        },
        password: {
            type: "string",
            format: "password",
            example: "MotDePasseExemple"
        }
    },
    required: ["email", "password"]
},

LoginResponse: {
    type: "object",
    properties: {
        accessToken: {
            type: "string",
            description: "Token JWT d'accès."
        },
        tokenType: {
            type: "string",
            example: "Bearer"
        }
    }
}
```

Dans les routes d'authentification :

```javascript
/**
 * @openapi
 * /api/auth/login:
 *   post:
 *     summary: Authentifier un utilisateur
 *     tags:
 *       - Authentication
 *
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/LoginRequest'
 *
 *     responses:
 *       200:
 *         description: Authentification réussie.
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/LoginResponse'
 *
 *       400:
 *         description: Données de connexion invalides.
 *
 *       401:
 *         description: Identifiants incorrects.
 */
```

**Attention :**

Dans l'atelier 9, SangConnect utilise également des refresh tokens et des cookies sécurisés.

Si votre endpoint de connexion place le refresh token dans un cookie `HttpOnly`, documentez aussi la réponse correspondante avec un en-tête `Set-Cookie`. Ne documentez pas un refresh token dans le corps JSON s'il n'est pas réellement retourné par votre application.

---

## 12. Documenter les réponses et les erreurs

Une documentation professionnelle doit décrire les erreurs que le client peut rencontrer.

### Question 15 — Définir un schéma d'erreur

**Travail demandé :**

Créer un schéma réutilisable `ErrorResponse` contenant :

- Un message d'erreur.
- Un code ou identifiant d'erreur éventuel.
- Les détails de validation éventuels.

**Solution :**

Dans `components.schemas` :

```javascript
ErrorResponse: {
    type: "object",

    properties: {
        message: {
            type: "string",
            example: "Donneur introuvable."
        },

        code: {
            type: "string",
            example: "DONOR_NOT_FOUND"
        },

        details: {
            type: "array",
            items: {
                type: "string"
            }
        }
    },

    required: ["message"]
}
```

Exemple de documentation d'une réponse 404 :

```yaml
404:
  description: Ressource introuvable.
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/ErrorResponse'
```

**Question de réflexion :**

Pourquoi est-il intéressant de définir un format de réponse d'erreur commun à toute l'API ?

---

## 13. Tester l'API depuis Swagger UI

Nous avons maintenant documenté plusieurs endpoints.

Il est temps de vérifier que la documentation est exploitable.

### Question 16 — Tester les endpoints

**Travail demandé :**

1. Ouvrir `http://localhost:3000/api-docs`.
2. Développer la catégorie `Donors`.
3. Sélectionner `GET /api/donors`.
4. Cliquer sur `Try it out`.
5. Cliquer sur `Execute`.
6. Observer la requête envoyée.
7. Observer le code HTTP et la réponse JSON.

Réaliser ensuite les mêmes opérations avec :

- `POST /api/donors`
- `GET /api/donors/{id}`
- `POST /api/auth/login`
- `GET /api/users/me`

**Question de réflexion :**

Quelle est la différence entre une documentation qui décrit simplement les endpoints et une documentation qui permet de les tester directement ?

### Question 17 — Tester une route protégée

**Travail demandé :**

1. Ouvrir la documentation Swagger UI.
2. Cliquer sur `Authorize`.
3. Saisir un token JWT valide.
4. Tester une route protégée.
5. Tester la même route sans token.
6. Comparer les réponses.

**Solution attendue :**

Swagger UI doit ajouter automatiquement l'en-tête :

```http
Authorization: Bearer VOTRE_TOKEN
```

Pour les routes protégées, le comportement doit correspondre à celui observé avec Postman ou cURL.

---

## 14. Finaliser la documentation de SangConnect

Vous devez maintenant compléter la documentation de l'ensemble de votre API.

### Question 18 — Documenter les autres ressources

**Travail demandé :**

En vous basant sur les endpoints réellement présents dans votre projet, documentez les ressources suivantes lorsqu'elles existent :

- Les utilisateurs.
- Les centres de collecte.
- Les rendez-vous.
- Les dons de sang.
- Les statistiques.
- Les opérations administratives.

Pour chaque endpoint, documentez obligatoirement :

1. La méthode HTTP.
2. Le chemin.
3. Le résumé.
4. La description.
5. Les paramètres éventuels.
6. Le corps de la requête éventuel.
7. Les réponses possibles.
8. Les mécanismes d'authentification et d'autorisation.

**Exemple de structure attendue :**

```text
src/
│
├── config/
│   └── swagger.js
│
├── routes/
│   ├── donor.routes.js
│   ├── user.routes.js
│   ├── auth.routes.js
│   ├── center.routes.js
│   └── donation.routes.js
│
├── controllers/
├── services/
├── repositories/
├── middlewares/
└── app.js
```

Chaque fichier de routes contient les annotations OpenAPI correspondantes.

---

## 15. Vérifier la qualité de la documentation

Une documentation Swagger doit rester synchronisée avec le code.

### Question 19 — Réaliser une vérification complète

**Travail demandé :**

Vérifier les points suivants :

- Tous les endpoints existants sont documentés.
- Les chemins correspondent aux véritables routes Express.
- Les méthodes HTTP sont correctes.
- Les paramètres obligatoires sont identifiés.
- Les schémas correspondent aux objets réellement échangés.
- Les codes HTTP sont exacts.
- Les routes protégées sont identifiées.
- Les routes publiques ne sont pas présentées comme protégées.
- Les exemples JSON sont cohérents.
- Aucun mot de passe ou secret réel n'est présent dans la documentation.

**Question de réflexion :**

Que se passe-t-il si un développeur modifie une route Express sans mettre à jour sa documentation Swagger ?

---

## 16. Livrables attendus

À la fin de l'atelier, chaque binôme doit fournir :

1. Le fichier `src/config/swagger.js`.
2. Les annotations OpenAPI dans les fichiers de routes.
3. Les schémas des principales ressources.
4. La documentation des endpoints d'authentification.
5. La documentation des routes protégées avec JWT.
6. Une capture de l'interface Swagger UI.
7. Une documentation complète des principales fonctionnalités de SangConnect.
8. Une démonstration de deux endpoints testés depuis Swagger UI, dont un endpoint protégé.

---

## 17. Questions de compréhension

1. Quelle est la différence entre Swagger et OpenAPI ?
2. Quel est le rôle de swagger-jsdoc ?
3. Quel est le rôle de swagger-ui-express ?
4. À quoi sert la propriété `requestBody` ?
5. Quelle est la différence entre un paramètre `path` et un paramètre `query` ?
6. Quel est le rôle de `components.schemas` ?
7. Pourquoi utilise-t-on `$ref` ?
8. À quoi sert `securitySchemes` ?
9. Pourquoi la déclaration JWT dans Swagger ne remplace-t-elle pas le middleware d'authentification Express ?
10. Pourquoi faut-il documenter les erreurs HTTP ?
11. Quelle est la différence entre une documentation générée automatiquement à partir des annotations et une documentation écrite manuellement ?
12. Pourquoi faut-il maintenir la documentation à jour avec le code ?

---

## 18. Travail à domicile

**Sujet : produire la documentation complète de SangConnect**

Votre travail consiste à terminer la documentation de votre API.

### Partie A — Documentation fonctionnelle

Documenter toutes les ressources présentes dans votre projet.

Chaque endpoint doit posséder :

- Un résumé.
- Une description.
- Un tag.
- Ses paramètres.
- Son corps de requête éventuel.
- Ses réponses.
- Ses exigences de sécurité.

### Partie B — Schémas

Créer les schémas OpenAPI réutilisables pour les principales entités du projet.

Par exemple :

- `Donor`
- `CreateDonor`
- `User`
- `LoginRequest`
- `LoginResponse`
- `ErrorResponse`

Les schémas doivent correspondre aux structures réellement utilisées dans l'application.

### Partie C — Démonstration

Préparer une démonstration de 5 à 10 minutes présentant :

1. L'accès à Swagger UI.
2. La liste des endpoints documentés.
3. La création d'un donneur.
4. La consultation d'un donneur.
5. L'authentification JWT.
6. Le test d'un endpoint protégé.
7. L'affichage d'une erreur.

### Partie D — Documentation du projet

Ajouter dans le dépôt Git un fichier `API_DOCUMENTATION.md` indiquant :

- L'adresse de Swagger UI.
- Les principales catégories de l'API.
- La méthode d'authentification.
- Les conventions de réponse.
- Les consignes permettant de lancer la documentation en local.

---

## 19. Documentation officielle

- OpenAPI Specification : https://spec.openapis.org/oas/latest.html
- Swagger : https://swagger.io/
- Swagger UI : https://swagger.io/tools/swagger-ui/
- swagger-jsdoc : https://github.com/Surnet/swagger-jsdoc
- swagger-ui-express : https://github.com/scottie1984/swagger-ui-express
- Express.js : https://expressjs.com/

---

## Bilan de l'atelier

Au cours de cet atelier, vous avez appris à documenter une API REST avec OpenAPI et Swagger.

Vous savez désormais décrire les endpoints, les paramètres, les données échangées, les réponses HTTP et les mécanismes d'authentification.

La documentation Swagger constitue une interface de communication entre les développeurs backend, les développeurs frontend, les testeurs et les autres utilisateurs de l'API.

**SangConnect dispose maintenant d'une documentation interactive permettant de découvrir et de tester ses fonctionnalités.**
