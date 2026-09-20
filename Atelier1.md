# Atelier 1 — Découverte de Node.js et cycle de vie HTTP

Module 1 : Moteur Node.js et architecture applicative

Nous allons reprendre la structure de l’atelier Symfony en l’adaptant à Node.js, conformément au document de référence « Ingénierie Backend ».

La différence principale est que nous allons étudier directement le fonctionnement de Node.js, notamment :

* le moteur d’exécution JavaScript côté serveur ;

* la boucle d’événements (Event Loop) ;

* les opérations d’entrées/sorties non bloquantes ;

* le serveur HTTP natif de Node.js ;

* la gestion des modules avec npm ;

* les premières notions d’architecture Backend.

Le projet fil rouge reste SangConnect, une application de gestion des dons de sang. Les étudiants utiliseront Node.js avec JavaScript, puis pourront découvrir Express ou Fastify dans les ateliers suivants.


## Fiche pédagogique

|
Élément

|

Description

|
| --- | --- |
|

Formation

|

4ᵉ année cycle ingénieur en informatique

|
|

Matière

|

Ingénierie Backend

|
|

Module

|

Moteur Node.js et architecture applicative

|
|

Atelier

|

1 — Découverte de Node.js et cycle de vie HTTP

|
|

Durée

|

3 heures

|
|

Niveau

|

Intermédiaire / avancé

|
|

Prérequis

|

JavaScript, PHP, POO, SQL, bases de HTTP

|
|

Environnement

|

Ubuntu 24.04, VS Code

|
|

Technologie

|

Node.js, JavaScript, npm

|
|

Projet

|

SangConnect

|

# 1. Présentation de l’atelier

Node.js permet d’exécuter du JavaScript en dehors d’un navigateur. Il est particulièrement utilisé pour développer des applications Backend, des API REST et des serveurs Web.

Dans cet atelier, l’étudiant va créer un premier serveur HTTP avec Node.js, comprendre son fonctionnement et développer les premières routes de l’application SangConnect.

L’objectif est de comprendre les mécanismes fondamentaux avant d’utiliser un framework comme Express ou Fastify.

## 1.1. Objectifs pédagogiques

À la fin de l’atelier, l’étudiant sera capable de :

* vérifier l’installation de Node.js et npm ;

* créer un projet Node.js ;

* comprendre le rôle de `package.json` ;

* distinguer les modules CommonJS et ES Modules ;

* exécuter un programme JavaScript avec Node.js ;

* créer un serveur HTTP natif ;

* comprendre les notions de requête et de réponse HTTP ;

* identifier le rôle de l’Event Loop ;

* gérer plusieurs routes simples ;

* retourner une réponse JSON ;

* tester un serveur avec un navigateur et `curl`.

# 2. Contexte du projet fil rouge

## 2.1. Présentation de SangConnect

SangConnect est une application Backend destinée à la gestion des dons de sang.

L’application devra progressivement permettre de :

* gérer les donneurs ;

* gérer les centres de collecte ;

* enregistrer les dons ;

* consulter les stocks de sang ;

* gérer les utilisateurs ;

* sécuriser l’accès aux fonctionnalités ;

* exposer des services à travers une API REST.

Durant cet atelier, nous allons construire une première version très simple du serveur, sans base de données et sans framework.

## 2.2. Fonctionnalités initiales

Le serveur devra proposer les routes suivantes :

|
Méthode

|

Route

|

Fonction

|
| --- | --- | --- |
|

`GET`

|

`/`

|

Page d’accueil de l’API

|
|

`GET`

|

`/api/health`

|

Vérifier l’état du serveur

|
|

`GET`

|

`/api/info`

|

Afficher les informations de l’application

|
|

`GET`

|

`/api/centres/demo`

|

Retourner des centres fictifs

|

Les données seront statiques. La persistance avec PostgreSQL sera étudiée dans le module consacré à la persistance.

# 3. Organisation temporelle de l’atelier

|
Activité

|

Durée

|
| --- | --- |
|

Introduction et rappels théoriques

|

20 min

|
|

Vérification de l’environnement

|

20 min

|
|

Création du projet Node.js

|

20 min

|
|

Premier programme et modules

|

20 min

|
|

Création du serveur HTTP

|

35 min

|
|

Routes et réponses JSON

|

30 min

|
|

Event Loop et I/O non bloquantes

|

20 min

|
|

Exercices et synthèse

|

15 min

|
|

Total

|

180 min

|

# 4. Résumé théorique : les concepts fondamentaux

## 4.1. Qu’est-ce que Node.js ?

