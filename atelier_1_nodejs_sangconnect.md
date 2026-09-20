# Atelier 1 — Découverte de Node.js et cycle de vie HTTP

**Matière :** Ingénierie Backend  
**Niveau :** 4e année cycle ingénieur  
**Durée :** 3 heures  
**Projet fil rouge :** SangConnect — API de gestion des dons de sang

---

## 1. Présentation de l’atelier

### 1.1. Contexte

Dans ce premier atelier, nous allons découvrir l’environnement Node.js et construire un premier serveur HTTP sans framework.

Le serveur développé servira de base au projet fil rouge **SangConnect**, une application backend destinée à gérer des informations relatives aux donneurs, aux centres de collecte, aux dons et aux stocks de sang.

### 1.2. Objectifs pédagogiques

À la fin de cet atelier, l’étudiant doit être capable de :

- vérifier l’installation de Node.js et de npm ;
- initialiser un projet Node.js ;
- comprendre le rôle de `package.json` ;
- utiliser les modules ES ;
- créer un serveur HTTP avec le module natif `node:http` ;
- distinguer une requête HTTP d’une réponse HTTP ;
- examiner la méthode et l’URL d’une requête ;
- créer des routes simples ;
- retourner des réponses au format JSON ;
- utiliser les codes de statut HTTP ;
- tester une API avec `curl` ;
- utiliser le mode `--watch` de Node.js ;
- identifier les limites d’un serveur construit uniquement avec le module natif.

### 1.3. Pré-requis

Les étudiants doivent disposer de :

- Ubuntu 24.04 ou d’un système Linux équivalent ;
- Visual Studio Code ;
- un terminal ;
- des notions de JavaScript ;
- des notions générales sur HTTP, HTML et les API web.

---

# 2. Rappels théoriques

## 2.1. Node.js

Node.js est un environnement d’exécution JavaScript basé sur le moteur V8. Il permet d’exécuter du JavaScript en dehors d’un navigateur, notamment pour développer des serveurs et des API backend.

Node.js fournit des modules natifs pour :

- manipuler les fichiers ;
- gérer les chemins ;
- créer des serveurs HTTP ;
- accéder au système ;
- gérer les événements ;
- travailler avec les flux et les opérations asynchrones.

## 2.2. npm

`npm` est le gestionnaire de paquets généralement fourni avec Node.js.

Il permet notamment de :

- créer un projet ;
- installer des dépendances ;
- gérer les scripts ;
- identifier les versions des paquets ;
- partager un projet avec d’autres développeurs.

## 2.3. Serveur HTTP

Un serveur HTTP reçoit des requêtes provenant de clients, puis renvoie des réponses.

Le client peut être :

- un navigateur ;
- l’outil `curl` ;
- une application mobile ;
- une autre API ;
- un logiciel de test comme Postman ou Insomnia.

Une requête HTTP contient notamment :

- une méthode HTTP ;
- une URL ;
- des en-têtes ;
- éventuellement un corps.

Une réponse HTTP contient notamment :

- un code de statut ;
- des en-têtes ;
- éventuellement un corps.

## 2.4. Quelques méthodes HTTP

| Méthode | Utilisation courante |
|---|---|
| `GET` | Consulter une ressource |
| `POST` | Créer une ressource ou envoyer des données |
| `PUT` | Remplacer une ressource |
| `PATCH` | Modifier partiellement une ressource |
| `DELETE` | Supprimer une ressource |

## 2.5. Quelques codes de statut

| Code | Signification |
|---|---|
| `200` | OK |
| `201` | Created |
| `400` | Bad Request |
| `404` | Not Found |
| `405` | Method Not Allowed |
| `500` | Internal Server Error |

---

# 3. Organisation temporelle

| Partie | Durée indicative |
|---|---:|
| Introduction et vérifications | 20 min |
| Initialisation du projet | 25 min |
| Premier programme Node.js | 20 min |
| Création du serveur HTTP | 45 min |
| Réponses JSON et codes HTTP | 40 min |
| Tests et synthèse | 30 min |
| **Total** | **3 h** |

---

# 4. Activité 1 — Vérifier l’environnement de travail

## Objectif

Vérifier que Node.js, npm et Visual Studio Code sont disponibles.

## 4.1. Vérifier Node.js

Ouvrez un terminal et exécutez :

```bash
node --version
```

Puis :

```bash
npm --version
```

Examinez les versions affichées.

