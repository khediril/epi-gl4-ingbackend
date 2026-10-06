# TP 3 — Concevoir et implémenter une API REST avec Node.js et Express.js

**Module :** Ingénierie Backend
**Projet fil rouge :** SangConnect — API de gestion des dons de sang
**Durée :** 3 heures
**Technologies :** Node.js, npm, Express.js
**Niveau :** 4ème année cycle ingénieur

---

# 1. Introduction

Dans les deux premiers TP, nous avons découvert l'environnement Node.js et commencé à structurer notre application **SangConnect**.

Nous avons notamment travaillé avec :

* Node.js ;
* le serveur HTTP natif de Node.js ;
* les modules JavaScript ;
* les fonctions asynchrones ;
* les `Promise` ;
* `async/await` ;
* les routes ;
* les contrôleurs ;
* les services ;
* l'organisation d'une application backend en couches.

Cependant, construire une application HTTP uniquement avec le module `http` de Node.js devient rapidement difficile lorsque le nombre de routes et de fonctionnalités augmente.

Par exemple, pour gérer les donneurs de sang, notre application devra répondre à des requêtes telles que :

```text
GET    /api/donneurs
GET    /api/donneurs/5
POST   /api/donneurs
PUT    /api/donneurs/5
DELETE /api/donneurs/5
```

Node.js fournit les mécanismes de bas niveau permettant de gérer ces requêtes, mais il ne fournit pas directement une architecture confortable pour construire une API REST.

C'est ici qu'intervient **Express.js**.

---

# 2. Objectifs du TP

À la fin de ce TP, l'étudiant devra être capable de :

* expliquer le rôle d'Express.js ;
* installer Express.js dans un projet Node.js ;
* créer une application Express ;
* démarrer un serveur Express ;
* créer des routes HTTP ;
* utiliser les méthodes HTTP `GET`, `POST`, `PUT`, `PATCH` et `DELETE` ;
* récupérer des paramètres d'URL ;
* récupérer des paramètres de requête ;
* récupérer les données envoyées dans le corps d'une requête ;
* retourner des réponses JSON ;
* utiliser les principaux codes HTTP ;
* construire une API REST simple ;
* organiser une API avec `routes`, `controllers` et `services` ;
* réaliser un CRUD sur la ressource `Donneur` ;
* tester une API avec `curl`.

---

# 3. Préparation de l'environnement

Nous allons poursuivre le projet commencé dans les TP précédents.

Placez-vous dans le dossier du projet :

```bash
cd ~/tpphp
```

ou dans le dossier dans lequel vous avez créé votre projet SangConnect.

Vérifiez Node.js :

```bash
node --version
```

Puis npm :

```bash
npm --version
```

### Question 1

1. Quelle version de Node.js utilisez-vous ?
2. Quelle version de npm utilisez-vous ?
3. Quelle est la différence entre Node.js et npm ?

---

# 4. Création du projet Express

Si vous poursuivez le projet du TP précédent, utilisez directement votre dossier SangConnect.

Sinon, créez un nouveau dossier :

```bash
mkdir sangconnect
cd sangconnect
```

Initialisez le projet npm :

```bash
npm init -y
```

La commande crée le fichier :

```text
package.json
```

---

# 5. Installation d'Express.js

## 5.1 Pourquoi Express.js ?

**Express.js** est un framework minimaliste pour Node.js permettant notamment de faciliter :

* la création d'un serveur HTTP ;
* la définition des routes ;
* le traitement des requêtes ;
* la génération des réponses ;
* l'utilisation de middlewares ;
* la construction d'API REST.

Avec Node.js seul, nous devons manipuler directement le module `http`.

Avec Express.js, nous pouvons écrire :

```javascript
app.get("/api/donneurs", ...);
```

au lieu de gérer manuellement la méthode HTTP et l'URL à partir de l'objet `request`.

---

## 5.2 Installation

Dans le terminal :

```bash
npm install express
```

### Question 2

Observez le fichier `package.json`.

1. Quelle nouvelle section apparaît ?
2. Quelle dépendance contient Express ?
3. Pourquoi Express est-il enregistré dans `dependencies` ?

