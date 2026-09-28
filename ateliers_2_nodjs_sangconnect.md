# Atelier 2 — Modules, programmation asynchrone et architecture en couches

**Matière :** Ingénierie Backend
**Niveau :** 4e année cycle ingénieur
**Durée :** 3 heures
**Projet fil rouge :** SangConnect — API de gestion des dons de sang
**Technologies :** Node.js, npm, JavaScript ES Modules

---

## 1. Présentation de l’atelier

Dans l’atelier 1, nous avons :

* installé et configuré un projet Node.js ;
* créé un serveur HTTP avec `node:http` ;
* manipulé les requêtes et réponses HTTP ;
* créé plusieurs routes ;
* produit des réponses JSON ;
* utilisé `curl` pour tester notre serveur.

Cependant, le fichier `server.js` commence déjà à contenir plusieurs responsabilités :

```text
server.js
 ├── démarrage du serveur
 ├── définition des routes
 ├── traitement des requêtes
 ├── préparation des réponses
 └── logique applicative
```

Cette organisation devient rapidement difficile à maintenir.

Dans cet atelier, nous allons donc apprendre à :

* découper une application Node.js en modules ;
* utiliser `import` et `export` ;
* comprendre les Promises ;
* utiliser `async/await` ;
* gérer les erreurs ;
* séparer les responsabilités ;
* introduire une architecture en couches ;
* préparer l’utilisation future d’Express.

---

# 2. Objectifs pédagogiques

À la fin de cet atelier, l’étudiant doit être capable de :

* créer et utiliser des modules ES ;
* exporter une fonction ou une donnée ;
* importer un module local ;
* expliquer le principe d’une Promise ;
* distinguer `pending`, `fulfilled` et `rejected` ;
* utiliser `async/await` ;
* gérer une erreur avec `try/catch` ;
* distinguer logique de routage et logique métier ;
* organiser une application en couches ;
* créer une première architecture `routes → controllers → services → repositories` ;
* appliquer cette architecture à SangConnect.

---

# 3. Rappel — Architecture cible

Nous allons progressivement arriver à une structure similaire à :

```text
sangconnect/
├── package.json
├── src/
│   ├── server.js
│   ├── routes/
│   │   └── centre.routes.js
│   ├── controllers/
│   │   └── centre.controller.js
│   ├── services/
│   │   └── centre.service.js
│   ├── repositories/
│   │   └── centre.repository.js
│   └── utils/
│       └── http.js
└── README.md
```

Cette architecture sera enrichie dans les prochains ateliers.

---

# 4. Activité 1 — Comprendre les modules Node.js

## Objectif

Comprendre pourquoi une application backend doit être découpée en plusieurs modules.

### 4.1. Première situation

Imaginez que `server.js` contienne :

```javascript
const centres = [
  {
    id: 1,
    nom: "Centre Tunis",
    ville: "Tunis"
  },
  {
    id: 2,
    nom: "Centre Sousse",
    ville: "Sousse"
  }
];
```

Puis :

```javascript
function getCentreById(id) {
  return centres.find(centre => centre.id === id);
}
```

Puis encore :

```javascript
function sendJson(res, statusCode, data) {
  // ...
}
```

Et finalement plusieurs dizaines de routes.

### Questions

1. Est-il facile de retrouver la fonction responsable de la recherche d’un centre ?
2. Que se passe-t-il si plusieurs fichiers ont besoin de `getCentreById()` ?
3. Pourquoi serait-il intéressant de placer cette fonction dans un fichier séparé ?
4. Quel pourrait être le rôle d’un module ?

---

# 5. Activité 2 — Créer son premier module ES

## Objectif

Créer un module contenant une fonction réutilisable.

Créez :

```text
src/utils/format.js
```

Ajoutez :

```javascript
export function formatCentreName(nom, ville) {
  return `${nom} - ${ville}`;
}
```

## 5.1. Importer le module

Dans `src/server.js` :

```javascript
import { formatCentreName } from "./utils/format.js";
```

Utilisez la fonction :

```javascript
const nom = formatCentreName("Centre Tunis", "Tunis");

console.log(nom);
```

Lancez :

```bash
node src/server.js
```

### Résultat attendu

```text
Centre Tunis - Tunis
```

---

