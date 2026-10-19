# Atelier 4 — Validation des données et gestion des erreurs avec Node.js, Express.js et Zod

**Module :** Ingénierie Backend
**Projet fil rouge :** SangConnect — API de gestion des dons de sang
**Durée :** 3 heures
**Technologies :** Node.js, Express.js, Zod
**Prérequis :** Ateliers 1, 2 et 3
**Niveau :** 4ème année cycle ingénieur

---

# 1. Introduction

Dans l'Atelier 3, nous avons construit notre première API REST avec **Node.js et Express.js**.

Nous avons notamment appris à :

* installer Express.js ;
* créer un serveur Express ;
* définir des routes ;
* utiliser les méthodes HTTP ;
* récupérer `req.params` ;
* récupérer `req.query` ;
* récupérer `req.body` ;
* retourner des réponses JSON ;
* utiliser les codes HTTP ;
* construire un CRUD ;
* organiser l'application en couches :

```text
Route
   ↓
Controller
   ↓
Service
   ↓
Données
```

Notre API permet maintenant de gérer les donneurs :

```text
GET    /api/donneurs
GET    /api/donneurs/:id
POST   /api/donneurs
PUT    /api/donneurs/:id
PATCH  /api/donneurs/:id
DELETE /api/donneurs/:id
```

Cependant, un problème important apparaît.

Que se passe-t-il si un client envoie :

```json
{
    "nom": "",
    "prenom": 25,
    "groupeSanguin": "XYZ"
}
```

Notre application ne doit pas accepter aveuglément ces données.

Nous devons donc mettre en place un mécanisme permettant de **valider les données reçues par l'API avant de les transmettre au contrôleur et au service**.

Dans l'Atelier 3, nous avons commencé à effectuer une validation manuelle.

Dans cet atelier, nous allons remplacer cette approche par une solution plus structurée utilisant **Zod**.

---

# 2. Objectifs de l'atelier

À la fin de cet atelier, l'étudiant devra être capable de :

* expliquer pourquoi la validation des données est nécessaire ;
* identifier les limites d'une validation manuelle ;
* installer et utiliser Zod ;
* définir un schéma de validation ;
* valider `req.body` ;
* valider `req.params` ;
* valider `req.query` ;
* créer un middleware de validation réutilisable ;
* distinguer validation et traitement métier ;
* gérer les erreurs de manière centralisée ;
* retourner des réponses HTTP cohérentes ;
* améliorer l'architecture de l'API SangConnect.

L'architecture finale sera :

```text
Client HTTP
     │
     ▼
   Route
     │
     ▼
 Validation
     │
     ▼
 Controller
     │
     ▼
  Service
     │
     ▼
 Données
```

---

# 3. Préparation : reprendre le projet de l'Atelier 3

Nous allons continuer directement avec le projet développé précédemment.

Placez-vous dans le dossier :

```bash
cd sangconnect
```

Vérifiez que le projet contient notamment :

```text
sangconnect/
├── package.json
├── package-lock.json
└── src/
    ├── server.js
    ├── routes/
    │   └── donneur.routes.js
    ├── controllers/
    │   └── donneur.controller.js
    └── services/
        └── donneur.service.js
```

---

# 4. Vérifier l'installation d'Express.js

Express.js a été installé dans l'Atelier 3.

Vérifiez simplement sa présence :

```bash
npm list express
```

Vous devriez obtenir une sortie indiquant qu'Express est installé.

### Question 1

Pourquoi ne devons-nous pas exécuter à nouveau :

```bash
npm install express
```

?

---

# 5. Rappel : la validation manuelle de l'Atelier 3

Dans l'Atelier 3, nous avons commencé à vérifier les données reçues lors de la création d'un donneur.

Par exemple :

```javascript
if (!req.body.nom ||
    !req.body.prenom ||
    !req.body.groupeSanguin) {

    return res.status(400).json({
        message: "Données invalides"
    });
}
```

Cette solution fonctionne pour un petit exemple.

Mais imaginons que nous ayons maintenant :

```text
nom
prenom
email
telephone
dateNaissance
groupeSanguin
adresse
ville
```

