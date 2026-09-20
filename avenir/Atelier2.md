# Atelier 2 — Modules, programmation asynchrone et architecture en couches

Module 1 : Moteur Node.js et architecture applicative

TP universitaire

Durée : 3 heures

Node.js

Projet fil rouge : SangConnect — API de gestion des dons de sang

## 1. Présentation de l’atelier

Dans l’atelier 1, les étudiants ont créé un serveur HTTP simple avec le module natif `node:http`. Cependant, le code du serveur devient rapidement difficile à maintenir lorsque le nombre de routes et les fonctionnalités augmentent.

Dans cet atelier, nous allons améliorer progressivement l’organisation du projet en introduisant :

* les modules JavaScript ;

* les exports et imports ;

* les fonctions asynchrones ;

* les `Promises` ;

* `async/await` ;

* la gestion des erreurs ;

* l’architecture en couches ;

* les premières notions de routes, contrôleurs et services.

Nous continuerons à utiliser Node.js sans framework afin de comprendre les mécanismes fondamentaux avant d’introduire Express ou Fastify.

# 2. Objectifs pédagogiques

À la fin de l’atelier, l’étudiant sera capable de :

* organiser une application Node.js en plusieurs modules ;

* utiliser `export` et `import` ;

* expliquer le principe d’une `Promise` ;

* utiliser `async/await` ;

* gérer les erreurs dans une fonction asynchrone ;

* distinguer une opération synchrone d’une opération asynchrone ;

* séparer le routage, le contrôleur et la logique métier ;

* créer une première architecture en couches ;

* expliquer les avantages et les limites de cette organisation.

# 3. Prérequis

L’étudiant doit avoir réalisé l’atelier 1 et disposer de connaissances en :

* JavaScript ;

* fonctions et objets ;

* programmation orientée objet ;

* HTTP ;

* commandes Linux ;

* utilisation de VS Code.

Le projet doit déjà contenir un serveur Node.js fonctionnel.

# 4. Résumé théorique

## 4.1. Pourquoi organiser le code en modules ?

Dans un petit programme, il est possible de placer tout le code dans un seul fichier :

```
server.js
```

Mais dans une véritable application Backend, plusieurs fonctionnalités doivent être gérées :

```
Gestion des donneurs
Gestion des centres
Gestion des dons
Gestion des utilisateurs
Authentification
Accès aux données
```

Placer toutes ces fonctionnalités dans un seul fichier entraîne plusieurs difficultés :

* fichier trop volumineux ;

* responsabilités mélangées ;

* maintenance difficile ;

* réutilisation limitée ;

* tests plus complexes ;

* travail en équipe moins efficace.

Les modules permettent de diviser l’application en unités logiques.

## 4.2. Architecture en couches

Une architecture en couches sépare les responsabilités de l’application.

```
Client HTTP
    |
    v
Routes
    |
    v
Controllers
    |
    v
Services
    |
    v
Data Access / Repositories
    |
    v
Base de données
```

### Responsabilité des différentes couches

|
Couche

|

Responsabilité

|
| --- | --- |
|

Routes

|

Associer une URL et une méthode HTTP à une fonctionnalité

|
|

Controllers

|

Recevoir la requête et construire la réponse

|
|

Services

|

Contenir la logique métier

|
|

Repositories

|

Accéder aux données

|
|

Base de données

|

Stocker les informations

|

Dans cet atelier, nous n’utiliserons pas encore de base de données. Nous simulerons les données avec des tableaux JavaScript.

# 5. Partie 1 — Organisation du projet

Durée indicative : 20 minutes

## Activité 1.1 — Examiner le projet existant

Accédez au projet :

Bash

```
cd ~/Documents/sang-connect
```

Affichez les fichiers :

Bash

```
find . -maxdepth 2 -type f
```

> Si le répertoire `node_modules` existe, vous pouvez l’exclure de l’analyse pour obtenir un affichage plus lisible.

### Travail demandé

Identifiez les fichiers suivants :

* `package.json` ;

* `server.js` ;

* `app.js` ;

* `event-loop.js`.

### Questions

1. Quel fichier démarre le serveur ?

2. Quel fichier contient les tests de l’Event Loop ?