Vous devriez obtenir quelque chose ressemblant à :

```json
{
  "dependencies": {
    "express": "..."
  }
}
```

Le fichier :

```text
package-lock.json
```

est également créé ou modifié.

---

# 6. Configuration des modules ES

Dans les TP précédents, nous avons utilisé les modules ES.

Dans `package.json`, ajoutez :

```json
"type": "module"
```

Par exemple :

```json
{
  "name": "sangconnect",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/server.js"
  },
  "dependencies": {
    "express": "..."
  }
}
```

Nous pouvons alors utiliser :

```javascript
import express from "express";
```

au lieu de :

```javascript
const express = require("express");
```

---

# 7. Première application Express

Créez le dossier :

```bash
mkdir src
```

Puis le fichier :

```text
src/server.js
```

Placez-y le code suivant :

```javascript
import express from "express";

const app = express();

const PORT = 3000;

app.get("/", (req, res) => {
    res.send("Bienvenue dans SangConnect");
});

app.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

Lancez le serveur :

```bash
node src/server.js
```

Vous devriez obtenir :

```text
Serveur démarré sur http://localhost:3000
```

Ouvrez :

```text
http://localhost:3000
```

---

# 8. Comprendre le code Express

Étudions maintenant le programme.

## 8.1 Importation

```javascript
import express from "express";
```

Nous importons le module Express.

---

## 8.2 Création de l'application

```javascript
const app = express();
```

Cette instruction crée notre application Express.

L'objet `app` permettra notamment de définir :

* les routes ;
* les middlewares ;
* le comportement du serveur.

---

## 8.3 Définition d'une route

```javascript
app.get("/", (req, res) => {
    res.send("Bienvenue dans SangConnect");
});
```

Cette route signifie :

> Lorsqu'une requête HTTP `GET` est envoyée vers `/`, exécuter cette fonction.

Les paramètres sont :

```javascript
req
```

pour **request**, la requête HTTP,

et :

```javascript
res
```

pour **response**, la réponse HTTP.

---

## 8.4 Démarrage du serveur

```javascript
app.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

Express démarre un serveur HTTP qui écoute sur le port `3000`.

---

# 9. Première modification : route `/api/health`

Ajoutez :

```javascript
app.get("/api/health", (req, res) => {
    res.json({
        status: "OK",
        service: "SangConnect"
    });
});
```

Testez :

```text
http://localhost:3000/api/health
```

La réponse doit être :

```json
{
  "status": "OK",
  "service": "SangConnect"
}
```

### Question 3

Quelle différence observez-vous entre :

```javascript
res.send(...)
```

et :

```javascript
res.json(...)
```

---

# 10. Créer une API REST

Notre objectif est maintenant de construire une API pour gérer les donneurs.

Une API REST manipule généralement des **ressources**.

Dans notre cas :

```text
Donneur
```

sera une ressource.

Nous utiliserons :

```text
/api/donneurs
```

comme URL principale.

---

# 11. Les principales méthodes HTTP

| Méthode | Utilisation                          |
| ------- | ------------------------------------ |
| GET     | récupérer des données                |
| POST    | créer une ressource                  |
| PUT     | remplacer une ressource              |
| PATCH   | modifier partiellement une ressource |
| DELETE  | supprimer une ressource              |

Pour notre ressource `Donneur` :

| Action                 | Méthode | URL                 |
| ---------------------- | ------- | ------------------- |
| Liste                  | GET     | `/api/donneurs`     |
| Détail                 | GET     | `/api/donneurs/:id` |
| Création               | POST    | `/api/donneurs`     |
| Modification           | PUT     | `/api/donneurs/:id` |
| Modification partielle | PATCH   | `/api/donneurs/:id` |
| Suppression            | DELETE  | `/api/donneurs/:id` |

---

# 12. Première route GET

Ajoutez :

```javascript
app.get("/api/donneurs", (req, res) => {
    res.json([
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
    ]);
});
```

Testez avec votre navigateur :

```text
http://localhost:3000/api/donneurs
```

ou avec :

```bash
curl http://localhost:3000/api/donneurs
```

