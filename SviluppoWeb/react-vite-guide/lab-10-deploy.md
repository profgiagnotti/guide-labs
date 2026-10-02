# Lab 10 — Deploy: Vercel + Render

> **Serie:** React + Vite + Node.js — DevNotes  
> **Lezione di riferimento:** [L10 — Deploy: Vercel + Render](blog-react-vite-nodejs-l10-deploy.html)  
> **Repository:** `profgiagnotti/react-vite-guide`  
> **Branch di partenza:** `l09-auth-jwt` → lavora su `l10-deploy`

---

## ✅ Obiettivi del laboratorio

- [ ] Verificare che la build Vite produca `dist/` senza errori
- [ ] Creare i file `.env.development` e `.env.production` nel frontend
- [ ] Aggiornare `notesApi.js` per usare `import.meta.env.VITE_API_URL`
- [ ] Configurare CORS dinamico nel backend con `ALLOWED_ORIGIN`
- [ ] Aggiungere `vercel.json` con il rewrite SPA
- [ ] Specificare `"engines"` e lo script `"start"` in `backend/package.json`
- [ ] Fare il deploy del backend su Render
- [ ] Fare il deploy del frontend su Vercel con le variabili d'ambiente corrette
- [ ] Verificare che l'app funzioni online: login, creazione nota, eliminazione
- [ ] Testare la CI/CD: push su main → deploy automatico

---

## 📋 Prerequisiti

| Requisito | Note |
|---|---|
| Account GitHub | Necessario per collegare Vercel e Render |
| Account Vercel | Registrazione gratuita su vercel.com |
| Account Render | Registrazione gratuita su render.com |
| Progetto DevNotes con auth (L09) | Sul branch `l09-auth-jwt` |
| Vercel CLI (opzionale) | `npm install -g vercel` |

```bash
# Windows / macOS / Linux
git checkout l09-auth-jwt
git checkout -b l10-deploy
```

---

## 📑 Indice