### Questions

1. Quelle version de Node.js est installée ?
2. Quelle version de npm est installée ?
3. Quelle commande permet de vérifier le chemin de l’exécutable Node.js ?
4. Quelle commande permet de vérifier le chemin de l’exécutable npm ?

### Commandes complémentaires

```bash
which node
```

```bash
which npm
```

Vous pouvez également afficher les informations détaillées :

```bash
node --help
```

## 4.2. Vérifier Visual Studio Code

Vérifiez que la commande `code` est disponible :

```bash
code --version
```

Si VS Code est installé mais que la commande n’est pas disponible, ouvrez l’application depuis le menu du système.

---

# 5. Activité 2 — Créer le projet Node.js

## Objectif

Créer une structure de projet propre pour l’application SangConnect.

## 5.1. Créer le dossier du projet

Placez-vous dans le dossier de travail de votre choix :

```bash
mkdir -p ~/projets
cd ~/projets
```

Créez ensuite le projet :

```bash
mkdir sangconnect
cd sangconnect
```

## 5.2. Initialiser npm

Exécutez :

```bash
npm init -y
```

Cette commande crée le fichier `package.json`.

## 5.3. Examiner `package.json`

Ouvrez le fichier `package.json`.

Il contient généralement des informations similaires à :

```json
{
  "name": "sangconnect",
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

### Questions

1. Quel est le rôle du champ `name` ?
2. Quel est le rôle du champ `version` ?
3. À quoi sert la section `scripts` ?
4. Pourquoi le fichier `package.json` est-il important dans un projet backend ?
5. Quelle différence existe-t-il entre le code de l’application et la configuration du projet ?

## 5.4. Configurer les modules ES

Modifiez `package.json` et ajoutez :

```json
"type": "module"
```

Exemple :

```json
{
  "name": "sangconnect",
  "version": "1.0.0",
  "description": "API de gestion des dons de sang",
  "type": "module",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js"
  }
}
```

### Questions

1. Pourquoi utilise-t-on `"type": "module"` ?
2. Quelle syntaxe permet d’importer un module ES ?
3. Quelle syntaxe permet d’exporter une fonction ?
4. Pourquoi les extensions `.js` sont-elles précisées dans les imports locaux ?

---

# 6. Activité 3 — Ouvrir le projet dans VS Code

## Objectif

Utiliser Visual Studio Code pour travailler dans le dossier du projet.

Depuis le dossier `sangconnect`, exécutez :

```bash
code .
```

## 6.1. Créer l’arborescence initiale

Créez l’organisation suivante :

```text
sangconnect/
├── package.json
├── src/
│   └── server.js
└── README.md
```

Commandes possibles :

```bash
mkdir -p src
touch src/server.js
touch README.md
```

## 6.2. Extensions conseillées pour VS Code

Les extensions suivantes peuvent être utiles :

- ESLint ;
- Prettier - Code formatter ;
- JavaScript and TypeScript Nightly, si nécessaire ;
- REST Client, pour tester des requêtes HTTP depuis VS Code.

L’installation d’extensions doit rester cohérente avec les règles et les outils utilisés dans le cours.

### Questions

1. Quel est le rôle d’un éditeur de code ?
2. Quelle différence existe-t-il entre un éditeur et un environnement d’exécution ?
3. Pourquoi est-il utile de travailler dans un dossier de projet ouvert dans VS Code ?
4. Quel est l’intérêt d’un outil de formatage automatique ?

---

# 7. Activité 4 — Écrire un premier programme Node.js

## Objectif

Exécuter un premier programme JavaScript avec Node.js.

Dans `src/server.js`, ajoutez :

```javascript
console.log("Bienvenue dans SangConnect");
console.log("Node.js est opérationnel");
```

Exécutez le programme depuis la racine du projet :

```bash
node src/server.js
```

### Résultat attendu

```text
Bienvenue dans SangConnect
Node.js est opérationnel
```

## 7.1. Utiliser une variable

Modifiez le programme :

```javascript
const applicationName = "SangConnect";
const applicationVersion = "1.0.0";

console.log(`Application : ${applicationName}`);
console.log(`Version : ${applicationVersion}`);
```

### Questions

1. Quelle différence existe-t-il entre `const` et `let` ?
2. Que signifie l’utilisation des accents graves dans une chaîne de caractères ?
3. Quel est le rôle de l’interpolation `${...}` ?
4. Pourquoi est-il préférable de déclarer une constante pour une valeur qui ne change pas ?

## 7.2. Ajouter une fonction

```javascript
function displayApplicationInfo() {
  console.log("Application : SangConnect");
  console.log("Version : 1.0.0");
}