# 6. Activité 3 — `export` et `import`

## Objectif

Comprendre les différentes formes d’exportation.

### 6.1. Export nommé

Dans un fichier :

```javascript
export function addition(a, b) {
  return a + b;
}
```

Import :

```javascript
import { addition } from "./math.js";
```

### 6.2. Plusieurs exports

Créez :

```text
src/utils/math.js
```

Ajoutez :

```javascript
export function addition(a, b) {
  return a + b;
}

export function multiplication(a, b) {
  return a * b;
}

export function soustraction(a, b) {
  return a - b;
}
```

Importez :

```javascript
import {
  addition,
  multiplication,
  soustraction
} from "./utils/math.js";
```

### Questions

1. Pourquoi les accolades sont-elles utilisées lors de l’import ?
2. Peut-on exporter plusieurs fonctions depuis un même fichier ?
3. Quel est l’intérêt de regrouper plusieurs fonctions liées dans un module ?

---

# 7. Activité 4 — Comprendre les fonctions asynchrones

## Objectif

Comprendre pourquoi l’asynchronisme est important dans une application backend.

Considérons :

```javascript
console.log("Début");

setTimeout(() => {
  console.log("Opération terminée");
}, 2000);

console.log("Fin");
```

Exécutez le programme.

### Résultat attendu

```text
Début
Fin
Opération terminée
```

### Questions

1. Pourquoi `"Fin"` apparaît-il avant `"Opération terminée"` ?
2. Le programme est-il bloqué pendant les deux secondes ?
3. Que représente l’opération programmée avec `setTimeout()` ?
4. Pourquoi ce comportement est-il intéressant pour un serveur ?

---

# 8. Activité 5 — Comprendre les Promises

## Objectif

Comprendre le fonctionnement d’une Promise.

Une Promise représente une opération dont le résultat sera disponible ultérieurement.

Elle possède trois états conceptuels :

```text
pending
   │
   ├── fulfilled
   │
   └── rejected
```

## 8.1. Première Promise

Créez :

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Centre récupéré");
  }, 1000);
});
```

Utilisez :

```javascript
promise.then(result => {
  console.log(result);
});
```

### Questions

1. Que contient la Promise au début ?
2. Que fait `resolve()` ?
3. Que fait `reject()` ?
4. Que se passe-t-il après une seconde ?
5. Pourquoi utilise-t-on `.then()` ?

---

# 9. Activité 6 — Promise réussie et Promise rejetée

## 6.1. Promise réussie

```javascript
function getCentre() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        id: 1,
        nom: "Centre Tunis",
        ville: "Tunis"
      });
    }, 1000);
  });
}
```

Utilisez :

```javascript
getCentre()
  .then(centre => {
    console.log(centre);
  });
```

### Résultat attendu

```text
{
  id: 1,
  nom: "Centre Tunis",
  ville: "Tunis"
}
```

## 6.2. Promise rejetée

Modifiez la fonction :

```javascript
function getCentre() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error("Impossible de récupérer le centre"));
    }, 1000);
  });
}
```

Utilisez :

```javascript
getCentre()
  .then(centre => {
    console.log(centre);
  })
  .catch(error => {
    console.error(error.message);
  });
```

### Questions

1. Pourquoi utilise-t-on `reject()` ?
2. Quel est le rôle de `.catch()` ?
3. Quelle différence existe-t-il entre une valeur retournée et une erreur ?
4. Pourquoi la gestion des erreurs est-elle importante dans une API backend ?

---

# 10. Activité 7 — Utiliser `async/await`

## Objectif

Simplifier l’écriture du code asynchrone.

Reprenez la fonction précédente :

```javascript
function getCentre() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        id: 1,
        nom: "Centre Tunis",
        ville: "Tunis"
      });
    }, 1000);
  });
}
```

Créez une fonction asynchrone :

```javascript
async function afficherCentre() {
  const centre = await getCentre();

  console.log(centre);
}

afficherCentre();
```

### Questions

1. Que signifie le mot-clé `async` ?
2. Que signifie `await` ?
3. Pourquoi `await` ne bloque-t-il pas tout le serveur pendant l’attente ?
4. Que se passe-t-il si la Promise est rejetée ?

---

# 11. Activité 8 — Gérer les erreurs avec `try/catch`

Modifiez la fonction :

```javascript
function getCentre() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error("Centre indisponible"));
    }, 1000);
  });
}
```

Puis :

```javascript
async function afficherCentre() {
  try {
    const centre = await getCentre();

    console.log(centre);
  } catch (error) {
    console.error("Erreur :", error.message);
  }
}