Nous devrions multiplier les conditions.

Exemple :

```javascript
if (!req.body.nom) {
    ...
}

if (!req.body.prenom) {
    ...
}

if (!req.body.email) {
    ...
}

if (!req.body.telephone) {
    ...
}
```

La validation devient rapidement difficile à maintenir.

---

# 6. Pourquoi utiliser une bibliothèque de validation ?

Une bibliothèque de validation permet de décrire les règles sous une forme structurée.

Nous voulons pouvoir écrire quelque chose comme :

```text
nom :
    chaîne de caractères
    minimum 2 caractères

prenom :
    chaîne de caractères
    minimum 2 caractères

groupeSanguin :
    une valeur parmi les groupes sanguins autorisés
```

C'est précisément ce que nous allons réaliser avec **Zod**.

---

# 7. Installation de Zod

Dans le terminal :

```bash
npm install zod
```

Vérifiez :

```bash
npm list zod
```

### Question 2

Dans quel fichier la dépendance Zod apparaît-elle ?

---

# 8. Première découverte de Zod

Créez temporairement :

```text
src/test-zod.js
```

Ajoutez :

```javascript
import { z } from "zod";

const schema = z.object({
    nom: z.string(),
    prenom: z.string()
});

const donneur = {
    nom: "Ben Ali",
    prenom: "Ahmed"
};

const resultat = schema.safeParse(donneur);

console.log(resultat);
```

Exécutez :

```bash
node src/test-zod.js
```

Observez le résultat.

---

# 9. Comprendre `z.object()`

Cette instruction :

```javascript
z.object({
    nom: z.string(),
    prenom: z.string()
});
```

décrit la structure attendue de l'objet.

Nous indiquons donc :

```text
nom    → chaîne de caractères
prenom → chaîne de caractères
```

Le schéma représente les règles auxquelles les données doivent satisfaire.

---

# 10. Comprendre `safeParse()`

Nous pouvons vérifier les données avec :

```javascript
schema.safeParse(donneur);
```

Le résultat contient notamment :

```javascript
success
```

Si la validation réussit :

```javascript
{
    success: true,
    ...
}
```

Sinon :

```javascript
{
    success: false,
    ...
}
```

Cette méthode est intéressante pour une API car elle permet de vérifier les données sans provoquer directement une exception.

---

# 11. Créer le schéma de validation d'un donneur

Supprimez le fichier de test lorsque vous avez terminé.

Créez :

```text
src/validators/
```

puis :

```text
src/validators/donneur.validator.js
```

Ajoutez :

```javascript
import { z } from "zod";

export const donneurSchema = z.object({

    nom: z.string().min(2),

    prenom: z.string().min(2),

    groupeSanguin: z.enum([
        "A+",
        "A-",
        "B+",
        "B-",
        "AB+",
        "AB-",
        "O+",
        "O-"
    ])

});
```

---

# 12. Comprendre le schéma

Nous avons maintenant exprimé les règles métier de base concernant les données reçues.

Pour :

```javascript
nom: z.string().min(2)
```

nous imposons :

* une chaîne de caractères ;
* au minimum deux caractères.

Pour :

```javascript
groupeSanguin: z.enum([...])
```

nous indiquons que seules certaines valeurs sont acceptées.

Par exemple :

```text
A+
A-
B+
B-
AB+
AB-
O+
O-
```

sont acceptées.

Mais :

```text
XYZ
AA
O
```

ne le sont pas.

---

# 13. Tester le schéma

Créez temporairement un petit test :

```javascript
const donneur = {
    nom: "Ben Ali",
    prenom: "Ahmed",
    groupeSanguin: "A+"
};

const resultat = donneurSchema.safeParse(donneur);

console.log(resultat.success);
```

Testez ensuite :

```javascript
const donneur = {
    nom: "",
    prenom: "Ahmed",
    groupeSanguin: "XYZ"
};
```

### Question 3

Que se passe-t-il dans les deux cas ?

---

# 14. Comprendre les erreurs de validation

Modifiez le test :

```javascript
const resultat = donneurSchema.safeParse(donneur);

if (!resultat.success) {
    console.log(resultat.error);
}
```

