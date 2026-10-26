# Atelier 5 — Persistance des données avec Prisma et MySQL

**Module :** Ingénierie Backend
**Projet fil rouge :** SangConnect — API de gestion des dons de sang
**Durée :** 3 heures
**Technologies :** Node.js, Express.js, Zod, Prisma ORM, MySQL
**Prérequis :** Ateliers 1 à 4
**Niveau :** 4ème année cycle ingénieur

---

# 1. Introduction

Dans les ateliers précédents, nous avons progressivement construit l'API **SangConnect**.

Nous avons commencé avec Node.js :

```text
Node.js
   ↓
Serveur HTTP
```

Puis nous avons introduit Express.js :

```text
Client
   ↓
Express.js
   ↓
Routes
   ↓
Controllers
   ↓
Services
```

Dans l'Atelier 4, nous avons amélioré cette architecture en ajoutant la validation avec Zod :

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
Données
```

Mais il existe encore une limitation importante.

Jusqu'à présent, les donneurs sont stockés dans un tableau JavaScript :

```javascript
const donneurs = [
    {
        id: 1,
        nom: "Ben Ali",
        prenom: "Ahmed",
        groupeSanguin: "A+"
    }
];
```

Ces données sont conservées uniquement en mémoire.

Si nous arrêtons le serveur :

```text
Arrêt de Node.js
       ↓
Mémoire supprimée
       ↓
Données perdues
```

Cette solution n'est donc pas adaptée à une véritable application backend.

Nous allons maintenant introduire une **base de données relationnelle MySQL**.

Pour communiquer avec MySQL depuis Node.js, nous utiliserons **Prisma ORM**.

---

# 2. Objectifs de l'atelier

À la fin de cet atelier, l'étudiant devra être capable de :

* expliquer pourquoi une base de données est nécessaire ;
* installer et vérifier MySQL ;
* créer une base de données MySQL ;
* créer un utilisateur MySQL pour l'application ;
* comprendre le rôle d'un ORM ;
* installer Prisma ;
* configurer Prisma avec MySQL ;
* définir un modèle Prisma ;
* créer une migration ;
* générer Prisma Client ;
* utiliser Prisma Client pour effectuer des requêtes ;
* créer une couche Repository ;
* connecter Repository, Service et Controller ;
* réaliser un CRUD persistant ;
* utiliser `findMany`, `findUnique`, `create`, `update` et `delete` ;
* utiliser Prisma Studio ;
* faire évoluer le schéma avec une migration.

L'architecture finale sera :

```text
Client HTTP
     │
     ▼
   Route
     │
     ▼
 Validation Zod
     │
     ▼
 Controller
     │
     ▼
  Service
     │
     ▼
 Repository
     │
     ▼
  Prisma
     │
     ▼
   MySQL
```

---

# 3. Rappel : la limite des données en mémoire

Dans l'Atelier 4, nous avions quelque chose comme :

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

Cette approche présente plusieurs problèmes :

* les données sont perdues lorsque le serveur est arrêté ;
* plusieurs instances de l'application ne partagent pas facilement ces données ;
* les données ne sont pas persistantes ;
* les recherches deviennent difficiles lorsque le volume augmente ;
* aucune relation entre les différentes ressources n'est encore gérée par un SGBD.

Nous allons donc remplacer progressivement :

```text
Tableau JavaScript
```

par :

```text
Table MySQL
```

---

# 4. MySQL et son rôle

**MySQL** est un système de gestion de base de données relationnelle.

Dans notre application, MySQL sera responsable de la conservation des données.

Nous aurons par exemple :

```text
Base de données : sangconnect

