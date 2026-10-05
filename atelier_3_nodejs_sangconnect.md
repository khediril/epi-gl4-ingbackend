# Atelier 3 — Concevoir et implémenter une API REST avec Node.js et Express

**Module :** Ingénierie Backend
**Technologies :** Node.js, Express.js, HTTP, JSON
**Projet fil rouge :** SangConnect — API de gestion des dons de sang
**Durée :** 3 heures
**Niveau :** 4ème année cycle ingénieur

---

# 1. Objectifs de l'atelier

À la fin de cet atelier, l'étudiant devra être capable de :

* comprendre les principes d'une API REST ;
* identifier les ressources d'une application ;
* associer les opérations CRUD aux méthodes HTTP ;
* concevoir des routes REST cohérentes ;
* utiliser les méthodes `GET`, `POST`, `PUT`, `PATCH` et `DELETE` ;
* récupérer des paramètres de route ;
* récupérer des paramètres de requête ;
* recevoir et traiter un corps JSON ;
* retourner des réponses JSON ;
* choisir correctement les codes HTTP ;
* distinguer les paramètres `params`, `query` et `body` ;
* tester une API avec `curl` ou un client HTTP ;
* organiser les routes, contrôleurs et services dans une architecture en couches.

---

# 2. Situation de départ

Dans les deux ateliers précédents, nous avons commencé la construction de **SangConnect**.

L'architecture obtenue est :

```text
Client HTTP
    │
    ▼
Routes
    │
    ▼
Controllers
    │
    ▼
Services
    │
    ▼
Données
```

Pour l'instant, les données des donneurs sont conservées temporairement en mémoire.

Exemple :

```javascript
const donneurs = [
    {
        id: 1,
        nom: "Ben Ali",
        prenom: "Ahmed",
        groupeSanguin: "A+"
    },
    {
        id: 2,
        nom: "Trabelsi",
        prenom: "Sami",
        groupeSanguin: "O+"
    }
];
```

Nous allons maintenant transformer cette organisation en une véritable **API REST**.

---

# 3. Rappel : qu'est-ce qu'une API REST ?

Une API REST permet à un client de communiquer avec un serveur en utilisant principalement le protocole HTTP.

Dans notre projet :

```text
Client
   │
   │ HTTP
   ▼
SangConnect API
   │
   ▼
Services
   │
   ▼
Données
```

Le client peut par exemple demander :

```http
GET /api/donneurs
```

Le serveur répond :

```json
[
    {
        "id": 1,
        "nom": "Ben Ali",
        "prenom": "Ahmed",
        "groupeSanguin": "A+"
    }
]
```

---

# 4. Les ressources de SangConnect

Une API REST est organisée autour de **ressources**.

Dans SangConnect, nous pouvons identifier :

* donneurs ;
* dons ;
* centres ;
* stocks ;
* utilisateurs.

Une ressource est généralement représentée par un nom au pluriel.

Exemples :

```text
/api/donneurs
/api/dons
/api/centres
/api/stocks
/api/utilisateurs
```

### Question 1

Pourquoi utilise-t-on :

```text
/api/donneurs
```

plutôt que :

```text
/api/getDonneurs
```

?

### Question 2

Quel serait le nom REST correspondant aux ressources suivantes ?

| Ressource    | URI |
| ------------ | --- |
| Donneurs     | ?   |
| Centres      | ?   |
| Dons         | ?   |
| Stocks       | ?   |
| Utilisateurs | ?   |

---

# 5. CRUD et méthodes HTTP

Une API REST utilise les méthodes HTTP pour représenter les opérations réalisées sur les ressources.

| Opération              | Méthode HTTP | Exemple           |
| ---------------------- | ------------ | ----------------- |
| Lire plusieurs         | GET          | `/api/donneurs`   |
| Lire un élément        | GET          | `/api/donneurs/5` |
| Créer                  | POST         | `/api/donneurs`   |
| Modifier complètement  | PUT          | `/api/donneurs/5` |
| Modifier partiellement | PATCH        | `/api/donneurs/5` |
| Supprimer              | DELETE       | `/api/donneurs/5` |

On obtient donc :

```text
GET       /api/donneurs
GET       /api/donneurs/:id
POST      /api/donneurs
PUT       /api/donneurs/:id
PATCH     /api/donneurs/:id
DELETE    /api/donneurs/:id
```

---

# 6. Activité 1 — Observer l'API actuelle