---

# 13. Les paramètres d'URL

Nous voulons maintenant récupérer un donneur particulier.

Nous utiliserons :

```text
/api/donneurs/1
```

La route est :

```javascript
app.get("/api/donneurs/:id", (req, res) => {

    const id = req.params.id;

    res.json({
        message: "Donneur demandé",
        id: id
    });
});
```

Testez :

```bash
curl http://localhost:3000/api/donneurs/1
```

Puis :

```bash
curl http://localhost:3000/api/donneurs/25
```

### Question 4

Que contient :

```javascript
req.params.id
```

dans chacun des deux cas ?

---

# 14. Transformer l'identifiant en nombre

Attention : les paramètres d'URL sont récupérés sous forme de chaîne de caractères.

Nous pouvons utiliser :

```javascript
const id = Number(req.params.id);
```

Puis :

```javascript
res.json({
    id: id,
    type: typeof id
});
```

Testez la route.

### Question 5

Quel est maintenant le type de :

```javascript
id
```

?

---

# 15. Simuler une base de données

Pour le moment, nous n'utilisons pas encore MySQL.

Nous allons utiliser un tableau JavaScript afin de simuler temporairement une source de données.

Créez :

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
    },
    {
        id: 3,
        nom: "Mansouri",
        prenom: "Amine",
        groupeSanguin: "B+"
    }
];
```

Puis :

```javascript
app.get("/api/donneurs", (req, res) => {
    res.json(donneurs);
});
```

---

# 16. Rechercher un donneur

Nous pouvons utiliser :

```javascript
find()
```

Exemple :

```javascript
app.get("/api/donneurs/:id", (req, res) => {

    const id = Number(req.params.id);

    const donneur = donneurs.find(
        donneur => donneur.id === id
    );

    res.json(donneur);
});
```

Testez :

```bash
curl http://localhost:3000/api/donneurs/2
```

---

# 17. Gestion du cas 404

Que se passe-t-il si nous demandons :

```bash
curl http://localhost:3000/api/donneurs/100
```

Il n'existe probablement aucun donneur avec l'identifiant `100`.

Nous devons donc retourner :

```text
404 Not Found
```

Modifiez la route :

```javascript
app.get("/api/donneurs/:id", (req, res) => {

    const id = Number(req.params.id);

    const donneur = donneurs.find(
        donneur => donneur.id === id
    );

    if (!donneur) {
        return res.status(404).json({
            message: "Donneur introuvable"
        });
    }

    res.json(donneur);
});
```

Testez :

```bash
curl -i http://localhost:3000/api/donneurs/100
```

L'option `-i` permet d'afficher également les informations HTTP.

---

# 18. Les codes HTTP

Dans une API REST, le code HTTP fournit une information importante au client.

Quelques codes essentiels :

| Code | Signification         |
| ---: | --------------------- |
|  200 | OK                    |
|  201 | Created               |
|  204 | No Content            |
|  400 | Bad Request           |
|  401 | Unauthorized          |
|  403 | Forbidden             |
|  404 | Not Found             |
|  409 | Conflict              |
|  500 | Internal Server Error |

Dans notre API :

```javascript
res.status(404).json({
    message: "Donneur introuvable"
});
```

permet de retourner explicitement le code `404`.

---

# 19. Les paramètres de requête — `req.query`

Nous voulons permettre au client de rechercher les donneurs appartenant à un groupe sanguin.

Exemple :

```text
/api/donneurs?groupeSanguin=A+
```

Nous pouvons récupérer la valeur avec :

```javascript
req.query.groupeSanguin
```

Exemple :

```javascript
app.get("/api/donneurs", (req, res) => {

    const groupe = req.query.groupeSanguin;

    if (!groupe) {
        return res.json(donneurs);
    }

    const resultat = donneurs.filter(
        donneur => donneur.groupeSanguin === groupe
    );

    res.json(resultat);
});
```

Testez :

```bash
curl "http://localhost:3000/api/donneurs?groupeSanguin=A%2B"
```

### Question 6

Quelle est la différence entre :

```javascript
req.params
```

et :

```javascript
req.query
```

---

# 20. Recevoir des données avec POST

Nous voulons maintenant créer un nouveau donneur.

Le client devra envoyer :

```json
{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}
```

Mais Express doit d'abord savoir comment interpréter les données JSON.

Ajoutez **avant les routes** :

```javascript
app.use(express.json());
```

Cette instruction est très importante.

Elle permet à Express d'interpréter les requêtes contenant du JSON.

---

# 21. Créer la route POST

Ajoutez :

```javascript
app.post("/api/donneurs", (req, res) => {

    const donneur = {
        id: donneurs.length + 1,
        nom: req.body.nom,
        prenom: req.body.prenom,
        groupeSanguin: req.body.groupeSanguin
    };

    donneurs.push(donneur);

    res.status(201).json(donneur);
});
```

---

# 22. Tester POST avec curl

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

Vous devez obtenir une réponse similaire à :

```json
{
    "id": 4,
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}
```

### Question 7

1. À quoi sert `req.body` ?
2. Pourquoi avons-nous ajouté `express.json()` ?
3. Pourquoi retournons-nous le code `201` plutôt que `200` ?

---

# 23. Modifier un donneur avec PUT

Créons maintenant une route permettant de modifier complètement un donneur.

```javascript
app.put("/api/donneurs/:id", (req, res) => {

    const id = Number(req.params.id);

    const donneur = donneurs.find(
        donneur => donneur.id === id
    );

    if (!donneur) {
        return res.status(404).json({
            message: "Donneur introuvable"
        });
    }

    donneur.nom = req.body.nom;
    donneur.prenom = req.body.prenom;
    donneur.groupeSanguin = req.body.groupeSanguin;

    res.json(donneur);
});
```

Test :

```bash
curl -X PUT http://localhost:3000/api/donneurs/1 \
-H "Content-Type: application/json" \
-d '{
    "nom": "Ben Salem",
    "prenom": "Ahmed",
    "groupeSanguin": "A+"
}'
```

---

# 24. Modifier partiellement avec PATCH

La méthode `PATCH` permet de modifier seulement certains attributs.

Exemple :

```javascript
app.patch("/api/donneurs/:id", (req, res) => {

    const id = Number(req.params.id);

    const donneur = donneurs.find(
        donneur => donneur.id === id
    );

    if (!donneur) {
        return res.status(404).json({
            message: "Donneur introuvable"
        });
    }

    if (req.body.nom !== undefined) {
        donneur.nom = req.body.nom;
    }

    if (req.body.prenom !== undefined) {
        donneur.prenom = req.body.prenom;
    }

    if (req.body.groupeSanguin !== undefined) {
        donneur.groupeSanguin = req.body.groupeSanguin;
    }

    res.json(donneur);
});
```

Testez :

```bash
curl -X PATCH http://localhost:3000/api/donneurs/1 \
-H "Content-Type: application/json" \
-d '{
    "nom": "NouveauNom"
}'
```

---

# 25. Supprimer un donneur

Ajoutez :

```javascript
app.delete("/api/donneurs/:id", (req, res) => {

    const id = Number(req.params.id);

    const index = donneurs.findIndex(
        donneur => donneur.id === id
    );

    if (index === -1) {
        return res.status(404).json({
            message: "Donneur introuvable"
        });
    }

    donneurs.splice(index, 1);

    res.status(204).send();
});
```

Test :

```bash
curl -i -X DELETE http://localhost:3000/api/donneurs/2
```

---

# 26. Notre première API CRUD

Nous avons maintenant :

| Fonction               | Méthode | URL                 |
| ---------------------- | ------- | ------------------- |
| Lister                 | GET     | `/api/donneurs`     |
| Consulter              | GET     | `/api/donneurs/:id` |
| Créer                  | POST    | `/api/donneurs`     |
| Modifier               | PUT     | `/api/donneurs/:id` |
| Modifier partiellement | PATCH   | `/api/donneurs/:id` |
| Supprimer              | DELETE  | `/api/donneurs/:id` |

Nous avons donc construit un premier **CRUD REST**.

CRUD signifie :

```text
Create
Read
Update
Delete
```

---

# 27. Organiser le code

Notre fichier `server.js` commence maintenant à devenir trop volumineux.

Nous allons appliquer l'architecture étudiée dans le TP 2.

Notre objectif est :

```text
Client HTTP
     │
     ▼
   Route
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

