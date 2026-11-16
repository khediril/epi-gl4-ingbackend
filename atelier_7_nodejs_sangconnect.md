# Atelier 7 — Authentification, JWT et gestion des utilisateurs

**Durée : 3 heures**
**Projet : SangConnect — API de gestion des dons de sang**
**Stack : Node.js + Express + Zod + Prisma + MySQL**

## 1. Introduction

Jusqu'à présent, notre API accepte des requêtes sans réellement savoir **qui** les effectue.

Par exemple, actuellement un client pourrait théoriquement appeler :

```http
DELETE /api/dons/15
```

sans fournir aucune information permettant de savoir qui est à l'origine de cette requête.

Dans une application Backend réelle, nous devons répondre à deux questions différentes :

### Question 1 — Qui êtes-vous ?

C'est le problème de **l'authentification**.

```text
Utilisateur
    ↓
Identifiant + mot de passe
    ↓
Authentification
    ↓
Utilisateur identifié
```

### Question 2 — Avez-vous le droit de faire cette opération ?

C'est le problème de **l'autorisation**.

```text
Utilisateur authentifié
        ↓
       rôle
        ↓
Autorisation
        ↓
Accès autorisé/refusé
```

Dans cet atelier, nous allons commencer par construire le système d'authentification de SangConnect.

---

# 2. Objectifs

À la fin de l'atelier, vous serez capable de :

* distinguer authentification et autorisation ;
* créer un modèle `User` avec Prisma ;
* stocker un mot de passe de manière sécurisée ;
* comprendre pourquoi on ne stocke jamais un mot de passe en clair ;
* utiliser `argon2` pour le hachage ;
* créer une inscription ;
* créer une connexion ;
* générer un JWT ;
* comprendre la structure d'un JWT ;
* créer un middleware d'authentification ;
* récupérer l'utilisateur connecté ;
* protéger une route ;
* gérer les erreurs d'authentification.

---

# 3. Architecture cible

Nous allons ajouter une nouvelle partie à notre architecture :

```text
Client
   │
   ▼
Route
   │
   ▼
Middleware Auth
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

Le middleware d'authentification sera placé **avant le Controller**.

---

# 4. Activité 1 — Identifier les utilisateurs

SangConnect possède plusieurs catégories d'utilisateurs :

```text
Administrateur
Agent de centre
Responsable de stock
Client API
```

Nous allons commencer par un modèle simple :

```text
User
--------------------
id
nom
email
password
role
createdAt
updatedAt
```

### Question

Pourquoi avons-nous besoin d'un `role` ?

Réfléchissez à la différence entre :

```text
Administrateur
```

et :

```text
Agent de centre
```

Un administrateur ne devrait probablement pas avoir exactement les mêmes droits qu'un agent.

---

# 5. Activité 2 — Créer le modèle User

Dans :

```text
prisma/schema.prisma
```

ajoutez :

```prisma
enum Role {
  ADMIN
  AGENT_CENTRE
  RESPONSABLE_STOCK
}