Table :
donneurs
```

Plus tard, nous aurons notamment :

```text
users
donneurs
centres
dons
stocks
```

---

# 5. Vérifier MySQL

Dans un terminal :

```bash
mysql --version
```

Vous devez obtenir une information indiquant que MySQL est installé.

Vérifiez également le service :

```bash
sudo systemctl status mysql
```

Si nécessaire :

```bash
sudo systemctl start mysql
```

Puis :

```bash
sudo systemctl status mysql
```

### Question 1

1. Quel est le rôle du service MySQL ?
2. Sur quel port MySQL écoute-t-il généralement ?
3. Pourquoi Node.js ne peut-il pas simplement stocker durablement les données dans ses variables JavaScript ?

---

# 6. Connexion à MySQL

Sur Ubuntu, vous pouvez tester :

```bash
sudo mysql
```

Vous devriez obtenir une console ressemblant à :

```text
mysql>
```

Testez :

```sql
SHOW DATABASES;
```

Puis quittez :

```sql
EXIT;
```

---

# 7. Créer la base de données SangConnect

Nous allons créer une base de données dédiée à notre application.

Connectez-vous :

```bash
sudo mysql
```

Puis :

```sql
CREATE DATABASE sangconnect;
```

Vérifiez :

```sql
SHOW DATABASES;
```

Vous devez retrouver :

```text
sangconnect
```

Sélectionnez la base :

```sql
USE sangconnect;
```

---

# 8. Créer un utilisateur pour l'application

Dans une véritable application, il est préférable de ne pas utiliser directement le compte administrateur MySQL.

Nous allons créer un utilisateur dédié à SangConnect.

Exemple :

```sql
CREATE USER 'sangconnect'@'localhost'
IDENTIFIED BY 'sangconnect123';
```

Puis :

```sql
GRANT ALL PRIVILEGES
ON sangconnect.*
TO 'sangconnect'@'localhost';
```

Actualisez les privilèges :

```sql
FLUSH PRIVILEGES;
```

Testez la connexion :

```bash
mysql -u sangconnect -p
```

Entrez le mot de passe :

```text
sangconnect123
```

Puis :

```sql
USE sangconnect;
```

### Question 2

Pourquoi est-il préférable que l'application utilise un utilisateur MySQL dédié plutôt que `root` ?

---

# 9. Créer une première table manuellement

Avant d'utiliser Prisma, nous allons rappeler le fonctionnement d'une base relationnelle.

Dans MySQL :

```sql
USE sangconnect;
```

Créons :

```sql
CREATE TABLE donneurs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100) NOT NULL,
    prenom VARCHAR(100) NOT NULL,
    groupe_sanguin VARCHAR(3) NOT NULL
);
```

Vérifiez :

```sql
SHOW TABLES;
```

Puis :

```sql
DESCRIBE donneurs;
```

---

# 10. Insérer quelques données

Ajoutez :

```sql
INSERT INTO donneurs
(nom, prenom, groupe_sanguin)
VALUES
('Ben Ali', 'Ahmed', 'A+');
```

Puis :

```sql
INSERT INTO donneurs
(nom, prenom, groupe_sanguin)
VALUES
('Trabelsi', 'Sami', 'O+');
```

Vérifiez :

```sql
SELECT * FROM donneurs;
```

---

# 11. Pourquoi utiliser Prisma ?

Nous pourrions écrire directement du SQL dans Node.js.

Par exemple :

```sql
SELECT * FROM donneurs;
```

Mais notre application deviendrait rapidement difficile à organiser.

Nous voulons plutôt pouvoir écrire côté JavaScript :

```javascript
const donneurs = await prisma.donneur.findMany();
```

Prisma est un **ORM**.

ORM signifie :

```text
Object-Relational Mapping
```

Il permet de travailler avec les données de la base de données à travers des objets et des méthodes JavaScript.

---

# 12. Architecture avec Prisma

Notre architecture va devenir :

```text
Controller
     │
     ▼
  Service
     │
     ▼
Repository
     │
     ▼
 Prisma Client
     │
     ▼
   MySQL
