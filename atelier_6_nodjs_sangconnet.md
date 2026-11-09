# Atelier 6 — Relations entre données et requêtes avancées avec Prisma

**Projet fil rouge : SangConnect — API de gestion des dons de sang**
**Durée : 3 heures**
**Technologies : Node.js · Express · Zod · Prisma ORM · MySQL**

## 1. Introduction

Dans l’atelier 5, nous avons remplacé les données stockées en mémoire par une véritable base **MySQL**, manipulée à travers **Prisma ORM**.

Nous avons notamment appris à :

* définir un modèle Prisma ;
* créer des migrations ;
* utiliser `PrismaClient` ;
* réaliser les opérations CRUD ;
* organiser l'accès aux données avec le **Repository Pattern** ;
* conserver l'architecture :

```text
Client HTTP
     ↓
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

Cependant, notre modèle reste encore très simple :

```text
Donneur
```

Dans une véritable application de gestion des dons de sang, un donneur peut effectuer plusieurs dons et chaque don est réalisé dans un centre.

Nous allons donc maintenant apprendre à représenter ces **relations entre les données**.

---

# 2. Objectifs de l'atelier

À la fin de cet atelier, vous devez être capable de :

* comprendre les relations entre tables SQL ;
* représenter une relation dans Prisma ;
* utiliser les clés étrangères ;
* créer une relation **1-N** ;
* utiliser `@relation` dans Prisma ;
* créer des données liées ;
* récupérer des données avec `include` ;
* limiter les champs retournés avec `select` ;
* effectuer des recherches et filtrages avec Prisma ;
* trier les résultats ;
* effectuer des requêtes combinées ;
* comprendre le problème des requêtes **N+1** ;
* organiser ces requêtes dans le Repository et le Service.

---

# 3. Mise en situation : SangConnect

Dans SangConnect, nous avons actuellement des donneurs :

```text
Donneur
--------------------
id
nom
prenom
groupeSanguin
telephone
```

Nous souhaitons maintenant gérer les dons.

Un don possède par exemple :

```text
Don
--------------------
id
dateDon
volume
donneurId
centreId
```

Un don appartient donc :

* à un **donneur** ;
* à un **centre**.

On obtient le modèle conceptuel suivant :

```text
DONNEUR
   │
   │ 1
   │
   │ N
   ▼
  DON
   │
   │ N
   │
   │ 1
   ▼
 CENTRE
```

---

# 4. Rappel : relation 1-N

Une relation **1-N** signifie qu'une occurrence d'une table peut être associée à plusieurs occurrences d'une autre table.

Par exemple :

> Un donneur peut effectuer plusieurs dons.

```text
Donneur 1 ─────────── N Don
```

Exemple :

```text
Donneur
id = 1
nom = Ben Ali
```

peut avoir :

```text
Don
id = 1
donneurId = 1

Don
id = 2
donneurId = 1

Don
id = 3
donneurId = 1
```

La colonne :

```text
donneurId
```

est une **clé étrangère**.

---

# 5. Activité 1 — Préparer les modèles Prisma

## Question 1

Ouvrez :

```text
prisma/schema.prisma
```

Vous devez déjà avoir un modèle `Donneur`.

Complétez le fichier avec les modèles suivants :

```prisma
model Donneur {
  id            Int    @id @default(autoincrement())
  nom           String
  prenom        String
  groupeSanguin String
  telephone     String?

  dons          Don[]
}

model Centre {
  id        Int    @id @default(autoincrement())
  nom       String
  ville     String

  dons      Don[]
}