1. [Build di produzione e preview locale](#1-build-di-produzione-e-preview-locale)
2. [Variabili d'ambiente frontend](#2-variabili-dambiente-frontend)
3. [Variabili d'ambiente backend](#3-variabili-dambiente-backend)
4. [CORS dinamico nel backend](#4-cors-dinamico-nel-backend)
5. [Preparare il backend per Render](#5-preparare-il-backend-per-render)
6. [vercel.json e preparare il frontend](#6-verceljson-e-preparare-il-frontend)
7. [Deploy del backend su Render](#7-deploy-del-backend-su-render)
8. [Deploy del frontend su Vercel](#8-deploy-del-frontend-su-vercel)
9. [Collegare frontend e backend in produzione](#9-collegare-frontend-e-backend-in-produzione)
10. [Test in produzione](#10-test-in-produzione)
11. [Esercizi di consolidamento](#11-esercizi-di-consolidamento)
12. [Checklist finale](#12-checklist-finale)

---

## 1. Build di produzione e preview locale

### 1.1 Esegui la build

```bash
# Windows / macOS / Linux
cd devnotes/frontend
npm run build
```

Output atteso (con eventuali numeri diversi):

```
✓ 34 modules transformed.
dist/index.html                   0.5 kB
dist/assets/index-BxKv4M5A.js   142.3 kB │ gzip: 45.8 kB
dist/assets/index-DcFzk9jO.css    3.1 kB │ gzip:  1.2 kB
✓ built in 1.2s
```

Se la build fallisce con errori TypeScript o import mancanti, correggi prima di procedere.

### 1.2 Preview della build di produzione

```bash
# Windows / macOS / Linux
npm run preview
# → Local: http://localhost:4173/
```

Apri `http://localhost:4173` e verifica che l'app si carichi. **Attenzione:** in questa fase le chiamate API falliranno perché il backend di produzione non è ancora configurato. È normale.

> 💡 **Differenza dev vs preview:** `npm run dev` usa HMR e il proxy Vite. `npm run preview` serve i file statici da `dist/` come farebbe un server reale — nessun proxy, nessun HMR.

---

## 2. Variabili d'ambiente frontend

### 2.1 Crea i file di environment

```bash
# Windows (PowerShell)
New-Item devnotes\frontend\.env.development -ItemType File
New-Item devnotes\frontend\.env.production  -ItemType File
New-Item devnotes\frontend\.gitignore       -ItemType File

# macOS / Linux
touch devnotes/frontend/.env.development
touch devnotes/frontend/.env.production
touch devnotes/frontend/.gitignore
```

### 2.2 Scrivi `.env.development`

```
# frontend/.env.development
# Con il proxy Vite, le chiamate /api/* vengono inoltrate a localhost:3001
# quindi VITE_API_URL può essere vuoto o puntare al dev server Vite stesso
VITE_API_URL=
```

### 2.3 Scrivi `.env.production`

```
# frontend/.env.production
# Sostituisci con l'URL del tuo backend su Render (lo ottieni al passo 7)
VITE_API_URL=https://devnotes-api.onrender.com
```

### 2.4 Scrivi `frontend/.gitignore`

```
# frontend/.gitignore
node_modules/
dist/
.env.local
```

> I file `.env.development` e `.env.production` **possono** essere committati se non contengono segreti (e qui non li contengono — l'URL pubblico del backend non è un segreto). I segreti stanno sempre solo nel `.env` del backend.

### 2.5 Aggiorna `notesApi.js` per usare la variabile

Modifica la riga `API_BASE` in `frontend/src/api/notesApi.js`:

```js
// frontend/src/api/notesApi.js
// In dev: VITE_API_URL è vuoto → API_BASE = '/api/notes' → proxy Vite gestisce
// In prod: VITE_API_URL = 'https://...' → API_BASE = 'https://.../api/notes'
const BASE_URL = import.meta.env.VITE_API_URL || ''
const API_BASE = `${BASE_URL}/api/notes`
```

Aggiorna anche `authRequest`:

```js
export async function authRequest(action, body) {
  const res = await fetch(`${BASE_URL}/api/auth/${action}`, {
    method:  'POST',
    headers: { 'Content-Type': 'application/json' },
    body:    JSON.stringify(body),
  })
  const data = await res.json().catch(() => ({}))
  if (!res.ok) throw new Error(data.messaggio || `Errore auth: ${res.status}`)
  return data
}
```

---

## 3. Variabili d'ambiente backend

### 3.1 Verifica il `.gitignore` del backend

Assicurati che `backend/.gitignore` contenga almeno:

```
node_modules/
.env
.env.local
```

### 3.2 Crea un file `.env.example`

Questo file **viene committato** e serve come documentazione delle variabili necessarie:

```bash
# Windows (PowerShell)
New-Item devnotes\backend\.env.example -ItemType File

# macOS / Linux
touch devnotes/backend/.env.example
```

```
# backend/.env.example
# Copia questo file come .env e compila i valori

PORT=3001
JWT_SECRET=genera_con_node_-e_require(crypto).randomBytes(48).toString(hex)
JWT_EXPIRES_IN=7d
NODE_ENV=development
ALLOWED_ORIGIN=http://localhost:5173
```

---

## 4. CORS dinamico nel backend

Aggiorna la configurazione CORS in `backend/server.js` per accettare sia l'origine di sviluppo sia quella di produzione:

```js
// backend/server.js — CORS aggiornato
require('dotenv').config()
const cors = require('cors')

const allowedOrigins = [
  'http://localhost:5173',               // sviluppo locale
  process.env.ALLOWED_ORIGIN,            // produzione (impostato in Render)
].filter(Boolean)   // rimuove undefined/null se ALLOWED_ORIGIN non è impostato

app.use(cors({
  origin: (origin, callback) => {
    // Permetti richieste senza origin (curl, Postman, health check)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error(`CORS bloccato: ${origin} non nella whitelist`))
    }
  },
  credentials: true,   // necessario se userai cookie in futuro
}))
```

---

## 5. Preparare il backend per Render

### 5.1 Aggiorna `backend/package.json`

Aggiungi il campo `engines` e assicurati che lo script `start` sia corretto:

```json
{
  "name": "devnotes-api",
  "version": "1.0.0",
  "engines": {
    "node": ">=18.0.0"
  },
  "scripts": {
    "start": "node server.js",
    "dev":   "nodemon server.js"
  },
  "dependencies": {
    "bcrypt":       "^5.0.0",
    "cors":         "^2.8.5",
    "dotenv":       "^16.0.0",
    "express":      "^4.18.0",
    "jsonwebtoken": "^9.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

### 5.2 Verifica che il server accetti PORT da environment

```js
// backend/server.js — assicurati che la porta sia dinamica
const PORT = process.env.PORT || 3001
app.listen(PORT, () => console.log(`✓ Server attivo su porta ${PORT}`))
```

### 5.3 Aggiungi un endpoint health check

Render usa questo endpoint per verificare che il server sia vivo:

```js
// backend/server.js
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() })
})
```

### 5.4 Commit e push su GitHub

```bash
# Windows / macOS / Linux
cd devnotes
git add .
git commit -m "feat: prepare for production deploy"
git push origin l10-deploy

# Merge su main per attivare la CI/CD
git checkout main
git merge l10-deploy
git push origin main
```

---

## 6. vercel.json e preparare il frontend

### 6.1 Crea `frontend/vercel.json`

```bash
# Windows (PowerShell)
New-Item devnotes\frontend\vercel.json -ItemType File

# macOS / Linux
touch devnotes/frontend/vercel.json
```

```json
{
  "rewrites": [
    {
      "source": "/((?!api/).*)",
      "destination": "/index.html"
    }
  ]
}
```

**Perché è necessario:** React Router usa la History API. Senza questo file, visitare direttamente `devnotes.vercel.app/login` farebbe cercare a Vercel un file `login/index.html` che non esiste → 404. Il rewrite dice: "qualunque path che non inizia con `/api/` → serve `index.html`".

### 6.2 Verifica la build con le variabili di produzione

```bash
# Simula una build di produzione con l'URL del backend
# Windows (PowerShell)
$env:VITE_API_URL="https://placeholder.onrender.com"; npm run build

# macOS / Linux
VITE_API_URL=https://placeholder.onrender.com npm run build
```

Se la build riesce, il setup delle env vars è corretto.

---

## 7. Deploy del backend su Render

### 7.1 Crea il Web Service

1. Vai su **[render.com](https://render.com)** → accedi/registrati
2. Clicca **"New +"** → **"Web Service"**
3. Scegli **"Build and deploy from a Git repository"** → connetti GitHub
4. Seleziona il tuo repository e il branch **`main`**

### 7.2 Configura il servizio

| Campo | Valore |
|---|---|
| Name | `devnotes-api` (o il nome che preferisci) |
| Region | Frankfurt EU Central (o la più vicina) |
| Root Directory | `backend` |
| Runtime | Node |
| Build Command | `npm install` |
| Start Command | `npm start` |
| Plan | Free |

### 7.3 Aggiungi le variabili d'ambiente

Scorri fino a "Environment Variables" e aggiungi:

| Key | Value |
|---|---|
| `JWT_SECRET` | Il tuo segreto generato con `crypto.randomBytes(48)` |
| `JWT_EXPIRES_IN` | `7d` |
| `NODE_ENV` | `production` |
| `ALLOWED_ORIGIN` | *(lo aggiornerai dopo il deploy di Vercel — metti un placeholder per ora)* |

### 7.4 Crea il servizio e attendi il deploy

Clicca **"Create Web Service"**. Il deploy richiede 2–5 minuti alla prima volta.

Quando il log mostra `✓ Server attivo su porta 10000` (Render assegna automaticamente la porta), il backend è online.

### 7.5 Salva l'URL del backend

L'URL ha la forma `https://devnotes-api-xxxx.onrender.com`. Copialo — ti serve nei passi successivi.

### 7.6 Verifica il backend con curl

```bash
# Windows (PowerShell)
Invoke-WebRequest -Uri https://devnotes-api-xxxx.onrender.com/api/health

# macOS / Linux
curl https://devnotes-api-xxxx.onrender.com/api/health
# Risposta: {"status":"ok","timestamp":"2026-04-18T..."}
```

---

## 8. Deploy del frontend su Vercel

### 8.1 Metodo A — CLI (da terminale)

```bash
# Installa la CLI globalmente (una volta sola)
npm install -g vercel

# Dalla cartella frontend/
cd devnotes/frontend
vercel

# Rispondi alle domande:
# Set up and deploy? → Y
# Link to existing project? → N
# Project name? → devnotes-frontend
# In which directory is your code located? → ./ (premi Enter)
# Want to override settings? → N
```

Aggiungi la variabile d'ambiente e deploya in produzione:

```bash
# Imposta la variabile d'ambiente su Vercel
vercel env add VITE_API_URL production
# Inserisci: https://devnotes-api-xxxx.onrender.com

# Deploy di produzione
vercel --prod
```

### 8.2 Metodo B — Interfaccia web (consigliato)

1. Vai su **[vercel.com](https://vercel.com)** → accedi con GitHub
2. Clicca **"Add New Project"** → importa il repository
3. Configura:
   - **Root Directory:** `frontend`
   - **Framework Preset:** Vite (rilevato automaticamente)
   - **Build Command:** `npm run build` (default Vite)
   - **Output Directory:** `dist` (default Vite)
4. In **"Environment Variables"** aggiungi:
   - Key: `VITE_API_URL`
   - Value: `https://devnotes-api-xxxx.onrender.com`
5. Clicca **"Deploy"**

Vercel completa il deploy in 1–2 minuti. L'URL ha la forma `https://devnotes-frontend-xxxx.vercel.app`.

---

## 9. Collegare frontend e backend in produzione

Ora che entrambi i servizi sono online, devi aggiornare `ALLOWED_ORIGIN` su Render con l'URL reale di Vercel.

### 9.1 Aggiorna ALLOWED_ORIGIN su Render

1. Vai su **render.com** → il tuo Web Service `devnotes-api`
2. Clicca **"Environment"** nel menu laterale
3. Modifica `ALLOWED_ORIGIN` con l'URL Vercel esatto:
   `https://devnotes-frontend-xxxx.vercel.app`
4. Clicca **"Save Changes"** → Render rideploya automaticamente

> ⚠️ Usa l'URL **esatto** come appare nella barra del browser, incluso l'`https://` e senza trailing slash finale.

### 9.2 Aggiorna `.env.production` nel frontend

```
# frontend/.env.production
VITE_API_URL=https://devnotes-api-xxxx.onrender.com
```

Poi fai un nuovo deploy su Vercel (o aspetta che la CI/CD lo faccia al prossimo push su main):

```bash
# Se usi la CLI
cd devnotes/frontend
vercel --prod
```

---

## 10. Test in produzione

Apri `https://devnotes-frontend-xxxx.vercel.app` nel browser.

### 10.1 Test registrazione in produzione

1. Vai all'URL del frontend
2. Verifica di essere reindirizzato a `/login` ✅
3. Registra un nuovo account
4. Verifica il messaggio di successo ✅

### 10.2 Test login

1. Accedi con le credenziali appena create
2. Verifica di essere reindirizzato alla pagina delle note ✅
3. Apri DevTools → Network → verifica che le richieste vadano a `https://devnotes-api-xxxx.onrender.com` ✅

### 10.3 Test CRUD

1. Aggiungi una nota tramite il form → verifica che appaia nella lista ✅
2. Elimina una nota → verifica che sparisca ✅

### 10.4 Test refresh diretto

1. Naviga a `https://devnotes-frontend-xxxx.vercel.app/login` (se hai altre route)
2. Premi F5/Refresh → deve caricare correttamente (verifica il `vercel.json`) ✅

### 10.5 Test CI/CD

1. Modifica un file del frontend (es. il titolo in `App.jsx`)
2. Fai commit e push su `main`
3. Vai su Vercel → il deploy parte automaticamente entro 30 secondi ✅
4. Dopo il deploy, ricarica la pagina → vedi la modifica ✅

---

## 11. Esercizi di consolidamento

### Esercizio A — Health check e monitoring ⭐

**Obiettivo:** Tenere sveglio il backend sul piano gratuito di Render.

**Attività:**

1. Registrati su **[uptimerobot.com](https://uptimerobot.com)** (gratuito)
2. Crea un "HTTP(s) Monitor" con:
   - URL: `https://devnotes-api-xxxx.onrender.com/api/health`
   - Intervallo: 5 minuti
3. Verifica che Render non entri più in modalità sleep

**Alternativa Node.js:** aggiungi un self-ping al backend:

```js
// backend/server.js — auto-ping ogni 14 minuti (solo in produzione)
if (process.env.NODE_ENV === 'production') {
  setInterval(() => {
    fetch(`https://${process.env.RENDER_EXTERNAL_HOSTNAME}/api/health`)
      .catch(() => {})   // ignora errori del ping
  }, 14 * 60 * 1000)
}
```

---

### Esercizio B — Variabili d'ambiente per ambiente staging ⭐⭐

**Obiettivo:** Capire come gestire più ambienti (dev/staging/prod).

**Attività:**

1. Crea un branch `staging` su GitHub
2. Su Vercel, configura un secondo deploy per il branch `staging`:
   - Settings → Git → "Preview Branches" oppure crea un secondo progetto
   - Imposta `VITE_API_URL` a un URL di backend diverso (o allo stesso Render per semplicità)
3. Crea il file `frontend/.env.staging`:
   ```
   VITE_API_URL=https://devnotes-api-staging.onrender.com
   ```
4. Verifica che i branch di staging usino la variabile corretta

---

### Esercizio C — Script di verifica pre-deploy ⭐⭐

**Obiettivo:** Automatizzare la checklist di verifica prima del deploy.

**Attività:**

Crea `devnotes/scripts/pre-deploy-check.js`:

```js
// scripts/pre-deploy-check.js
const fs   = require('fs')
const path = require('path')

const checks = []

function check(name, condition, fix) {
  const ok = condition()
  checks.push({ name, ok, fix })
}

// 1. .env nel .gitignore backend
check(
  '.env nel .gitignore backend',
  () => {
    const gi = path.join(__dirname, '../backend/.gitignore')
    if (!fs.existsSync(gi)) return false
    return fs.readFileSync(gi, 'utf8').includes('.env')
  },
  'Aggiungi .env al backend/.gitignore'
)

// 2. vercel.json esiste
check(
  'vercel.json presente',
  () => fs.existsSync(path.join(__dirname, '../frontend/vercel.json')),
  'Crea frontend/vercel.json con il rewrite SPA'
)

// 3. engines in backend/package.json
check(
  'engines in backend/package.json',
  () => {
    const pkg = JSON.parse(fs.readFileSync(path.join(__dirname, '../backend/package.json'), 'utf8'))
    return !!pkg.engines?.node
  },
  'Aggiungi "engines": { "node": ">=18.0.0" } al backend/package.json'
)

// Report
let failed = 0
checks.forEach(c => {
  const icon = c.ok ? '✅' : '❌'
  console.log(`${icon} ${c.name}${!c.ok ? `\n   Fix: ${c.fix}` : ''}`)
  if (!c.ok) failed++
})

console.log(`\n${failed === 0 ? '✅ Tutti i check superati!' : `❌ ${failed} check falliti — correggi prima di deployare.`}`)
process.exit(failed > 0 ? 1 : 0)
```

Aggiungi lo script al `package.json` della root:

```json
"scripts": {
  "pre-deploy": "node scripts/pre-deploy-check.js",
  "dev": "...",
  "build": "..."
}
```

Eseguilo con:

```bash
# Windows / macOS / Linux
npm run pre-deploy
```

---

## 12. Checklist finale

### Backend (Render)

- [ ] `require('dotenv').config()` è la prima riga di `server.js`
- [ ] `app.listen(process.env.PORT || 3001, ...)` — porta dinamica
- [ ] `engines.node` in `backend/package.json`
- [ ] Script `"start": "node server.js"` in `backend/package.json`
- [ ] `.env` nel `.gitignore` del backend (**non** nel repository!)
- [ ] `.env.example` committato con i nomi delle variabili senza valori
- [ ] Endpoint `/api/health` risponde con `{"status":"ok"}` ✅
- [ ] `ALLOWED_ORIGIN` in Render = URL esatto del frontend Vercel
- [ ] Login funziona dal frontend in produzione ✅

### Frontend (Vercel)

- [ ] `frontend/vercel.json` con rewrite `→ /index.html`
- [ ] `frontend/.env.production` con `VITE_API_URL` = URL Render
- [ ] `VITE_API_URL` impostato anche nel dashboard Vercel → Environment Variables
- [ ] `npm run build` completa senza errori
- [ ] `npm run preview` mostra l'app (anche senza backend locale)
- [ ] Navigazione diretta a `/login` funziona (non dà 404) ✅
- [ ] DevTools Network mostra richieste all'URL Render (non localhost) ✅

### CI/CD

- [ ] Push su `main` trigghera il deploy su Vercel ✅
- [ ] Push su `main` trigghera il deploy su Render ✅
- [ ] Dopo un push, la modifica è visibile online entro 3 minuti ✅

---

## 🏆 Progetto completato!

Hai costruito e deployato **DevNotes** — un'applicazione web full-stack completa:

| Livello | Tecnologia | Funzionalità |
|---|---|---|
| Frontend | React + Vite | SPA con routing, hooks, Context API |
| Stile | CSS Modules + Tailwind | Responsive, design system |
| Autenticazione | JWT + bcrypt | Login/logout, route protette |
| Backend | Node.js + Express | API REST CRUD, middleware auth |
| Deploy | Vercel + Render | HTTPS, CDN, CI/CD automatico |

### Prossimi passi consigliati

1. **Database reale:** Sostituisci gli store in-memory con MongoDB (Mongoose) o PostgreSQL (Prisma)
2. **Testing:** Aggiungi test con Vitest (frontend) e Jest/Supertest (backend)
3. **TypeScript:** Migra gradualmente il progetto a TypeScript per type safety
4. **Autenticazione avanzata:** Refresh token, OAuth con Google/GitHub, cookie HttpOnly
5. **Performance:** React Query o SWR per data fetching con cache, React.lazy per code splitting