Observez les informations fournies par Zod.

Nous pouvons également utiliser :

```javascript
resultat.error.issues
```

pour accéder aux différentes erreurs.

---

# 15. Problème : où effectuer la validation ?

Nous pourrions écrire directement dans le contrôleur :

```javascript
export function create(req, res) {

    const resultat = donneurSchema.safeParse(req.body);

    if (!resultat.success) {
        ...
    }

    ...
}
```

Mais nous allons avoir le même problème si plusieurs contrôleurs doivent effectuer des validations.

Nous voulons donc créer un **middleware réutilisable**.

---

# 16. Introduction aux middlewares Express

Un middleware Express est une fonction qui intervient pendant le traitement d'une requête.

On peut représenter le traitement ainsi :

```text
Requête HTTP
     │
     ▼
 Middleware
     │
     ▼
 Controller
     │
     ▼
 Réponse
```

Un middleware peut :

* examiner la requête ;
* modifier la requête ;
* arrêter le traitement ;
* transmettre la requête au middleware suivant ;
* appeler le contrôleur.

La fonction :

```javascript
next()
```

permet de poursuivre le traitement.

---

# 17. Créer le middleware de validation

Créez :

```text
src/middlewares/
```

puis :

```text
src/middlewares/validate.js
```

Ajoutez :

```javascript
export function validate(schema) {

    return (req, res, next) => {

        const result = schema.safeParse(req.body);

        if (!result.success) {

            return res.status(400).json({
                message: "Données invalides",
                errors: result.error.issues
            });
        }

        req.body = result.data;

        next();
    };
}
```

---

# 18. Comprendre le middleware

La fonction :

```javascript
validate(schema)
```

reçoit un schéma Zod.

Par exemple :

```javascript
validate(donneurSchema)
```

Le middleware effectue ensuite :

```javascript
schema.safeParse(req.body)
```

Si les données sont invalides :

```text
400 Bad Request
```

est retourné.

Si les données sont valides :

```javascript
next();
```

permet de poursuivre vers le contrôleur.

---

# 19. Modifier la route POST

Dans :

```text
src/routes/donneur.routes.js
```

importez :

```javascript
import { validate } from "../middlewares/validate.js";

import { donneurSchema } from "../validators/donneur.validator.js";
```

Puis modifiez la route POST.

Avant :

```javascript
router.post("/", create);
```

Après :

```javascript
router.post(
    "/",
    validate(donneurSchema),
    create
);
```

Nous avons maintenant :

```text
POST /api/donneurs
        │
        ▼
validate(donneurSchema)
        │
        ▼
     create()
```

---

# 20. Tester avec des données valides

Utilisez :

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}'
```

La validation doit être réussie.

---

# 21. Tester avec des données invalides

Testez :

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{
    "nom": "",
    "prenom": "Yassine",
    "groupeSanguin": "XYZ"
}'
```

La réponse doit être une erreur HTTP `400`.

### Question 4

Pourquoi le contrôleur n'est-il pas exécuté lorsque la validation échoue ?

---

# 22. Pourquoi valider avant le contrôleur ?

Considérons :

```text
Client
   │
   ▼
Validation
   │
   ├── invalide → 400
   │
   ▼
Controller
   │
   ▼
Service
```

Cette organisation permet au contrôleur de travailler avec des données déjà contrôlées.

Le contrôleur n'a donc plus besoin de vérifier chaque champ.

Cela permet de respecter une séparation claire des responsabilités :

| Composant  | Responsabilité                   |
| ---------- | -------------------------------- |
| Route      | définir l'URL et la méthode HTTP |
| Validation | vérifier les données             |
| Controller | gérer la requête/réponse         |
| Service    | logique métier                   |

---

# 23. Validation des paramètres d'URL

Nous devons également valider :

```text
/api/donneurs/:id
```

Dans ce cas, la donnée se trouve dans :

```javascript
req.params
```

Nous voulons vérifier que `id` est un entier positif.

Créons dans :

```text
src/validators/donneur.validator.js
```

le schéma :