model Don {
  id        Int      @id @default(autoincrement())
  dateDon   DateTime @default(now())
  volume    Int

  donneurId Int
  centreId  Int

  donneur   Donneur  @relation(fields: [donneurId], references: [id])
  centre    Centre   @relation(fields: [centreId], references: [id])
}
```

### Questions

1. Quel est le rôle de :

```prisma
dons Don[]
```

2. Quel est le rôle de :

```prisma
donneurId Int
```

3. Quelle est la différence entre :

```prisma
donneurId Int
```

et :

```prisma
donneur Donneur
```

4. Que signifie :

```prisma
@relation(fields: [donneurId], references: [id])
```

---

# 6. Comprendre la relation Prisma

La relation entre `Don` et `Donneur` est définie par :

```prisma
donneurId Int

donneur Donneur @relation(
    fields: [donneurId],
    references: [id]
)
```

Cela signifie que :

```text
Don.donneurId
```

référence :

```text
Donneur.id
```

On peut représenter cette relation ainsi :

```text
Donneur
+----------------+
| id             |
| nom            |
| prenom         |
+----------------+
       ▲
       │
       │ donneurId
       │
+----------------+
| Don            |
+----------------+
| id             |
| dateDon        |
| volume         |
| donneurId      |
| centreId       |
+----------------+
```

---

# 7. Activité 2 — Créer la migration

Après avoir modifié le schéma :

```bash
npx prisma migrate dev --name add_dons_centres
```

Puis :

```bash
npx prisma generate
```

Vérifiez ensuite la base MySQL.

```bash
mysql -u sangconnect -p
```

Puis :

```sql
USE sangconnect;

SHOW TABLES;
```

Vous devriez retrouver notamment :

```text
Donneur
Centre
Don
```

Selon la configuration Prisma/MySQL, les noms exacts peuvent respecter la convention générée par Prisma.

---

# 8. Activité 3 — Comprendre les clés étrangères

Examinez la table `Don`.

Elle doit contenir notamment :

```text
id
dateDon
volume
donneurId
centreId
```

### Questions

1. Pourquoi `donneurId` est-il nécessaire ?
2. Pourquoi ne stockons-nous pas directement le nom du donneur dans `Don` ?
3. Que se passerait-il si nous supprimions la relation entre `Don` et `Donneur` ?
4. Quel avantage apporte une clé étrangère ?

---

# 9. Activité 4 — Créer des centres

Ajoutez quelques centres dans la base.

Vous pouvez utiliser Prisma ou directement MySQL pour comprendre les données.

Exemple :

```text
Centre 1
Nom : Centre régional de Sousse
Ville : Sousse

Centre 2
Nom : Centre régional de Tunis
Ville : Tunis

Centre 3
Nom : Centre régional de Sfax
Ville : Sfax
```

L'objectif est d'obtenir plusieurs centres avant de créer les dons.

---

# 10. Activité 5 — Créer un don avec Prisma

Dans un script de test, créez un don :

```javascript
import prisma from "./database/prisma.js";

const don = await prisma.don.create({
    data: {
        volume: 450,
        donneurId: 1,
        centreId: 1
    }
});

console.log(don);

await prisma.$disconnect();
```

Exécutez le script.

### Question

Que représente :

```javascript
donneurId: 1
```

et :

```javascript
centreId: 1
```

---

# 11. Créer plusieurs dons

Créez plusieurs dons.

Par exemple :

```javascript
await prisma.don.create({
    data: {
        volume: 450,
        donneurId: 1,
        centreId: 1
    }
});

await prisma.don.create({
    data: {
        volume: 450,
        donneurId: 1,
        centreId: 2
    }
});

await prisma.don.create({
    data: {
        volume: 500,
        donneurId: 2,
        centreId: 1
    }
});
```

Nous avons maintenant une situation de ce type :

```text
Donneur 1
   │
   ├── Don 1 → Centre 1
   │
   └── Don 2 → Centre 2

Donneur 2
   │
   └── Don 3 → Centre 1
```

---

# 12. Activité 6 — Récupérer les dons

Commencez par une requête simple :

```javascript
const dons = await prisma.don.findMany();