```

Chaque couche possède une responsabilité.

| Couche     | Responsabilité       |
| ---------- | -------------------- |
| Route      | URL + méthode HTTP   |
| Validation | vérifier les données |
| Controller | HTTP                 |
| Service    | logique métier       |
| Repository | accès aux données    |
| Prisma     | communication ORM    |
| MySQL      | stockage             |

---

# 13. Installation de Prisma

Nous allons utiliser une version Prisma 6.x pour conserver la configuration pédagogique simple utilisée dans cet atelier.

Dans le projet :

```bash
npm install prisma@6 @prisma/client@6
```

Vérifiez :

```bash
npx prisma --version
```

---

# 14. Initialiser Prisma

Exécutez :

```bash
npx prisma init
```

Prisma crée notamment :

```text
prisma/
└── schema.prisma
```

et un fichier :

```text
.env
```

---

# 15. Le fichier `.env`

Ouvrez :

```text
.env
```

Nous allons définir l'URL de connexion à MySQL.

Par exemple :

```env
DATABASE_URL="mysql://sangconnect:sangconnect123@localhost:3306/sangconnect"
```

Décomposition :

```text
mysql://
    sangconnect
    :
    sangconnect123
    @
    localhost
    :
    3306
    /
    sangconnect
```

Cela signifie :

```text
SGBD       : MySQL
Utilisateur: sangconnect
Mot de passe: sangconnect123
Serveur    : localhost
Port       : 3306
Base       : sangconnect
```

### Attention

Le fichier `.env` peut contenir des informations sensibles.

Il ne doit généralement pas être envoyé dans un dépôt Git public.

Ajoutez donc :

```text
.env
```

dans :

```text
.gitignore
```

---

# 16. Configurer Prisma

Ouvrez :

```text
prisma/schema.prisma
```

Pour cette version pédagogique de Prisma, nous allons utiliser :

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```

Le point important est :

```prisma
provider = "mysql"
```

Cela indique à Prisma que notre base de données est MySQL.

---

# 17. Définir le modèle Donneur

Ajoutez :

```prisma
model Donneur {
  id            Int    @id @default(autoincrement())
  nom           String
  prenom        String
  groupeSanguin String
}
```

Notre fichier devient :

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model Donneur {
  id            Int    @id @default(autoincrement())
  nom           String
  prenom        String
  groupeSanguin String
}
```

---

# 18. Comprendre le modèle Prisma

La ligne :

```prisma
id Int @id @default(autoincrement())
```

signifie :

```text
id
 ↓
entier
 ↓
clé primaire
 ↓
auto-incrémenté
```

La ligne :

```prisma
nom String
```

signifie que `nom` est une chaîne.

---

# 19. Prisma et la table MySQL

Le modèle :

```prisma
model Donneur {
    ...
}
```

représente une table dans MySQL.

On peut avoir conceptuellement :

```text
Prisma

Donneur
   │
   ▼
MySQL

donneurs
```

Prisma se charge de faire correspondre le modèle de l'application avec la structure de la base.

---

# 20. Migration initiale

Nous allons maintenant demander à Prisma de créer la table correspondant au modèle.

Exécutez :

```bash
npx prisma migrate dev --name init
```

Prisma va notamment :

* créer une migration ;
* appliquer la migration à MySQL ;
* créer la table correspondante.

Vous devriez obtenir un dossier :

```text
prisma/
├── schema.prisma
└── migrations/
    └── ..._init/
        └── migration.sql
