# Depitcho IA

Chatbot RAG de l'agence web **Depitcho**. Le bot répond aux visiteurs en s'appuyant uniquement sur une base de connaissances interne (services, tarifs, horaires), connaît la date/heure en Europe/Paris et affiche un statut ouvert / fermé selon les horaires de l'agence.

## Fonctionnement

- L'utilisateur envoie un message depuis l'interface de chat (`public/index.html`).
- Le serveur crée une session de chat Gemini (modèle `gemini-3.5-flash-lite`) avec une **instruction système** contenant :
  - la date et l'heure actuelles (fuseau Europe/Paris) ;
  - le statut de l'agence : **OUVERT** du lundi au vendredi de 08h00 à 18h00, **FERMÉ** sinon ;
  - la base de connaissances chargée depuis `data/test.txt` (RAG) ;
  - des consignes strictes (saluer une seule fois, répondre uniquement depuis la base, rester précis et concis).
- Si l'agence est **FERMÉE**, le bot répond uniquement : « L'agence est actuellement fermée. Veuillez laisser votre message, nous vous répondrons le plus tôt possible. »
- L'historique de conversation est conservé **en mémoire** par session (`sessionId` généré côté navigateur).

## Structure du projet

```
exo-IA/
│
├── src/
│   └── index.js          Serveur Express : API chat, sessions, instruction système, RAG
│
├── public/
│   └── index.html        Interface du chatbot (HTML / CSS / JS, appels fetch)
│
├── data/
│   └── test.txt          Base de connaissances Depitcho (services, tarifs, horaires)
│
├── .env                  Clé API Gemini (non versionnée, à créer)
├── .gitignore            Ignore node_modules/ et .env
├── package.json          Dépendances et script de démarrage
└── package-lock.json     Versions exactes des dépendances
```

- **`src/index.js`** — cœur du projet : charge la clé API, lit `data/test.txt`, construit l'instruction système et expose les routes.
- **`public/index.html`** — tout le front (pas de build) : génère un `sessionId` et appelle `POST /api/chat`.
- **`data/test.txt`** — c'est ce fichier qui alimente le RAG : modifie-le pour changer les réponses du bot.
- **`.env`** — secret : jamais poussé sur GitHub (voir `.gitignore`).

## Installation

### Prérequis

- [Node.js](https://nodejs.org) version 18 ou supérieure (LTS recommandée)
- Une clé API [Google AI Studio](https://aistudio.google.com/apikey)

### Étapes

1. Créer un fichier `.env` à la racine :

```env
GEMINI_API_KEY=ta_cle_api_ici
```

2. Installer et lancer :

```bash
npm install
npm run dev
```

3. Ouvrir http://localhost:3000

> `npm run dev` utilise `node --watch` : le serveur redémarre automatiquement à chaque modification de `src/index.js`.

## API du serveur

| Méthode | Route | Rôle |
|---|---|---|
| GET | `/health` | Vérifie que le serveur répond |
| POST | `/api/chat` | Envoie un message, renvoie la réponse du bot |

Corps attendu pour `POST /api/chat` :

```json
{
  "message": "Quels sont vos tarifs ?",
  "sessionId": "session_abc123"
}
```

Réponse :

```json
{
  "sessionId": "session_abc123",
  "reply": "Notre offre de site vitrine démarre à 500 € HT..."
}
```

## Personnalisation

Tout se règle dans `src/index.js` :

- **Réponses du bot** : modifier `data/test.txt` (services, tarifs, horaires).
- **Instruction système** : le bloc `systemInstruction` de `ai.chats.create` (ton, règles, statut ouvert/fermé).
- **Horaires d'ouverture** : fonction `getCurrentTimeInfo` (actuellement lun–ven, 08h00–18h00, fuseau Europe/Paris).
- **Modèle IA** : propriété `model` (actuellement `gemini-3.5-flash-lite`).

## Limites à connaître

- Les sessions et l'historique sont en mémoire : tout est perdu au redémarrage du serveur.
- La base de connaissances est relue à la création de chaque session (ajoute ou retire des sessions pour la recharger).
- Ne jamais committer le fichier `.env` : la clé API doit rester secrète.

## Technologies

Node.js, Express 5, `@google/genai` (Gemini), dotenv, HTML / CSS / JavaScript.