console.log(dons);
```

Vous obtenez les informations du don :

```text
id
dateDon
volume
donneurId
centreId
```

Mais pas les informations du donneur ni du centre.

Nous allons utiliser `include`.

---

# 13. Utiliser `include`

Essayez :

```javascript
const dons = await prisma.don.findMany({
    include: {
        donneur: true,
        centre: true
    }
});
```

Affichez :

```javascript
console.log(dons);
```

Vous devriez maintenant obtenir une structure proche de :

```javascript
[
    {
        id: 1,
        dateDon: "...",
        volume: 450,
        donneurId: 1,
        centreId: 1,

        donneur: {
            id: 1,
            nom: "...",
            prenom: "...",
            groupeSanguin: "A+"
        },

        centre: {
            id: 1,
            nom: "...",
            ville: "Sousse"
        }
    }
]
```

---

# 14. Question importante

Comparez :

```javascript
const dons = await prisma.don.findMany();
```

avec :

```javascript
const dons = await prisma.don.findMany({
    include: {
        donneur: true,
        centre: true
    }
});
```

### Question

Dans quel cas utiliseriez-vous `include` ?

---

# 15. Activité 7 — Utiliser `select`

`include` peut retourner beaucoup d'informations.

Nous pouvons limiter les champs :

```javascript
const dons = await prisma.don.findMany({
    select: {
        id: true,
        dateDon: true,
        volume: true,

        donneur: {
            select: {
                nom: true,
                prenom: true,
                groupeSanguin: true
            }
        },

        centre: {
            select: {
                nom: true,
                ville: true
            }
        }
    }
});
```

Le résultat est plus précis.

### Question

Pourquoi est-il intéressant de ne pas retourner systématiquement toutes les colonnes ?

Réfléchissez notamment :

* aux performances ;
* à la quantité de données transférées ;
* à la sécurité ;
* à la conception d'une API REST.

---

# 16. Activité 8 — Récupérer les dons d'un donneur

Nous voulons maintenant obtenir :

```http
GET /api/donneurs/1/dons
```

La requête Prisma peut être :

```javascript
const dons = await prisma.don.findMany({
    where: {
        donneurId: 1
    }
});
```

Mais nous pouvons également récupérer les dons directement à partir du donneur :

```javascript
const donneur = await prisma.donneur.findUnique({
    where: {
        id: 1
    },
    include: {
        dons: true
    }
});
```

### Question

Comparez les deux approches.

Quelle différence existe entre :

```text
Don → rechercher les dons du donneur
```

et :

```text
Donneur → charger ses dons
```

---

# 17. Activité 9 — Rechercher selon le groupe sanguin

Nous souhaitons maintenant trouver les dons effectués par les donneurs du groupe `A+`.

Une première approche consiste à filtrer à travers la relation :

```javascript
const dons = await prisma.don.findMany({
    where: {
        donneur: {
            groupeSanguin: "A+"
        }
    },
    include: {
        donneur: true,
        centre: true
    }
});
```

### Question

Expliquez la structure :

```javascript
where: {
    donneur: {
        groupeSanguin: "A+"
    }
}
```

Nous réalisons ici un **filtrage sur une donnée appartenant à une autre table**.

---

# 18. Activité 10 — Filtrer par centre

Recherchez tous les dons réalisés dans le centre numéro `1`.

```javascript
const dons = await prisma.don.findMany({
    where: {
        centreId: 1
    },
    include: {
        donneur: true,
        centre: true
    }
});
```

Puis recherchez les dons réalisés dans la ville de Sousse :

```javascript
const dons = await prisma.don.findMany({
    where: {
        centre: {
            ville: "Sousse"
        }
    },
    include: {
        donneur: true,
        centre: true
    }
});
```

### Question

Quelle différence existe entre :

```javascript
centreId: 1
```

et :

```javascript
centre: {
    ville: "Sousse"
}
```

---

# 19. Activité 11 — Trier les dons

Nous souhaitons afficher les dons du plus récent au plus ancien.

```javascript
const dons = await prisma.don.findMany({
    orderBy: {
        dateDon: "desc"
    }
});
```

Pour trier par volume :

```javascript
const dons = await prisma.don.findMany({
    orderBy: {
        volume: "desc"
    }
});
```

### Travail demandé

Testez :

```text
dateDon ASC
dateDon DESC
volume ASC
volume DESC
```

Expliquez la différence entre :

```text
asc
```

et :

```text
desc
```

---

# 20. Activité 12 — Combiner filtrage et tri

Construisez une requête permettant d'obtenir :

> Tous les dons du groupe sanguin A+, réalisés à Sousse, classés du plus récent au plus ancien.

Vous disposez des éléments suivants :

```javascript
where: {
    donneur: {
        groupeSanguin: "A+"
    },
    centre: {
        ville: "Sousse"
    }
}
```

et :

```javascript
orderBy: {
    dateDon: "desc"
}
```

### Travail demandé

Construisez la requête complète.

---

# 21. Activité 13 — Pagination

Une API réelle ne doit généralement pas retourner plusieurs milliers de résultats en une seule réponse.

Prisma permet d'utiliser :

```javascript
skip
```

et :

```javascript
take
```

Exemple :

```javascript
const dons = await prisma.don.findMany({
    skip: 0,
    take: 10
});
```

Cela permet de récupérer les 10 premiers résultats.

Pour la page suivante :

```javascript
const dons = await prisma.don.findMany({
    skip: 10,
    take: 10
});
```

### Question

Si nous voulons afficher :

```text
page = 3
limit = 10
```

quelle valeur faut-il donner à :

```javascript
skip
```

---

# 22. Activité 14 — Pagination dans une API

Nous voulons maintenant :

```http
GET /api/dons?page=2&limit=10
```

Dans Express :

```javascript
const page = Number(req.query.page || 1);
const limit = Number(req.query.limit || 10);
```

Puis :

```javascript
const skip = (page - 1) * limit;
```

Enfin :

```javascript
const dons = await prisma.don.findMany({
    skip,
    take: limit
});
```

### Travail demandé

Complétez cette fonctionnalité pour obtenir :

```http
GET /api/dons?page=1&limit=10
GET /api/dons?page=2&limit=10
GET /api/dons?page=3&limit=10
```

---

# 23. Activité 15 — Compter les résultats

Pour connaître le nombre total de dons :

```javascript
const total = await prisma.don.count();
```

Nous pouvons donc préparer une réponse API :

```javascript
res.json({
    data: dons,
    pagination: {
        page,
        limit,
        total
    }
});
```

### Question

Pourquoi est-il utile de retourner `total` avec les résultats paginés ?

---

# 24. Activité 16 — Créer le Repository des dons

Nous allons maintenant intégrer les requêtes dans notre architecture.

Créez :

```text
src/repositories/don.repository.js
```

Commencez par :

```javascript
import prisma from "../database/prisma.js";
```

Puis créez :

```javascript
export function findAll() {
    return prisma.don.findMany({
        include: {
            donneur: true,
            centre: true
        }
    });
}
```

Ajoutez ensuite :

```javascript
export function findById(id) {
    return prisma.don.findUnique({
        where: {
            id
        },
        include: {
            donneur: true,
            centre: true
        }
    });
}
```

---

# 25. Activité 17 — Ajouter la recherche par donneur

Dans le Repository, ajoutez :

```javascript
export function findByDonneur(donneurId) {
    return prisma.don.findMany({
        where: {
            donneurId
        },
        include: {
            centre: true
        },
        orderBy: {
            dateDon: "desc"
        }
    });
}
```

### Question

Pourquoi le Repository est-il un bon endroit pour placer cette requête ?

---

# 26. Activité 18 — Ajouter le Service

Créez :

```text
src/services/don.service.js
```

Ajoutez :

```javascript
import * as donRepository
    from "../repositories/don.repository.js";