# 28. Organisation proposée

Créez :

```text
sangconnect/
│
├── package.json
├── package-lock.json
│
└── src/
    ├── server.js
    │
    ├── routes/
    │   └── donneur.routes.js
    │
    ├── controllers/
    │   └── donneur.controller.js
    │
    └── services/
        └── donneur.service.js
```

Cette organisation prépare l'application à l'arrivée prochaine de :

```text
Prisma
   ↓
MySQL
```

---

# 29. Le service

Créez :

```text
src/services/donneur.service.js
```

Déplacez les opérations sur les données dans ce fichier.

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

export function getAll() {
    return donneurs;
}

export function getById(id) {
    return donneurs.find(
        donneur => donneur.id === id
    );
}
```

---

# 30. Le contrôleur

Créez :

```text
src/controllers/donneur.controller.js
```

```javascript
import * as donneurService from "../services/donneur.service.js";

export function getAll(req, res) {

    const donneurs = donneurService.getAll();

    res.json(donneurs);
}

export function getById(req, res) {

    const id = Number(req.params.id);

    const donneur = donneurService.getById(id);

    if (!donneur) {
        return res.status(404).json({
            message: "Donneur introuvable"
        });
    }

    res.json(donneur);
}
```

---

# 31. Les routes

Créez :

```text
src/routes/donneur.routes.js
```

```javascript
import express from "express";