```

---

# 21. Observer la base de données

Retournez dans MySQL :

```bash
mysql -u sangconnect -p
```

Puis :

```sql
USE sangconnect;
```

Listez les tables :

```sql
SHOW TABLES;
```

Vous devriez notamment trouver :

```text
Donneur
```

et une table utilisée par Prisma pour suivre les migrations.

### Question 3

Quel est l'intérêt d'une migration ?

---

# 22. Pourquoi utiliser les migrations ?

Imaginons que l'équipe de développement ajoute :

```text
telephone
```

au donneur.

Il faut alors modifier la structure de la base.

Au lieu de modifier directement la base à la main, nous allons modifier :

```text
schema.prisma
```

puis créer une nouvelle migration.

Cela permet de conserver un historique des évolutions de la base.

---

# 23. Générer Prisma Client

Après la définition du modèle :

```bash
npx prisma generate
```

Prisma génère le client JavaScript permettant d'interagir avec la base.

Nous pourrons ensuite écrire :

```javascript
prisma.donneur.findMany()
```

ou :

```javascript
prisma.donneur.create(...)
```

---

# 24. Créer le module Prisma

Nous allons centraliser l'accès à Prisma.

Créez :

```text
src/database/
```

puis :

```text
src/database/prisma.js
```

Ajoutez :

```javascript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default prisma;
```

---

# 25. Pourquoi créer un module `prisma.js` ?

Nous pourrions créer :

```javascript
new PrismaClient()
```

dans chaque service.

Mais cela rendrait l'application difficile à maintenir.

Nous allons donc centraliser l'instance :

```text
src/database/prisma.js
```

Puis les repositories utiliseront cette instance.

---

# 26. Premier test de Prisma

Créez temporairement :

```text
src/test-prisma.js
```

avec :

```javascript
import prisma from "./database/prisma.js";

const donneurs = await prisma.donneur.findMany();

console.log(donneurs);

await prisma.$disconnect();
```

Exécutez :

```bash
node src/test-prisma.js
```

Vous devriez obtenir les données présentes dans la table `Donneur`.

---

# 27. Premier constat

Nous avons maintenant remplacé :

```javascript
const donneurs = [];
```

par :

```javascript
await prisma.donneur.findMany();
```

Nous sommes donc passés de :

```text
Mémoire
```

à :

```text
MySQL
```

---

# 28. Prisma CRUD

Prisma fournit des méthodes correspondant aux principales opérations CRUD.

| Opération      | Prisma         |
| -------------- | -------------- |
| Lire plusieurs | `findMany()`   |
| Lire un        | `findUnique()` |
| Créer          | `create()`     |
| Modifier       | `update()`     |
| Supprimer      | `delete()`     |

---

# 29. Lire tous les donneurs

```javascript
const donneurs = await prisma.donneur.findMany();
```

Cela correspond conceptuellement à :

```sql
SELECT * FROM Donneur;
```

---

# 30. Lire un donneur

Pour récupérer le donneur dont l'identifiant vaut `1` :

```javascript
const donneur = await prisma.donneur.findUnique({
    where: {
        id: 1
    }
});
```

Si aucun donneur ne correspond, Prisma retourne :

```text
null
```

---

# 31. Créer un donneur

```javascript
const donneur = await prisma.donneur.create({
    data: {
        nom: "Khelifi",
        prenom: "Yassine",
        groupeSanguin: "AB+"
    }
});
```

---

# 32. Modifier un donneur

```javascript
const donneur = await prisma.donneur.update({
    where: {
        id: 1
    },
    data: {
        nom: "Ben Salem"
    }
});
```

---

# 33. Supprimer un donneur

```javascript
await prisma.donneur.delete({
    where: {
        id: 1
    }
});
```

---

# 34. Introduire le Repository Pattern

Dans les ateliers précédents, nous avions :

```text
Controller
    ↓
Service
    ↓
Données
```

Nous allons maintenant remplacer :

```text
Données
```

par :

```text
Repository
```

L'architecture devient :

```text
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

---

# 35. Créer le repository Donneur

Créez :

```text
src/repositories/
```

puis :

```text
src/repositories/donneur.repository.js
```

Ajoutez :

```javascript
import prisma from "../database/prisma.js";

export function findAll() {
    return prisma.donneur.findMany();
}

export function findById(id) {
    return prisma.donneur.findUnique({
        where: {
            id: id
        }
    });
}

export function create(data) {
    return prisma.donneur.create({
        data: data
    });
}

export function update(id, data) {
    return prisma.donneur.update({
        where: {
            id: id
        },
        data: data
    });
}

export function remove(id) {
    return prisma.donneur.delete({
        where: {
            id: id
        }
    });
}
```

---

# 36. Pourquoi un Repository ?

Le Repository est responsable de l'accès aux données.

Le Service ne doit pas avoir besoin de connaître directement Prisma.

Le Service doit pouvoir dire :