export async function getAll() {
    return donRepository.findAll();
}

export async function getById(id) {
    const don = await donRepository.findById(id);

    if (!don) {
        const error = new Error("Don introuvable");
        error.statusCode = 404;
        throw error;
    }

    return don;
}
```

Puis :

```javascript
export async function getByDonneur(donneurId) {
    return donRepository.findByDonneur(donneurId);
}
```

---

# 27. Activité 19 — Créer le Controller

Créez :

```text
src/controllers/don.controller.js
```

Exemple :

```javascript
import * as donService
    from "../services/don.service.js";

export async function getAll(req, res, next) {
    try {
        const dons = await donService.getAll();

        res.json(dons);
    } catch (error) {
        next(error);
    }
}
```

Puis :

```javascript
export async function getById(req, res, next) {
    try {
        const id = Number(req.params.id);

        const don = await donService.getById(id);

        res.json(don);
    } catch (error) {
        next(error);
    }
}
```

---

# 28. Activité 20 — Créer les routes

Créez :

```text
src/routes/don.routes.js
```

Puis :

```javascript
import express from "express";

import {
    getAll,
    getById
} from "../controllers/don.controller.js";

const router = express.Router();

router.get("/", getAll);
router.get("/:id", getById);

export default router;
```

Dans `server.js` :

```javascript
import donRoutes from "./routes/don.routes.js";