## 6.1 Démarrer le serveur

Depuis le projet :

```bash
node --watch src/server.js
```

Tester :

```bash
curl http://localhost:3000/api/donneurs
```

Observer la réponse.

---

## 6.2 Examiner la route

Dans le fichier :

```text
src/routes/donneur.routes.js
```

identifier la route permettant de récupérer les donneurs.

Elle doit être proche de :

```javascript
router.get("/donneurs", getDonneurs);
```

### Question 3

Quelle méthode HTTP est utilisée ?

### Question 4

Quelle est l'URI finale ?

### Question 5

Quel contrôleur est exécuté ?

### Question 6

Quelle couche contient la logique de récupération des donneurs ?

---

# 7. Activité 2 — Récupérer un donneur

Nous voulons maintenant récupérer un donneur particulier.

La requête sera :

```http
GET /api/donneurs/1
```

Dans Express, on utilise un paramètre de route :

```javascript
router.get("/donneurs/:id", getDonneurById);
```

Le paramètre peut être récupéré dans le contrôleur avec :

```javascript
req.params.id
```

Exemple :

```javascript
export async function getDonneurById(req, res) {

    const id = Number(req.params.id);

    // ...
}
```

---

## 7.1 Tester

```bash
curl http://localhost:3000/api/donneurs/1
```

Puis :

```bash
curl http://localhost:3000/api/donneurs/999
```

### Question 7

Quelle différence doit-il y avoir entre les deux réponses ?

### Question 8

Quel code HTTP doit être utilisé lorsque le donneur existe ?

### Question 9

Quel code HTTP doit être utilisé lorsque le donneur n'existe pas ?

---

# 8. Activité 3 — Comprendre les paramètres de route

Considérons :

```http
GET /api/donneurs/15
```

et :

```javascript
router.get("/donneurs/:id", getDonneurById);
```

Dans le contrôleur :

```javascript
console.log(req.params);
```

Observer le résultat.

On obtient :

```javascript
{
    id: "15"
}
```

### Attention

Les paramètres de route reçus par Express sont des chaînes de caractères.

Il faut donc effectuer une conversion lorsque l'on souhaite manipuler un nombre :

```javascript
const id = Number(req.params.id);
```

### Question 10

Pourquoi est-il nécessaire de convertir :

```javascript
req.params.id
```

en nombre ?

---

# 9. Activité 4 — Les paramètres de requête

Les paramètres de requête permettent notamment de filtrer ou rechercher des ressources.

Exemple :

```http
GET /api/donneurs?groupeSanguin=A+
```

Dans Express :

```javascript
req.query
```

permet de récupérer ces paramètres.

Exemple :

```javascript
const groupe = req.query.groupeSanguin;
```

---

## 9.1 Implémenter un filtre

Modifier le service des donneurs afin de pouvoir rechercher les donneurs appartenant à un groupe sanguin donné.

Exemple :

```http
GET /api/donneurs?groupeSanguin=A+
```

Le serveur devra retourner uniquement les donneurs :

```text
groupeSanguin = A+
```

---

## 9.2 Plusieurs paramètres

Nous pouvons également utiliser :

```http
GET /api/donneurs?groupeSanguin=A+&nom=Ben
```

Dans le contrôleur :

```javascript
const { groupeSanguin, nom } = req.query;
```

### Question 11

Quelle est la différence entre :

```javascript
req.params
```

et :

```javascript
req.query
```

?

---

# 10. Activité 5 — Concevoir l'endpoint POST

Nous allons maintenant permettre la création d'un donneur.

La requête sera :

```http
POST /api/donneurs
```

Le client transmettra les données dans le corps de la requête.

Exemple JSON :

```json
{
    "nom": "Khelifi",
    "prenom": "Ali",
    "groupeSanguin": "B+"
}
```

---

# 11. Activité 6 — Recevoir du JSON avec Express

Pour permettre à Express de lire automatiquement les requêtes JSON, le serveur doit utiliser :

```javascript
app.use(express.json());
```

Cette instruction doit être placée avant les routes.

Exemple :

```javascript
const app = express();

app.use(express.json());

app.use("/api", donneurRoutes);
```

---

## 11.1 Observer `req.body`

Créer temporairement une route :

```javascript
router.post("/donneurs", (req, res) => {

    console.log(req.body);

    res.json(req.body);
});
```