```javascript
donneurRepository.findById(id)
```

sans connaître les détails de la requête Prisma.

Cela permet de séparer :

```text
Logique métier
```

de :

```text
Accès aux données
```

---

# 37. Modifier le Service

Dans :

```text
src/services/donneur.service.js
```

nous allons maintenant utiliser le repository.

Exemple :

```javascript
import * as donneurRepository
    from "../repositories/donneur.repository.js";

export async function getAll() {

    return donneurRepository.findAll();
}
```

Pour rechercher un donneur :

```javascript
export async function getById(id) {

    const donneur =
        await donneurRepository.findById(id);

    if (!donneur) {

        const error = new Error(
            "Donneur introuvable"
        );

        error.statusCode = 404;

        throw error;
    }

    return donneur;
}
```

---

# 38. Pourquoi le `404` appartient au Service ?

Le repository répond à une question technique :

```text
Existe-t-il un donneur avec cet identifiant ?
```

Le service peut ensuite appliquer une règle métier :

```text
Si le donneur n'existe pas,
considérer cela comme une erreur métier 404.
```

Nous conservons donc :

```text
Repository
    ↓
accès aux données

Service
    ↓
logique métier
```

---

# 39. Modifier le Controller

Dans :

```text
src/controllers/donneur.controller.js
```

nous allons utiliser le service.

Exemple :

```javascript
import * as donneurService
    from "../services/donneur.service.js";

export async function getAll(req, res, next) {

    try {

        const donneurs =
            await donneurService.getAll();

        res.json(donneurs);

    } catch (error) {

        next(error);
    }
}
```

---

# 40. Controller — rechercher un donneur

```javascript
export async function getById(req, res, next) {

    try {

        const id = Number(req.params.id);

        const donneur =
            await donneurService.getById(id);

        res.json(donneur);

    } catch (error) {

        next(error);
    }
}
```

Remarquez que le contrôleur :

* récupère la requête ;
* appelle le service ;
* retourne la réponse ;
* transmet les erreurs.

Il ne contient pas de requête Prisma.

---

# 41. Controller — créer un donneur

```javascript
export async function create(req, res, next) {

    try {

        const donneur =
            await donneurService.create(req.body);

        res.status(201).json(donneur);

    } catch (error) {

        next(error);
    }
}
```

---

# 42. Service — créer un donneur

Dans le service :

```javascript
export async function create(data) {

    return donneurRepository.create(data);
}
```

Le flux devient :

```text
POST /api/donneurs
       │
       ▼
Validation Zod
       │
       ▼
Controller
       │
       ▼
Service
       │
       ▼
Repository
       │
       ▼
Prisma
       │
       ▼
MySQL
```

---

# 43. Controller — modifier

```javascript
export async function update(req, res, next) {

    try {

        const id = Number(req.params.id);

        const donneur =
            await donneurService.update(
                id,
                req.body
            );

        res.json(donneur);

    } catch (error) {

        next(error);
    }
}
```

---

# 44. Service — modifier

```javascript
export async function update(id, data) {

    await getById(id);

    return donneurRepository.update(
        id,
        data
    );
}
```

Cette organisation permet au service de vérifier d'abord que le donneur existe.

---

# 45. Controller — supprimer

```javascript
export async function remove(req, res, next) {

    try {

        const id = Number(req.params.id);

        await donneurService.remove(id);

        res.status(204).send();

    } catch (error) {

        next(error);
    }
}
```

---

# 46. Service — supprimer

```javascript
export async function remove(id) {

    await getById(id);

    await donneurRepository.remove(id);
}
```

---

# 47. Routes finales

Le fichier :

```text
src/routes/donneur.routes.js
```

doit maintenant conserver la validation du TP4.

Exemple :

```javascript
import express from "express";

import {
    getAll,
    getById,
    create,
    update,
    remove
} from "../controllers/donneur.controller.js";

import {
    donneurSchema,
    idSchema,
    donneurQuerySchema,
    updateDonneurSchema
} from "../validators/donneur.validator.js";

import { validate }
from "../middlewares/validate.js";

const router = express.Router();

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

router.delete(
    "/:id",
    validate(idSchema, "params"),
    remove
);

export default router;
```