3. Pourquoi est-il préférable de séparer le code du serveur et le code de test ?

4. Quels nouveaux répertoires pourrions-nous créer pour organiser l’application ?

## Activité 1.2 — Créer les répertoires

À la racine du projet, exécutez :

Bash

```
mkdir -p src/routes
mkdir -p src/controllers
mkdir -p src/services
mkdir -p src/repositories
```

La structure initiale devient :

```
sang-connect/
├── src/
│   ├── routes/
│   ├── controllers/
│   ├── services/
│   └── repositories/
├── app.js
├── server.js
├── package.json
└── package-lock.json
```

### Questions

1. Pourquoi créer plusieurs répertoires plutôt que plusieurs fichiers à la racine ?

2. Quelle responsabilité attribuez-vous à `services/` ?

3. Quelle responsabilité attribuez-vous à `repositories/` ?

4. Est-il obligatoire d’utiliser cette architecture pour tous les projets Node.js ?

> À retenir : l’architecture doit être adaptée à la taille et aux exigences du projet. Une petite application ne nécessite pas toujours une structure complexe.

# 6. Partie 2 — Modules ES Modules

Durée indicative : 25 minutes

## Activité 2.1 — Créer un module utilitaire

Créez le fichier :

```
src/utils.js
```

Ajoutez :

JavaScript

```
export function formatApplicationName(name) {
    return `[SangConnect] ${name}`;
}
```

Créez ensuite le fichier :

```
module-test.js
```

Ajoutez :

JavaScript

```
import { formatApplicationName } from "./src/utils.js";

const message = formatApplicationName("Serveur démarré");

console.log(message);
```

Exécutez :

Bash

```
node module-test.js
```

### Résultat attendu

```
[SangConnect] Serveur démarré
```

## 6.2. Analyse

Dans le fichier `utils.js` :

JavaScript

```
export function formatApplicationName(name) {
    return `[SangConnect] ${name}`;
}
```

Le mot-clé `export` rend la fonction disponible pour d’autres modules.

Dans `module-test.js` :

JavaScript

```
import { formatApplicationName } from "./src/utils.js";
```

Le mot-clé `import` permet d’utiliser la fonction exportée.

### Questions

1. Quel est le rôle de `export` ?

2. Quel est le rôle de `import` ?

3. Pourquoi l’extension `.js` est-elle indiquée dans le chemin du module ?

4. Quelle erreur peut apparaître si le chemin du fichier est incorrect ?

5. Pourquoi la fonction peut-elle être réutilisée dans plusieurs fichiers ?

## Activité 2.2 — Export nommé et export par défaut

Créez le fichier :

```
src/constants.js
```

Ajoutez :

JavaScript

```
export const APPLICATION_NAME = "SangConnect";

export const APPLICATION_VERSION = "1.0.0";
```

Importez les constantes :

JavaScript

```
import {
    APPLICATION_NAME,
    APPLICATION_VERSION
} from "./src/constants.js";

console.log(APPLICATION_NAME);
console.log(APPLICATION_VERSION);
```

### Question

Quelle différence existe-t-il entre :

JavaScript

```
export const APPLICATION_NAME = "SangConnect";
```

et un export par défaut ?

Faites une recherche dans la documentation JavaScript ou consultez les exemples fournis par l’enseignant pour proposer une réponse.

# 7. Partie 3 — Comprendre les Promises

Durée indicative : 30 minutes

## 7.1. Définition

Une Promise représente le résultat futur d’une opération asynchrone.

Elle peut se trouver dans trois états :

|
État

|

Signification

|
| --- | --- |
|

`pending`

|

Opération en cours

|
|

`fulfilled`

|

Opération réussie

|
|

`rejected`

|

Opération échouée

|

Une Promise ne garantit pas que l’opération réussira. Elle permet de représenter son résultat futur ou son échec.

## Activité 3.1 — Créer une Promise simple

Créez :

```
promises.js
```

Ajoutez :

JavaScript

```
const operation = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("Opération terminée");
    }, 1000);
});

operation.then((result) => {
    console.log(result);
});
```

Exécutez :

Bash

```
node promises.js
```

### Questions

1. Quel est le rôle de `resolve()` ?

2. Quel est le rôle de `reject()` ?