Tester avec :

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{"nom":"Khelifi","prenom":"Ali","groupeSanguin":"B+"}'
```

Observer le résultat.

---

# 12. Activité 7 — Créer réellement un donneur

Créer dans le service :

```javascript
export function createDonneur(data) {
    // ...
}
```

Le service devra :

1. générer un identifiant ;
2. créer le donneur ;
3. ajouter le donneur au tableau ;
4. retourner le donneur créé.

Exemple :

```javascript
{
    id: 3,
    nom: "Khelifi",
    prenom: "Ali",
    groupeSanguin: "B+"
}
```

---

# 13. Activité 8 — Créer le contrôleur POST

Dans :

```text
src/controllers/donneur.controller.js
```

ajouter :

```javascript
export async function createDonneur(req, res) {

    const donneur = createDonneurService(req.body);

    res.status(201).json(donneur);
}
```

Adapter le nom de la fonction importée afin d'éviter une collision entre le contrôleur et le service.

---

# 14. Le code HTTP 201

Lorsqu'une ressource est créée avec succès, l'API doit généralement retourner :

```http
201 Created
```

et non :

```http
200 OK
```

Exemple :

```javascript
res.status(201).json(donneur);
```

### Question 12

Pourquoi est-il préférable d'utiliser `201` pour une création ?

---

# 15. Activité 9 — Tester POST

Tester :

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{"nom":"Khelifi","prenom":"Ali","groupeSanguin":"B+"}'
```

Puis :

```bash
curl http://localhost:3000/api/donneurs
```

Vérifier que le nouveau donneur apparaît dans la liste.

---

# 16. Activité 10 — Gérer une requête invalide

Que se passe-t-il si le client envoie :

```json
{
    "nom": "Ali"
}
```

Le champ :

```text
prenom
```

est absent.

De même, le groupe sanguin peut être absent :

```json
{
    "nom": "Ali",
    "prenom": "Ahmed"
}
```

Nous devons donc contrôler les données reçues.

---

## 16.1 Première validation manuelle

Dans le contrôleur :

```javascript
if (!nom || !prenom || !groupeSanguin) {
    return res.status(400).json({
        message: "Données invalides"
    });
}
```

### Question 13

Pourquoi utilise-t-on le code :

```http
400 Bad Request
```

?

---

# 17. Activité 11 — Séparer validation et logique métier

Le contrôleur commence progressivement à devenir complexe.

Exemple :

```javascript
export async function createDonneur(req, res) {

    const { nom, prenom, groupeSanguin } = req.body;

    if (!nom || !prenom || !groupeSanguin) {
        return res.status(400).json({
            message: "Données invalides"
        });
    }

    // logique métier...

    // création...

    // réponse HTTP...
}
```

### Question 14

Le contrôleur doit-il être responsable de tout ?

Identifier les responsabilités suivantes :

| Responsabilité       | Couche |
| -------------------- | ------ |
| Lire `req.body`      | ?      |
| Vérifier les données | ?      |
| Créer le donneur     | ?      |
| Accéder aux données  | ?      |
| Choisir le code HTTP | ?      |

---

# 18. Activité 12 — Implémenter DELETE

Nous voulons maintenant supprimer un donneur.

La route sera :

```http
DELETE /api/donneurs/:id
```

Exemple :

```http
DELETE /api/donneurs/3
```

Dans Express :

```javascript
router.delete("/donneurs/:id", deleteDonneur);
```

Le service devra :

1. rechercher le donneur ;
2. vérifier qu'il existe ;
3. le supprimer ;
4. retourner une information permettant au contrôleur de construire la réponse.

---

# 19. Choisir le code HTTP pour DELETE

Deux situations sont possibles.

### Donneur trouvé

```http
204 No Content
```

Le serveur peut répondre :

```javascript
res.status(204).send();
```

### Donneur inexistant

```http
404 Not Found
```

Exemple :

```json
{
    "message": "Donneur introuvable"
}
```

---

# 20. Activité 13 — Implémenter PUT

Nous souhaitons permettre la modification complète d'un donneur.

Requête :

```http
PUT /api/donneurs/2
```

Corps :

```json
{
    "nom": "Trabelsi",
    "prenom": "Sami",
    "groupeSanguin": "O-"
}
```

Créer :

```javascript
router.put("/donneurs/:id", updateDonneur);
```

Le contrôleur devra récupérer :

```javascript
req.params.id
```

et :

```javascript
req.body
```

---

# 21. PUT versus PATCH