Node.js est un environnement d’exécution permettant d’exécuter du JavaScript côté serveur.

Il repose notamment sur :

* un moteur JavaScript ;

* un système d’accès aux fichiers et au réseau ;

* une boucle d’événements (Event Loop) ;

* des API permettant de réaliser des opérations asynchrones ;

* un système de modules ;

* un gestionnaire de paquets, npm.

### Node.js dans une architecture Backend

```
Client Web / Application mobile
            |
            | Requête HTTP
            v
       Serveur Node.js
            |
            v
     Logique applicative
            |
            v
  Base de données / Services externes
```

### Question 1

Répondez aux questions suivantes :

1. Quelle est la différence entre JavaScript exécuté dans un navigateur et JavaScript exécuté avec Node.js ?

2. Node.js est-il un langage de programmation ou un environnement d’exécution ?

3. Pourquoi Node.js est-il adapté au développement de serveurs Web ?

4. Node.js remplace-t-il nécessairement une base de données ?

5. Quel est le rôle d’un framework comme Express dans une application Node.js ?

## 4.2. Le modèle d’exécution de Node.js

Node.js utilise un modèle d’exécution principalement basé sur :

* un thread JavaScript principal ;

* une boucle d’événements ;

* des opérations d’entrée/sortie asynchrones ;

* des mécanismes internes permettant de gérer certaines opérations coûteuses.

L’objectif est d’éviter de bloquer inutilement l’exécution du programme pendant l’attente d’une opération d’entrée/sortie.

Exemples d’opérations d’entrée/sortie :

* lecture d’un fichier ;

* accès à une base de données ;

* requête réseau ;

* communication avec une API externe.

### Attention

Le caractère non bloquant de nombreuses API Node.js ne signifie pas que toutes les opérations JavaScript sont automatiquement parallèles.

Un calcul JavaScript long exécuté sur le thread principal peut empêcher le traitement d’autres événements.

# 5. Partie 1 — Vérification de l’environnement

Durée indicative : 20 minutes

## Activité 1.1 — Vérifier Node.js

Ouvrez un terminal :

Bash

```
Ctrl + Alt + T
```

Exécutez :

Bash

```
node --version
```

Puis :

Bash

```
npm --version
```

Vous pouvez également utiliser :

Bash

```
node -v
npm -v
```

### Travail demandé

Notez :

* la version de Node.js ;

* la version de npm ;

* le chemin de l’exécutable Node.js.

Pour obtenir le chemin de Node.js :

Bash

```
which node
```

Pour obtenir le chemin de npm :

Bash

```
which npm
```

### Questions

1. Quelle version de Node.js est installée ?

2. À quoi sert npm ?

3. Pourquoi est-il important d’utiliser une version de Node.js compatible avec le projet ?

4. Quelle différence existe-t-il entre `node` et `npm` ?

## Activité 1.2 — Vérifier l’exécution JavaScript

Exécutez :

Bash

```
node
```

Vous entrez alors dans le mode interactif Node.js, appelé REPL (Read-Eval-Print Loop).

Testez les instructions suivantes :

JavaScript

```
2 + 3
```

JavaScript

```
console.log("Bonjour depuis Node.js");
```

JavaScript

```
typeof process
```

Pour quitter le REPL :

JavaScript

```
.exit
```

Ou utilisez :

```
Ctrl + D
```

### Questions

1. Que signifie REPL ?

2. Quel est le rôle de l’objet `process` ?

3. Quelle différence existe-t-il entre le REPL et l’exécution d’un fichier `.js` ?

# 6. Partie 2 — Création du projet Node.js

Durée indicative : 20 minutes

## Activité 2.1 — Créer le répertoire du projet

Placez-vous dans votre répertoire de travail :

Bash

```
cd ~/Documents
```

Créez le dossier du projet :

Bash

```
mkdir sang-connect
```

Accédez au dossier :

Bash

```
cd sang-connect
```

Ouvrez le projet avec VS Code :

Bash

```
code .
```

Si la commande `code` n’est pas disponible, ouvrez le dossier manuellement dans Visual Studio Code.

## Activité 2.2 — Initialiser le projet npm

Exécutez :

Bash

```
npm init -y
```

Cette commande crée un fichier :

```
package.json
```

Examinez son contenu :

Bash

```
cat package.json
```

Le contenu initial peut ressembler à ceci :

JSON