afficherCentre();
```

### Questions

1. Quel est le rôle de `try` ?
2. Quel est le rôle de `catch` ?
3. Quelle variable contient l’erreur ?
4. Pourquoi ne doit-on pas laisser les erreurs asynchrones sans traitement ?

---

# 12. Activité 9 — Créer un repository de centres

## Objectif

Commencer à séparer les données de la logique applicative.

Créez :

```text
src/repositories/centre.repository.js
```

Ajoutez :

```javascript
const centres = [
  {
    id: 1,
    nom: "Centre de transfusion de Tunis",
    ville: "Tunis"
  },
  {
    id: 2,
    nom: "Centre régional de Sousse",
    ville: "Sousse"
  },
  {
    id: 3,
    nom: "Centre régional de Sfax",
    ville: "Sfax"
  }
];

export async function findAll() {
  return centres;
}

export async function findById(id) {
  return centres.find(centre => centre.id === id);
}
```

### Questions

1. Où sont maintenant stockées les données ?
2. Pourquoi ce fichier est-il appelé `repository` ?
3. Le repository connaît-il HTTP ?
4. Le repository connaît-il `req` et `res` ?
5. Pourquoi cette séparation est-elle intéressante ?

---

# 13. Activité 10 — Créer le service métier

## Objectif

Créer une couche contenant la logique métier.

Créez :

```text
src/services/centre.service.js
```

Ajoutez :

```javascript
import {
  findAll,
  findById
} from "../repositories/centre.repository.js";

export async function getAllCentres() {
  return await findAll();
}

export async function getCentreById(id) {
  const centre = await findById(id);

  if (!centre) {
    throw new Error("Centre introuvable");
  }

  return centre;
}
```

### Questions

1. Quel est le rôle du service ?
2. Pourquoi le service utilise-t-il le repository ?
3. Pourquoi le service ne manipule-t-il pas directement `res` ?
4. Où doit être placée une règle métier ?
5. Pourquoi séparer le service du repository ?

---

# 14. Activité 11 — Créer le contrôleur

## Objectif

Faire le lien entre HTTP et la logique métier.

Créez :

```text
src/controllers/centre.controller.js
```

Ajoutez :

```javascript
import {
  getAllCentres,
  getCentreById
} from "../services/centre.service.js";

import { sendJson } from "../utils/http.js";

export async function listCentres(req, res) {
  try {
    const centres = await getAllCentres();

    sendJson(res, 200, {
      data: centres
    });
  } catch (error) {
    sendJson(res, 500, {
      error: "Erreur interne du serveur"
    });
  }
}

export async function showCentre(req, res, id) {
  try {
    const centre = await getCentreById(id);

    sendJson(res, 200, {
      data: centre
    });
  } catch (error) {
    sendJson(res, 404, {
      error: error.message
    });
  }
}
```

### Questions

1. Quel est le rôle du contrôleur ?
2. Pourquoi le contrôleur connaît-il HTTP ?
3. Pourquoi appelle-t-il le service ?
4. Pourquoi le service ne doit-il pas appeler `sendJson()` ?
5. Où est gérée l’erreur « Centre introuvable » ?

---

# 15. Activité 12 — Créer les routes des centres

## Objectif

Connecter les URL aux contrôleurs.

Créez :

```text
src/routes/centre.routes.js
```

Pour l’instant, nous utilisons toujours le module HTTP natif.

Ajoutez :

```javascript
import {
  listCentres,
  showCentre
} from "../controllers/centre.controller.js";