```javascript
export const idSchema = z.object({
    id: z.coerce.number().int().positive()
});
```

---

# 24. Adapter le middleware pour plusieurs sources

Notre middleware actuel ne valide que :

```javascript
req.body
```

Nous allons le rendre générique.

Remplacez son contenu par :

```javascript
export function validate(schema, source) {

    return (req, res, next) => {

        const result = schema.safeParse(req[source]);

        if (!result.success) {

            return res.status(400).json({
                message: "Données invalides",
                errors: result.error.issues
            });
        }

        req[source] = result.data;

        next();
    };
}
```

Nous pouvons maintenant écrire :

```javascript
validate(donneurSchema, "body")
```

ou :

```javascript
validate(idSchema, "params")
```

---

# 25. Appliquer la validation à `GET /api/donneurs/:id`

Dans :

```text
src/routes/donneur.routes.js
```

utilisez :

```javascript
router.get(
    "/:id",
    validate(idSchema, "params"),
    getById
);
```

Le traitement devient :

```text
GET /api/donneurs/5
        │
        ▼
Validation de req.params
        │
        ▼
     Controller
        │
        ▼
      Service
```

---

# 26. Tester l'identifiant

Test valide :

```bash
curl http://localhost:3000/api/donneurs/1
```

Test invalide :

```bash
curl http://localhost:3000/api/donneurs/abc
```

Test invalide :

```bash
curl http://localhost:3000/api/donneurs/0
```

Test invalide :

```bash
curl http://localhost:3000/api/donneurs/-5
```

### Question 5

Pourquoi est-il préférable de vérifier `id` avant d'appeler le service ?

---

# 27. Validation des paramètres de requête

Dans l'Atelier 3, nous avons également utilisé :

```text
GET /api/donneurs?groupeSanguin=A+
```

La valeur se trouve dans :

```javascript
req.query
```

Créons :

```javascript
export const donneurQuerySchema = z.object({
    groupeSanguin: z.enum([
        "A+",
        "A-",
        "B+",
        "B-",
        "AB+",
        "AB-",
        "O+",
        "O-"
    ]).optional()
});
```

Le `optional()` signifie que le paramètre n'est pas obligatoire.

---

# 28. Appliquer la validation à `req.query`

Dans les routes :

```javascript
router.get(
    "/",
    validate(donneurQuerySchema, "query"),
    getAll
);
```

Nous avons maintenant :

```text
GET /api/donneurs
        │
        ▼
Validation query
        │
        ▼
Controller
```

---

# 29. Tester les paramètres de requête

Test valide :

```bash
curl "http://localhost:3000/api/donneurs?groupeSanguin=A%2B"
```

Test invalide :

```bash
curl "http://localhost:3000/api/donneurs?groupeSanguin=XYZ"
```

### Question 6

Pourquoi avons-nous utilisé :

```javascript
.optional()
```

?

---

# 30. Validation de PUT

La méthode `PUT` représente une modification complète.

Nous pouvons donc utiliser le même schéma :

```javascript
donneurSchema
```

Exemple :

```javascript
router.put(
    "/:id",
    validate(idSchema, "params"),
    validate(donneurSchema, "body"),
    update
);
```

Le traitement est alors :

```text
PUT /api/donneurs/5
       │
       ▼
Validation ID
       │
       ▼
Validation Body
       │
       ▼
Controller
       │
       ▼
Service
```

---

# 31. Validation de PATCH

`PATCH` permet une modification partielle.

Nous avons donc besoin d'un schéma dans lequel les propriétés peuvent être absentes.

Ajoutez :

```javascript
export const updateDonneurSchema =
    donneurSchema.partial();
```

Ainsi :

```json
{
    "nom": "NouveauNom"
}
```

peut être accepté.

Mais :

```json
{
    "groupeSanguin": "XYZ"
}
```

doit être refusé.

---

# 32. Route PATCH

```javascript
router.patch(
    "/:id",
    validate(idSchema, "params"),
    validate(updateDonneurSchema, "body"),
    update
);
```

---

# 33. Architecture des routes

Notre fichier de routes commence maintenant à exprimer clairement le processus de validation :