import {
    getAll,
    getById
} from "../controllers/donneur.controller.js";

const router = express.Router();

router.get("/", getAll);

router.get("/:id", getById);

export default router;
```

---

# 32. Le serveur principal

Dans :

```text
src/server.js
```

nous pouvons maintenant écrire :

```javascript
import express from "express";

import donneurRoutes from "./routes/donneur.routes.js";

const app = express();

const PORT = 3000;

app.use(express.json());

app.get("/", (req, res) => {
    res.json({
        message: "Bienvenue dans l'API SangConnect"
    });
});

app.use("/api/donneurs", donneurRoutes);

app.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

---

# 33. Comprendre `app.use()`

Cette instruction :

```javascript
app.use("/api/donneurs", donneurRoutes);
```

signifie que les routes définies dans :

```text
donneur.routes.js
```

seront préfixées par :

```text
/api/donneurs
```

Ainsi :

```javascript
router.get("/", getAll);
```

devient :

```text
GET /api/donneurs
```

et :

```javascript
router.get("/:id", getById);
```

devient :

```text
GET /api/donneurs/:id
```

---

# 34. Architecture obtenue

Nous avons maintenant :

```text
                   Client HTTP
                       │
                       ▼
                  Express.js
                       │
                       ▼
                     Route
                       │
                       ▼
                  Controller
                       │
                       ▼
                    Service
                       │
                       ▼
                 Données mémoire
```

Dans le prochain TP, nous remplacerons les données en mémoire par une véritable validation des données.

Puis, dans l'atelier suivant, nous connecterons l'application à :

```text
Prisma
   ↓
MySQL
```

---

# 35. Exercice 1 — Ajouter la ressource Centre

Créer une nouvelle ressource :

```text
Centre
```

avec les propriétés :

```text
id
nom
ville
adresse
telephone
```

Créer :

```text
src/routes/centre.routes.js
src/controllers/centre.controller.js
src/services/centre.service.js
```

Implémenter au minimum :

```text
GET /api/centres
GET /api/centres/:id
```

### Questions

1. Comment organiseriez-vous les fichiers ?
2. Quelle fonction doit se trouver dans le service ?
3. Quelle fonction doit se trouver dans le contrôleur ?
4. Où doit être définie la route ?

---

# 36. Exercice 2 — Compléter le CRUD des donneurs

Compléter l'architecture précédente afin d'obtenir :

```text
GET    /api/donneurs
GET    /api/donneurs/:id
POST   /api/donneurs
PUT    /api/donneurs/:id
PATCH  /api/donneurs/:id
DELETE /api/donneurs/:id
```