export async function handleCentreRoutes(req, res) {
  if (req.url === "/api/centres" && req.method === "GET") {
    await listCentres(req, res);
    return true;
  }

  const match = req.url.match(/^\/api\/centres\/(\d+)$/);

  if (match && req.method === "GET") {
    const id = Number(match[1]);

    await showCentre(req, res, id);
    return true;
  }

  return false;
}
```

### Questions

1. Quel est le rôle de `handleCentreRoutes()` ?
2. Pourquoi cette fonction retourne-t-elle `true` ou `false` ?
3. Que représente `match[1]` ?
4. Pourquoi utilise-t-on `Number()` ?
5. Pourquoi la route `/api/centres/1` doit-elle être distinguée de `/api/centres` ?

---

# 16. Activité 13 — Connecter les routes au serveur

## Objectif

Faire communiquer les différentes couches.

Dans `src/server.js`, importez :

```javascript
import http from "node:http";
import { handleCentreRoutes } from "./routes/centre.routes.js";
import { sendJson } from "./utils/http.js";

const port = 3000;

const server = http.createServer(async (req, res) => {
  const handled = await handleCentreRoutes(req, res);

  if (handled) {
    return;
  }

  if (req.url === "/" && req.method === "GET") {
    sendJson(res, 200, {
      message: "Bienvenue dans SangConnect"
    });

    return;
  }

  sendJson(res, 404, {
    error: "Route non trouvée",
    path: req.url,
    method: req.method
  });
});

server.listen(port, () => {
  console.log(`Serveur démarré sur http://localhost:${port}`);
});
```

---

# 17. Activité 14 — Tester l’architecture

## 17.1. Tester la liste des centres

```bash
curl -i http://localhost:3000/api/centres
```

Résultat attendu :

```json
{
  "data": [
    {
      "id": 1,
      "nom": "Centre de transfusion de Tunis",
      "ville": "Tunis"
    }
  ]
}
```

La réponse complète doit contenir les trois centres.

## 17.2. Tester un centre

```bash
curl -i http://localhost:3000/api/centres/1
```

## 17.3. Tester un centre inexistant

```bash
curl -i http://localhost:3000/api/centres/999
```

La réponse attendue doit utiliser le code :

```text
404 Not Found
```

### Questions

1. Quelle couche reçoit la requête HTTP en premier ?
2. Quelle couche contient les données ?
3. Quelle couche contient la logique métier ?
4. Quelle couche produit la réponse HTTP ?
5. Peut-on remplacer les données fictives par une base de données sans modifier les routes ?

---

# 18. Activité 15 — Ajouter une recherche de centres

## Objectif

Introduire une première règle métier.

Dans :

```text
src/repositories/centre.repository.js
```

ajoutez :

```javascript
export async function searchByCity(city) {
  return centres.filter(
    centre => centre.ville.toLowerCase() === city.toLowerCase()
  );
}
```

Dans le service :

```javascript
import {
  findAll,
  findById,
  searchByCity
} from "../repositories/centre.repository.js";

export async function searchCentresByCity(city) {
  if (!city || city.trim() === "") {
    throw new Error("La ville est obligatoire");
  }

  return await searchByCity(city);
}
```

### Question

Pourquoi la vérification :

```javascript
if (!city || city.trim() === "")
```

doit-elle se trouver dans le service plutôt que dans le repository ?

---

# 19. Activité 16 — Simuler une erreur asynchrone

## Objectif

Comprendre le comportement d’une erreur provenant d’une opération asynchrone.

Modifiez temporairement `findAll()` :

```javascript
export async function findAll() {
  throw new Error("Base de données indisponible");
}
```

Testez :

```bash
curl -i http://localhost:3000/api/centres
```

### Questions

1. Quelle couche génère l’erreur ?
2. Quelle couche la récupère ?
3. Quel code HTTP doit être retourné ?
4. Pourquoi ne faut-il pas retourner le message technique complet d’une erreur interne au client dans une véritable application ?

Rétablissez ensuite le code initial.

---

# 20. Activité 17 — Observer l’architecture finale

À la fin de l’activité, vous devez obtenir une organisation proche de :

```text
sangconnect/
├── package.json
├── src/
│   ├── server.js
│   │
│   ├── routes/
│   │   └── centre.routes.js
│   │
│   ├── controllers/
│   │   └── centre.controller.js
│   │
│   ├── services/
│   │   └── centre.service.js
│   │
│   ├── repositories/
│   │   └── centre.repository.js
│   │
│   └── utils/
│       ├── http.js
│       └── format.js
│
└── README.md
```

## Responsabilités

```text
HTTP
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
Repositories
 │
 ▼