app.use("/api/dons", donRoutes);
```

---

# 29. Tester l'API

Démarrez le serveur :

```bash
npm run dev
```

ou :

```bash
node --watch src/server.js
```

Testez :

```bash
curl http://localhost:3000/api/dons
```

Puis :

```bash
curl http://localhost:3000/api/dons/1
```

---

# 30. Activité 21 — Comprendre le problème N+1

Considérons le code suivant :

```javascript
const donneurs = await prisma.donneur.findMany();

for (const donneur of donneurs) {

    const dons = await prisma.don.findMany({
        where: {
            donneurId: donneur.id
        }
    });

    console.log(donneur.nom, dons);
}
```

Supposons qu'il existe 100 donneurs.

On réalise :

```text
1 requête pour récupérer les donneurs
+
100 requêtes pour récupérer leurs dons
=
101 requêtes
```

C'est le problème **N+1**.

---

# 31. Activité 22 — Éviter N+1 avec `include`

Essayez plutôt :

```javascript
const donneurs = await prisma.donneur.findMany({
    include: {
        dons: true
    }
});
```

Nous demandons directement à Prisma de récupérer les données liées.

### Question

Comparez les deux approches :

### Approche A

```javascript
findMany()
```

puis une requête dans une boucle.

### Approche B

```javascript
findMany({
    include: {
        dons: true
    }
})
```

Pourquoi l'approche B est-elle généralement préférable ?

---

# 32. Activité 23 — `include` ou `select` ?

Comparez :

```javascript
include: {
    donneur: true,
    centre: true
}
```

avec :

```javascript
select: {
    id: true,
    volume: true,

    donneur: {
        select: {
            nom: true,
            groupeSanguin: true
        }
    },

    centre: {
        select: {
            nom: true
        }
    }
}
```

### Travail demandé

Pour une API publique, choisissez l'approche que vous considérez comme la plus adaptée et justifiez votre choix.

---

# 33. Activité 24 — Concevoir une API plus complète

Votre API doit maintenant proposer :

```text
GET /api/dons
GET /api/dons/:id
GET /api/dons?centreId=1
GET /api/dons?donneurId=2
GET /api/dons?ville=Sousse
GET /api/dons?groupeSanguin=A+
GET /api/dons?page=1&limit=10
```

### Travail demandé

Concevez la logique permettant de combiner plusieurs paramètres.

Exemple :

```http
GET /api/dons?ville=Sousse&groupeSanguin=A+&page=1&limit=10
```

La requête doit :

1. filtrer par ville ;
2. filtrer par groupe sanguin ;
3. trier les résultats ;
4. appliquer la pagination ;
5. retourner le nombre total de résultats.

---

# 34. Synthèse de l'atelier

Nous avons progressivement enrichi SangConnect.

Au début :

```text
Donneur
```

Puis :

```text
Donneur
   │
   │ 1-N
   ▼
  Don