Les opérations doivent être réparties correctement entre :

```text
routes
controllers
services
```

---

# 37. Exercice 3 — Recherche par groupe sanguin

Ajouter :

```text
GET /api/donneurs?groupeSanguin=A+
```

Le résultat doit contenir uniquement les donneurs appartenant au groupe demandé.

Tester également :

```text
GET /api/donneurs?groupeSanguin=O+
```

### Question

Dans quelle partie de l'architecture doit être placée la logique de filtrage ?

Justifiez votre réponse.

---

# 38. Exercice 4 — Validation simple

Avant l'installation de la bibliothèque **Zod** dans le prochain TP, réaliser une première validation manuelle.

Lors de la création d'un donneur :

```json
{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}
```

vérifier que :

* `nom` est présent ;
* `prenom` est présent ;
* `groupeSanguin` est présent.

Si une donnée manque, retourner :

```http
400 Bad Request
```

avec :

```json
{
    "message": "Données invalides"
}
```

### Question

Pourquoi cette méthode de validation risque-t-elle de devenir difficile à maintenir lorsque l'application grandit ?

---

# 39. Tests à réaliser

L'étudiant doit tester au minimum les requêtes suivantes :

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

### Créer

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}'
```

### Modifier

```bash
curl -X PUT http://localhost:3000/api/donneurs/1 \
-H "Content-Type: application/json" \
-d '{
    "nom": "Ben Salem",
    "prenom": "Ahmed",
    "groupeSanguin": "A+"
}'
```

### Supprimer

```bash
curl -X DELETE http://localhost:3000/api/donneurs/1
```

---

# 40. Bilan du TP

À ce stade, nous avons installé et utilisé **Express.js** pour construire notre première API REST.

Nous avons appris à :

* installer Express avec npm ;
* créer une application Express ;
* démarrer un serveur ;
* créer des routes ;
* utiliser les méthodes HTTP ;
* récupérer `req.params` ;
* récupérer `req.query` ;
* récupérer `req.body` ;
* retourner des réponses JSON ;
* utiliser les codes HTTP ;
* construire un CRUD ;
* organiser le projet en routes, contrôleurs et services.

L'architecture actuelle est :

```text
Client
   │
   ▼
Express
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
Données en mémoire
```

Elle sera progressivement transformée en :

```text
Client
   │
   ▼
Routes
   │
   ▼
Validation
   │
   ▼
Controllers
   │
   ▼
Services
   │
   ▼
Repositories
   │
   ▼
Prisma
   │
   ▼
MySQL
```

---

# 41. Travail pratique à faire à la maison

## Sujet : API REST des centres de collecte

Compléter l'API SangConnect en ajoutant la gestion des centres de collecte.

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

L'architecture devra respecter :

```text
Route
   ↓
Controller
   ↓
Service
   ↓
Données
```

Ajouter également :

```text
GET /api/centres?ville=Sousse
```

pour rechercher les centres d'une ville.

### Travail demandé

L'étudiant doit fournir :

1. l'arborescence du projet ;
2. les fichiers `routes`, `controllers` et `services` ;
3. les différentes routes REST ;
4. les tests `curl` ;
5. quelques captures d'écran des résultats ;
6. une courte explication de l'architecture retenue.

### Question de réflexion

Pourquoi est-il préférable de séparer :

```text
Routes
Controllers
Services
```

plutôt que de placer toute la logique dans `server.js` ?

---

# 42. Préparation du prochain TP

Dans ce TP, nous avons volontairement réalisé une première validation **manuelle**.

Dans le TP suivant, nous allons résoudre ce problème avec une bibliothèque spécialisée :

```text
Zod
```

Nous apprendrons notamment à valider :

```text
req.body
req.params
req.query
```

et à mettre en place une gestion centralisée des erreurs.

L'architecture deviendra :

```text
Route
   ↓
Validation
   ↓
Controller
   ↓
Service
   ↓
Données
```

Puis, dans l'atelier consacré à la persistance :

```text
Route
   ↓
Validation
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