---

# 48. Architecture complète de SangConnect

Nous avons maintenant :

```text
                         CLIENT
                           │
                           ▼
                    ┌──────────────┐
                    │   Express    │
                    └──────┬───────┘
                           │
                           ▼
                        ROUTES
                           │
                           ▼
                     VALIDATION
                        ZOD
                           │
                           ▼
                     CONTROLLER
                           │
                           ▼
                       SERVICE
                           │
                           ▼
                     REPOSITORY
                           │
                           ▼
                    PRISMA CLIENT
                           │
                           ▼
                         MYSQL
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

# 49. Tester le CRUD complet

## 49.1 Lister

```bash
curl http://localhost:3000/api/donneurs
```

---

## 49.2 Consulter

```bash
curl http://localhost:3000/api/donneurs/1
```

---

## 49.3 Créer

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}'
```

---

## 49.4 Modifier

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

## 49.5 Modifier partiellement

```bash
curl -X PATCH http://localhost:3000/api/donneurs/1 \
-H "Content-Type: application/json" \
-d '{
    "nom": "NouveauNom"
}'
```

---

## 49.6 Supprimer

```bash
curl -i -X DELETE \
http://localhost:3000/api/donneurs/1
```

---

# 50. Vérifier directement MySQL

Après avoir créé un donneur avec l'API :

```bash
mysql -u sangconnect -p
```

Puis :

```sql
USE sangconnect;
```

et :

```sql
SELECT * FROM Donneur;
```

Vous devez retrouver le donneur créé depuis l'API.

### Question 4

Expliquez le chemin parcouru par la donnée suivante :

```json
{
    "nom": "Khelifi",
    "prenom": "Yassine",
    "groupeSanguin": "AB+"
}
```

lors d'une requête :

```text
POST /api/donneurs
```

---

# 51. Prisma Studio

Prisma fournit un outil permettant d'examiner les données de la base.

Lancez :

```bash
npx prisma studio
```

Une interface permettant d'explorer les données s'ouvre.

Vous pouvez notamment :

* consulter les donneurs ;
* ajouter des données ;
* modifier des données ;
* supprimer des données.

### Question 5

Quelle différence existe-t-il entre :

```text
Prisma Studio
```

et :

```text
MySQL
```

?

---

# 52. Ajouter un champ `telephone`

Notre modèle `Donneur` est actuellement :

```prisma
model Donneur {
  id            Int    @id @default(autoincrement())
  nom           String
  prenom        String
  groupeSanguin String
}
```

Nous voulons maintenant ajouter :

```text
telephone
```

Comme tous les donneurs existants n'ont pas nécessairement un téléphone enregistré, nous allons rendre le champ optionnel :

```prisma
telephone String?
```

Le modèle devient :

```prisma
model Donneur {
  id            Int     @id @default(autoincrement())
  nom           String
  prenom        String
  groupeSanguin String
  telephone     String?
}
```

---

# 53. Créer une nouvelle migration

Exécutez :

```bash
npx prisma migrate dev --name add_telephone
```

Prisma va générer une nouvelle migration.

Nous avons maintenant :

```text
prisma/
├── schema.prisma
└── migrations/
    ├── ..._init/
    └── ..._add_telephone/
```

### Question 6

Pourquoi ne modifions-nous pas directement la table MySQL ?

---

# 54. Adapter la validation Zod

La base accepte maintenant :

```text
telephone
```

Nous devons également adapter le schéma Zod.

Dans :

```text
src/validators/donneur.validator.js
```

ajoutez :

```javascript
telephone: z.string().min(8).optional()
```

Le schéma devient par exemple :

```javascript
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
    ]),

    telephone: z.string().min(8).optional()
});
```

---

# 55. Tester la nouvelle propriété

Créer :

