# Lab 09 — Autenticazione base con JWT

> **Serie:** React + Vite + Node.js — DevNotes  
> **Lezione di riferimento:** [L09 — Autenticazione base con JWT](blog-react-vite-nodejs-l09-auth-jwt.html)  
> **Repository:** `profgiagnotti/react-vite-guide`  
> **Branch di partenza:** `l08-fullstack` → lavora su `l09-auth-jwt`

---

## ✅ Obiettivi del laboratorio

- [ ] Installare `jsonwebtoken`, `bcrypt` e `dotenv` nel backend
- [ ] Creare il file `.env` con `JWT_SECRET` e aggiungerlo al `.gitignore`
- [ ] Implementare lo store utenti e le route `/api/auth/register` e `/api/auth/login`
- [ ] Scrivere il middleware `authMiddleware.js` che verifica il JWT
- [ ] Proteggere le route `/api/notes` con il middleware
- [ ] Creare `AuthContext` con `useAuth`, `login`, `logout` e inizializzazione lazy
- [ ] Aggiornare `notesApi.js` per inviare `Authorization: Bearer <token>`
- [ ] Costruire la pagina `LoginPage` con form register/login
- [ ] Implementare `ProtectedRoute` e aggiornare il routing in `main.jsx`
- [ ] Testare il flusso completo: register → login → note → logout

---

## 📋 Prerequisiti

| Requisito | Versione minima |
|---|---|
| Node.js | ≥ 18.x |
| Progetto DevNotes con backend Express funzionante (L08) | — |
| React Router v6 configurato (L05) | — |
| Layer API `notesApi.js` e hook `useNotes` (L08) | — |

```bash
# Windows / macOS / Linux
git checkout l08-fullstack
git checkout -b l09-auth-jwt
```

---

## 📑 Indice