Données
```

### Tableau de synthèse

| Couche     | Responsabilité                     |
| ---------- | ---------------------------------- |
| Routes     | Identifier la route et la méthode  |
| Controller | Gérer la requête/réponse HTTP      |
| Service    | Contenir la logique métier         |
| Repository | Accéder aux données                |
| Utils      | Fonctions génériques réutilisables |

---

# 21. Activité 18 — Questions de synthèse

Répondez individuellement aux questions suivantes.

### Question 1

Quelle différence existe entre :

```javascript
function getCentre() {}
```

et :

```javascript
async function getCentre() {}
```

### Question 2

Quelle différence existe entre :

```javascript
const result = await operation();
```

et :

```javascript
operation().then(result => {});
```

### Question 3

Quels sont les trois états principaux d’une Promise ?

### Question 4

Quel est le rôle de `try/catch` avec `async/await` ?

### Question 5

Pourquoi faut-il éviter de mettre toute la logique dans `server.js` ?

### Question 6

Quelle est la responsabilité du repository ?

### Question 7

Quelle est la responsabilité du service ?

### Question 8

Quelle est la responsabilité du controller ?

### Question 9

Pourquoi cette architecture facilite-t-elle les tests ?

### Question 10

Quelle partie de l’architecture pourrait être remplacée lorsqu’une vraie base PostgreSQL sera introduite ?

---

# 22. Travail pratique à faire à la maison

## Sujet

### Gestion des donneurs — SangConnect

Vous devez reproduire l’architecture utilisée pour les centres afin de gérer les donneurs.

Créez :

```text
src/
├── routes/
│   └── donneur.routes.js
├── controllers/
│   └── donneur.controller.js
├── services/
│   └── donneur.service.js
└── repositories/
    └── donneur.repository.js
```

## 22.1. Données

Utilisez au minimum cinq donneurs fictifs.

Chaque donneur doit posséder :

```javascript
{
  id: 1,
  nom: "Ben Ali",
  prenom: "Ahmed",
  groupeSanguin: "O+",
  ville: "Tunis"
}
```

Les données doivent rester fictives.

## 22.2. Route `/api/donneurs`

Implémenter :

```text
GET /api/donneurs
```

Elle doit retourner la liste des donneurs.

## 22.3. Route `/api/donneurs/:id`

Implémenter :

```text
GET /api/donneurs/1
```

Elle doit retourner le donneur correspondant.

Si le donneur n’existe pas :

```text
404 Not Found
```

## 22.4. Recherche

Ajouter une recherche par ville :

```text
GET /api/donneurs?ville=Tunis
```

La recherche doit être réalisée dans la couche appropriée.

## 22.5. Gestion des erreurs

Prévoir au minimum :

* donneur inexistant ;
* ville vide ;
* erreur simulée du repository.

## 22.6. Tests

Tester :

```bash
curl -i http://localhost:3000/api/donneurs
```

```bash
curl -i http://localhost:3000/api/donneurs/1
```

```bash
curl -i http://localhost:3000/api/donneurs/999
```

### Livrables

```text
src/
├── routes/
├── controllers/
├── services/
└── repositories/
```

avec :

* le code source ;
* les données fictives ;
* les tests `curl` ;
* un `README.md` expliquant l’architecture.

---

# 23. Critères d’évaluation du travail maison

| Critère                   | Points |
| ------------------------- | -----: |
| Organisation en couches   |      4 |
| Repository                |      3 |
| Service                   |      3 |
| Controller                |      3 |
| Routes                    |      3 |
| Gestion des erreurs       |      2 |
| Tests HTTP                |      1 |
| README et qualité du code |      1 |
| **Total**                 | **20** |

---

# 24. Bilan de l’atelier

À la fin de cet atelier, vous devez avoir compris que l’objectif d’une architecture backend n’est pas simplement de faire fonctionner une route.

Il faut également organiser le code afin que chaque partie possède une responsabilité clairement définie.

Vous avez découvert :

* les modules ES ;
* `import` et `export` ;
* les opérations asynchrones ;
* les Promises ;
* `async/await` ;
* `try/catch` ;
* les repositories ;
* les services ;
* les controllers ;
* les routes ;
* la séparation des responsabilités.

L’architecture obtenue est :

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
Repositories
     │
     ▼
   Données
```