model User {
  id        Int      @id @default(autoincrement())
  nom       String
  email     String   @unique
  password  String
  role      Role     @default(AGENT_CENTRE)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### Questions

1. Pourquoi `email` est-il `@unique` ?
2. Pourquoi `password` est-il de type `String` ?
3. Pourquoi le rôle est-il un `enum` ?
4. Quelle valeur sera utilisée par défaut ?

---

# 6. Activité 3 — Migration

Créez une migration :

```bash
npx prisma migrate dev --name add_users
```

Puis :

```bash
npx prisma generate
```

Vérifiez la structure de la base MySQL.

Vous devez maintenant avoir une table correspondant à `User`.

---

# 7. Activité 4 — Le problème du mot de passe

Imaginons que l'utilisateur saisisse :

```text
MotDePasse123
```

Il serait extrêmement dangereux d'enregistrer directement :

```text
password = "MotDePasse123"
```

dans MySQL.

### Pourquoi ?

Si la base de données est compromise, l'attaquant obtient directement les mots de passe.

Nous devons donc utiliser un **hachage cryptographique adapté aux mots de passe**.

Nous utiliserons :

```text
Argon2
```

---

# 8. Activité 5 — Installer Argon2

Dans le projet :

```bash
npm install argon2
```

Puis testez l'installation.

Créez :

```text
src/utils/password.js
```

Commencez par :

```javascript
import argon2 from "argon2";
```

Puis créez :

```javascript
export async function hashPassword(password) {
    return argon2.hash(password);
}
```

---

# 9. Activité 6 — Hacher un mot de passe

Testez :

```javascript
const hash = await hashPassword("MotDePasse123");

console.log(hash);
```

Vous devriez obtenir une chaîne ressemblant à :

```text
$argon2id$v=19$m=...
```

### Question

Comparez :

```text
MotDePasse123
```

et :

```text
$argon2id$v=19$...
```

Pourquoi ne peut-on pas retrouver simplement le mot de passe original à partir du hash ?

---

# 10. Activité 7 — Vérifier un mot de passe

Ajoutez :

```javascript
export async function verifyPassword(hash, password) {
    return argon2.verify(hash, password);
}
```

Test :

```javascript
const hash = await hashPassword("MotDePasse123");

const correct = await verifyPassword(
    hash,
    "MotDePasse123"
);

const incorrect = await verifyPassword(
    hash,
    "AutreMotDePasse"
);

console.log(correct);
console.log(incorrect);
```

Vous devez obtenir :

```text
true
false
```

---

# 11. Activité 8 — Créer le Repository User

Créez :

```text
src/repositories/user.repository.js
```

Ajoutez :

```javascript
import prisma from "../database/prisma.js";
```

Puis :

```javascript
export function findByEmail(email) {
    return prisma.user.findUnique({
        where: {
            email
        }
    });
}
```

Ajoutez :

```javascript
export function findById(id) {
    return prisma.user.findUnique({
        where: {
            id
        }
    });
}
```

Et :

```javascript
export function create(data) {
    return prisma.user.create({
        data
    });
}
```

---

# 12. Activité 9 — Service d'inscription

Créez :

```text
src/services/auth.service.js
```

Importez :

```javascript
import * as userRepository
    from "../repositories/user.repository.js";

import {
    hashPassword,
    verifyPassword
} from "../utils/password.js";
```

Commencez par :

```javascript
export async function register(data) {

    const existingUser =
        await userRepository.findByEmail(data.email);

    if (existingUser) {
        const error = new Error(
            "Cette adresse email est déjà utilisée"
        );

        error.statusCode = 409;

        throw error;
    }

    const hashedPassword =
        await hashPassword(data.password);

    const user =
        await userRepository.create({
            nom: data.nom,
            email: data.email,
            password: hashedPassword,
            role: data.role
        });

    return user;
}
```

---

# 13. Activité 10 — Ne jamais retourner le mot de passe

Le Controller ne doit pas renvoyer :

```json
{
    "id": 1,
    "nom": "Ali",
    "email": "ali@example.com",
    "password": "..."
}
```

Même si le mot de passe est haché.

Nous pouvons construire un objet public :

```javascript
const publicUser = {
    id: user.id,
    nom: user.nom,
    email: user.email,
    role: user.role
};
```

### Question

Pourquoi est-il préférable de ne pas exposer le hash du mot de passe dans l'API ?

---

# 14. Activité 11 — Validation avec Zod

Créez :

```text
src/validators/auth.validator.js
```

Puis :

```javascript
import { z } from "zod";

export const registerSchema = z.object({
    nom: z.string().min(2),
    email: z.string().email(),
    password: z.string().min(8),
    role: z.enum([
        "ADMIN",
        "AGENT_CENTRE",
        "RESPONSABLE_STOCK"
    ]).optional()
});
```

### Travail demandé

Ajoutez une validation plus stricte du mot de passe.

Par exemple, imposez progressivement :

* longueur minimale ;
* au moins une lettre ;
* au moins un chiffre.

---

# 15. Activité 12 — Controller d'inscription

Créez :

```text
src/controllers/auth.controller.js
```

Puis :

```javascript
import * as authService
    from "../services/auth.service.js";

export async function register(req, res, next) {

    try {

        const user =
            await authService.register(req.body);

        res.status(201).json({
            id: user.id,
            nom: user.nom,
            email: user.email,
            role: user.role
        });

    } catch (error) {

        next(error);
    }
}
```

---

# 16. Activité 13 — Route d'inscription

Créez :

```text
src/routes/auth.routes.js
```

Puis :

```javascript
import express from "express";

import {
    register
} from "../controllers/auth.controller.js";

import { validate } from "../middlewares/validate.js";

import {
    registerSchema
} from "../validators/auth.validator.js";

const router = express.Router();

router.post(
    "/register",
    validate(registerSchema, "body"),
    register
);

export default router;
```

Dans `server.js` :

```javascript
import authRoutes
    from "./routes/auth.routes.js";

app.use(
    "/api/auth",
    authRoutes
);
```

---

# 17. Activité 14 — Tester l'inscription

Testez :

```bash
curl -X POST http://localhost:3000/api/auth/register \
-H "Content-Type: application/json" \
-d '{
    "nom": "Ahmed",
    "email": "ahmed@sangconnect.tn",
    "password": "Password123"
}'
```

Vérifiez ensuite dans MySQL.

### Question importante

Que trouvez-vous dans la colonne `password` ?

Vous devez trouver un **hash Argon2**, et non :

```text
Password123
```

---

# 18. Activité 15 — Créer la connexion

Nous allons maintenant implémenter :

```http
POST /api/auth/login
```

Le client envoie :

```json
{
    "email": "ahmed@sangconnect.tn",
    "password": "Password123"
}
```

Le serveur doit :

```text
1. rechercher l'utilisateur
2. vérifier le mot de passe
3. générer un token
4. retourner le token
```

---

# 19. Activité 16 — Installer JWT

Installez :

```bash
npm install jsonwebtoken
```

Ajoutez une variable d'environnement dans `.env` :

```text
JWT_SECRET="une-cle-secrete-longue-et-aleatoire"
```

### Question

Pourquoi ne faut-il pas écrire directement la clé secrète dans le code source ?

---

# 20. Activité 17 — Comprendre un JWT

Un JWT contient trois parties :

```text
HEADER.PAYLOAD.SIGNATURE
```

Par exemple :

```text
xxxxx.yyyyy.zzzzz
```

Le Payload peut contenir :

```json
{
    "sub": 15,
    "email": "ahmed@sangconnect.tn",
    "role": "AGENT_CENTRE"
}
```

### Attention

Le payload d'un JWT n'est **pas chiffré**.

Il est encodé.

Il ne faut donc jamais y placer :

```text
mot de passe
numéro de carte bancaire
information secrète
```

---

# 21. Activité 18 — Générer le JWT

Créez :

```text
src/utils/jwt.js
```

Puis :

```javascript
import jwt from "jsonwebtoken";

export function generateToken(user) {

    return jwt.sign(
        {
            sub: user.id,
            email: user.email,
            role: user.role
        },
        process.env.JWT_SECRET,
        {
            expiresIn: "1h"
        }
    );
}
```

---

# 22. Activité 19 — Compléter le login

Dans `auth.service.js` :

```javascript
import {
    generateToken
} from "../utils/jwt.js";
```

Puis :

```javascript
export async function login(email, password) {

    const user =
        await userRepository.findByEmail(email);

    if (!user) {
        const error =
            new Error("Email ou mot de passe incorrect");

        error.statusCode = 401;

        throw error;
    }

    const valid =
        await verifyPassword(
            user.password,
            password
        );

    if (!valid) {
        const error =
            new Error("Email ou mot de passe incorrect");

        error.statusCode = 401;

        throw error;
    }

    const token =
        generateToken(user);

    return {
        token,
        user: {
            id: user.id,
            nom: user.nom,
            email: user.email,
            role: user.role
        }
    };
}
```

---

# 23. Activité 20 — Controller Login

Dans `auth.controller.js` :

```javascript
export async function login(req, res, next) {

    try {

        const result =
            await authService.login(
                req.body.email,
                req.body.password
            );

        res.json(result);

    } catch (error) {

        next(error);
    }
}
```

Ajoutez ensuite la route :

```javascript
router.post(
    "/login",
    validate(loginSchema, "body"),
    login
);
```

avec :

```javascript
export const loginSchema = z.object({
    email: z.string().email(),
    password: z.string().min(1)
});
```

---

# 24. Activité 21 — Tester le login

Testez :

```bash
curl -X POST http://localhost:3000/api/auth/login \
-H "Content-Type: application/json" \
-d '{
    "email": "ahmed@sangconnect.tn",
    "password": "Password123"
}'
```

Vous devriez obtenir une réponse proche de :

```json
{
    "token": "eyJ...",
    "user": {
        "id": 1,
        "nom": "Ahmed",
        "email": "ahmed@sangconnect.tn",
        "role": "AGENT_CENTRE"
    }
}
```

Copiez le token.

---

# 25. Activité 22 — Créer le middleware d'authentification

Créez :

```text
src/middlewares/auth.js
```

Le client enverra :

```http
Authorization: Bearer <TOKEN>
```

Commencez par :

```javascript
import jwt from "jsonwebtoken";

export function authenticate(req, res, next) {

    try {

        const header =
            req.headers.authorization;

        if (!header) {
            const error =
                new Error("Token manquant");

            error.statusCode = 401;

            throw error;
        }

        const [type, token] =
            header.split(" ");

        if (type !== "Bearer" || !token) {
            const error =
                new Error("Format du token invalide");

            error.statusCode = 401;

            throw error;
        }

        const payload =
            jwt.verify(
                token,
                process.env.JWT_SECRET
            );

        req.user = payload;

        next();

    } catch (error) {

        if (!error.statusCode) {
            error.statusCode = 401;
        }

        next(error);
    }
}
```

---

# 26. Activité 23 — Protéger une route

Prenons :

```http
GET /api/dons
```

Dans la route :

```javascript
router.get(
    "/",
    authenticate,
    getAll
);
```

Le flux devient :

```text
Client
   │
   │ Authorization: Bearer JWT
   ▼
authenticate
   │
   ├── Token invalide → 401
   │
   └── Token valide
            ↓
        Controller
```

---

# 27. Activité 24 — Tester sans token

Essayez :

```bash
curl http://localhost:3000/api/dons
```

Vous devez obtenir une erreur :

```http
401 Unauthorized
```

---

# 28. Activité 25 — Tester avec le token

Utilisez :

```bash
curl http://localhost:3000/api/dons \
-H "Authorization: Bearer VOTRE_TOKEN"
```

Cette fois, la requête doit pouvoir atteindre le Controller.

---

# 29. Activité 26 — Qui est l'utilisateur connecté ?

Dans le middleware, nous avons écrit :

```javascript
req.user = payload;
```

Le Controller peut donc accéder à :

```javascript
req.user
```

Par exemple :

```javascript
console.log(req.user);
```

Le résultat peut contenir :

```json
{
    "sub": 1,
    "email": "ahmed@sangconnect.tn",
    "role": "AGENT_CENTRE"
}
```

### Question

Pourquoi est-il intéressant de placer l'utilisateur authentifié dans :

```javascript
req.user
```

?

---

# 30. Activité 27 — Créer `/me`

Nous allons créer :

```http
GET /api/auth/me
```

Cette route devra retourner l'utilisateur actuellement authentifié.

Exemple :

```json
{
    "id": 1,
    "email": "ahmed@sangconnect.tn",
    "role": "AGENT_CENTRE"
}
```

La route devra utiliser :

```javascript
authenticate
```

avant le Controller.

---

# 31. Activité 28 — Authentification ≠ autorisation

Nous avons maintenant :

```text
Authentification
      ↓
Qui êtes-vous ?
```

Mais cela ne suffit pas.

Supposons :

```text
Ahmed → AGENT_CENTRE
```

Il est authentifié.

Mais doit-il pouvoir :

```http
DELETE /api/users/15
```

?

Pas nécessairement.

Nous devons ajouter une deuxième couche :

```text
Authentification
       ↓
Utilisateur identifié
       ↓
Autorisation
       ↓
Vérification du rôle
```

---

# 32. Activité 29 — Middleware de rôle

Créez :

```text
src/middlewares/authorize.js
```

Nous voulons pouvoir écrire :

```javascript
authorize("ADMIN")
```

ou :

```javascript
authorize(
    "ADMIN",
    "RESPONSABLE_STOCK"
)
```

Le middleware devra vérifier :

```javascript
req.user.role
```

et refuser l'accès si le rôle n'est pas autorisé.

Le code HTTP attendu sera :

```http
403 Forbidden
```

### Question

Pourquoi utiliser :

```text
401
```

pour l'authentification et :

```text
403
```

pour l'autorisation ?

---

# 33. Activité 30 — Protéger les routes selon les rôles

Imaginez les règles suivantes :

| Ressource                | ADMIN | AGENT_CENTRE | RESPONSABLE_STOCK |
| ------------------------ | ----: | -----------: | ----------------: |
| Consulter les dons       |     ✅ |            ✅ |                 ✅ |
| Créer un don             |     ✅ |            ✅ |                 ❌ |
| Modifier le stock        |     ✅ |            ❌ |                 ✅ |
| Gérer les utilisateurs   |     ✅ |            ❌ |                 ❌ |
| Supprimer un utilisateur |     ✅ |            ❌ |                 ❌ |

### Travail demandé

Transformez ce tableau en règles de middleware.

Par exemple :

```javascript
router.delete(
    "/:id",
    authenticate,
    authorize("ADMIN"),
    deleteUser
);
```

---

# 34. Activité 31 — Tester les scénarios

Testez au minimum :

### Cas 1

Pas de token :

```text
→ 401
```

### Cas 2

Token invalide :

```text
→ 401
```

### Cas 3

Token valide mais mauvais rôle :

```text
→ 403
```

### Cas 4

Token valide et rôle autorisé :

```text
→ accès accordé
```

---

# 35. Architecture de sécurité obtenue

Nous avons maintenant :

```text
                   CLIENT
                      │
                      │ JWT
                      ▼
                   ROUTE
                      │
                      ▼
             ┌─────────────────┐
             │ authenticate    │
             │                 │
             │ Vérifie JWT     │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ authorize       │
             │                 │
             │ Vérifie rôle    │
             └────────┬────────┘
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
                   PRISMA
                      │
                      ▼
                   MYSQL
```

---

# 36. Travail pratique à faire à la maison

## Mini-projet — Authentification SangConnect

Complétez le système d'authentification.

### Partie 1 — Utilisateurs

Implémentez :

```http
POST /api/auth/register
```

et :

```http
POST /api/auth/login
```

---

### Partie 2 — JWT

Le JWT devra contenir au minimum :

```text
sub
email
role
```

avec une durée d'expiration définie.

---

### Partie 3 — Middleware

Implémentez :

```text
authenticate
```

permettant de protéger les routes.

---

### Partie 4 — `/me`

Implémentez :

```http
GET /api/auth/me
```

Cette route doit être accessible uniquement aux utilisateurs authentifiés.

---

### Partie 5 — Autorisation

Implémentez :

```text
authorize(...)
```

et protégez les routes selon les rôles.

---

### Partie 6 — Tests

Testez au minimum :

```text
Inscription réussie
Email déjà utilisé
Mot de passe incorrect
Utilisateur inexistant
Login réussi
Token absent
Token invalide
Token expiré
Rôle insuffisant
Accès autorisé
```

---

# 37. Bilan

Nous avons ajouté à SangConnect une première véritable couche de sécurité :

```text
Ateliers 5–7
Persistance
Prisma + MySQL
        ↓
Atelier 8
Authentification
        ↓
JWT
        ↓
Middleware
        ↓
Autorisation
        ↓
RBAC
```

### Point important pour la suite

L'authentification de cet atelier constitue une **première version pédagogique**. Dans l'atelier suivant, il sera intéressant d'aller plus loin avec :

* **refresh tokens** ;
* expiration et renouvellement des tokens ;
* révocation de sessions ;
* contrôle d'accès plus fin ;
* sécurisation des routes ;
* **CORS** ;
* **Helmet** ;
* **rate limiting** ;
* prévention des injections ;
* bonnes pratiques OWASP ;
* et tests des mécanismes de sécurité.