1. [Dipendenze e file .env](#1-dipendenze-e-file-env)
2. [Store utenti e route auth nel backend](#2-store-utenti-e-route-auth-nel-backend)
3. [Middleware di autenticazione](#3-middleware-di-autenticazione)
4. [Proteggere le route notes](#4-proteggere-le-route-notes)
5. [AuthContext e useAuth nel frontend](#5-authcontext-e-useauth-nel-frontend)
6. [Aggiornare notesApi.js](#6-aggiornare-notesapijs)
7. [Costruire LoginPage](#7-costruire-loginpage)
8. [ProtectedRoute e routing](#8-protectedroute-e-routing)
9. [Test del flusso completo](#9-test-del-flusso-completo)
10. [Esercizi di consolidamento](#10-esercizi-di-consolidamento)
11. [Checklist finale](#11-checklist-finale)

---

## 1. Dipendenze e file .env

### 1.1 Installa i pacchetti nel backend

```bash
# Windows / macOS / Linux
cd devnotes/backend
npm install jsonwebtoken bcrypt dotenv
```

Verifica che le dipendenze siano in `backend/package.json`:

```json
"dependencies": {
  "bcrypt":         "^5.0.0",
  "cors":           "^2.8.5",
  "dotenv":         "^16.0.0",
  "express":        "^4.18.0",
  "jsonwebtoken":   "^9.0.0"
}
```

### 1.2 Crea il file `.env`

```bash
# Windows (PowerShell)
New-Item devnotes\backend\.env -ItemType File

# macOS / Linux
touch devnotes/backend/.env
```

Incolla il contenuto:

```
PORT=3001
JWT_SECRET=cambia_questa_stringa_ora_deve_essere_lunga_almeno_32_caratteri_e_casuale
JWT_EXPIRES_IN=7d
NODE_ENV=development
ALLOWED_ORIGIN=http://localhost:5173
```

> 🔑 **Genera un JWT_SECRET sicuro** con Node.js:
> ```bash
> node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
> ```
> Copia il risultato nel campo `JWT_SECRET`.

### 1.3 Aggiungi `.env` al `.gitignore`

```bash
# Windows (PowerShell)
Add-Content devnotes\backend\.gitignore "`n.env`n.env.local"

# macOS / Linux
echo -e "\n.env\n.env.local" >> devnotes/backend/.gitignore
```

Crea il file se non esiste:

```
# backend/.gitignore
node_modules/
.env
.env.local
```

---

## 2. Store utenti e route auth nel backend

### 2.1 Crea la struttura

```bash
# Windows (PowerShell)
New-Item devnotes\backend\data\users.js -ItemType File
New-Item devnotes\backend\routes\auth.js -ItemType File

# macOS / Linux
touch devnotes/backend/data/users.js
touch devnotes/backend/routes/auth.js
```

### 2.2 Scrivi `backend/data/users.js`

```js
// backend/data/users.js
// Store in-memory degli utenti.
// In produzione questo viene sostituito con un database (MongoDB, PostgreSQL, ecc.)
const users  = []
let   nextId = 1

/**
 * Cerca un utente per email (case-insensitive).
 * @param {string} email
 * @returns {object|undefined}
 */
function findByEmail(email) {
  return users.find(u => u.email === email.toLowerCase())
}

/**
 * Crea e salva un nuovo utente.
 * @param {{ email: string, passwordHash: string }} param0
 * @returns {object} l'utente appena creato
 */
function createUser({ email, passwordHash }) {
  const user = { id: nextId++, email: email.toLowerCase(), passwordHash }
  users.push(user)
  return user
}

module.exports = { findByEmail, createUser }
```

### 2.3 Scrivi `backend/routes/auth.js`

```js
// backend/routes/auth.js
const router = require('express').Router()
const bcrypt = require('bcrypt')
const jwt    = require('jsonwebtoken')
const { findByEmail, createUser } = require('../data/users')

const SALT_ROUNDS = 10   // ~100ms su hardware moderno

// ─── POST /api/auth/register ──────────────────────────────────────────
router.post('/register', async (req, res, next) => {
  try {
    const { email, password } = req.body

    // Validazione input
    if (!email || !password)
      return res.status(400).json({ messaggio: 'Email e password obbligatorie' })
    if (!/\S+@\S+\.\S+/.test(email))
      return res.status(422).json({ messaggio: 'Formato email non valido' })
    if (password.length < 8)
      return res.status(422).json({ messaggio: 'Password min 8 caratteri' })

    // Controlla se l'email è già registrata
    if (findByEmail(email))
      return res.status(409).json({ messaggio: 'Email già registrata' })

    // Hash della password (bcrypt aggiunge automaticamente il salt)
    const passwordHash = await bcrypt.hash(password, SALT_ROUNDS)
    const user         = createUser({ email, passwordHash })

    res.status(201).json({ messaggio: 'Registrazione completata', id: user.id })
  } catch (err) { next(err) }
})

// ─── POST /api/auth/login ─────────────────────────────────────────────
router.post('/login', async (req, res, next) => {
  try {
    const { email, password } = req.body
    const user = findByEmail(email)

    // Risposta generica: non rivelare se l'email esiste (anti-user enumeration)
    if (!user) return res.status(401).json({ messaggio: 'Credenziali non valide' })

    const match = await bcrypt.compare(password, user.passwordHash)
    if (!match)  return res.status(401).json({ messaggio: 'Credenziali non valide' })

    // Firma il JWT con sub (subject = userId), email, iat (auto), exp (auto)
    const token = jwt.sign(
      { sub: String(user.id), email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: process.env.JWT_EXPIRES_IN || '7d' }
    )

    res.json({ token, email: user.email })
  } catch (err) { next(err) }
})

module.exports = router
```

---

## 3. Middleware di autenticazione

### 3.1 Crea il file

```bash
# Windows (PowerShell)
New-Item devnotes\backend\middleware\authMiddleware.js -ItemType File

# macOS / Linux
mkdir -p devnotes/backend/middleware
touch devnotes/backend/middleware/authMiddleware.js
```

### 3.2 Scrivi `authMiddleware.js`

```js
// backend/middleware/authMiddleware.js
const jwt = require('jsonwebtoken')

/**
 * Middleware che verifica il JWT nell'header Authorization.
 * Se valido, aggiunge req.user = { sub, email, iat, exp }.
 * Se non valido, risponde con 401.
 */
function authMiddleware(req, res, next) {
  const authHeader = req.headers['authorization']

  // Formato atteso: "Bearer eyJ..."
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ messaggio: 'Token mancante' })
  }

  const token = authHeader.slice(7)   // rimuove i primi 7 caratteri "Bearer "

  try {
    // jwt.verify() controlla firma E scadenza in un'unica chiamata
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    req.user = payload    // disponibile nelle route come req.user.sub, req.user.email
    next()
  } catch (err) {
    const messaggio = err.name === 'TokenExpiredError'
      ? 'Token scaduto — effettua nuovamente il login'
      : 'Token non valido'
    res.status(401).json({ messaggio })
  }
}

module.exports = authMiddleware
```

---

## 4. Proteggere le route notes

### 4.1 Aggiorna `backend/server.js`

Aggiungi `require('dotenv').config()` **in cima** al file (prima di tutto il resto) e aggiorna il montaggio delle route:

```js
// backend/server.js — versione aggiornata
require('dotenv').config()   // DEVE essere la prima riga! Carica .env in process.env

const express        = require('express')
const cors           = require('cors')
const authMiddleware = require('./middleware/authMiddleware')
const authRoutes     = require('./routes/auth')
const notesRoutes    = require('./routes/notes')

const app = express()

app.use(cors({
  origin:      process.env.ALLOWED_ORIGIN || 'http://localhost:5173',
  credentials: true,
}))
app.use(express.json())

// ─── Route pubbliche (senza auth) ──────────────────────────────────────
app.get('/api/health', (req, res) => res.json({ status: 'ok' }))
app.use('/api/auth', authRoutes)

// ─── Route protette (auth richiesta) ───────────────────────────────────
// authMiddleware viene eseguito come secondo argomento, prima di notesRoutes
app.use('/api/notes', authMiddleware, notesRoutes)

// ─── Middleware errori (alla fine) ─────────────────────────────────────
app.use((req, res) => res.status(404).json({ messaggio: 'Risorsa non trovata' }))
app.use((err, req, res, next) => {
  console.error('[SERVER ERROR]', err.message)
  res.status(err.status || 500).json({
    messaggio: err.message || 'Errore interno del server',
  })
})

const PORT = process.env.PORT || 3001
app.listen(PORT, () => console.log(`✓ Server su porta ${PORT}`))
```

### 4.2 Verifica con curl

```bash
# Richiesta senza token → 401
# Windows (PowerShell)
Invoke-WebRequest -Uri http://localhost:3001/api/notes -Method GET

# macOS / Linux
curl -s http://localhost:3001/api/notes
# Risposta attesa: {"messaggio":"Token mancante"}

# Register un utente
curl -s -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@devnotes.it","password":"password123"}'
# Risposta: {"messaggio":"Registrazione completata","id":1}

# Login e salva il token
curl -s -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@devnotes.it","password":"password123"}'
# Risposta: {"token":"eyJ...","email":"test@devnotes.it"}

# Richiesta con token → 200
curl -s http://localhost:3001/api/notes \
  -H "Authorization: Bearer eyJ..."
# Risposta: [] (array note)
```

---

## 5. AuthContext e useAuth nel frontend

### 5.1 Crea i file

```bash
# Windows (PowerShell)
New-Item devnotes\frontend\src\context\AuthContext.jsx -ItemType File

# macOS / Linux
mkdir -p devnotes/frontend/src/context
touch devnotes/frontend/src/context/AuthContext.jsx
```

### 5.2 Scrivi `AuthContext.jsx`

```jsx
// frontend/src/context/AuthContext.jsx
import { createContext, useContext, useState, useCallback } from 'react'

const AuthContext = createContext(null)

/**
 * Provider che avvolge l'intera app e rende disponibile
 * lo stato di autenticazione a tutti i componenti figli.
 */
export function AuthProvider({ children }) {
  // Inizializzazione lazy: la funzione viene eseguita solo al primo render
  // evitando di chiamare localStorage ad ogni re-render
  const [token,     setToken]     = useState(() => localStorage.getItem('devnotes_token'))
  const [userEmail, setUserEmail] = useState(() => localStorage.getItem('devnotes_email'))

  const login = useCallback((newToken, email) => {
    localStorage.setItem('devnotes_token', newToken)
    localStorage.setItem('devnotes_email', email)
    setToken(newToken)
    setUserEmail(email)
  }, [])

  const logout = useCallback(() => {
    localStorage.removeItem('devnotes_token')
    localStorage.removeItem('devnotes_email')
    setToken(null)
    setUserEmail(null)
  }, [])

  const value = {
    token,
    userEmail,
    login,
    logout,
    isAuth: !!token,   // true se token non è null/undefined/""
  }

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  )
}

/**
 * Hook che espone il contesto auth.
 * Lancia un errore se usato fuori dall'AuthProvider (fail-fast).
 */
export function useAuth() {
  const ctx = useContext(AuthContext)
  if (!ctx) throw new Error('useAuth deve essere usato dentro <AuthProvider>')
  return ctx
}
```

---

## 6. Aggiornare notesApi.js

Modifica `frontend/src/api/notesApi.js` per includere il token in ogni richiesta:

```js
// frontend/src/api/notesApi.js — versione con auth
const API_BASE = '/api/notes'

/**
 * Costruisce gli header con il token JWT se disponibile.
 * Legge il token da localStorage ad ogni chiamata (non da un parametro)
 * così rileva sempre il token più aggiornato dopo login/logout.
 */
function authHeaders(extra = {}) {
  const token = localStorage.getItem('devnotes_token')
  return {
    'Content-Type': 'application/json',
    ...(token && { 'Authorization': `Bearer ${token}` }),
    ...extra,
  }
}

export async function fetchNotes() {
  const res = await fetch(API_BASE, { headers: authHeaders() })
  if (!res.ok) throw new Error(`Errore GET note: ${res.status}`)
  return res.json()
}

export async function createNote(nota) {
  const res = await fetch(API_BASE, {
    method:  'POST',
    headers: authHeaders(),
    body:    JSON.stringify(nota),
  })
  if (!res.ok) {
    const errore = await res.json().catch(() => ({}))
    throw new Error(errore.messaggio || `Errore POST: ${res.status}`)
  }
  return res.json()
}

export async function deleteNote(id) {
  const res = await fetch(`${API_BASE}/${id}`, {
    method:  'DELETE',
    headers: authHeaders(),
  })
  if (!res.ok) throw new Error(`Errore DELETE ${id}: ${res.status}`)
}

/**
 * Helper per le chiamate alle route auth (no token richiesto).
 * @param {'register'|'login'} action
 * @param {{ email: string, password: string }} body
 */
export async function authRequest(action, body) {
  const res = await fetch(`/api/auth/${action}`, {
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

## 7. Costruire LoginPage

### 7.1 Crea i file

```bash
# Windows (PowerShell)
New-Item devnotes\frontend\src\pages\LoginPage.jsx -ItemType File
New-Item devnotes\frontend\src\pages\LoginPage.module.css -ItemType File

# macOS / Linux
mkdir -p devnotes/frontend/src/pages
touch devnotes/frontend/src/pages/LoginPage.jsx
touch devnotes/frontend/src/pages/LoginPage.module.css
```

### 7.2 Scrivi `LoginPage.jsx`

```jsx
// frontend/src/pages/LoginPage.jsx
import { useState } from 'react'
import { useNavigate } from 'react-router-dom'
import { useAuth } from '../context/AuthContext'
import { authRequest } from '../api/notesApi'
import styles from './LoginPage.module.css'

export default function LoginPage() {
  const { login }    = useAuth()
  const navigate     = useNavigate()

  // Modalità: 'login' o 'register'
  const [mode,       setMode]       = useState('login')
  const [form,       setForm]       = useState({ email: '', password: '' })
  const [submitting, setSubmitting] = useState(false)
  const [errMsg,     setErrMsg]     = useState('')
  const [okMsg,      setOkMsg]      = useState('')

  function handleChange(e) {
    setForm(prev => ({ ...prev, [e.target.name]: e.target.value }))
  }

  async function handleSubmit(e) {
    e.preventDefault()
    setErrMsg('')
    setOkMsg('')

    // Validazione base
    if (!form.email || !form.password) {
      setErrMsg('Compila tutti i campi')
      return
    }

    try {
      setSubmitting(true)

      if (mode === 'register') {
        // Registrazione: non fa il login automatico
        await authRequest('register', form)
        setOkMsg('Registrazione completata! Ora puoi fare il login.')
        setMode('login')
        setForm(prev => ({ ...prev, password: '' }))
      } else {
        // Login: riceve il token e reindirizza
        const { token, email } = await authRequest('login', form)
        login(token, email)
        navigate('/', { replace: true })
      }
    } catch (err) {
      setErrMsg(err.message)
    } finally {
      setSubmitting(false)
    }
  }

  return (
    <div className={styles.page}>
      <div className={styles.card}>
        <h1 className={styles.logo}>DevNotes</h1>
        <p className={styles.subtitle}>
          {mode === 'login' ? 'Accedi al tuo account' : 'Crea un nuovo account'}
        </p>

        <form onSubmit={handleSubmit} className={styles.form}>
          <input
            type="email"
            name="email"
            placeholder="la-tua@email.it"
            value={form.email}
            onChange={handleChange}
            className={styles.input}
            autoComplete="email"
          />
          <input
            type="password"
            name="password"
            placeholder="Password (min 8 caratteri)"
            value={form.password}
            onChange={handleChange}
            className={styles.input}
            autoComplete={mode === 'login' ? 'current-password' : 'new-password'}
          />

          {errMsg && <p className={styles.errMsg}>⚠ {errMsg}</p>}
          {okMsg  && <p className={styles.okMsg}>✓ {okMsg}</p>}

          <button type="submit" disabled={submitting} className={styles.btn}>
            {submitting
              ? 'Attendere…'
              : mode === 'login' ? 'Accedi' : 'Registrati'
            }
          </button>
        </form>

        <p className={styles.toggle}>
          {mode === 'login' ? 'Non hai un account?' : 'Hai già un account?'}
          {' '}
          <button
            className={styles.toggleBtn}
            onClick={() => { setMode(m => m === 'login' ? 'register' : 'login'); setErrMsg(''); setOkMsg('') }}
          >
            {mode === 'login' ? 'Registrati' : 'Accedi'}
          </button>
        </p>
      </div>
    </div>
  )
}
```

### 7.3 Scrivi `LoginPage.module.css`

```css
/* frontend/src/pages/LoginPage.module.css */
.page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--deep-void);
  padding: 1.5rem;
}

.card {
  background: var(--surface-1);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 2.5rem 2rem;
  width: 100%;
  max-width: 400px;
}

.logo {
  font-size: 1.8rem;
  font-weight: 700;
  color: var(--blue);
  text-align: center;
  margin-bottom: 0.35rem;
}

.subtitle {
  color: var(--slate);
  text-align: center;
  font-size: 0.9rem;
  margin-bottom: 1.8rem;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
}

.input {
  background: var(--surface-2);
  border: 1px solid var(--border);
  border-radius: 7px;
  color: var(--ice);
  font-family: inherit;
  font-size: 0.95rem;
  padding: 0.75rem 1rem;
  transition: border-color 0.2s;
  width: 100%;
}

.input:focus {
  border-color: var(--blue);
  outline: none;
}

.errMsg { color: var(--red);   font-size: 0.85rem; margin: 0; }
.okMsg  { color: var(--green); font-size: 0.85rem; margin: 0; }

.btn {
  background: var(--blue);
  border: none;
  border-radius: 7px;
  color: #001824;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 700;
  padding: 0.8rem;
  margin-top: 0.3rem;
  transition: background 0.2s, opacity 0.2s;
}

.btn:hover:not(:disabled) { background: #33dcff; }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }

.toggle {
  color: var(--slate);
  font-size: 0.85rem;
  margin-top: 1.2rem;
  text-align: center;
}

.toggleBtn {
  background: none;
  border: none;
  color: var(--blue);
  cursor: pointer;
  font-size: 0.85rem;
  padding: 0;
  text-decoration: underline;
}

.toggleBtn:hover { color: #33dcff; }
```

---

## 8. ProtectedRoute e routing

### 8.1 Crea `ProtectedRoute.jsx`

```bash
# Windows (PowerShell)
New-Item devnotes\frontend\src\components\ProtectedRoute.jsx -ItemType File

# macOS / Linux
touch devnotes/frontend/src/components/ProtectedRoute.jsx
```

```jsx
// frontend/src/components/ProtectedRoute.jsx
import { Navigate, Outlet } from 'react-router-dom'
import { useAuth } from '../context/AuthContext'

/**
 * Route wrapper: se l'utente non è autenticato, reindirizza a /login.
 * Le route figlie vengono renderizzate tramite <Outlet />.
 *
 * Uso in main.jsx:
 *   <Route element={<ProtectedRoute />}>
 *     <Route path="/" element={<App />} />
 *   </Route>
 */
export default function ProtectedRoute() {
  const { isAuth } = useAuth()
  return isAuth
    ? <Outlet />
    : <Navigate to="/login" replace />
}
```

### 8.2 Aggiorna `frontend/src/main.jsx`

```jsx
// frontend/src/main.jsx
import React       from 'react'
import ReactDOM    from 'react-dom/client'
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { AuthProvider }  from './context/AuthContext'
import ProtectedRoute    from './components/ProtectedRoute'
import App               from './App'
import LoginPage         from './pages/LoginPage'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          {/* Route pubblica */}
          <Route path="/login" element={<LoginPage />} />

          {/* Route protette: richiedono autenticazione */}
          <Route element={<ProtectedRoute />}>
            <Route path="/"    element={<App />} />
            {/* Aggiungi qui altre route che richiedono login */}
          </Route>
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  </React.StrictMode>
)
```

### 8.3 Aggiorna l'header in `App.jsx` per mostrare il bottone logout

```jsx
// frontend/src/App.jsx — aggiunta pulsante logout
import { useAuth } from './context/AuthContext'

export default function App() {
  const { userEmail, logout } = useAuth()
  const { notes, loading, error, addNote, removeNote, reload } = useNotes()

  // ... resto del componente ...

  return (
    <div className={styles.layout}>
      <header className={styles.header}>
        <h1>DevNotes</h1>
        <div style={{ display: 'flex', alignItems: 'center', gap: '1rem' }}>
          <span style={{ color: 'var(--slate)', fontSize: '0.85rem' }}>{userEmail}</span>
          <button onClick={logout} style={{
            background: 'transparent', border: '1px solid var(--border)',
            borderRadius: '5px', color: 'var(--slate)', cursor: 'pointer',
            fontSize: '0.82rem', padding: '0.35rem 0.8rem',
          }}>
            Logout
          </button>
        </div>
      </header>
      {/* ... resto JSX ... */}
    </div>
  )
}
```

---

## 9. Test del flusso completo

### 9.1 Avvia backend e frontend

```bash
# dalla root devnotes/
npm run dev
```

### 9.2 Test registrazione

1. Apri `http://localhost:5173`
2. Vieni reindirizzato a `/login` (ProtectedRoute funziona) ✅
3. Clicca "Registrati" per passare alla modalità register
4. Inserisci email e password valide → clicca "Registrati"
5. Verifica il messaggio di successo e il ritorno alla modalità login ✅

### 9.3 Test login

1. Inserisci le credenziali appena registrate → clicca "Accedi"
2. Vieni reindirizzato a `/` con le note ✅
3. Apri DevTools → Application → localStorage
4. Verifica che `devnotes_token` e `devnotes_email` siano presenti ✅
5. Copia il token e incollalo su **jwt.io** per ispezionare il payload ✅

### 9.4 Test richieste autenticate (DevTools Network)

1. Con il login effettuato, apri Network → Fetch/XHR
2. Ricarica la pagina
3. Seleziona la richiesta `GET /api/notes`
4. Tab "Headers" → verifica che sia presente `Authorization: Bearer eyJ…` ✅

### 9.5 Test token scaduto/manomesso

1. In DevTools → Application → localStorage
2. Modifica `devnotes_token` aggiungendo un carattere alla fine
3. Ricarica la pagina
4. Il backend risponde 401, la UI deve mostrare l'errore ✅
   (opzionale: implementa il reindirizzamento automatico a /login in `useNotes`)

### 9.6 Test logout

1. Clicca il pulsante "Logout" nell'header
2. Vieni reindirizzato a `/login` ✅
3. Verifica in localStorage che `devnotes_token` sia stato rimosso ✅

### 9.7 Test errori backend

```bash
# Credenziali errate → 401
curl -s -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@devnotes.it","password":"sbagliata"}'
# {"messaggio":"Credenziali non valide"}

# Email già registrata → 409
curl -s -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@devnotes.it","password":"nuovaPassword123"}'
# {"messaggio":"Email già registrata"}
```

---

## 10. Esercizi di consolidamento

### Esercizio A — Reindirizzamento automatico su 401 ⭐⭐

**Obiettivo:** Se il backend risponde con 401, l'utente viene reindirizzato automaticamente al login.

**Attività:**

Modifica `useNotes.js` per gestire gli errori 401:

```js
// In loadNotes() del hook useNotes
} catch (err) {
  if (err.message.includes('401') || err.message.toLowerCase().includes('token')) {
    // Token scaduto o non valido: forza il logout
    // Nota: non puoi chiamare useAuth direttamente da questo file
    // perché è una funzione asincrona, non un componente/hook.
    // Soluzione: passa la funzione logout come parametro al hook.
  }
  setError(err.message)
}
```

**Soluzione guidata:**

Modifica `useNotes` per accettare una callback `onUnauthorized`:

```js
export function useNotes(onUnauthorized) {
  // ...
  async function loadNotes() {
    try {
      // ...
    } catch (err) {
      if (err.message.includes('401') && onUnauthorized) {
        onUnauthorized()
      }
      setError(err.message)
    }
  }
}
```

In `App.jsx`:

```jsx
const { logout } = useAuth()
const { notes, /* ... */ } = useNotes(logout)
```

**Verifica:** Manometti il token in localStorage → ricarica → devi essere reindirizzato a `/login`.

---

### Esercizio B — Pagina profilo utente ⭐⭐

**Obiettivo:** Aggiungere una route `/profilo` che mostra l'email dell'utente e il numero di note.

**Attività:**

1. Crea `src/pages/ProfilePage.jsx` che usa `useAuth()` e `useNotes()`:

```jsx
export default function ProfilePage() {
  const { userEmail } = useAuth()
  const { notes }     = useNotes()

  return (
    <div>
      <h1>Profilo</h1>
      <p>Email: {userEmail}</p>
      <p>Note salvate: {notes.length}</p>
    </div>
  )
}
```

2. Aggiungi la route `/profilo` dentro `<ProtectedRoute>` in `main.jsx`
3. Aggiungi un link `<Link to="/profilo">` nell'header di `App.jsx`

---

### Esercizio C — Refresh token simulato ⭐⭐⭐

**Obiettivo:** Capire il problema del token scaduto e come risolverlo senza un vero refresh token.

**Attività:**

1. Cambia `JWT_EXPIRES_IN=30s` nel file `.env` backend (scade in 30 secondi)
2. Fai il login e aspetta 35 secondi
3. Prova ad aggiungere una nota → il backend risponde 401
4. Aggiungi in `LoginPage.jsx` un avviso che informa l'utente che le sessioni durano 30 secondi
5. Ripristina `JWT_EXPIRES_IN=7d`

**Domanda di riflessione:** Come funziona il refresh token in produzione? Perché è necessario un secondo token separato con durata maggiore (es. 30 giorni)?

---

## 11. Checklist finale

- [ ] `.env` presente in `backend/` e nel `.gitignore`
- [ ] `require('dotenv').config()` è la **prima riga** di `server.js`
- [ ] `POST /api/auth/register` risponde 201 con utente valido
- [ ] `POST /api/auth/register` risponde 409 se email già registrata
- [ ] `POST /api/auth/login` risponde 401 con messaggio generico per email/password errate
- [ ] `POST /api/auth/login` risponde con `{ token, email }` con credenziali corrette
- [ ] `GET /api/notes` senza token → 401
- [ ] `GET /api/notes` con token valido → 200 con array
- [ ] `AuthContext` inizializza stato da `localStorage` con funzione lazy
- [ ] `notesApi.js` include `Authorization: Bearer <token>` in tutte le chiamate
- [ ] `LoginPage` gestisce entrambe le modalità register/login nella stessa pagina
- [ ] `ProtectedRoute` reindirizza a `/login` se `isAuth === false`
- [ ] Logout rimuove token da `localStorage` e dallo stato React
- [ ] Token ispezionato su jwt.io mostra payload corretto ✅

---

## 📊 Tabella riepilogo concetti chiave

| Concetto | Dove | Perché |
|---|---|---|
| `bcrypt.hash(pw, 10)` | `routes/auth.js` | Hashing one-way con salt automatico |
| `bcrypt.compare()` | `routes/auth.js` | Confronto sicuro password/hash |
| Risposta 401 generica | `routes/auth.js` | Anti-user enumeration |
| `jwt.sign(payload, secret, options)` | `routes/auth.js` | Firma + aggiunge iat/exp |
| `jwt.verify(token, secret)` | `authMiddleware.js` | Verifica firma + scadenza |
| `req.user = payload` | `authMiddleware.js` | Propaga dati utente alle route |
| Inizializzazione lazy useState | `AuthContext.jsx` | Legge localStorage una sola volta |
| `!!token` | `AuthContext.jsx` | Converte stringa → booleano |
| `<Outlet />` | `ProtectedRoute.jsx` | Renderizza le route figlie se autenticato |
| `<Navigate replace />` | `ProtectedRoute.jsx` | Reindirizza senza aggiungere alla history |

---

## ➡️ Prossimo laboratorio

[Lab 10 — Deploy: Vercel + Render](lab-10-deploy.md)

Preparerai la build di produzione, configurerai le variabili d'ambiente per Vercel e Render,
eseguirai il deploy di frontend e backend e verificherai che l'app funzioni online con HTTPS.