```bash
curl -X POST http://localhost:3000/api/donneurs \
-H "Content-Type: application/json" \
-d '{
    "nom": "Mansouri",
    "prenom": "Amine",
    "groupeSanguin": "O+",
    "telephone": "73123456"
}'
```

Puis :

```bash
curl http://localhost:3000/api/donneurs
```

---

# 56. Exercice 1 — Comprendre Prisma

Répondre aux questions suivantes.

### Question 1

Quelle est la différence entre :

```text
MySQL
```

et :

```text
Prisma
```

### Question 2

Quel est le rôle de :

```text
schema.prisma
```

?

### Question 3

Quel est le rôle de :

```text
Prisma Client
```

?

### Question 4

Quel est le rôle d'une migration ?

---

# 57. Exercice 2 — Repository

Compléter le repository afin de disposer des opérations :

```text
findAll()
findById()
create()
update()
remove()
```

Puis expliquer pourquoi ces opérations ne doivent pas être placées directement dans le contrôleur.

---

# 58. Exercice 3 — Recherche par groupe sanguin

Nous avions déjà implémenté :

```text
GET /api/donneurs?groupeSanguin=A+
```

Dans l'Atelier 3, cette recherche était effectuée sur un tableau JavaScript.

Nous devons maintenant effectuer cette recherche directement dans MySQL à travers Prisma.

Modifier le repository.

Indice :

```javascript
prisma.donneur.findMany({
    where: {
        groupeSanguin: ...
    }
});
```

Le service doit pouvoir recevoir :

```javascript
groupeSanguin
```

et demander au repository les donneurs correspondants.

---

# 59. Exercice 4 — Trier les donneurs

Ajouter la possibilité de demander :

```text
GET /api/donneurs?sort=nom
```

ou :

```text
GET /api/donneurs?sort=prenom
```

Réfléchir à la manière de transmettre le paramètre :

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Prisma
```

### Question

Pourquoi est-il préférable de réaliser le tri au niveau de la base de données plutôt que de récupérer toutes les données puis de les trier en JavaScript ?

---

# 60. Exercice 5 — Rechercher par nom

Ajouter :

```text
GET /api/donneurs?nom=Ben
```

Le résultat doit contenir les donneurs dont le nom correspond au critère.

Utiliser les possibilités de filtrage de Prisma.

---

# 61. Exercice 6 — Ajouter la ressource Centre

Nous allons commencer à préparer la suite de SangConnect.

Créer le modèle :

```prisma
model Centre {
  id        Int    @id @default(autoincrement())
  nom       String
  ville     String
  adresse   String
  telephone String
}
```

Créer une migration :

```bash
npx prisma migrate dev --name add_centre
```

Puis créer :

```text
src/repositories/centre.repository.js
src/services/centre.service.js
src/controllers/centre.controller.js
src/routes/centre.routes.js
src/validators/centre.validator.js
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

---

# 62. Exercice 7 — Comparer l'ancienne et la nouvelle architecture

Avant l'Atelier 5 :

```text
Route
  ↓
Validation
  ↓
Controller
  ↓
Service
  ↓
Tableau JavaScript
```

Après l'Atelier 5 :

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

### Travail demandé

Expliquer précisément pourquoi l'ajout de `Repository` améliore l'architecture.

---

# 63. Exercice 8 — Gestion des erreurs Prisma

Tester une situation dans laquelle une opération Prisma échoue.

Par exemple, essayer de modifier ou supprimer un identifiant inexistant.

Observer l'erreur.

Puis adapter le Service afin de transformer cette erreur technique en réponse HTTP appropriée.

L'objectif est de ne pas exposer directement les détails internes de Prisma au client.

---

# 64. Bilan de l'atelier

Nous avons effectué une évolution importante de SangConnect.

Au départ :

```text
Données en mémoire
```

À la fin :

```text
MySQL
```

Nous avons introduit :

```text
Prisma ORM
```

et :

```text
Repository Pattern
```