3. Pourquoi le message apparaît-il après environ une seconde ?

4. Quel est le rôle de `.then()` ?

5. Que se passe-t-il si la Promise est rejetée ?

## Activité 3.2 — Gérer le rejet

Modifiez le code :

JavaScript

```
const operation = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(new Error("Une erreur est survenue"));
    }, 1000);
});

operation
    .then((result) => {
        console.log(result);
    })
    .catch((error) => {
        console.error("Erreur :", error.message);
    });
```

### Questions

1. Dans quel cas le bloc `.then()` est-il exécuté ?

2. Dans quel cas le bloc `.catch()` est-il exécuté ?

3. Pourquoi faut-il traiter les erreurs asynchrones ?

4. Que risque-t-on si un rejet n’est pas correctement traité ?

# 8. Partie 4 — Utiliser `async/await`

Durée indicative : 25 minutes

## 8.1. Présentation

`async/await` fournit une syntaxe permettant d’écrire du code utilisant des Promises d’une manière plus lisible.

Une fonction déclarée avec `async` retourne une Promise.

Le mot-clé `await` permet d’attendre le résultat d’une Promise dans une fonction asynchrone.

## Activité 4.1 — Transformer une Promise

Créez :

```
async-await.js
```

Ajoutez :

JavaScript

```
function waitOneSecond() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve("Une seconde est passée");
        }, 1000);
    });
}

async function main() {
    const result = await waitOneSecond();

    console.log(result);
}

main();
```

Exécutez :

Bash

```
node async-await.js
```

### Questions

1. Pourquoi la fonction `main()` est-elle déclarée avec `async` ?

2. Que fait `await waitOneSecond()` ?

3. Est-ce que `await` bloque l’ensemble du processus Node.js ?

4. Pourquoi est-il nécessaire de gérer les erreurs dans `main()` ?

## Activité 4.2 — Utiliser `try/catch`

Modifiez le code :

JavaScript

```
function loadData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(new Error("Impossible de charger les données"));
        }, 500);
    });
}

async function main() {
    try {
        const data = await loadData();

        console.log("Données :", data);
    } catch (error) {
        console.error("Erreur :", error.message);
    }
}

main();
```

### Questions

1. Pourquoi utilise-t-on `try/catch` ?

2. Quelle différence existe-t-il entre `.catch()` et `try/catch` ?

3. Que devrait retourner un service lorsqu’une opération métier échoue ?

4. Pourquoi ne faut-il pas toujours afficher directement les détails internes d’une erreur au client HTTP ?

# 9. Partie 5 — Créer le service des centres

Durée indicative : 30 minutes

Nous allons maintenant appliquer l’architecture en couches au projet SangConnect.

## Activité 5.1 — Créer les données fictives

Créez le fichier :

```
src/services/centre.service.js
```

Ajoutez :

JavaScript

```
const centres = [
    {
        id: 1,
        name: "Centre de collecte Tunis",
        city: "Tunis"
    },
    {
        id: 2,
        name: "Centre de collecte Sousse",
        city: "Sousse"
    },
    {
        id: 3,
        name: "Centre de collecte Sfax",
        city: "Sfax"
    }
];

export function getAllCentres() {
    return centres;
}
```

Cette première version retourne un tableau de données fictives.

## Activité 5.2 — Créer un contrôleur

Créez le fichier :

```
src/controllers/centre.controller.js
```

Ajoutez :

JavaScript

```
import { getAllCentres } from "../services/centre.service.js";

export function getCentres(request, response) {
    const centres = getAllCentres();

    response.statusCode = 200;
    response.setHeader(
        "Content-Type",
        "application/json; charset=utf-8"
    );

    response.end(JSON.stringify({
        centres: centres
    }));
}
```

### Analyse

Le contrôleur :

1. importe le service ;

2. appelle la fonction métier ;

3. prépare la réponse HTTP ;

4. retourne les données au format JSON.

### Questions

1. Le contrôleur contient-il directement les données des centres ?

2. Quelle fonction est responsable de récupérer les centres ?

3. Quel est l’intérêt de séparer le service du contrôleur ?

4. Si les données sont ensuite récupérées depuis PostgreSQL, quelle couche devra principalement être modifiée ?