displayApplicationInfo();
```

### Travail demandé

Modifiez la fonction pour qu’elle reçoive :

- le nom de l’application ;
- sa version.

Exemple d’appel :

```javascript
displayApplicationInfo("SangConnect", "1.0.0");
```

---

# 8. Activité 5 — Créer un serveur HTTP avec `node:http`

## Objectif

Créer un premier serveur HTTP sans utiliser de framework.

## 8.1. Importer le module HTTP

Remplacez le contenu de `src/server.js` par :

```javascript
import http from "node:http";

const port = 3000;

const server = http.createServer((req, res) => {
  res.writeHead(200, {
    "Content-Type": "text/plain; charset=utf-8"
  });

  res.end("Bienvenue dans SangConnect");
});

server.listen(port, () => {
  console.log(`Serveur démarré sur http://localhost:${port}`);
});
```

## 8.2. Lancer le serveur

Exécutez :

```bash
node src/server.js
```

Vous devez obtenir un message similaire à :

```text
Serveur démarré sur http://localhost:3000
```

Ouvrez ensuite l’adresse suivante dans un navigateur :

```text
http://localhost:3000
```

## 8.3. Comprendre le code

### Import du module

```javascript
import http from "node:http";
```

Le module natif `node:http` permet de créer un serveur HTTP.

### Définition du port

```javascript
const port = 3000;
```

Le serveur écoutera les requêtes sur le port `3000`.

### Création du serveur

```javascript
const server = http.createServer((req, res) => {
  // traitement de la requête
});
```

La fonction passée à `createServer()` est exécutée à chaque requête reçue.

Elle reçoit deux objets :

- `req` : la requête entrante ;
- `res` : la réponse à envoyer au client.

### Définition de la réponse

```javascript
res.writeHead(200, {
  "Content-Type": "text/plain; charset=utf-8"
});
```

Cette instruction définit :

- le code de statut `200` ;
- le type de contenu ;
- l’encodage utilisé.

### Envoi du contenu

```javascript
res.end("Bienvenue dans SangConnect");
```

Cette instruction envoie la réponse et termine le traitement.

### Démarrage de l’écoute

```javascript
server.listen(port, () => {
  console.log(`Serveur démarré sur http://localhost:${port}`);
});
```

Le serveur commence à écouter les requêtes sur le port indiqué.

## 8.4. Questions guidées

1. Quel est le rôle de `http.createServer()` ?
2. À quel moment la fonction `(req, res) => { ... }` est-elle exécutée ?
3. Quelle est la différence entre `req` et `res` ?
4. Quel est le rôle de `res.writeHead()` ?
5. Quel est le rôle de `res.end()` ?
6. Que se passe-t-il si le port `3000` est déjà utilisé ?
7. Comment arrêter le serveur depuis le terminal ?

Pour arrêter le serveur :

```text
Ctrl + C
```

---

# 9. Activité 6 — Créer des routes HTTP simples

## Objectif

Répondre différemment selon l’URL demandée et la méthode HTTP utilisée.

## 9.1. Lire l’URL et la méthode

Ajoutez temporairement :

```javascript
console.log("Méthode :", req.method);
console.log("URL :", req.url);
```

Testez plusieurs adresses depuis un navigateur ou avec `curl`.

### Questions

1. Quelle propriété permet de connaître la méthode HTTP ?
2. Quelle propriété permet de connaître l’URL demandée ?
3. Que renvoie `req.method` lorsque vous ouvrez une page dans un navigateur ?
4. Que renvoie `req.url` lorsque vous consultez `/api/health` ?

## 9.2. Créer la route `/`

Modifiez le serveur :

```javascript
if (req.url === "/" && req.method === "GET") {
  res.writeHead(200, {
    "Content-Type": "text/plain; charset=utf-8"
  });

  res.end("Bienvenue dans SangConnect");
  return;
}
```

## 9.3. Créer la route `/api/health`

Ajoutez :

```javascript
if (req.url === "/api/health" && req.method === "GET") {
  res.writeHead(200, {
    "Content-Type": "text/plain; charset=utf-8"
  });

  res.end("API opérationnelle");
  return;
}
```

## 9.4. Créer la route `/api/info`

Ajoutez :

```javascript
if (req.url === "/api/info" && req.method === "GET") {
  res.writeHead(200, {
    "Content-Type": "text/plain; charset=utf-8"
  });

  res.end("SangConnect - API de gestion des dons de sang");
  return;
}
```

## 9.5. Gérer les routes inconnues

À la fin du traitement, ajoutez :

```javascript
res.writeHead(404, {
  "Content-Type": "text/plain; charset=utf-8"
});