Il est important de distinguer :

### PUT

Représente généralement une modification complète de la ressource.

```http
PUT /api/donneurs/2
```

```json
{
    "nom": "Trabelsi",
    "prenom": "Sami",
    "groupeSanguin": "O-"
}
```

### PATCH

Permet une modification partielle.

```http
PATCH /api/donneurs/2
```

Par exemple :

```json
{
    "groupeSanguin": "O+"
}
```

---

# 22. Activité 14 — Implémenter PATCH

Ajouter :

```javascript
router.patch("/donneurs/:id", patchDonneur);
```

Tester :

```bash
curl -X PATCH http://localhost:3000/api/donneurs/2 \
-H "Content-Type: application/json" \
-d '{"groupeSanguin":"O+"}'
```

Vérifier :

```bash
curl http://localhost:3000/api/donneurs/2
```

---

# 23. Activité 15 — Construire la table des endpoints

Compléter la table suivante.

| Méthode | URI                 | Fonction |
| ------- | ------------------- | -------- |
| GET     | `/api/donneurs`     | Lister   |
| GET     | `/api/donneurs/:id` | ?        |
| POST    | `/api/donneurs`     | ?        |
| PUT     | `/api/donneurs/:id` | ?        |
| PATCH   | `/api/donneurs/:id` | ?        |
| DELETE  | `/api/donneurs/:id` | ?        |

Puis compléter :

| Situation                        | Code HTTP |
| -------------------------------- | --------: |
| Requête réussie                  |         ? |
| Ressource créée                  |         ? |
| Ressource inexistante            |         ? |
| Données invalides                |         ? |
| Suppression réussie sans contenu |         ? |

---

# 24. Activité 16 — Vérifier l'architecture

À ce stade, l'organisation du projet doit ressembler à :

```text
sangconnect/
│
├── package.json
│
└── src/
    │
    ├── server.js
    │
    ├── routes/
    │   ├── info.routes.js
    │   └── donneur.routes.js
    │
    ├── controllers/
    │   ├── info.controller.js
    │   └── donneur.controller.js
    │
    └── services/
        ├── info.service.js
        └── donneur.service.js
```

Le flux d'une requête doit être :

```text
                     HTTP Request
                          │
                          ▼
                    ┌──────────┐
                    │  Route   │
                    └────┬─────┘
                         │
                         ▼
                  ┌─────────────┐
                  │ Controller  │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │   Service   │
                  └──────┬──────┘
                         │
                         ▼
                       Data
```

---

# 25. Activité 17 — Identifier les erreurs d'architecture

Considérer le code suivant :

```javascript
router.post("/donneurs", (req, res) => {

    const { nom, prenom, groupeSanguin } = req.body;

    if (!nom || !prenom || !groupeSanguin) {
        return res.status(400).json({
            message: "Données invalides"
        });
    }

    donneurs.push({
        id: donneurs.length + 1,
        nom,
        prenom,
        groupeSanguin
    });

    res.status(201).json(donneurs[donneurs.length - 1]);
});
```

### Questions

1. Où se trouve la validation ?
2. Où se trouve la logique métier ?
3. Où se trouve l'accès aux données ?
4. Pourquoi ce code peut-il devenir difficile à maintenir ?
5. Comment le répartir entre Route, Controller et Service ?

---

# 26. Activité 18 — Tester l'API avec curl

Réaliser une campagne de tests.

### Lister

```bash
curl http://localhost:3000/api/donneurs
```

### Consulter

```bash
curl http://localhost:3000/api/donneurs/1
```

### Consulter un donneur inexistant

```bash
curl http://localhost:3000/api/donneurs/999
```

### Filtrer

```bash
curl "http://localhost:3000/api/donneurs?groupeSanguin=A+"
```