## Activité 5.3 — Créer le module de routage

Créez :

```
src/routes/centre.routes.js
```

Ajoutez :

JavaScript

```
import { getCentres } from "../controllers/centre.controller.js";

export function handleCentreRoutes(request, response) {
    if (
        request.method === "GET" &&
        request.url === "/api/centres"
    ) {
        getCentres(request, response);
        return true;
    }

    return false;
}
```

La fonction retourne :

* `true` si la route a été traitée ;

* `false` si aucune route ne correspond.

# 10. Partie 6 — Intégrer les routes dans le serveur

Durée indicative : 25 minutes

## Activité 6.1 — Modifier `server.js`

Modifiez le fichier `server.js` :

JavaScript

```
import http from "node:http";

import { handleCentreRoutes } from "./src/routes/centre.routes.js";

const PORT = 3000;

function sendJson(response, statusCode, data) {
    response.statusCode = statusCode;

    response.setHeader(
        "Content-Type",
        "application/json; charset=utf-8"
    );

    response.end(JSON.stringify(data));
}

const server = http.createServer((request, response) => {
    const routeHandled = handleCentreRoutes(request, response);

    if (routeHandled) {
        return;
    }

    if (request.method === "GET" && request.url === "/api/health") {
        sendJson(response, 200, {
            status: "ok",
            application: "SangConnect"
        });

        return;
    }

    sendJson(response, 404, {
        error: "Route introuvable"
    });
});

server.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

## Activité 6.2 — Tester la route

Démarrez le serveur :

Bash

```
node server.js
```

Dans un autre terminal :

Bash

```
curl -i http://localhost:3000/api/centres
```

### Résultat attendu

JSON

```
{
  "centres": [
    {
      "id": 1,
      "name": "Centre de collecte Tunis",
      "city": "Tunis"
    },
    {
      "id": 2,
      "name": "Centre de collecte Sousse",
      "city": "Sousse"
    },
    {
      "id": 3,
      "name": "Centre de collecte Sfax",
      "city": "Sfax"
    }
  ]
}
```

### Questions

1. Quelle fonction est exécutée pour `/api/centres` ?

2. Quelle fonction récupère les centres ?

3. Pourquoi la route retourne-t-elle `true` ?

4. Que se passe-t-il lorsque l’URL ne correspond pas à la route des centres ?

5. Quelle responsabilité reste encore dans `server.js` ?

# 11. Partie 7 — Améliorer la séparation des responsabilités

Durée indicative : 20 minutes

Dans la version précédente, `server.js` contient encore :

* la création du serveur ;

* la gestion de la route `/api/health` ;

* la gestion des erreurs 404 ;

* la fonction `sendJson()`.

Nous allons déplacer progressivement certaines responsabilités.

## Activité 7.1 — Créer un utilitaire de réponse

Créez :

```
src/utils/http-response.js
```

Ajoutez :

JavaScript

```
export function sendJson(response, statusCode, data) {
    response.statusCode = statusCode;

    response.setHeader(
        "Content-Type",
        "application/json; charset=utf-8"
    );

    response.end(JSON.stringify(data));
}
```

## Activité 7.2 — Utiliser l’utilitaire dans le contrôleur

Modifiez `centre.controller.js` :

JavaScript

```
import { getAllCentres } from "../services/centre.service.js";
import { sendJson } from "../utils/http-response.js";

export function getCentres(request, response) {
    const centres = getAllCentres();

    sendJson(response, 200, {
        centres: centres
    });
}
```

### Questions

1. Quel code a été supprimé du contrôleur ?

2. Pourquoi centraliser la création des réponses JSON ?

3. Dans quelles autres routes l’utilitaire pourra-t-il être utilisé ?

4. Quels problèmes peuvent apparaître si plusieurs développeurs créent chacun leur propre fonction de réponse ?

# 12. Partie 8 — Introduction au traitement asynchrone dans un service

Durée indicative : 15 minutes

Pour le moment, les données sont récupérées de manière synchrone à partir d’un tableau en mémoire.

Dans une application réelle, les données peuvent être récupérées :

* depuis PostgreSQL ;

* depuis MongoDB ;

* depuis un service distant ;

* depuis un fichier ;

* depuis une API externe.

Ces opérations peuvent être asynchrones.

## Activité 8.1 — Transformer le service

Modifiez `centre.service.js` :

JavaScript

```
const centres = [
    {
        id: 1,
        name: "Centre de collecte Tunis",
        city: "Tunis"
    },
    {
        id: 2,
        name: "Centre de collecte Sousse",
        city: "Sousse"
    },
    {
        id: 3,
        name: "Centre de collecte Sfax",
        city: "Sfax"
    }
];