```

Puis :

```text
Donneur
   │
   │
   ▼
  Don
   │
   │
   ▼
 Centre
```

Nous avons appris à exploiter ces relations avec Prisma :

```javascript
include
```

```javascript
select
```

```javascript
where
```

```javascript
orderBy
```

```javascript
skip
```

```javascript
take
```

```javascript
count
```

Nous avons également commencé à prendre en compte les problèmes de performance, notamment :

```text
N+1 queries
```

---

# 35. Architecture obtenue

À la fin de l'atelier, notre architecture devient :

```text
                    Client
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
                  Repository
                      │
                      ▼
                    Prisma
                      │
                      ▼
                    MySQL
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     Donneur         Don          Centre
```

Cette organisation permet de séparer clairement :

| Couche     | Responsabilité       |
| ---------- | -------------------- |
| Route      | Définir les URL      |
| Validation | Vérifier les données |
| Controller | Gérer HTTP           |
| Service    | Logique métier       |
| Repository | Accès aux données    |
| Prisma     | ORM                  |
| MySQL      | Stockage             |

---

# 36. Travail pratique à faire à la maison

## Mini-projet — Gestion des dons

Complétez l'API SangConnect afin de gérer complètement la ressource **Don**.

### Partie 1 — Modèle

Le modèle `Don` doit contenir au minimum :

```text
id
dateDon
volume
donneurId
centreId
```

---

### Partie 2 — CRUD

Implémentez :

```http
GET    /api/dons
GET    /api/dons/:id
POST   /api/dons
PUT    /api/dons/:id
PATCH  /api/dons/:id
DELETE /api/dons/:id
```

---

### Partie 3 — Relations

Lors de l'affichage d'un don, retournez également :

```text
Donneur
Centre
```

mais uniquement avec les champs nécessaires.

---

### Partie 4 — Recherche

Implémentez :

```http
GET /api/dons?ville=Sousse
```

```http
GET /api/dons?groupeSanguin=A+
```

```http
GET /api/dons?donneurId=2
```

---

### Partie 5 — Tri

Permettez de trier par :

```text
dateDon
volume
```

avec :

```text
asc
desc
```

---

### Partie 6 — Pagination

Implémentez :

```http
GET /api/dons?page=2&limit=10
```

La réponse devra avoir une structure proche de :

```json
{
    "data": [],
    "pagination": {
        "page": 2,
        "limit": 10,
        "total": 35
    }
}
```

---

### Partie 7 — Analyse

Dans votre compte rendu, expliquez :

1. le rôle d'une clé étrangère ;
2. le rôle de `@relation` ;
3. la différence entre `include` et `select` ;
4. l'intérêt de `where` ;
5. le fonctionnement de `skip` et `take` ;
6. le problème N+1 ;
7. pourquoi les requêtes Prisma doivent être placées dans le Repository plutôt que directement dans le Controller.

---

## Préparation de l'Atelier 7

L'étape suivante pourra naturellement porter sur **les requêtes avancées, transactions et performance avec Prisma/MySQL**, notamment :

* requêtes complexes ;
* agrégations ;
* `count`, `sum`, `avg` ;
* transactions Prisma ;
* cohérence des opérations métier ;
* gestion des erreurs de base de données ;
* optimisation des requêtes ;
* approfondissement du problème N+1 ;
* éventuellement introduction au **Repository Pattern avancé**.

Cela préparera ensuite la partie **authentification, autorisation et sécurité** du module Backend.