res.end("Route non trouvée");
```

### Questions

1. Pourquoi utilise-t-on `return` après chaque réponse ?
2. Pourquoi la réponse `404` doit-elle être placée après les routes connues ?
3. Que se passe-t-il lorsqu’une URL n’est pas définie ?
4. Une URL identique peut-elle être utilisée avec plusieurs méthodes HTTP ?

---

# 10. Activité 6.1 — Créer une fonction utilitaire JSON

## Objectif

Créer une fonction permettant d’envoyer une réponse HTTP au format JSON afin d’éviter de répéter le même code dans chaque route.

## 10.1. Identifier les répétitions

Repérez le code suivant :

```javascript
res.writeHead(200, {
  "Content-Type": "application/json"
});

res.end(JSON.stringify({
  message: "Bienvenue dans SangConnect"
}));
```

### Questions

1. Quelle instruction transforme un objet JavaScript en chaîne JSON ?
2. Quel est le rôle de `Content-Type` ?
3. Pourquoi est-il intéressant de regrouper ces instructions dans une fonction ?
4. Quels paramètres la fonction devrait-elle recevoir ?

La fonction recevra :

- l’objet `ServerResponse` ;
- le code de statut HTTP ;
- les données à convertir en JSON.

## 10.2. Créer le dossier utilitaire

```bash
mkdir -p src/utils
```

Créez le fichier :

```text
src/utils/http.js
```

## 10.3. Implémenter `sendJson`

Dans `src/utils/http.js` :

```javascript
export function sendJson(res, statusCode, data) {
  res.writeHead(statusCode, {
    "Content-Type": "application/json; charset=utf-8"
  });

  res.end(JSON.stringify(data));
}
```

## 10.4. Importer la fonction

Dans `src/server.js` :

```javascript
import { sendJson } from "./utils/http.js";
```

## 10.5. Utiliser la fonction

```javascript
if (req.url === "/" && req.method === "GET") {
  sendJson(res, 200, {
    message: "Bienvenue dans SangConnect",
    application: "Gestion des dons de sang"
  });

  return;
}
```

### Comprendre le code

| Élément | Rôle |
|---|---|
| `export function` | Rend la fonction disponible dans d’autres fichiers |
| `res` | Objet représentant la réponse HTTP |
| `statusCode` | Code de statut HTTP |
| `data` | Données JavaScript à convertir en JSON |
| `res.writeHead()` | Définit le statut et les en-têtes |
| `JSON.stringify()` | Convertit les données en chaîne JSON |
| `res.end()` | Termine et envoie la réponse |

### Questions

1. Pourquoi faut-il utiliser `return` après l’appel à `sendJson()` ?
2. Que se passe-t-il si plusieurs réponses sont envoyées pour une même requête ?
3. Quel code de statut est utilisé pour une réponse réussie ?
4. Comment retourner une réponse avec le code `404` ?

## 10.6. Tester la fonction

Lancez le serveur :

```bash
node --watch src/server.js
```

Dans un autre terminal :

```bash
curl http://localhost:3000/
```

Résultat attendu :

```json
{
  "message": "Bienvenue dans SangConnect",
  "application": "Gestion des dons de sang"
}
```

---

# 11. Activité 6.2 — Utiliser différents codes de statut HTTP

## Objectif

Associer un code de statut cohérent à chaque réponse.

## 11.1. Codes à connaître

| Code | Signification | Exemple |
|---|---|---|
| `200` | OK | Requête réussie |
| `201` | Created | Ressource créée |
| `400` | Bad Request | Requête incorrecte |
| `404` | Not Found | Route inexistante |
| `405` | Method Not Allowed | Méthode non autorisée |
| `500` | Internal Server Error | Erreur interne |

## 11.2. Modifier `/api/health`

```javascript
if (req.url === "/api/health" && req.method === "GET") {
  sendJson(res, 200, {
    status: "ok",
    service: "SangConnect"
  });

  return;
}
```

## 11.3. Modifier la réponse 404

```javascript
sendJson(res, 404, {
  error: "Route non trouvée",
  path: req.url,
  method: req.method,
  timestamp: new Date().toISOString()
});
```

## 11.4. Tester

```bash
curl -i http://localhost:3000/api/health
```

```bash
curl -i http://localhost:3000/api/inconnue
```

### Questions

1. Quel code est renvoyé par `/api/health` ?
2. Quel code est renvoyé par une route inexistante ?
3. Pourquoi ne faut-il pas retourner `200` lorsqu’une route n’existe pas ?
4. Quelle différence existe-t-il entre le code HTTP et le contenu JSON ?

---

# 12. Activité 6.3 — Ajouter une route `/api/info`

## Objectif

Retourner des informations sur l’application et l’environnement Node.js.

Ajoutez :

```javascript
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