```javascript
router.get(
    "/",
    validate(donneurQuerySchema, "query"),
    getAll
);

router.get(
    "/:id",
    validate(idSchema, "params"),
    getById
);

router.post(
    "/",
    validate(donneurSchema, "body"),
    create
);

router.put(
    "/:id",
    validate(idSchema, "params"),
    validate(donneurSchema, "body"),
    update
);

router.patch(
    "/:id",
    validate(idSchema, "params"),
    validate(updateDonneurSchema, "body"),
    update
);
```

---

# 34. Exercice 1 — Tester toutes les validations

Réalisez les tests suivants.

### Cas 1 — Donneur valide

```json
{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "A+"
}
```

### Cas 2 — Nom trop court

```json
{
    "nom": "A",
    "prenom": "Yassine",
    "groupeSanguin": "A+"
}
```

### Cas 3 — Groupe sanguin invalide

```json
{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "XYZ"
}
```

### Cas 4 — Donneur sans prénom

```json
{
    "nom": "Khelifi",
    "groupeSanguin": "A+"
}
```

### Cas 5 — Identifiant invalide

```text
/api/donneurs/abc
```

### Cas 6 — Groupe sanguin invalide dans la requête

```text
/api/donneurs?groupeSanguin=XYZ
```

Pour chaque cas, notez :

* la requête ;
* le code HTTP ;
* le résultat obtenu ;
* la raison du rejet éventuel.

---

# 35. Problème suivant : la gestion des erreurs

Jusqu'à présent, nous avons utilisé :

```javascript
try {
    ...
}
catch (error) {
    ...
}
```

dans certains traitements.

Mais si plusieurs contrôleurs doivent gérer leurs erreurs de la même manière, nous allons répéter du code.

Nous allons donc mettre en place un **middleware centralisé de gestion des erreurs**.

---

# 36. Middleware de gestion des erreurs

Créez :

```text
src/middlewares/errorHandler.js
```

Ajoutez :

```javascript
export function errorHandler(error, req, res, next) {

    console.error(error);

    const statusCode = error.statusCode || 500;

    res.status(statusCode).json({
        message: error.message || "Erreur interne du serveur"
    });
}
```

---

# 37. Pourquoi quatre paramètres ?

Un middleware d'erreur Express possède une signature particulière :

```javascript
(error, req, res, next)
```

La présence du premier paramètre :

```javascript
error
```

permet à Express de reconnaître qu'il s'agit d'un middleware destiné aux erreurs.

---

# 38. Ajouter le middleware dans `server.js`

Dans :

```text
src/server.js
```

ajoutez le middleware **après les routes** :

```javascript
app.use("/api/donneurs", donneurRoutes);

app.use(errorHandler);
```

N'oubliez pas l'import :

```javascript
import { errorHandler }
from "./middlewares/errorHandler.js";
```

L'ordre est important.

Nous obtenons :

```text
Middleware JSON
      ↓
Routes
      ↓
Controllers
      ↓
Services
      ↓
Error Handler
```

---

# 39. Transmettre une erreur au middleware

Dans un contrôleur, nous pouvons utiliser :

```javascript
next(error);
```

Exemple :

```javascript
export function example(req, res, next) {

    try {

        // traitement

    } catch (error) {

        next(error);
    }
}
```

Express transmettra alors l'erreur au middleware :

```javascript
errorHandler
```

---

# 40. Créer une erreur métier

Nous pouvons par exemple créer une erreur lorsque le donneur n'existe pas.

Dans le contrôleur :

```javascript
export function getById(req, res, next) {

    try {

        const id = req.params.id;

        const donneur = donneurService.getById(id);

        if (!donneur) {

            const error = new Error(
                "Donneur introuvable"
            );

            error.statusCode = 404;

            throw error;
        }

        res.json(donneur);

    } catch (error) {

        next(error);
    }
}
```

Le contrôleur ne construit donc plus directement la réponse d'erreur.

Il transmet l'erreur au système centralisé.

---

# 41. Le flux complet

Nous avons maintenant :