export async function getAllCentres() {
    return centres;
}
```

La fonction retourne maintenant une Promise, car elle est déclarée avec `async`.

## Activité 8.2 — Adapter le contrôleur

Modifiez le contrôleur :

JavaScript

```
import { getAllCentres } from "../services/centre.service.js";
import { sendJson } from "../utils/http-response.js";

export async function getCentres(request, response) {
    try {
        const centres = await getAllCentres();

        sendJson(response, 200, {
            centres: centres
        });
    } catch (error) {
        console.error("Erreur lors de la récupération des centres :", error);

        sendJson(response, 500, {
            error: "Erreur interne du serveur"
        });
    }
}
```

### Questions

1. Pourquoi le contrôleur est-il maintenant déclaré avec `async` ?

2. Quel est le rôle de `await` ?

3. Pourquoi avons-nous ajouté `try/catch` ?

4. Quel code HTTP est utilisé en cas d’erreur interne ?

5. Pourquoi le message envoyé au client est-il différent du message affiché dans le terminal ?

# 13. Partie 9 — Exercices de consolidation

## Exercice 1 — Créer une route `/api/centres/{id}`

Le serveur natif ne dispose pas automatiquement d’un système de paramètres de route comme Express.

Vous devez utiliser une URL de la forme :

```
/api/centres/2
```

### Travail demandé

1. Récupérer l’identifiant dans l’URL.

2. Rechercher le centre correspondant.

3. Retourner le centre au format JSON.

4. Retourner une erreur `404` si le centre n’existe pas.

5. Retourner une erreur `400` si l’identifiant est invalide.

### Exemple de réponse

JSON

```
{
  "id": 2,
  "name": "Centre de collecte Sousse",
  "city": "Sousse"
}
```

### Questions guidées

1. Comment découper une URL en JavaScript ?

2. Comment convertir une chaîne en nombre ?

3. Comment rechercher un élément dans un tableau ?

4. Pourquoi faut-il vérifier que l’identifiant est valide ?

5. Quelle est la différence entre une donnée absente et une donnée invalide ?

## Exercice 2 — Ajouter une fonction de recherche

Dans le service, créez :

JavaScript

```
export async function getCentreById(id) {
    // À compléter
}
```

La fonction doit rechercher un centre par son identifiant.

Le contrôleur devra utiliser cette fonction au lieu d’accéder directement au tableau.

### Objectif architectural

```
Route
  ↓
Controller
  ↓
Service
  ↓