### Créer

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{"nom":"Khelifi","prenom":"Ali","groupeSanguin":"B+"}'
```

### Modifier

```bash
curl -X PATCH http://localhost:3000/api/donneurs/1 \
-H "Content-Type: application/json" \
-d '{"groupeSanguin":"O+"}'
```

### Supprimer

```bash
curl -X DELETE http://localhost:3000/api/donneurs/1
```

---

# 27. Activité 19 — Concevoir l'API des centres

Nous allons appliquer les mêmes principes à la ressource `Centre`.

Un centre contient :

```javascript
{
    id: 1,
    nom: "Centre régional de transfusion",
    ville: "Sousse"
}
```

Concevoir les endpoints :

```text
GET     /api/centres
GET     /api/centres/:id
POST    /api/centres
PUT     /api/centres/:id
PATCH   /api/centres/:id
DELETE  /api/centres/:id
```

### Travail demandé

Compléter :

| Endpoint           | Méthode | Rôle |
| ------------------ | ------- | ---- |
| `/api/centres`     | GET     | ?    |
| `/api/centres`     | POST    | ?    |
| `/api/centres/:id` | GET     | ?    |
| `/api/centres/:id` | PUT     | ?    |
| `/api/centres/:id` | PATCH   | ?    |
| `/api/centres/:id` | DELETE  | ?    |

---

# 28. Synthèse de l'atelier

Nous avons construit progressivement une API REST.

Les notions essentielles à retenir sont :

## Ressource

Une ressource représente un élément manipulé par l'API :

```text
donneurs
centres
dons
stocks
```

## URI

Une URI identifie une ressource :

```text
/api/donneurs
/api/donneurs/5
```

## Méthodes HTTP

```text
GET       → consulter
POST      → créer
PUT       → remplacer/modifier complètement
PATCH     → modifier partiellement
DELETE    → supprimer
```

## Paramètres

### Paramètres de route

```javascript
req.params
```

Exemple :

```text
/api/donneurs/5
```

### Paramètres de requête

```javascript
req.query
```

Exemple :

```text
/api/donneurs?groupeSanguin=A+
```

### Corps de requête

```javascript
req.body
```

Exemple :

```json
{
    "nom": "Ali",
    "prenom": "Ahmed"
}
```

---

# 29. Les principaux codes HTTP rencontrés

| Code | Signification         | Exemple               |
| ---: | --------------------- | --------------------- |
|  200 | OK                    | récupération réussie  |
|  201 | Created               | création réussie      |
|  204 | No Content            | suppression réussie   |
|  400 | Bad Request           | données invalides     |
|  404 | Not Found             | ressource inexistante |
|  500 | Internal Server Error | erreur serveur        |

---

# 30. Architecture obtenue

À la fin de l'atelier, l'étudiant doit comprendre le chemin suivant :

```text
                       Client
                         │
                         │ HTTP
                         ▼
                    ┌─────────┐
                    │ Express │
                    └────┬────┘
                         │
                         ▼
                      Routes
                         │
                         ▼
                    Controllers
                         │
                         ▼
                      Services
                         │
                         ▼
                       Data
```

Cette architecture sera conservée pour les prochains ateliers.

---

# 31. Travail pratique à faire à la maison

## Sujet — Compléter l'API SangConnect

Développer entièrement l'API REST de la ressource **Centre**.

Chaque centre possède :

```javascript
{
    id,
    nom,
    ville,
    adresse
}
```

Implémenter :

```text
GET     /api/centres
GET     /api/centres/:id
POST    /api/centres
PUT     /api/centres/:id
PATCH   /api/centres/:id
DELETE  /api/centres/:id
```

### Contraintes

L'application doit :

1. respecter l'architecture :

```text
routes
   ↓
controllers
   ↓
services
   ↓
data
```

2. retourner les codes HTTP appropriés ;
3. gérer le cas d'une ressource inexistante ;
4. vérifier les données minimales lors de la création ;
5. accepter les données JSON ;
6. permettre la modification partielle avec `PATCH` ;
7. tester chaque endpoint avec `curl`.

### Travail supplémentaire

Ajouter :

```http
GET /api/centres?ville=Sousse
```

pour filtrer les centres par ville.

---

# 32. Préparation de l'atelier suivant

Dans cet atelier, la validation des données a été volontairement réalisée de manière simple.

Dans le prochain atelier, nous allons rencontrer une problématique importante :

> **Comment valider proprement les données reçues par une API ?**

Nous introduirons alors une véritable stratégie de validation et nous commencerons à traiter :

* validation des champs ;
* types de données ;
* contraintes ;
* messages d'erreur ;
* validation des `body`, `params` et `query` ;
* gestion centralisée des erreurs.

La prochaine étape sera donc :

```text
API REST
   ↓
Validation
   ↓
Gestion des erreurs
   ↓
API plus robuste
```

Cet atelier 3 prépare ainsi directement **l’atelier 4 sur la validation des entrées et la gestion des erreurs**, sans encore introduire la base de données : cela permet de garder la progression pédagogique centrée sur l’API avant d’aborder la persistance.