```text
                Requête HTTP
                     │
                     ▼
                   Route
                     │
                     ▼
               Validation
                     │
             ┌───────┴───────┐
             │               │
          invalide          valide
             │               │
             ▼               ▼
            400          Controller
                             │
                             ▼
                          Service
                             │
                     ┌───────┴───────┐
                     │               │
                   erreur          succès
                     │               │
                     ▼               ▼
               Error Handler      Response
                     │
                     ▼
                  Réponse
```

Cette architecture est beaucoup plus propre que de gérer chaque erreur directement dans chaque route.

---

# 42. Exercice 2 — Gestion centralisée des erreurs

Modifier le contrôleur `Donneur` afin que :

* une erreur `404` soit générée lorsqu'un donneur n'existe pas ;
* l'erreur soit transmise avec `next(error)` ;
* le middleware `errorHandler` retourne la réponse.

Tester :

```bash
curl http://localhost:3000/api/donneurs/999
```

La réponse attendue est similaire à :

```json
{
    "message": "Donneur introuvable"
}
```

avec :

```text
404 Not Found
```

---

# 43. Exercice 3 — Ajouter une ressource Centre

Nous allons maintenant appliquer les concepts à une nouvelle ressource.

Un centre possède :

```text
id
nom
ville
adresse
telephone
```

Créer :

```text
src/validators/centre.validator.js
```

Définir un schéma Zod.

Exemple :

```javascript
import { z } from "zod";

export const centreSchema = z.object({

    nom: z.string().min(2),

    ville: z.string().min(2),

    adresse: z.string().min(3),

    telephone: z.string().min(8)

});
```

Créer également :

```text
centre.routes.js
centre.controller.js
centre.service.js
```

---

# 44. Exercice 4 — Valider les centres

Implémenter :

```text
POST /api/centres
```

avec validation Zod.

Tester un centre valide :

```json
{
    "nom": "Centre Régional de Sousse",
    "ville": "Sousse",
    "adresse": "Rue de la République",
    "telephone": "73123456"
}
```

Puis tester des données invalides.

---

# 45. Exercice 5 — Améliorer les messages d'erreur

Actuellement, nous retournons :

```json
{
    "message": "Données invalides",
    "errors": [...]
}
```

Observer le contenu de :

```javascript
result.error.issues
```

Puis améliorer la réponse afin que le client puisse identifier clairement les champs incorrects.

Par exemple :

```json
{
    "message": "Données invalides",
    "errors": [
        {
            "field": "nom",
            "message": "Le nom doit contenir au moins 2 caractères"
        }
    ]
}
```

---

# 46. Exercice 6 — Validation d'un email

Ajouter un champ :

```text
email
```

dans le donneur.

Utiliser une validation Zod adaptée.

Tester :

```text
ahmed@example.com
```

puis :

```text
email-invalide
```

### Question

Pourquoi est-il préférable de réaliser cette validation dans le middleware plutôt que dans le service ?

---

# 47. Exercice 7 — Validation d'un numéro de téléphone

Ajouter :

```text
telephone
```

au schéma du donneur.

Définir une règle permettant de refuser les valeurs manifestement incorrectes.

Tester plusieurs valeurs.

Réfléchir aux limites d'une validation trop restrictive.

---

# 48. Synthèse des concepts

Dans cet atelier, nous avons ajouté plusieurs concepts importants à notre API.

## Zod

Permet de définir les règles de validation.

```javascript
z.object(...)
```

---

## `safeParse()`

Permet de tester les données :

```javascript
schema.safeParse(data)
```

---

## Middleware

Permet d'intercepter une requête avant le contrôleur.

```javascript
validate(schema, "body")
```

---

## `req.body`

Contient les données envoyées dans le corps de la requête.

---

## `req.params`

Contient les paramètres de l'URL.

Exemple :

```text
/api/donneurs/5
```

```javascript
req.params.id
```

---

## `req.query`

Contient les paramètres de requête.

Exemple :

```text
/api/donneurs?groupeSanguin=A+
```

```javascript
req.query.groupeSanguin
```

---

## `next(error)`

Permet de transmettre une erreur au gestionnaire central.

---

# 49. Architecture finale de l'Atelier 4