Données fictives
```

Le contrôleur ne doit pas contenir la logique de recherche.

## Exercice 3 — Simuler une erreur

Modifiez temporairement le service afin de provoquer une erreur :

JavaScript

```
export async function getAllCentres() {
    throw new Error("Erreur simulée");
}
```

Testez la route :

Bash

```
curl -i http://localhost:3000/api/centres
```

### Questions

1. Le serveur doit-il s’arrêter lorsqu’une erreur est générée ?

2. Quel code HTTP doit être retourné ?

3. Quelle information doit être conservée dans les logs ?

4. Pourquoi faut-il éviter d’envoyer la trace complète de l’erreur au client en production ?

N’oubliez pas de restaurer le service après le test.

# 14. Partie 10 — Synthèse et discussion

## 14.1. Architecture obtenue

À la fin de l’atelier, l’organisation peut être la suivante :

```
sang-connect/
├── src/
│   ├── controllers/
│   │   └── centre.controller.js
│   ├── routes/
│   │   └── centre.routes.js
│   ├── services/
│   │   └── centre.service.js
│   └── utils/
│       ├── http-response.js
│       └── constants.js
├── server.js
├── app.js
├── event-loop.js
├── module-test.js
├── async-await.js
├── promises.js
├── package.json
└── package-lock.json
```

## 14.2. Questions de synthèse

Répondez de manière argumentée :

1. Pourquoi est-il déconseillé de placer la logique métier dans le serveur HTTP ?

2. Quelle est la différence entre une route et un contrôleur ?

3. Quelle est la différence entre un contrôleur et un service ?

4. Pourquoi les services sont-ils généralement plus faciles à tester lorsqu’ils sont indépendants du protocole HTTP ?

5. Quel est l’intérêt de `async/await` dans une application qui utilise une base de données ?

6. Pourquoi une erreur doit-elle être gérée à plusieurs niveaux ?

7. Quels avantages une architecture en couches apporte-t-elle à une équipe de développeurs ?

# 15. Travail pratique à réaliser à la maison

## Sujet : Gestion des donneurs — première version

Vous devez ajouter une fonctionnalité de gestion des donneurs au projet SangConnect.

### Fonctionnalité 1 — Lister les donneurs

Créer la route :

http

```
GET /api/donneurs
```

Les données peuvent être fictives :

JSON

```
{
  "donneurs": [
    {
      "id": 1,
      "firstName": "Ahmed",
      "lastName": "Ben Ali",
      "bloodType": "A+"
    },
    {
      "id": 2,
      "firstName": "Sarra",
      "lastName": "Trabelsi",
      "bloodType": "O+"
    }
  ]
}
```

### Fonctionnalité 2 — Consulter un donneur

Créer :

http

```
GET /api/donneurs/1
```

Retourner le donneur correspondant.

Si le donneur n’existe pas :

JSON

```
{
  "error": "Donneur introuvable"
}
```

Le code HTTP attendu est :

```
404 Not Found
```

### Fonctionnalité 3 — Architecture imposée

Organisez votre code comme suit :

```
src/
├── controllers/
│   └── donor.controller.js
├── routes/
│   └── donor.routes.js
└── services/
    └── donor.service.js
```

Les responsabilités doivent être séparées :

* Route : détecter la route ;

* Contrôleur : gérer la requête et la réponse ;

* Service : gérer la logique de recherche des donneurs.

### Fonctionnalité 4 — Gestion des erreurs

Simulez une erreur dans le service et vérifiez que :

* le serveur ne s’arrête pas brutalement ;

* une erreur est enregistrée dans le terminal ;

* le client reçoit une réponse HTTP adaptée ;

* les détails internes ne sont pas exposés dans la réponse.

## Livrable

Ajoutez au fichier `README.md` :

* la structure de votre projet ;

* les routes développées ;

* les commandes de test ;

* des exemples de réponses ;

* un schéma représentant le cheminement d’une requête ;

* une explication de l’utilisation de `async/await` ;

* une explication du rôle du service.

# 16. Critères d’évaluation

|
Critère

|

Points

|
| --- | --- |
|

Utilisation correcte des modules

|

3

|
|

Utilisation de Promises et `async/await`

|

3

|
|

Gestion des erreurs

|

3

|
|

Architecture routes / contrôleurs / services

|

4

|
|

Fonctionnalités donneurs

|

3

|
|

Codes HTTP et réponses JSON

|

2

|
|

Organisation du code

|

1

|
|

Documentation README

|

1

|
|

Total

|

20

|

# 17. À retenir

* Les modules permettent de diviser une application Node.js en unités réutilisables.

* ES Modules utilise `import` et `export`.

* Une Promise représente le résultat futur d’une opération.

* `async/await` facilite l’écriture du code asynchrone.

* Les erreurs asynchrones doivent être traitées avec `.catch()` ou `try/catch`.

* Le contrôleur gère les échanges HTTP, tandis que le service contient la logique métier.

* Une architecture en couches facilite la maintenance et les tests.

* Le serveur HTTP natif permet de comprendre les mécanismes fondamentaux, mais demande beaucoup de code manuel.

* Les frameworks tels qu’Express et Fastify simplifient le routage et la gestion des requêtes.

## Transition vers l’atelier 3

Le prochain atelier sera consacré à :

Event Loop, I/O non bloquantes, gestion des erreurs et introduction à Express

Nous approfondirons le fonctionnement asynchrone de Node.js et nous commencerons à comparer le serveur HTTP natif avec un framework Backend.