L'architecture complète est maintenant :

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
                    Validation Zod
                           │
                           ▼
                       Controller
                           │
                           ▼
                        Service
                           │
                           ▼
                      Repository
                           │
                           ▼
                      Prisma ORM
                           │
                           ▼
                         MySQL
```

---

# 65. Ce que l'étudiant doit retenir

## MySQL

MySQL assure la persistance des données.

```text
Application
    ↓
MySQL
```

---

## Prisma

Prisma fournit une abstraction permettant à Node.js d'interagir avec la base.

```text
Node.js
   ↓
Prisma
   ↓
MySQL
```

---

## Migration

Une migration permet de faire évoluer la structure de la base de manière contrôlée.

```text
schema.prisma
      ↓
migration
      ↓
MySQL
```

---

## Repository

Le Repository centralise l'accès aux données.

```text
Service
   ↓
Repository
   ↓
Prisma
```

---

# 66. Architecture globale des cinq premiers ateliers

Nous pouvons maintenant voir la progression des ateliers.

## Atelier 1

```text
Node.js
   ↓
Serveur HTTP
```

## Atelier 2

```text
Node.js
   ↓
Express / architecture en couches
   ↓
Route
   ↓
Controller
   ↓
Service
```

## Atelier 3

```text
Express
   ↓
API REST
   ↓
CRUD
```

## Atelier 4

```text
API REST
   ↓
Validation Zod
   ↓
Gestion centralisée des erreurs
```

## Atelier 5

```text
API REST
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

La progression est donc :

```text
Node.js
   ↓
Express
   ↓
REST
   ↓
Validation
   ↓
Persistance
```

---

# 67. Préparation de l'Atelier 6

Notre modèle commence à devenir intéressant.

Nous avons actuellement :

```text
Donneur
Centre
```

Mais dans une application de gestion des dons de sang, un donneur peut effectuer plusieurs dons.

Nous devons donc pouvoir représenter des relations entre les entités.

Nous allons introduire progressivement :

```text
Donneur
   │
   │ 1
   │
   └───────────< Don
                  │
                  │
                  ▼
                Centre
```

L'Atelier 6 portera notamment sur :

* les relations Prisma ;
* les clés étrangères ;
* les relations `1-N` ;
* `include` ;
* `select` ;
* les repositories relationnels ;
* les requêtes plus complexes ;
* le filtrage ;
* le tri ;
* la préparation aux problèmes de performance et au phénomène N+1.

---

# 68. Travail pratique à faire à la maison

## Sujet : persistance complète de la ressource Centre

Compléter SangConnect afin de gérer les centres de collecte avec MySQL et Prisma.

### 1. Modèle

Créer :

```prisma
model Centre {
    id        Int    @id @default(autoincrement())
    nom       String
    ville     String
    adresse   String
    telephone String
}
```

### 2. Migration

Créer la migration :

```bash
npx prisma migrate dev --name add_centre
```

### 3. Validation

Créer un schéma Zod :

```text
centre.validator.js
```

### 4. Repository

Créer :

```text
centre.repository.js
```

avec :

```text
findAll()
findById()
create()
update()
remove()
```

### 5. Service

Créer :

```text
centre.service.js
```

### 6. Controller

Créer :

```text
centre.controller.js
```

### 7. Routes

Implémenter :

```text
GET    /api/centres
GET    /api/centres/:id
POST   /api/centres
PUT    /api/centres/:id
PATCH  /api/centres/:id
DELETE /api/centres/:id
```

### 8. Tests

Tester toutes les opérations avec `curl`.

### 9. Base de données

Vérifier les données avec :

```bash
npx prisma studio
```

et directement avec MySQL :

```sql
SELECT * FROM Centre;
```

### 10. Compte rendu

Le compte rendu doit contenir :

* l'arborescence du projet ;
* le modèle Prisma ;
* la migration ;
* le Repository ;
* le Service ;
* le Controller ;
* les routes ;
* les tests ;
* quelques captures de Prisma Studio ;
* une explication du flux :

```text
HTTP
 → Route
 → Validation
 → Controller
 → Service
 → Repository
 → Prisma
 → MySQL
```