L'application SangConnect possède maintenant une architecture plus complète :

```text
sangconnect/
│
├── package.json
├── package-lock.json
│
└── src/
    │
    ├── server.js
    │
    ├── routes/
    │   └── donneur.routes.js
    │
    ├── controllers/
    │   └── donneur.controller.js
    │
    ├── services/
    │   └── donneur.service.js
    │
    ├── validators/
    │   └── donneur.validator.js
    │
    └── middlewares/
        ├── validate.js
        └── errorHandler.js
```

Le flux d'une requête est désormais :

```text
Client
  │
  ▼
Express
  │
  ▼
Route
  │
  ▼
Validation
  │
  ▼
Controller
  │
  ▼
Service
  │
  ▼
Données
```

En cas d'erreur :

```text
Controller / Service
        │
        ▼
    next(error)
        │
        ▼
 Error Handler
        │
        ▼
 Réponse HTTP
```

---

# 50. Comparaison Atelier 3 → Atelier 4

## Atelier 3

Nous avions :

```text
Client
   ↓
Route
   ↓
Controller
   ↓
Service
   ↓
Données en mémoire
```

La validation était principalement manuelle.

---

## Atelier 4

Nous avons ajouté :

```text
Client
   ↓
Route
   ↓
Validation Zod
   ↓
Controller
   ↓
Service
   ↓
Données en mémoire
```

et :

```text
Erreur
   ↓
Error Handler
   ↓
Réponse HTTP
```

L'architecture devient donc plus robuste.

---

# 51. Préparation de l'Atelier 5

Jusqu'à présent, les données sont conservées en mémoire dans des tableaux JavaScript.

Cette solution présente une limitation évidente :

```text
Arrêt du serveur
      ↓
Données perdues
```

Dans l'Atelier 5, nous allons résoudre ce problème.

Nous allons introduire :

```text
Prisma ORM
     ↓
MySQL
```

L'architecture deviendra :

```text
Client
   ↓
Route
   ↓
Validation Zod
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Prisma
   ↓
MySQL
```

Nous conserverons donc tout ce qui a été construit dans les Ateliers 3 et 4.

Nous ajouterons simplement la **couche de persistance**.

---

# 52. Travail pratique à faire à la maison

## Sujet : validation complète de la ressource Centre

Compléter l'API `Centre` en utilisant Zod.

Chaque centre possède :

```text
id
nom
ville
adresse
telephone
```

Implémenter :

```text
GET    /api/centres
GET    /api/centres/:id
POST   /api/centres
PUT    /api/centres/:id
PATCH  /api/centres/:id
DELETE /api/centres/:id
```

Mettre en place :

```text
routes
controllers
services
validators
middlewares
```

La validation doit concerner :

* `req.body` ;
* `req.params` ;
* `req.query`.

Ajouter également :

```text
GET /api/centres?ville=Sousse
```

et valider le paramètre `ville`.

---

# 53. Travail de réflexion

Répondre aux questions suivantes :

### Question 1

Pourquoi ne faut-il pas faire toute la validation dans le contrôleur ?

### Question 2

Quelle est la différence entre :

```text
Validation
```

et :

```text
Logique métier
```

### Question 3

Quel est l'intérêt d'un middleware générique :

```javascript
validate(schema, source)
```

?

### Question 4

Pourquoi centraliser la gestion des erreurs ?

### Question 5

Quelle couche sera ajoutée dans l'Atelier 5 pour remplacer les données en mémoire ?

---

# 54. Compétences acquises

À la fin de l'Atelier 4, l'étudiant doit être capable de :

* installer et utiliser Zod ;
* créer un schéma de validation ;
* valider un `body` ;
* valider des paramètres d'URL ;
* valider des paramètres de requête ;
* créer un middleware ;
* utiliser `next()` ;
* utiliser `next(error)` ;
* gérer les erreurs de façon centralisée ;
* séparer validation, contrôleur et logique métier ;
* expliquer l'architecture de l'API SangConnect.

Vous êtes est maintenant prêt à aborder la **persistance des données avec Prisma et MySQL** dans l'Atelier 5.
