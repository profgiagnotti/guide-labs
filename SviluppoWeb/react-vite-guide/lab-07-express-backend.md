# Lab 07 — Backend con Node.js ed Express

> **Serie:** React + Vite + Node.js — Guida completa  
> **Repository:** `profgiagnotti/react-vite-guide`  
> **File:** `lab-07-express-backend.md`

---

## ✅ Obiettivi

- [ ] Inizializzare il package del server con `npm init`
- [ ] Installare Express e le dipendenze necessarie
- [ ] Creare la struttura `server/` con `index.js`, `routes/` e `data/`
- [ ] Implementare le 5 route CRUD per `/api/notes`
- [ ] Configurare CORS, body parser JSON e un logger minimale
- [ ] Testare tutte le route con curl o REST Client
- [ ] Aggiungere un middleware di validazione e il gestore errori globale
- [ ] Configurare nodemon per il reload automatico in sviluppo

---

## 📋 Prerequisiti

- Lab 06 completato: `useFetch`, `notesApi.js`, `LoadingSpinner`, `ErrorMessage` presenti in `src/`
- Node.js ≥ 18 installato (`node --version` per verificare)
- Il frontend DevNotes funzionante su `http://localhost:5173`

---

## 📑 Indice

1. [Struttura cartelle server](#1-struttura-cartelle-server)
2. [Inizializzazione package](#2-inizializzazione-package)
3. [Dati in-memory](#3-dati-in-memory)
4. [Router delle note](#4-router-delle-note)
5. [Entry point index.js](#5-entry-point-indexjs)
6. [Avviare il server](#6-avviare-il-server)
7. [Testare le route con curl](#7-testare-le-route-con-curl)
8. [Testare con REST Client per VS Code](#8-testare-con-rest-client-per-vs-code)
9. [Middleware di validazione](#9-middleware-di-validazione)
10. [Esercizi di consolidamento](#10-esercizi-di-consolidamento)
11. [Checklist finale](#11-checklist-finale)

---

## 1. Struttura cartelle server

Dalla **root del progetto** (la stessa cartella di `vite.config.js`), crea la struttura:

**Windows (PowerShell):**
```powershell
mkdir server
mkdir server\routes
mkdir server\data
```

**macOS / Linux:**
```bash
mkdir -p server/routes server/data
```

La struttura completa del progetto sarà:

```
devnotes/
├── src/                    ← frontend React (già esistente)
│   ├── hooks/
│   ├── services/
│   ├── components/
│   └── pages/
├── server/                 ← NUOVO
│   ├── index.js
│   ├── routes/
│   │   └── notes.js
│   └── data/
│       └── notes.js
├── .env
├── package.json            ← package del frontend
└── vite.config.js
```

---

## 2. Inizializzazione package

Entra nella cartella `server/` e inizializza il suo package:

**Windows / macOS / Linux:**
```bash
cd server
npm init -y
```

Questo crea `server/package.json`. Installa Express e il middleware CORS:

```bash
npm install express cors
```

Installa nodemon come dipendenza di sviluppo (ricarica il server automaticamente ad ogni modifica):

```bash
npm install --save-dev nodemon
```

Modifica `server/package.json` aggiungendo gli script:

```json
{
  "name": "devnotes-server",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.18.3"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

---

## 3. Dati in-memory

Crea `server/data/notes.js`:

```javascript
// server/data/notes.js
// Array che simula il database. I dati si perdono ad ogni riavvio del server —
// in L09 li sostituiremo con una persistenza reale (file JSON o SQLite).

let notes = [
  {
    id: 1,
    title: 'Setup del progetto',
    body: 'Installato Node.js 20 LTS, Vite 5 e VS Code con le estensioni giuste: ESLint, Prettier, ES7+ React Snippets.',
    tag: 'lavoro',
    createdAt: '2026-01-10T09:00:00.000Z',
  },
  {
    id: 2,
    title: 'Componenti JSX e props',
    body: "Header, NoteCard e NoteList. Le props fluiscono dall'alto verso il basso. Il componente non può modificare le props ricevute.",
    tag: 'studio',
    createdAt: '2026-01-12T10:30:00.000Z',
  },
  {
    id: 3,
    title: 'useState e useEffect',
    body: 'Lo stato è privato del componente. useEffect con array vuoto equivale a componentDidMount. Ricorda il cleanup.',
    tag: 'studio',
    createdAt: '2026-01-15T14:00:00.000Z',
  },
];

// nextId simula l'auto-increment di un database
let nextId = 4;

module.exports = {
  /** Restituisce il riferimento all'array (mutabile) */
  get notes() {
    return notes;
  },
  /** Restituisce l'ID successivo e incrementa il contatore */
  getNextId() {
    return nextId++;
  },
};
```

> **Nota:** usando `get notes()` come getter, le route ottengono sempre il riferimento all'array corrente, anche dopo aggiunte o rimozioni.

---

## 4. Router delle note

Crea `server/routes/notes.js`:

```javascript
// server/routes/notes.js
const express = require('express');
const db = require('../data/notes');

const router = express.Router();

// ─── GET /api/notes ────────────────────────────────────────────────────────
// Query param opzionale: ?tag=lavoro  → filtra per tag
// Query param opzionale: ?q=testo    → ricerca nel titolo (case-insensitive)
router.get('/', (req, res) => {
  let result = db.notes;

  if (req.query.tag) {
    result = result.filter((n) => n.tag === req.query.tag);
  }

  if (req.query.q) {
    const q = req.query.q.toLowerCase();
    result = result.filter((n) => n.title.toLowerCase().includes(q));
  }

  res.json(result);
});

// ─── GET /api/notes/:id ────────────────────────────────────────────────────
router.get('/:id', (req, res) => {
  const note = db.notes.find((n) => n.id === Number(req.params.id));
  if (!note) {
    return res.status(404).json({ error: 'Nota non trovata' });
  }
  res.json(note);
});

// ─── POST /api/notes ───────────────────────────────────────────────────────
// Body JSON atteso: { title: string, body: string, tag?: string }
router.post('/', (req, res) => {
  const { title, body, tag = 'personale' } = req.body;

  if (!title?.trim() || !body?.trim()) {
    return res.status(400).json({ error: 'title e body sono obbligatori e non possono essere vuoti' });
  }

  const newNote = {
    id: db.getNextId(),
    title: title.trim(),
    body: body.trim(),
    tag,
    createdAt: new Date().toISOString(),
  };

  db.notes.push(newNote);
  // 201 Created: la risorsa è stata creata. Includiamo la nota nel body.
  res.status(201).json(newNote);
});

// ─── PUT /api/notes/:id ────────────────────────────────────────────────────
// Sostituzione completa. Body JSON: { title, body, tag }
router.put('/:id', (req, res) => {
  const idx = db.notes.findIndex((n) => n.id === Number(req.params.id));
  if (idx === -1) {
    return res.status(404).json({ error: 'Nota non trovata' });
  }

  const { title, body, tag } = req.body;
  if (!title?.trim() || !body?.trim()) {
    return res.status(400).json({ error: 'title e body sono obbligatori' });
  }

  db.notes[idx] = {
    ...db.notes[idx],
    title: title.trim(),
    body: body.trim(),
    tag: tag ?? db.notes[idx].tag,
    updatedAt: new Date().toISOString(),
  };

  res.json(db.notes[idx]);
});

// ─── PATCH /api/notes/:id ──────────────────────────────────────────────────
// Aggiornamento parziale: invia solo i campi da modificare
router.patch('/:id', (req, res) => {
  const idx = db.notes.findIndex((n) => n.id === Number(req.params.id));
  if (idx === -1) {
    return res.status(404).json({ error: 'Nota non trovata' });
  }

  const allowed = ['title', 'body', 'tag'];
  const updates = {};
  allowed.forEach((field) => {
    if (req.body[field] !== undefined) updates[field] = req.body[field];
  });

  db.notes[idx] = { ...db.notes[idx], ...updates, updatedAt: new Date().toISOString() };
  res.json(db.notes[idx]);
});

// ─── DELETE /api/notes/:id ─────────────────────────────────────────────────
router.delete('/:id', (req, res) => {
  const idx = db.notes.findIndex((n) => n.id === Number(req.params.id));
  if (idx === -1) {
    return res.status(404).json({ error: 'Nota non trovata' });
  }

  db.notes.splice(idx, 1);
  // 204 No Content: successo senza body da restituire
  res.status(204).send();
});

module.exports = router;
```

---

## 5. Entry point index.js

Crea `server/index.js`:

```javascript
// server/index.js
const express = require('express');
const cors = require('cors');
const notesRouter = require('./routes/notes');

const app = express();
const PORT = process.env.PORT || 3001;

// ─── MIDDLEWARE GLOBALI ────────────────────────────────────────────────────

// 1. CORS — consente al frontend Vite (porta 5173) di chiamare il backend
app.use(
  cors({
    origin: process.env.FRONTEND_ORIGIN || 'http://localhost:5173',
    methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization'],
  })
);

// 2. Body parser JSON — popola req.body per POST/PUT/PATCH
app.use(express.json());

// 3. Logger minimale — stampa timestamp, metodo e URL ad ogni richiesta
app.use((req, res, next) => {
  const ts = new Date().toISOString().replace('T', ' ').slice(0, 19);
  console.log(`[${ts}] ${req.method.padEnd(7)} ${req.url}`);
  next();
});

// ─── ROUTE ────────────────────────────────────────────────────────────────

// Health check — utile per monitoraggio e per verificare che il server risponda
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    uptime: Math.floor(process.uptime()),
    timestamp: new Date().toISOString(),
  });
});

// Router delle note: tutte le route sono montate sotto /api/notes
app.use('/api/notes', notesRouter);

// ─── CATCH-ALL 404 ────────────────────────────────────────────────────────
// Va registrato DOPO tutte le route, ma PRIMA del gestore errori
app.use((req, res) => {
  res.status(404).json({ error: `Route non trovata: ${req.method} ${req.url}` });
});

// ─── GESTORE ERRORI GLOBALE ───────────────────────────────────────────────
// Firma con 4 parametri: Express lo riconosce come error handler
// Viene chiamato con next(err) da qualsiasi middleware o route
app.use((err, req, res, next) => { // eslint-disable-line no-unused-vars
  console.error('[ERROR]', err.stack);
  res.status(err.status || 500).json({
    error: err.message || 'Errore interno del server',
  });
});

// ─── AVVIO ────────────────────────────────────────────────────────────────
app.listen(PORT, () => {
  console.log(`\n🚀 Server DevNotes avviato`);
  console.log(`   http://localhost:${PORT}`);
  console.log(`   Health: http://localhost:${PORT}/health`);
  console.log(`   Notes:  http://localhost:${PORT}/api/notes\n`);
});
```

---

## 6. Avviare il server

Apri un **secondo terminale** (il primo serve per il frontend con `npm run dev`).

Entra nella cartella server e avvialo in modalità sviluppo:

```bash
cd server
npm run dev
```

Dovresti vedere:

```
🚀 Server DevNotes avviato
   http://localhost:3001
   Health: http://localhost:3001/health
   Notes:  http://localhost:3001/api/notes
```

> **Due terminali aperti in contemporanea:**
> - Terminale 1: `npm run dev` nella root → Vite su :5173
> - Terminale 2: `npm run dev` in `server/` → Express su :3001

---

## 7. Testare le route con curl

Verifica che ogni route risponda correttamente prima di collegarla al frontend.

**Windows (PowerShell) — usa `Invoke-WebRequest` o installa curl nativo:**
```powershell
# Health check
curl.exe http://localhost:3001/health

# Lista note
curl.exe http://localhost:3001/api/notes

# Filtra per tag
curl.exe "http://localhost:3001/api/notes?tag=studio"

# Nota singola
curl.exe http://localhost:3001/api/notes/1

# Crea nota
curl.exe -X POST http://localhost:3001/api/notes `
  -H "Content-Type: application/json" `
  -d '{"title":"Nuova nota","body":"Corpo della nota","tag":"lavoro"}'

# Aggiorna nota
curl.exe -X PUT http://localhost:3001/api/notes/1 `
  -H "Content-Type: application/json" `
  -d '{"title":"Titolo aggiornato","body":"Corpo aggiornato","tag":"studio"}'

# Elimina nota
curl.exe -X DELETE http://localhost:3001/api/notes/1
```

**macOS / Linux:**
```bash
# Health check
curl http://localhost:3001/health

# Lista note
curl http://localhost:3001/api/notes | json_pp

# Filtra per tag
curl "http://localhost:3001/api/notes?tag=studio"

# Nota singola
curl http://localhost:3001/api/notes/2

# Crea nota (201 → nota con id generato)
curl -X POST http://localhost:3001/api/notes \
  -H "Content-Type: application/json" \
  -d '{"title":"Nuova nota","body":"Corpo della nota","tag":"lavoro"}'

# Aggiornamento parziale con PATCH
curl -X PATCH http://localhost:3001/api/notes/2 \
  -H "Content-Type: application/json" \
  -d '{"tag":"personale"}'

# Elimina nota (204 → nessun body)
curl -X DELETE http://localhost:3001/api/notes/3
```

---

## 8. Testare con REST Client per VS Code

Installa l'estensione **REST Client** (Huachao Mao) da VS Code Marketplace.

Crea il file `server/test.http` nella root di `server/`:

```http
### Health check
GET http://localhost:3001/health

### Lista tutte le note
GET http://localhost:3001/api/notes

### Filtra per tag
GET http://localhost:3001/api/notes?tag=studio

### Cerca per testo
GET http://localhost:3001/api/notes?q=componenti

### Nota con id 1
GET http://localhost:3001/api/notes/1

### Nota inesistente (404)
GET http://localhost:3001/api/notes/999

### Crea una nota
POST http://localhost:3001/api/notes
Content-Type: application/json

{
  "title": "React Router v6",
  "body": "BrowserRouter > Routes > Route. useParams per i parametri URL. useNavigate per la navigazione programmatica.",
  "tag": "studio"
}

### Aggiorna completamente la nota 1
PUT http://localhost:3001/api/notes/1
Content-Type: application/json

{
  "title": "Setup progetto — aggiornato",
  "body": "Incluse anche le istruzioni per nvm e la configurazione di Prettier.",
  "tag": "lavoro"
}

### Aggiornamento parziale (solo tag)
PATCH http://localhost:3001/api/notes/2
Content-Type: application/json

{
  "tag": "personale"
}

### Elimina nota con id 3
DELETE http://localhost:3001/api/notes/3

### Validazione: body senza title (400)
POST http://localhost:3001/api/notes
Content-Type: application/json

{
  "body": "Manca il title"
}
```

Clicca su **Send Request** sopra ogni blocco per eseguire la richiesta e vedere la risposta nel pannello laterale.

---

## 9. Middleware di validazione

Anziché ripetere i controlli di validazione in ogni route, crea un middleware riutilizzabile. Aggiungi questo file `server/middleware/validateNote.js`:

```javascript
// server/middleware/validateNote.js

/**
 * Middleware di validazione per le route POST e PUT delle note.
 * Verifica che title e body siano presenti e non vuoti.
 * Chiama next() se la validazione passa, altrimenti risponde con 400.
 */
function validateNote(req, res, next) {
  const { title, body } = req.body;

  const errors = [];
  if (!title?.trim()) errors.push('title è obbligatorio');
  if (!body?.trim()) errors.push('body è obbligatorio');
  if (title && title.trim().length > 200) errors.push('title non può superare 200 caratteri');

  if (errors.length > 0) {
    return res.status(400).json({ error: errors.join('; ') });
  }

  next();
}

module.exports = validateNote;
```

Usa il middleware nelle route POST e PUT in `server/routes/notes.js`:

```javascript
const validateNote = require('../middleware/validateNote');

// Applica il middleware solo alle route che ne hanno bisogno
router.post('/', validateNote, (req, res) => { /* ... */ });
router.put('/:id', validateNote, (req, res) => { /* ... */ });
```

Ora puoi rimuovere i controlli di validazione duplicati dai route handler.

---

## 10. Esercizi di consolidamento

### Esercizio A — Route `GET /api/notes/stats`

Aggiungi una route che restituisce statistiche aggregate sulle note:

```json
{
  "total": 3,
  "byTag": {
    "lavoro": 1,
    "studio": 2
  },
  "lastCreated": "2026-01-15T14:00:00.000Z"
}
```

> **Attenzione:** registra questa route PRIMA di `/:id`, altrimenti Express interpreterà `stats` come un ID numerico.

---

### Esercizio B — Paginazione

Aggiungi il supporto per la paginazione nella route `GET /api/notes`:

- Query params: `?page=1&limit=5`
- La risposta deve includere i metadati di paginazione:

```json
{
  "data": [ /* array note */ ],
  "pagination": {
    "total": 10,
    "page": 1,
    "limit": 5,
    "totalPages": 2
  }
}
```

---

### Esercizio C — Persistenza su file JSON

Usa il modulo `fs` di Node.js per leggere e scrivere le note su un file `server/data/notes.json` anziché usare un array in-memory. In questo modo le note sopravvivono ai riavvii del server.

```javascript
const fs = require('fs');
const path = require('path');

const DB_PATH = path.join(__dirname, 'notes.json');

function readNotes() {
  if (!fs.existsSync(DB_PATH)) return [];
  return JSON.parse(fs.readFileSync(DB_PATH, 'utf8'));
}

function writeNotes(notes) {
  fs.writeFileSync(DB_PATH, JSON.stringify(notes, null, 2), 'utf8');
}
```

---

## 11. Checklist finale

| Controllo | OK? |
|-----------|-----|
| `server/package.json` con script `start` e `dev` | ☐ |
| `express` e `cors` installati in `server/` | ☐ |
| `nodemon` installato come devDependency | ☐ |
| `server/data/notes.js` con array e getter | ☐ |
| `server/routes/notes.js` con le 5 route CRUD | ☐ |
| `server/index.js` con cors, json, logger, route, error handler | ☐ |
| Il server parte senza errori con `npm run dev` | ☐ |
| `GET /health` risponde `{ status: 'ok' }` | ☐ |
| `GET /api/notes` restituisce l'array delle note | ☐ |
| `POST /api/notes` crea una nota e risponde 201 | ☐ |
| `PUT /api/notes/1` aggiorna la nota e risponde 200 | ☐ |
| `DELETE /api/notes/1` risponde 204 senza body | ☐ |
| Route inesistente risponde 404 con JSON | ☐ |
| POST senza `title` risponde 400 con messaggio errore | ☐ |
| File `server/test.http` con tutte le richieste testate | ☐ |

---

## 📊 Riepilogo file prodotti

| File | Tipo | Descrizione |
|------|------|-------------|
| `server/package.json` | Config | Dipendenze e script del server |
| `server/index.js` | Entry point | App Express con middleware e avvio |
| `server/routes/notes.js` | Router | 5 route CRUD per /api/notes |
| `server/data/notes.js` | Dati | Array in-memory con getter |
| `server/middleware/validateNote.js` | Middleware | Validazione body delle note |
| `server/test.http` | Test | Richieste REST Client per VS Code |

---

## ➡️ Prossimo lab

**[Lab 08 — Connettere frontend e backend](https://github.com/profgiagnotti/react-vite-guide/blob/main/lab-08-fullstack.md)**  
Configureremo il proxy Vite (`server.proxy`) per far parlare React con Express senza errori CORS, aggiorneremo `VITE_API_URL` e testeremo il CRUD completo end-to-end.