## Test

```bash
curl -i http://localhost:3000/api/info
```

### Questions

1. Quel objet Node.js permet d’accéder aux informations du processus ?
2. Quelle propriété permet d’obtenir la version de Node.js ?
3. Pourquoi la version de l’application est-elle distincte de celle de Node.js ?
4. Quelles autres informations pourraient être retournées ?

### Travail demandé

Ajoutez :

- le nom du projet ;
- la version de l’API ;
- une description ;
- l’environnement ;
- la version de Node.js.

**Attention :** ne retournez jamais de mots de passe, de clés secrètes ou de variables d’environnement confidentielles.

---

# 13. Activité 6.4 — Examiner la méthode HTTP

## Objectif

Distinguer les requêtes selon leur méthode HTTP.

Ajoutez :

```javascript
if (req.url === "/api/diagnostic" && req.method === "GET") {
  sendJson(res, 200, {
    method: req.method,
    url: req.url,
    headers: req.headers
  });

  return;
}
```

Testez :

```bash
curl -i http://localhost:3000/api/diagnostic
```

Puis :

```bash
curl -i -X POST http://localhost:3000/api/diagnostic
```

## 13.1. Gérer une méthode non autorisée

Pour distinguer la route inexistante de la méthode non autorisée :

```javascript
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
    url: req.url,
    headers: req.headers
  });

  return;
}
```

### Questions

1. Quelle propriété permet de connaître la méthode HTTP ?
2. Quelle propriété permet de connaître l’URL ?
3. Quelle différence existe-t-il entre `GET` et `POST` ?
4. Pourquoi une route doit-elle tenir compte de la méthode ?
5. Une même URL peut-elle être associée à plusieurs méthodes ?

---

# 14. Activité 7 — Tester l’API avec cURL

## Objectif

Tester les routes et vérifier les codes de statut ainsi que les en-têtes HTTP.

## 14.1. Démarrer le serveur

```bash
node --watch src/server.js
```

## 14.2. Tester les routes

```bash
curl -i http://localhost:3000/
```

```bash
curl -i http://localhost:3000/api/health
```

```bash
curl -i http://localhost:3000/api/info
```

```bash
curl -i http://localhost:3000/api/diagnostic
```

```bash
curl -i http://localhost:3000/api/inconnue
```

```bash
curl -i -X POST http://localhost:3000/api/diagnostic
```

## 14.3. Tableau de validation

Complétez le tableau :

| URL | Méthode | Code attendu | Code observé | Résultat |
|---|---|---:|---:|---|
| `/` | GET | 200 | | |
| `/api/health` | GET | 200 | | |
| `/api/info` | GET | 200 | | |
| `/api/diagnostic` | GET | 200 | | |
| `/api/inconnue` | GET | 404 | | |
| `/api/diagnostic` | POST | 405 | | |

### Questions de synthèse

1. Toutes les réponses possèdent-elles un `Content-Type` ?
2. Les codes HTTP sont-ils cohérents ?
3. Que se passe-t-il si le serveur est arrêté ?
4. Pourquoi tester une route existante et une route inexistante ?
5. Quel est l’intérêt de l’option `-i` ?

---

# 15. Activité 8 — Créer un mini health check

## Objectif

Créer une route rapide permettant de vérifier que le serveur est opérationnel.

## 15.1. Implémentation

