# mern-auth-starter

Starter MERN d'authentification : inscription, connexion, profil et suppression de compte.
Backend Node + Express + MongoDB, frontend React + Redux. Pensé comme **base pédagogique**.

## Objectif
Fournir une base MERN minimale et lisible pour un flux d'auth JWT :
- Inscription et connexion avec password hashé (bcrypt)
- Session persistée côté client (JWT dans localStorage)
- Route protégée (profil utilisateur) côté serveur
- Dashboard et navigation conditionnelle côté React

## Données
MongoDB — une seule collection `users` :

| Champ | Type | Notes |
|---|---|---|
| `_id` | ObjectId | généré par MongoDB |
| `name` | String | fourni à l'inscription |
| `email` | String | fourni à l'inscription (unicité vérifiée côté controller) |
| `password` | String | **hash bcrypt** (salt rounds = 10) |
| `createdAt`, `updatedAt` | Date | timestamps Mongoose |

> Le schéma Mongoose utilise `require: true` au lieu de `required: true` — les contraintes
> schéma sont donc inactives, la validation réelle est faite en runtime dans le controller
> (type String, trim, password ≥ 8 chars).

## Méthode

### API (backend)
Préfixe `/api/users` :

| Méthode | Chemin | Rôle | Auth |
|---|---|---|---|
| POST | `/` | Inscription | public |
| POST | `/login` | Connexion | public |
| GET | `/me` | Récupère le profil courant (hors password) | **privé** |
| DELETE | `/:id` | Supprime un compte (uniquement son propre id) | **privé** |

- JWT signé avec `JWT_SECRET`, `expiresIn: "30d"`.
- Transporté via header **`x-auth-token`** (header custom).
- Password hashé via `bcrypt.genSalt(10)` + `bcrypt.hash`.
- Pas de refresh token, pas de reset password, pas de vérification d'email.

### Front (React)
- Routes : `/` (Dashboard, requiert login), `/login`, `/register`.
- `localStorage.user` stocke `{_id, name, email, token}` pour rehydrater Redux au boot.
- Redux (createStore classic + redux-thunk) : `authReducer` + `countReducer` (démo).
- Chaque action async a un délai artificiel `setTimeout(2000)` pour afficher le spinner — à
  retirer en prod.
- Logout : nettoyage `localStorage.user` + `AUTH_LOGOUT`.

## Stack
**Backend** : Node, Express 4, Mongoose 8, bcryptjs 2.4, jsonwebtoken 9, dotenv,
express-async-handler, express-validator (dép. listée, non utilisée).
Dev : nodemon, concurrently.

**Frontend** : React 18.3, React-Router 6.24, Redux 5 + redux-thunk 3, React-Redux 9,
React-Bootstrap 2.10, Bootstrap 5.3, axios 1.7, react-icons.

## Résultats
Starter fonctionnel pour les flux register / login / me / delete. **Pas destiné à la prod en
l'état** — voir "Limites connues" ci-dessous.

### Limites connues
- Bug schéma : `require` au lieu de `required` → contraintes DB inactives.
- Pas d'index `unique` sur `email` : race condition possible sur deux inscriptions simultanées.
- Header JWT non standard (`x-auth-token` au lieu de `Authorization: Bearer`).
- Pas de CORS configuré (OK en dev via proxy React → `http://localhost:5000`, à ajouter pour un déploiement).
- Pas de rate-limiting sur login/register.
- Pas de reset password, pas de vérification d'email, pas de rôles.
- `setTimeout(2000)` artificiel dans les actions Redux (démo UX).
- Expiration JWT non détectée côté client avant la prochaine requête.
- Composant `Counter` de démonstration (reducer async non lié à l'auth) laissé dans `App.js`.
- Test `App.test.js` = défaut Create React App, pas de vrais tests.

## Comment lancer

### Prérequis
- Node ≥ 18, npm
- MongoDB local (ou URL cluster MongoDB Atlas)

### Variables d'environnement (`.env` à créer à la racine)
```
MONGO_URL=mongodb://localhost:27017/mern-auth
JWT_SECRET=change-me
PORT=5000
```

### Installation
```bash
npm install                      # dépendances backend
cd frontend && npm install       # dépendances frontend
cd ..
```

### Lancement
Mode dev avec concurrently (backend + frontend) :
```bash
npm run dev
```

Ou séparément :
```bash
npm run server                   # backend sur http://localhost:5000
npm run client                   # frontend sur http://localhost:3000 (proxy vers :5000)
```

### Endpoints (exemples curl)
```bash
# Register
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@test.com","password":"motdepasse"}'

# Login
curl -X POST http://localhost:5000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@test.com","password":"motdepasse"}'

# Me (coller le token récupéré ci-dessus)
curl http://localhost:5000/api/users/me \
  -H "x-auth-token: <TOKEN>"
```