```
{
  "name": "sang-connect",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Le contenu exact peut varier selon la version de npm et les options utilisées.

## 6.3. Le fichier `package.json`

Le fichier `package.json` décrit notamment :

* le nom du projet ;

* la version du projet ;

* les scripts npm ;

* les dépendances ;

* les informations générales du projet.

### Question 2

Expliquez le rôle des propriétés suivantes :

JSON

```
{
  "name": "sang-connect",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {}
}
```

## Activité 2.3 — Configurer le projet en ES Modules

Dans cet atelier, nous allons utiliser la syntaxe ES Modules.

Modifiez le fichier `package.json` afin d’ajouter :

JSON

```
"type": "module"
```

Exemple :

JSON

```
{
  "name": "sang-connect",
  "version": "1.0.0",
  "description": "API de gestion des dons de sang",
  "type": "module",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### Explication

La propriété :

JSON

```
"type": "module"
```

indique que les fichiers JavaScript du projet utilisent par défaut le système ES Modules.

Nous pourrons alors écrire :

JavaScript

```
import http from "node:http";
```

au lieu de :

JavaScript

```
const http = require("http");
```

### Comparaison

|
CommonJS

|

ES Modules

|
| --- | --- |
|

`require()`

|

`import`

|
|

`module.exports`

|

`export`

|
|

Système historique de modules Node.js

|

Système standard JavaScript moderne

|
|

Fichiers `.js` généralement CommonJS par défaut selon configuration

|

Activé avec `"type": "module"`

|

### Question 3

1. Quelle syntaxe de modules utilisons-nous dans cet atelier ?

2. Quel est le rôle de `"type": "module"` ?

3. Quelle différence existe-t-il entre `require()` et `import` ?

4. Pourquoi faut-il éviter de mélanger les deux systèmes sans comprendre leur configuration ?

# 7. Partie 3 — Premier programme Node.js

Durée indicative : 20 minutes

## Activité 3.1 — Créer un programme

Dans VS Code, créez le fichier :

```
app.js
```

Ajoutez :

JavaScript

```
console.log("Bienvenue dans SangConnect");
console.log("Node.js est opérationnel");
```

Exécutez le programme :

Bash

```
node app.js
```

### Résultat attendu

```
Bienvenue dans SangConnect
Node.js est opérationnel
```

## Activité 3.2 — Utiliser l’objet `process`

Modifiez `app.js` :

JavaScript

```
console.log("Nom du fichier :", process.argv[1]);
console.log("Version de Node.js :", process.version);
console.log("Système d'exploitation :", process.platform);
```

Exécutez :

Bash

```
node app.js
```

### Questions

1. Que contient `process.version` ?

2. Que contient `process.platform` ?

3. Quel est le rôle de `process.argv` ?

4. Quelle différence existe-t-il entre les arguments de la ligne de commande et les variables d’environnement ?

## Activité 3.3 — Ajouter un script npm

Dans `package.json`, ajoutez un script :

JSON

```
"scripts": {
  "start": "node app.js"
}
```

Exécutez :

Bash

```
npm start
```

### Questions

1. Quel est l’intérêt de déclarer des scripts npm ?

2. Quelle différence existe-t-il entre `node app.js` et `npm start` ?

3. Pourquoi les scripts sont-ils utiles dans un projet développé par plusieurs personnes ?

# 8. Partie 4 — Créer un serveur HTTP natif

Durée indicative : 35 minutes

Nous allons créer un serveur HTTP sans utiliser Express ni Fastify.

Cette étape est importante pour comprendre les mécanismes de base sur lesquels les frameworks Backend s’appuient.

## Activité 4.1 — Créer le fichier du serveur

Créez le fichier :

```
server.js
```

Ajoutez le code suivant :

JavaScript

```
import http from "node:http";

const PORT = 3000;

const server = http.createServer((request, response) => {
    response.statusCode = 200;
    response.setHeader("Content-Type", "text/plain; charset=utf-8");

    response.end("Bienvenue sur le serveur SangConnect");
});

server.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

## 8.2. Analyse du code

### Importer le module HTTP

JavaScript

```
import http from "node:http";
```

Le module natif `node:http` fournit des fonctionnalités permettant de créer un serveur HTTP.

Il ne nécessite pas l’installation d’une dépendance externe.

### Définir le port

JavaScript

```
const PORT = 3000;
```

Le serveur écoutera sur le port `3000`.

Un port est un point logique utilisé pour recevoir des communications réseau sur une machine.

### Créer le serveur

JavaScript

```
const server = http.createServer((request, response) => {
    // Traitement de la requête
});
```

La fonction `http.createServer()` crée un serveur HTTP.

La fonction passée en argument est appelée lorsqu’une requête HTTP est reçue.

Elle reçoit deux objets :

|
Objet

|

Rôle

|
| --- | --- |
|

`request`

|

Informations sur la requête reçue

|
|

`response`

|

Objet utilisé pour construire la réponse

|

### Définir le code de statut

JavaScript

```
response.statusCode = 200;
```

Le code `200` indique que la requête a été traitée avec succès.

### Définir le type de contenu

JavaScript

```
response.setHeader(
    "Content-Type",
    "text/plain; charset=utf-8"
);
```

Cette instruction définit le type de contenu de la réponse.

Ici, nous indiquons que la réponse contient du texte.

### Terminer la réponse

JavaScript

```
response.end("Bienvenue sur le serveur SangConnect");
```

La méthode `response.end()` termine la réponse HTTP et transmet son contenu au client.

### Démarrer l’écoute

JavaScript

```
server.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

Le serveur commence à écouter les requêtes entrantes sur le port choisi.

## Activité 4.2 — Lancer le serveur

Dans le terminal :

Bash

```
node server.js
```

Vous devez obtenir un message similaire à :

```
Serveur démarré sur http://localhost:3000
```

Ouvrez un navigateur et accédez à :

```
http://localhost:3000
```

### Question 4

1. Que se passe-t-il lorsque vous ouvrez l’URL dans le navigateur ?

2. Quelle partie du code est exécutée lors de la réception de la requête ?

3. Pourquoi le programme reste-t-il actif après l’exécution de `node server.js` ?

4. Comment arrêter le serveur depuis le terminal ?

Pour arrêter le serveur :

```
Ctrl + C
```

## Activité 4.3 — Tester avec `curl`

Dans un autre terminal :

Bash

```
curl http://localhost:3000
```

Pour afficher les en-têtes HTTP :

Bash

```
curl -i http://localhost:3000
```

### Résultat attendu

La réponse doit contenir des informations similaires à :

http

```
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
```

suivies du message :

```
Bienvenue sur le serveur SangConnect
```

### Questions

1. Quel code de statut est retourné ?

2. Quel est le type de contenu ?

3. Pourquoi `curl -i` permet-il d’obtenir davantage d’informations qu’un simple `curl` ?

4. Quelle différence existe-t-il entre le contenu de la réponse et ses en-têtes ?

# 9. Partie 5 — Comprendre les routes HTTP

Durée indicative : 30 minutes

Dans un serveur HTTP natif Node.js, le développeur doit généralement examiner lui-même :

* la méthode HTTP ;

* le chemin demandé ;

* les paramètres éventuels ;

* les en-têtes ;

* le corps de la requête.

Les frameworks comme Express et Fastify simplifient ensuite cette tâche grâce à des mécanismes de routage dédiés.

## Activité 5.1 — Afficher la méthode et l’URL

Modifiez le serveur :

JavaScript

```
import http from "node:http";

const PORT = 3000;

const server = http.createServer((request, response) => {
    console.log("Méthode :", request.method);
    console.log("URL :", request.url);

    response.statusCode = 200;
    response.setHeader("Content-Type", "text/plain; charset=utf-8");

    response.end("Requête reçue");
});

server.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

Lancez le serveur :

Bash

```
node server.js
```

Dans un autre terminal, exécutez :

Bash

```
curl http://localhost:3000
```

Puis :

Bash

```
curl http://localhost:3000/api/health
```

Observez les informations affichées dans le terminal du serveur.

### Questions

1. Quelle valeur prend `request.method` ?

2. Quelle valeur prend `request.url` ?

3. Le serveur distingue-t-il automatiquement `/` et `/api/health` ?

4. Quelle logique faut-il ajouter pour traiter les chemins différemment ?

## Activité 5.2 — Créer plusieurs routes

Remplacez le contenu de `server.js` par :

JavaScript

```
import http from "node:http";

const PORT = 3000;

const server = http.createServer((request, response) => {
    const { method, url } = request;

    response.setHeader("Content-Type", "text/plain; charset=utf-8");

    if (method === "GET" && url === "/") {
        response.statusCode = 200;
        response.end("Bienvenue dans l'API SangConnect");
        return;
    }

    if (method === "GET" && url === "/api/health") {
        response.statusCode = 200;
        response.end("Serveur opérationnel");
        return;
    }

    if (method === "GET" && url === "/api/info") {
        response.statusCode = 200;
        response.end("API SangConnect - Version 1.0.0");
        return;
    }

    response.statusCode = 404;
    response.end("Route introuvable");
});

server.listen(PORT, () => {
    console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

### Tests à réaliser

Bash

```
curl -i http://localhost:3000/
```

Bash

```
curl -i http://localhost:3000/api/health
```

Bash

```
curl -i http://localhost:3000/api/info
```

Bash

```
curl -i http://localhost:3000/inconnue
```

### Questions

1. Pourquoi utilisons-nous `return` après `response.end()` ?

2. Que se passe-t-il si aucune condition ne correspond ?

3. Quel code HTTP est utilisé pour une route inexistante ?

4. Pourquoi faut-il tester également des URLs invalides ?

5. Que se passe-t-il si nous envoyons une requête `POST` sur une route qui accepte uniquement `GET` ?

# 10. Partie 6 — Retourner des réponses JSON

Durée indicative : 25 minutes

Une API Backend retourne généralement des données structurées, souvent au format JSON.

## 10.1. Le format JSON

JSON permet de représenter :

* des objets ;

* des tableaux ;

* des chaînes ;

* des nombres ;

* des valeurs booléennes ;

* la valeur `null`.

Exemple :

JSON

```
{
  "status": "ok",
  "application": "SangConnect"
}
```

En JavaScript, l’objet équivalent est :

JavaScript

```
{
    status: "ok",
    application: "SangConnect"
}
```

Pour transformer un objet JavaScript en chaîne JSON, nous utilisons :

JavaScript

```
JSON.stringify()
```

Dans les activités précédentes, vous avez créé un serveur HTTP avec le module natif `node:http`. Vous allez maintenant améliorer la manière dont le serveur renvoie les réponses au client.

L'objectif est de créer une fonction réutilisable pour envoyer des réponses JSON, puis de l'utiliser dans les différentes routes de l'application SangConnect.

## Activité 6.1 — Créer une fonction utilitaire JSON

### Objectif

Créer une fonction permettant d'envoyer une réponse HTTP au format JSON, afin d'éviter de répéter le même code dans chaque route.

### Étape 1 — Ouvrir le fichier principal

Ouvrez le fichier :

```
src/server.js
```

Repérez le code utilisé pour envoyer une réponse JSON, par exemple :

JavaScript

```
res.writeHead(200, {
  "Content-Type": "application/json"
});

res.end(JSON.stringify({
  message: "Bienvenue dans SangConnect"
}));
```

### Étape 2 — Identifier les répétitions

Répondez aux questions suivantes :

1. Quelle instruction transforme un objet JavaScript en chaîne JSON ?

2. Quel est le rôle de l'en-tête `Content-Type` ?

3. Pourquoi serait-il intéressant de regrouper ces instructions dans une fonction ?

4. Quels paramètres cette fonction devrait-elle recevoir ?

Réflexion attendue :

La fonction devrait pouvoir recevoir :

* L'objet représentant les données à envoyer.

* Le code de statut HTTP.

* L'objet `ServerResponse` permettant d'envoyer la réponse au client.

### Étape 3 — Créer le dossier des utilitaires

À la racine du projet, exécutez :

Bash

```
mkdir -p src/utils
```

Créez ensuite le fichier :

```
src/utils/http.js
```

### Étape 4 — Écrire la fonction `sendJson`

Ajoutez le code suivant dans `src/utils/http.js` :

JavaScript

```
export function sendJson(res, statusCode, data) {
  res.writeHead(statusCode, {
    "Content-Type": "application/json; charset=utf-8"
  });

  res.end(JSON.stringify(data));
}
```

### Comprendre le code

|
Élément

|

Rôle

|
| --- | --- |
|

`export function`

|

Rend la fonction utilisable dans d'autres fichiers

|
|

`res`

|

Objet représentant la réponse HTTP

|
|

`statusCode`

|

Code de statut HTTP, par exemple `200` ou `404`

|
|

`data`

|

Données JavaScript à convertir en JSON

|
|

`res.writeHead()`

|

Définit le statut et les en-têtes HTTP

|
|

`JSON.stringify()`

|

Transforme les données JavaScript en chaîne JSON

|
|

`res.end()`

|

Termine et envoie la réponse

|

### Étape 5 — Tester la fonction avec une route

Dans `src/server.js`, importez la fonction :

JavaScript

```
import { sendJson } from "./utils/http.js";
```

Modifiez ensuite une route pour utiliser cette fonction :

JavaScript

```
if (req.url === "/" && req.method === "GET") {
  sendJson(res, 200, {
    message: "Bienvenue dans SangConnect",
    application: "Gestion des dons de sang"
  });

  return;
}
```

### Questions guidées

1. Pourquoi faut-il utiliser `return` après l'appel à `sendJson()` ?

2. Que se passe-t-il si plusieurs réponses sont envoyées pour la même requête ?

3. Quel code de statut est utilisé pour une réponse réussie ?

4. Comment modifier le code afin de retourner une réponse `404` ?

### Test

Démarrez le serveur :

Bash

```
node --watch src/server.js
```

Dans un autre terminal :

Bash

```
curl http://localhost:3000/
```

Résultat attendu, sous forme similaire :

JSON

```
{
  "message": "Bienvenue dans SangConnect",
  "application": "Gestion des dons de sang"
}
```

## Activité 6.2 — Utiliser différents codes de statut HTTP

### Objectif

Associer les codes de statut HTTP appropriés aux réponses du serveur.

### Rappel théorique

|
Code

|

Signification

|

Exemple d'utilisation

|
| --- | --- | --- |
|

`200`

|

OK

|

Requête traitée avec succès

|
|

`201`

|

Created

|

Ressource créée

|
|

`400`

|

Bad Request

|

Requête incorrecte

|
|

`404`

|

Not Found

|

Route ou ressource inexistante

|
|

`500`

|

Internal Server Error

|

Erreur interne du serveur

|

### Étape 1 — Créer une route de vérification

Dans `src/server.js`, ajoutez une route :

JavaScript

```
if (req.url === "/api/health" && req.method === "GET") {
  sendJson(res, 200, {
    status: "ok",
    service: "SangConnect"
  });

  return;
}
```

### Étape 2 — Gérer une route inexistante

À la fin de la fonction de traitement des requêtes, ajoutez :

JavaScript

```
sendJson(res, 404, {
  error: "Route non trouvée",
  path: req.url,
  method: req.method
});
```

### Étape 3 — Tester

Testez une route existante :

Bash

```
curl -i http://localhost:3000/api/health
```

Puis testez une route inexistante :

Bash

```
curl -i http://localhost:3000/api/inconnue
```

L'option `-i` permet d'afficher les en-têtes HTTP.

### Questions

1. Quel code de statut est renvoyé pour `/api/health` ?

2. Quel code de statut est renvoyé pour `/api/inconnue` ?

3. Pourquoi le code `404` est-il préférable à un code `200` lorsqu'une route n'existe pas ?

4. Quelle différence existe-t-il entre le statut HTTP et le contenu JSON de la réponse ?

### Travail demandé

Modifiez la réponse `404` pour ajouter :

* La date et l'heure de la requête.

* Un message destiné au développeur.

* Un message destiné au client.

Exemple de structure :

JSON

```
{
  "error": "Route non trouvée",
  "message": "La ressource demandée n'existe pas",
  "path": "/api/inconnue",
  "timestamp": "..."
}
```

Pour générer la date :

JavaScript

```
new Date().toISOString()
```

## Activité 6.3 — Ajouter une route `/api/info`

### Objectif

Créer une route renvoyant des informations sur l'application et l'environnement Node.js.

### Étape 1 — Ajouter la route

Dans `src/server.js` :

JavaScript

```
if (req.url === "/api/info" && req.method === "GET") {
  sendJson(res, 200, {
    application: "SangConnect",
    version: "1.0.0",
    environment: "development",
    nodeVersion: process.version
  });

  return;
}
```

### Étape 2 — Tester la route

Bash

```
curl http://localhost:3000/api/info
```

### Questions

1. Quel objet Node.js permet d'accéder aux informations du processus en cours ?

2. Quelle propriété permet d'obtenir la version de Node.js ?

3. Pourquoi la version de l'application devrait-elle être distincte de la version de Node.js ?

4. Quelles autres informations pourraient être retournées par cette route ?

### Travail demandé

Ajoutez les informations suivantes :

* Le nom du projet.

* La version de l'API.

* Le nom du développeur ou de l'équipe.

* L'environnement d'exécution.

* Une description courte de l'application.

Exemple :

JavaScript

```
{
  application: "SangConnect",
  apiVersion: "v1",
  description: "API de gestion des dons de sang",
  environment: "development",
  nodeVersion: process.version
}
```

Attention : une route d'information destinée au public ne doit pas exposer de données sensibles, de mots de passe, de clés secrètes ou de variables d'environnement confidentielles.

## Activité 6.4 — Comprendre la méthode HTTP utilisée

### Objectif

Distinguer les requêtes HTTP en fonction de leur méthode.

Jusqu'à présent, les routes ont principalement utilisé la méthode `GET`. Vous allez maintenant afficher la méthode reçue par le serveur.

### Étape 1 — Créer une route de diagnostic

Ajoutez la route suivante :

JavaScript

```
if (req.url === "/api/diagnostic" && req.method === "GET") {
  sendJson(res, 200, {
    method: req.method,
    url: req.url,
    headers: req.headers
  });

  return;
}
```

### Étape 2 — Tester avec `curl`

Bash

```
curl http://localhost:3000/api/diagnostic
```

### Étape 3 — Tester une autre méthode

Bash

```
curl -X POST http://localhost:3000/api/diagnostic
```

La route précédente ne correspond pas à la méthode `POST`. Le serveur devrait donc retourner la réponse `404` générale, sauf si vous avez ajouté une gestion particulière des méthodes.

### Questions

1. Quelle propriété permet de connaître la méthode HTTP ?

2. Quelle propriété permet de connaître l'URL demandée ?

3. Quelle différence existe-t-il entre une requête `GET` et une requête `POST` ?

4. Pourquoi une route doit-elle tenir compte de la méthode HTTP ?

5. Une même URL peut-elle être associée à plusieurs méthodes HTTP ?

### Travail demandé

Ajoutez une réponse spécifique lorsqu'un client utilise une méthode non autorisée :

JavaScript

```
if (req.url === "/api/diagnostic") {
  if (req.method !== "GET") {
    sendJson(res, 405, {
      error: "Méthode non autorisée",
      method: req.method,
      allowedMethods: ["GET"]
    });

    return;
  }

  sendJson(res, 200, {
    method: req.method,
    url: req.url
  });

  return;
}
```

Le code `405` signifie Method Not Allowed.

# Activité 7 — Tester l'API avec cURL

## Objectif

Utiliser l'outil `curl` pour tester les différentes routes du serveur HTTP.

### Étape 1 — Vérifier que le serveur fonctionne

Lancez le serveur :

Bash

```
node --watch src/server.js
```

Dans un deuxième terminal, exécutez :

Bash

```
curl -i http://localhost:3000/
```

### Étape 2 — Tester les routes existantes

Exécutez successivement :

Bash

```
curl -i http://localhost:3000/
```

Bash

```
curl -i http://localhost:3000/api/health
```

Bash

```
curl -i http://localhost:3000/api/info
```

Bash

```
curl -i http://localhost:3000/api/diagnostic
```

Bash

```
curl -i http://localhost:3000/api/inconnue
```

### Tableau de validation

Complétez le tableau suivant :

|
URL

|

Méthode

|

Code attendu

|

Résultat observé

|
| --- | --- | --- | --- |
|

`/`

|

GET

|

200

|  |
|

`/api/health`

|

GET

|

200

|  |
|

`/api/info`

|

GET

|

200

|  |
|

`/api/diagnostic`

|

GET

|

200

|  |
|

`/api/inconnue`

|

GET

|

404

|  |
|

`/api/diagnostic`

|

POST

|

405 ou 404 selon l'implémentation

|  |

### Questions de synthèse

1. Toutes les réponses possèdent-elles un en-tête `Content-Type` ?

2. Le code HTTP est-il cohérent avec le contenu de la réponse ?

3. Que se passe-t-il si le serveur est arrêté pendant l'exécution de `curl` ?

4. Pourquoi est-il utile de tester une route existante et une route inexistante ?

5. Quel est l'intérêt de l'option `-i` ?

# Activité 8 — Réaliser un mini-diagnostic du serveur

## Objectif

Créer une route permettant de vérifier que le serveur est opérationnel.

Cette route constituera un premier exemple de health check, couramment utilisé dans les applications backend et les environnements de déploiement.

### Étape 1 — Définir les informations de santé

La route `/api/health` doit retourner :

* L'état du serveur.

* Le nom de l'application.

* La date du diagnostic.

* La version de Node.js.

### Étape 2 — Implémenter la route

JavaScript

```
if (req.url === "/api/health" && req.method === "GET") {
  sendJson(res, 200, {
    status: "ok",
    application: "SangConnect",
    timestamp: new Date().toISOString(),
    nodeVersion: process.version
  });

  return;
}
```

### Étape 3 — Tester

Bash

```
curl -i http://localhost:3000/api/health
```

### Questions

1. Pourquoi utilise-t-on généralement une route de santé ?

2. Quelle différence existe-t-il entre une vérification de disponibilité et une vérification complète de l'état d'une application ?

3. Pourquoi la réponse d'une route de santé doit-elle rester rapide ?

4. Quelles vérifications supplémentaires pourrait-on effectuer dans une application réelle ?

Exemples :

* Vérification de la connexion à la base de données.

* Vérification de la disponibilité d'un service externe.

* Vérification de l'espace disque.

* Vérification de la présence de configurations nécessaires.

Attention : dans cet atelier, la route réalise uniquement un diagnostic simple. Elle ne vérifie pas encore de base de données ni de service externe.

# Activité 9 — Structurer le serveur et vérifier le code

## Objectif

Vérifier que le serveur reste compréhensible et que les différentes parties du code ont des responsabilités distinctes.

À ce stade, le fichier `src/server.js` peut commencer à contenir plusieurs routes et beaucoup de conditions `if`.

### Questions de réflexion

1. Quel est le rôle du fichier `src/server.js` ?

2. Que se passe-t-il si l'on ajoute 20 ou 30 routes dans le même fichier ?

3. Pourquoi la répétition de conditions `if` peut-elle rendre le code difficile à maintenir ?

4. Comment pourrait-on séparer les routes dans plusieurs fichiers ?

5. Quelles responsabilités pourrait-on confier à des fichiers distincts ?

Une architecture possible, qui sera approfondie dans les ateliers suivants, est :

```
sangconnect/
├── package.json
├── src/
│   ├── server.js
│   ├── routes/
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   └── utils/
└── README.md
```

Pour le moment, vous devez principalement vous assurer que :

* Le serveur démarre correctement.

* Les routes répondent correctement.

* Les codes HTTP sont adaptés.

* Les réponses sont au format JSON.

* La fonction `sendJson()` est réutilisée.

* Aucune erreur de syntaxe ne subsiste.

# Travail pratique à faire à la maison

## Sujet : Ajouter des routes à l'application SangConnect

Vous devez enrichir le serveur HTTP créé pendant l'atelier.

### 1. Ajouter la route `/api/welcome`

La route doit répondre uniquement aux requêtes `GET`.

Elle doit retourner :

JSON

```
{
  "message": "Bienvenue dans l'API SangConnect",
  "description": "Gestion des dons de sang"
}
```

### 2. Ajouter la route `/api/version`

Cette route doit retourner :

* Le nom de l'application.

* La version de l'API.

* La version de Node.js.

* L'environnement d'exécution.

### 3. Ajouter la route `/api/diagnostic`

Cette route doit retourner :

* La méthode HTTP utilisée.

* L'URL demandée.

* La date du diagnostic.

* Les en-têtes HTTP reçus.

La route doit accepter uniquement la méthode `GET`.

Pour les autres méthodes, elle doit retourner :

* Le code HTTP `405`.

* Un message explicatif.

* La liste des méthodes autorisées.

### 4. Améliorer la gestion des routes inconnues

Pour toute route non définie, retourner :

JSON

```
{
  "error": "Route non trouvée",
  "path": "...",
  "method": "...",
  "timestamp": "..."
}
```

avec le code HTTP `404`.

### 5. Réaliser les tests

Pour chaque route, fournir les commandes `curl` utilisées.

Exemple :

Bash

```
curl -i http://localhost:3000/api/version
```

Vous devez tester :

* Une route existante avec `GET`.

* Une route inexistante.

* Une route existante avec une méthode non autorisée.

* Plusieurs routes à la suite.

## Livrables attendus

À remettre :

```
src/
├── server.js
└── utils/
    └── http.js
```

Ainsi qu'un fichier `README.md` contenant :

1. Les instructions de lancement du serveur.

2. La liste des routes créées.

3. Les méthodes HTTP acceptées.

4. Les codes de statut utilisés.

5. Les commandes de test `curl`.

6. Une courte explication de la fonction `sendJson()`.

## Critères d'évaluation

|
Critère

|

Points

|
| --- | --- |
|

Serveur fonctionnel

|

4

|
|

Fonction `sendJson()` correctement implémentée

|

4

|
|

Routes correctement créées

|

4

|
|

Codes HTTP appropriés

|

3

|
|

Tests avec `curl`

|

3

|
|

Organisation et lisibilité du code

|

2

|
|

Total

|

20

|

## Bilan de l'atelier 1

À la fin de cet atelier, vous devez être capable de :

* Vérifier une installation Node.js.

* Initialiser un projet avec npm.

* Utiliser les modules ES.

* Créer un serveur HTTP avec `node:http`.

* Comprendre le rôle d'une requête et d'une réponse HTTP.

* Définir des routes simples.

* Renvoyer des réponses JSON.

* Utiliser les codes de statut HTTP.

* Tester une API avec `curl`.

* Utiliser le mode `--watch`.

* Identifier les limites d'un serveur HTTP construit uniquement avec le module natif.

Transition vers l'atelier 2 : vous allez maintenant découvrir les modules, les fonctions asynchrones, les Promises et l'organisation en couches d'une application backend Node.js.