```javascript
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

## 15.2. Test

```bash
curl -i http://localhost:3000/api/health
```

### Questions

1. Pourquoi utilise-t-on une route de santé ?
2. Pourquoi cette route doit-elle rester rapide ?
3. Quelle différence existe-t-il entre une vérification de disponibilité et une vérification complète ?
4. Quelles vérifications supplémentaires seraient possibles dans une application réelle ?

Exemples :

- vérifier la connexion à une base de données ;
- vérifier un service externe ;
- vérifier l’espace disque ;
- vérifier la présence des configurations nécessaires.

Dans cet atelier, le health check reste volontairement simple.

---

# 16. Activité 9 — Vérifier l’organisation du code

## Objectif

Identifier les limites d’un fichier contenant toutes les routes.

À ce stade, le fichier `server.js` peut contenir plusieurs conditions `if`.

### Questions

1. Quel est le rôle de `server.js` ?
2. Que se passe-t-il si l’application possède 30 routes ?
3. Pourquoi de nombreuses conditions `if` compliquent-elles la maintenance ?
4. Comment pourrait-on séparer les routes dans plusieurs fichiers ?
5. Quelles responsabilités pourraient être confiées à des fichiers distincts ?

Une architecture possible pour les prochains ateliers :

```text
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

Pour cet atelier, vérifiez que :

- le serveur démarre ;
- les routes répondent correctement ;
- les codes HTTP sont adaptés ;
- les réponses sont au format JSON ;
- `sendJson()` est réutilisée ;
- aucune erreur de syntaxe ne subsiste.

---

# 17. Travail pratique à faire à la maison

## Sujet

Ajouter de nouvelles routes à l’application SangConnect.

## 17.1. Route `/api/welcome`

La route doit accepter uniquement `GET`.

Elle doit retourner :

```json
{
  "message": "Bienvenue dans l’API SangConnect",
  "description": "Gestion des dons de sang"
}
```

## 17.2. Route `/api/version`

La route doit retourner :

- le nom de l’application ;
- la version de l’API ;
- la version de Node.js ;
- l’environnement d’exécution.

## 17.3. Route `/api/diagnostic`

La route doit retourner :

- la méthode HTTP ;
- l’URL demandée ;
- la date du diagnostic ;
- les en-têtes HTTP reçus.

La route doit accepter uniquement `GET`.

Pour les autres méthodes, retourner :

- le code `405` ;
- un message explicatif ;
- les méthodes autorisées.

## 17.4. Route inconnue

Pour toute route non définie, retourner :

```json
{
  "error": "Route non trouvée",
  "path": "...",
  "method": "...",
  "timestamp": "..."
}
```

avec le code HTTP `404`.

## 17.5. Tests obligatoires

Tester :

- une route existante avec `GET` ;
- une route inexistante ;
- une route existante avec une méthode non autorisée ;
- plusieurs routes à la suite.

Exemple :

```bash
curl -i http://localhost:3000/api/version
```

## 17.6. Livrables

Remettre :

```text
src/
├── server.js
└── utils/
    └── http.js
```

Ainsi qu’un fichier `README.md` contenant :

1. les instructions de lancement ;
2. la liste des routes ;
3. les méthodes acceptées ;
4. les codes de statut utilisés ;
5. les commandes `curl` ;
6. une explication de `sendJson()`.

## 17.7. Critères d’évaluation

| Critère | Points |
|---|---:|
| Serveur fonctionnel | 4 |
| Fonction `sendJson()` | 4 |
| Routes correctement créées | 4 |
| Codes HTTP appropriés | 3 |
| Tests avec `curl` | 3 |
| Organisation et lisibilité | 2 |
| **Total** | **20** |

---

# 18. Bilan de l’atelier

À la fin de l’atelier, l’étudiant doit être capable de :

- vérifier une installation Node.js ;
- initialiser un projet npm ;
- comprendre le rôle de `package.json` ;
- utiliser les modules ES ;
- créer un serveur HTTP avec `node:http` ;
- comprendre le rôle de `req` et `res` ;
- créer des routes simples ;
- retourner des réponses JSON ;
- utiliser les codes de statut HTTP ;
- tester une API avec `curl` ;
- utiliser `node --watch` ;
- identifier les limites du module HTTP natif.

## Transition vers l’atelier 2

Dans l’atelier suivant, nous allons étudier :

- les modules JavaScript ;
- `import` et `export` ;
- les Promises ;
- `async/await` ;
- la gestion des erreurs ;
- l’organisation en couches ;
- la séparation entre routes, contrôleurs, services et repositories.

Le projet SangConnect sera progressivement structuré pour devenir une véritable application backend maintenable.
