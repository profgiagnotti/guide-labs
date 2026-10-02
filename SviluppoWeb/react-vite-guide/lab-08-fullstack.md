# Lab 08 — Connettere frontend e backend

> **Serie:** React + Vite + Node.js — DevNotes  
> **Lezione di riferimento:** [L08 — Connettere frontend e backend](blog-react-vite-nodejs-l08-fullstack.html)  
> **Repository:** `profgiagnotti/react-vite-guide`  
> **Branch di partenza:** `l07-express-backend` → lavora su `l08-fullstack`

---

## ✅ Obiettivi del laboratorio

- [ ] Configurare il proxy Vite in `vite.config.js` per inviare `/api/*` al backend
- [ ] Creare il modulo `src/api/notesApi.js` con `fetchNotes`, `createNote`, `deleteNote`
- [ ] Implementare il custom hook `useNotes` con stati `loading`, `error`, `notes`
- [ ] Costruire il componente `NoteForm` con validazione client e feedback visivo
- [ ] Aggiornare `App.jsx` per usare il hook e passare le props corrette
- [ ] Aggiungere il middleware di errore globale in Express
- [ ] Avviare backend e frontend insieme con `concurrently`
- [ ] Testare l'intera catena GET / POST / DELETE con i DevTools del browser

---

## 📋 Prerequisiti

| Requisito | Versione minima |
|---|---|
| Node.js | ≥ 18.x |
| npm | ≥ 9.x |
| Progetto frontend React + Vite (L01–L06) | — |
| Progetto backend Express (L07) | — |
| Struttura cartelle `devnotes/frontend` e `devnotes/backend` | — |

Se parti da zero, clona il branch di partenza:

```bash
# Windows / macOS / Linux
git clone https://github.com/profgiagnotti/react-vite-guide.git devnotes
cd devnotes
git checkout l07-express-backend
```

---

## 📑 Indice

1. [Setup del branch e struttura cartelle](#1-setup-del-branch-e-struttura-cartelle)
2. [Configurare il proxy Vite](#2-configurare-il-proxy-vite)
3. [Creare il layer API](#3-creare-il-layer-api)
4. [Implementare il custom hook useNotes](#4-implementare-il-custom-hook-usenotes)
5. [Costruire NoteForm](#5-costruire-noteform)
6. [Aggiornare App.jsx](#6-aggiornare-appjsx)
7. [Middleware errori in Express](#7-middleware-errori-in-express)
8. [Avvio concorrente con concurrently](#8-avvio-concorrente-con-concurrently)
9. [Test completo dell'integrazione](#9-test-completo-dellintegrazione)
10. [Esercizi di consolidamento](#10-esercizi-di-consolidamento)
11. [Checklist finale](#11-checklist-finale)

---

## 1. Setup del branch e struttura cartelle

### 1.1 Crea il branch di lavoro

```bash
# Windows / macOS / Linux
git checkout -b l08-fullstack
```

### 1.2 Struttura finale che otterrai

```
devnotes/
├── package.json               ← root (lo crei al passo 8)
├── backend/
│   ├── server.js              ← aggiungi middleware errori
│   ├── routes/
│   │   └── notes.js
│   └── package.json
└── frontend/
    ├── vite.config.js         ← aggiungi server.proxy
    ├── src/
    │   ├── api/
    │   │   └── notesApi.js    ← nuovo
    │   ├── hooks/
    │   │   └── useNotes.js    ← nuovo
    │   ├── components/
    │   │   ├── NoteCard.jsx
    │   │   ├── NoteCard.module.css
    │   │   ├── NoteList.jsx
    │   │   ├── NoteList.module.css
    │   │   ├── NoteForm.jsx   ← nuovo
    │   │   └── NoteForm.module.css ← nuovo
    │   ├── App.jsx            ← aggiornato
    │   └── App.module.css     ← aggiornato
    └── package.json
```

### 1.3 Installa le dipendenze dei sotto-progetti (se non già fatto)

```bash
# Windows / macOS / Linux — backend
cd devnotes/backend
npm install

# Windows / macOS / Linux — frontend
cd devnotes/frontend
npm install
```

---

## 2. Configurare il proxy Vite

### 2.1 Apri `frontend/vite.config.js`

Modifica il file aggiungendo la sezione `server.proxy`:

```js
// frontend/vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],

  server: {
    proxy: {
      // Tutte le richieste /api/* vengono inoltrate al backend
      '/api': {
        target:      'http://localhost:3001',
        changeOrigin: true,
        // Decommenta la riga seguente SOLO se il backend usa HTTPS con certificato self-signed
        // secure: false,
      }
    }
  }
})
```

### 2.2 Riavvia il dev server

> ⚠️ Le modifiche a `vite.config.js` richiedono un riavvio manuale — l'HMR non le rileva.

```bash
# Se il dev server è attivo: Ctrl+C per fermarlo, poi:
# Windows / macOS / Linux
cd devnotes/frontend
npm run dev
```

### 2.3 Verifica rapida del proxy

Con il **backend avviato** (`node backend/server.js`), apri nel browser:

```
http://localhost:5173/api/notes
```

Se vedi la risposta JSON del backend (array di note), il proxy funziona. ✅

---

## 3. Creare il layer API

### 3.1 Crea la cartella e il file

```bash
# Windows (PowerShell)
mkdir devnotes\frontend\src\api
New-Item devnotes\frontend\src\api\notesApi.js -ItemType File

# macOS / Linux
mkdir -p devnotes/frontend/src/api
touch devnotes/frontend/src/api/notesApi.js
```

### 3.2 Scrivi `notesApi.js`

```js
// frontend/src/api/notesApi.js

// Grazie al proxy Vite usiamo percorsi relativi:
// /api/notes viene inoltrato a http://localhost:3001/api/notes
const API_BASE = '/api/notes'

/**
 * Recupera tutte le note dal backend.
 * @returns {Promise<Array>}
 */
export async function fetchNotes() {
  const res = await fetch(API_BASE)
  // fetch() NON lancia eccezioni per 4xx/5xx: controlla sempre res.ok!
  if (!res.ok) throw new Error(`Errore GET note: ${res.status}`)
  return res.json()
}

/**
 * Crea una nuova nota.
 * @param {{ titolo: string, contenuto: string, tag: string }} nota
 * @returns {Promise<Object>} la nota con id assegnato dal server
 */
export async function createNote(nota) {
  const res = await fetch(API_BASE, {
    method:  'POST',
    headers: { 'Content-Type': 'application/json' },
    body:    JSON.stringify(nota),
  })
  if (!res.ok) {
    // Prova a leggere il messaggio di errore inviato dal server
    const errore = await res.json().catch(() => ({}))
    throw new Error(errore.messaggio || `Errore POST: ${res.status}`)
  }
  return res.json()
}

/**
 * Elimina una nota per ID.
 * @param {string|number} id
 * @returns {Promise<void>}
 */
export async function deleteNote(id) {
  const res = await fetch(`${API_BASE}/${id}`, { method: 'DELETE' })
  if (!res.ok) throw new Error(`Errore DELETE ${id}: ${res.status}`)
}
```

> 💡 **Perché un file separato?** Se in futuro cambi le URL del backend o passi da `fetch()` a una libreria HTTP (es. `axios`), modifichi solo questo file — i componenti non cambiano.

---

## 4. Implementare il custom hook useNotes

### 4.1 Crea la cartella e il file

```bash
# Windows (PowerShell)
mkdir devnotes\frontend\src\hooks
New-Item devnotes\frontend\src\hooks\useNotes.js -ItemType File

# macOS / Linux
mkdir -p devnotes/frontend/src/hooks
touch devnotes/frontend/src/hooks/useNotes.js
```

### 4.2 Scrivi `useNotes.js`

```js
// frontend/src/hooks/useNotes.js
import { useState, useEffect, useCallback } from 'react'
import { fetchNotes, createNote, deleteNote } from '../api/notesApi'

export function useNotes() {
  const [notes,   setNotes]   = useState([])
  const [loading, setLoading] = useState(true)   // true: mostra spinner al mount
  const [error,   setError]   = useState(null)

  // Carica le note al primo render del componente che usa il hook
  useEffect(() => {
    loadNotes()
  }, [])   // [] = esegui solo al mount

  async function loadNotes() {
    try {
      setLoading(true)
      setError(null)
      const data = await fetchNotes()
      setNotes(data)
    } catch (err) {
      setError(err.message)
    } finally {
      // finally viene eseguito sia in caso di successo sia di errore
      setLoading(false)
    }
  }

  // useCallback: la funzione non viene ricreata ad ogni render di chi usa il hook
  const addNote = useCallback(async (formData) => {
    const nuovaNota = await createNote(formData)
    // Aggiornamento ottimistico: aggiunge subito in testa alla lista
    setNotes(prev => [nuovaNota, ...prev])
  }, [])

  const removeNote = useCallback(async (id) => {
    await deleteNote(id)
    // Rimuove dalla lista locale (senza re-fetch)
    setNotes(prev => prev.filter(n => n.id !== id))
  }, [])

  return { notes, loading, error, addNote, removeNote, reload: loadNotes }
}
```

---

## 5. Costruire NoteForm

### 5.1 Crea i file

```bash
# Windows (PowerShell)
New-Item devnotes\frontend\src\components\NoteForm.jsx -ItemType File
New-Item devnotes\frontend\src\components\NoteForm.module.css -ItemType File

# macOS / Linux
touch devnotes/frontend/src/components/NoteForm.jsx
touch devnotes/frontend/src/components/NoteForm.module.css
```

### 5.2 Scrivi `NoteForm.jsx`

```jsx
// frontend/src/components/NoteForm.jsx
import { useState } from 'react'
import styles from './NoteForm.module.css'

const TAGS_DISPONIBILI = ['React', 'JavaScript', 'CSS', 'Node.js', 'Altro']

export default function NoteForm({ onAdd }) {
  // Stato del form: un oggetto per tutti i campi
  const [form, setForm] = useState({
    titolo:    '',
    contenuto: '',
    tag:       'React',
  })
  const [submitting, setSubmitting] = useState(false)
  const [errMsg,     setErrMsg]     = useState('')
  const [success,    setSuccess]    = useState(false)

  // Handler generico: aggiorna il campo il cui name coincide con e.target.name
  function handleChange(e) {
    setForm(prev => ({ ...prev, [e.target.name]: e.target.value }))
  }

  async function handleSubmit(e) {
    e.preventDefault()   // blocca il comportamento nativo del browser

    // Validazione client-side: non sprecare una richiesta HTTP per dati incompleti
    if (!form.titolo.trim()) {
      setErrMsg('Il titolo è obbligatorio')
      return
    }

    try {
      setSubmitting(true)
      setErrMsg('')
      await onAdd(form)              // delega al genitore (App)
      setForm({ titolo: '', contenuto: '', tag: 'React' })   // reset
      setSuccess(true)
      setTimeout(() => setSuccess(false), 2500)    // nascondi dopo 2.5s
    } catch (err) {
      setErrMsg(err.message)         // mostra l'errore dal server
    } finally {
      setSubmitting(false)
    }
  }

  return (
    <form onSubmit={handleSubmit} className={styles.form}>
      <h2 className={styles.title}>Nuova nota</h2>

      <input
        type="text"
        name="titolo"
        placeholder="Titolo nota…"
        value={form.titolo}
        onChange={handleChange}
        className={styles.input}
        autoComplete="off"
      />

      <textarea
        name="contenuto"
        placeholder="Scrivi qui il contenuto…"
        value={form.contenuto}
        onChange={handleChange}
        rows={4}
        className={styles.textarea}
      />

      <select
        name="tag"
        value={form.tag}
        onChange={handleChange}
        className={styles.select}
      >
        {TAGS_DISPONIBILI.map(t => (
          <option key={t} value={t}>{t}</option>
        ))}
      </select>

      {/* Messaggi di feedback */}
      {errMsg  && <p className={styles.errMsg}>⚠ {errMsg}</p>}
      {success && <p className={styles.okMsg}>✓ Nota salvata con successo!</p>}

      <button type="submit" disabled={submitting} className={styles.btn}>
        {submitting ? 'Salvataggio…' : 'Aggiungi nota'}
      </button>
    </form>
  )
}
```

### 5.3 Scrivi `NoteForm.module.css`

```css
/* frontend/src/components/NoteForm.module.css */
.form {
  background: var(--surface-1);
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
}

.title {
  font-size: 1rem;
  font-weight: 600;
  color: var(--ice);
  margin: 0;
}

.input,
.textarea,
.select {
  background: var(--surface-2);
  border: 1px solid var(--border);
  border-radius: 6px;
  color: var(--ice);
  font-family: inherit;
  font-size: 0.9rem;
  padding: 0.65rem 0.9rem;
  transition: border-color 0.2s;
  width: 100%;
}

.input:focus,
.textarea:focus,
.select:focus {
  border-color: var(--blue);
  outline: none;
}

.textarea {
  resize: vertical;
  min-height: 80px;
}

.errMsg {
  color: var(--red);
  font-size: 0.85rem;
  margin: 0;
}

.okMsg {
  color: var(--green);
  font-size: 0.85rem;
  margin: 0;
}

.btn {
  background: var(--blue);
  border: none;
  border-radius: 6px;
  color: #001824;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 700;
  padding: 0.7rem 1.4rem;
  align-self: flex-start;
  transition: background 0.2s, opacity 0.2s;
}

.btn:hover:not(:disabled) { background: #33dcff; }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
```

---

## 6. Aggiornare App.jsx

Sostituisci il contenuto di `App.jsx` con la versione che usa il hook:

```jsx
// frontend/src/App.jsx
import { useNotes } from './hooks/useNotes'
import NoteList     from './components/NoteList'
import NoteForm     from './components/NoteForm'
import styles       from './App.module.css'

export default function App() {
  const { notes, loading, error, addNote, removeNote, reload } = useNotes()

  // App non conosce i dettagli di fetch: delega tutto al hook
  async function handleAdd(formData) {
    await addNote(formData)
  }

  // Stato: caricamento in corso
  if (loading) {
    return <div className={styles.loading}>⏳ Caricamento note…</div>
  }

  // Stato: errore di rete o del server
  if (error) {
    return (
      <div className={styles.errorBox}>
        <p>⚠ {error}</p>
        <button onClick={reload}>Riprova</button>
      </div>
    )
  }

  // Stato: dati caricati correttamente
  return (
    <div className={styles.layout}>
      <header className={styles.header}>
        <h1>DevNotes</h1>
        <span className={styles.counter}>{notes.length} note</span>
      </header>
      <NoteForm onAdd={handleAdd} />
      <NoteList notes={notes} onElimina={removeNote} />
    </div>
  )
}
```

Aggiungi le classi mancanti in `App.module.css`:

```css
/* frontend/src/App.module.css — aggiunte */
.layout {
  max-width: 860px;
  margin: 0 auto;
  padding: 2rem 1.25rem 4rem;
}

.header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 1.5rem;
}

.header h1 {
  font-size: 1.8rem;
  font-weight: 700;
  color: var(--ice);
}

.counter {
  color: var(--slate);
  font-size: 0.88rem;
}

.loading {
  padding: 3rem;
  text-align: center;
  color: var(--slate);
}

.errorBox {
  background: rgba(255,92,92,0.08);
  border: 1px solid var(--red);
  border-radius: 8px;
  margin: 2rem auto;
  max-width: 400px;
  padding: 1.5rem;
  text-align: center;
  color: var(--red);
}

.errorBox button {
  background: var(--red);
  border: none;
  border-radius: 5px;
  color: white;
  cursor: pointer;
  font-weight: 600;
  margin-top: 1rem;
  padding: 0.5rem 1.2rem;
}
```

---

## 7. Middleware errori in Express

### 7.1 Apri `backend/server.js`

Aggiungi i due middleware **alla fine del file**, dopo tutte le route esistenti:

```js
// backend/server.js — aggiunta in fondo al file

// ────────────────────────────────────────────
// Middleware 404 — route non trovata
// ────────────────────────────────────────────
app.use((req, res) => {
  res.status(404).json({ messaggio: 'Risorsa non trovata' })
})

// ────────────────────────────────────────────
// Middleware di errore globale
// I 4 parametri (err, req, res, next) sono OBBLIGATORI:
// senza di essi Express non lo riconosce come error handler
// ────────────────────────────────────────────
app.use((err, req, res, next) => {
  console.error('[SERVER ERROR]', err.stack)

  res
    .status(err.status || 500)
    .json({
      messaggio: err.message || 'Errore interno del server',
      // In sviluppo puoi esporre lo stack; in produzione omettilo
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
    })
})
```

### 7.2 Usa `next(err)` nelle route per propagare gli errori

Nei controller, invece di gestire gli errori in ogni route, passali al middleware:

```js
// backend/routes/notes.js — pattern consigliato
router.get('/', async (req, res, next) => {
  try {
    // ... logica ...
    res.json(notes)
  } catch (err) {
    next(err)   // propaga al middleware globale
  }
})
```

---

## 8. Avvio concorrente con concurrently

### 8.1 Crea il `package.json` nella root

```bash
# Windows / macOS / Linux — nella cartella devnotes/ (root)
cd devnotes
npm init -y
```

### 8.2 Installa concurrently

```bash
# Windows / macOS / Linux
npm install -D concurrently
```

### 8.3 Aggiungi gli script

Apri `devnotes/package.json` e sostituisci la sezione `scripts`:

```json
{
  "name": "devnotes",
  "scripts": {
    "dev": "concurrently -n \"FE,BE\" -c \"cyan,green\" \"npm run dev --prefix frontend\" \"node backend/server.js\"",
    "dev:watch": "concurrently -n \"FE,BE\" -c \"cyan,green\" \"npm run dev --prefix frontend\" \"npx nodemon backend/server.js\"",
    "build": "npm run build --prefix frontend"
  },
  "devDependencies": {
    "concurrently": "^8.0.0"
  }
}
```

### 8.4 Installa nodemon nel backend (opzionale ma consigliato)

```bash
# Windows / macOS / Linux
cd devnotes/backend
npm install -D nodemon
```

Aggiungi lo script nel `backend/package.json`:

```json
{
  "scripts": {
    "start":  "node server.js",
    "dev":    "nodemon server.js"
  }
}
```

### 8.5 Avvia tutto con un comando

```bash
# Windows / macOS / Linux — dalla root devnotes/
cd devnotes
npm run dev
```

Dovresti vedere due stream di log colorati: ciano per il frontend (Vite), verde per il backend (Express).

---

## 9. Test completo dell'integrazione

### 9.1 Apri i DevTools del browser

Vai su `http://localhost:5173`, apri **F12 → Network** e filtra per **Fetch/XHR**.

### 9.2 Test GET — caricamento note

1. Ricarica la pagina
2. Verifica che appaia una richiesta `GET /api/notes` → **200 OK**
3. Nella tab **Response** vedi l'array JSON delle note

### 9.3 Test POST — creazione nota

1. Compila il form con titolo, contenuto e tag
2. Clicca "Aggiungi nota"
3. Verifica la richiesta `POST /api/notes` → **201 Created**
4. La nuova nota appare in cima alla lista senza refresh

### 9.4 Test DELETE — eliminazione nota

1. Clicca il bottone elimina su una nota
2. Verifica `DELETE /api/notes/:id` → **200 OK** o **204 No Content**
3. La nota sparisce dalla lista

### 9.5 Test gestione errore — backend spento

1. Ferma il backend con `Ctrl+C` nel terminale dedicato
2. Ricarica la pagina
3. Verifica che appaia il componente di errore con messaggio e bottone "Riprova"
4. Riavvia il backend, clicca "Riprova" → le note si ricaricano

### 9.6 Test validazione — form vuoto

1. Lascia il titolo vuoto
2. Clicca "Aggiungi nota"
3. Verifica che appaia il messaggio di errore client-side senza che parta alcuna richiesta network

---

## 10. Esercizi di consolidamento

### Esercizio A — Modifica ottimistica con rollback ⭐⭐

**Obiettivo:** migliorare la UX dell'eliminazione.

**Attività:**

Modifica `removeNote` nel hook `useNotes` per implementare il rollback in caso di errore:

```js
const removeNote = useCallback(async (id) => {
  // Salva lo stato precedente
  const prev = notes  // ATTENZIONE: è uno snapshot, non la ref!
  
  // 1. Aggiorna IMMEDIATAMENTE la UI (ottimistico)
  setNotes(current => current.filter(n => n.id !== id))
  
  try {
    // 2. Chiama il backend
    await deleteNote(id)
  } catch (err) {
    // 3. Se fallisce, ripristina lo stato precedente
    setNotes(prev)
    setError(err.message)
  }
}, [notes])
```

**Testa che:**
- La nota sparisce subito dalla UI
- Se simuli un errore (es. modificando temporaneamente l'URL nel modulo API), la nota ritorna

---

### Esercizio B — Filtro note per tag ⭐⭐

**Obiettivo:** aggiungere un filtro visivo senza toccare il backend.

**Attività:**

1. In `App.jsx` aggiungi uno stato `filtroTag` inizializzato a `'Tutti'`
2. Calcola le note filtrate con `useMemo`:

```js
import { useMemo } from 'react'

const noteFiltrate = useMemo(() =>
  filtroTag === 'Tutti'
    ? notes
    : notes.filter(n => n.tag === filtroTag),
  [notes, filtroTag]
)
```

3. Mostra i bottoni filtro — uno per ogni tag unico presente nelle note più "Tutti":

```jsx
{['Tutti', ...new Set(notes.map(n => n.tag))].map(tag => (
  <button
    key={tag}
    onClick={() => setFiltroTag(tag)}
    style={{ fontWeight: filtroTag === tag ? 700 : 400 }}
  >
    {tag}
  </button>
))}
```

4. Passa `noteFiltrate` (invece di `notes`) a `<NoteList />`

**Requisiti:** il contatore nell'header deve mostrare il numero di note filtrate, non il totale.

---

### Esercizio C — Persistenza con localStorage come fallback ⭐⭐⭐

**Obiettivo:** se il backend non è raggiungibile, mostrare le ultime note dalla cache locale.

**Attività:**

Modifica `useNotes.js` per salvare le note nel `localStorage` ogni volta che vengono caricate con successo:

```js
async function loadNotes() {
  try {
    setLoading(true)
    setError(null)
    const data = await fetchNotes()
    setNotes(data)
    // Salva in cache
    localStorage.setItem('devnotes_cache', JSON.stringify(data))
  } catch (err) {
    // Prova a usare la cache
    const cache = localStorage.getItem('devnotes_cache')
    if (cache) {
      setNotes(JSON.parse(cache))
      setError('Backend non raggiungibile — visualizzazione dalla cache locale')
    } else {
      setError(err.message)
    }
  } finally {
    setLoading(false)
  }
}
```

**Testa che:**
1. Con il backend attivo, carica e salva la cache
2. Ferma il backend
3. Ricarica la pagina: vedi le note dalla cache con il messaggio di avviso
4. Il messaggio di errore deve essere giallo (warning), non rosso (errore critico)

---

## 11. Checklist finale

Verifica di aver completato tutti i passi prima di procedere alla L09:

- [ ] `vite.config.js` contiene `server.proxy` con `target: 'http://localhost:3001'`
- [ ] Riavviando Vite il proxy funziona: `localhost:5173/api/notes` restituisce JSON
- [ ] `src/api/notesApi.js` esporta `fetchNotes`, `createNote`, `deleteNote`
- [ ] Tutte le funzioni API controllano `res.ok` prima di chiamare `res.json()`
- [ ] `src/hooks/useNotes.js` gestisce gli stati `loading`, `error`, `notes`
- [ ] `NoteForm.jsx` ha validazione client, stato `submitting`, messaggi di feedback
- [ ] `App.jsx` usa `useNotes` e mostra spinner / errore / contenuto in base agli stati
- [ ] Il middleware errore Express ha 4 parametri e risponde con JSON strutturato
- [ ] `concurrently` è installato e il comando `npm run dev` dalla root avvia entrambi
- [ ] Test GET, POST, DELETE verificati con DevTools → tutti 200/201 ✅
- [ ] Test "backend spento": compare errore con bottone "Riprova" ✅
- [ ] Test "form vuoto": validazione client blocca la richiesta ✅

---

## 📊 Tabella riepilogo concetti chiave

| Concetto | Dove | Perché |
|---|---|---|
| `server.proxy` | `vite.config.js` | Aggira Same-Origin Policy in sviluppo |
| `if (!res.ok) throw` | `notesApi.js` | `fetch()` non lancia per 4xx/5xx |
| `try/catch/finally` | `useNotes.js`, `NoteForm.jsx` | Gestione uniforme degli errori async |
| `useCallback` | `useNotes.js` | Evita ricreazioni inutili delle funzioni |
| Aggiornamento ottimistico | `addNote`, `removeNote` | UI istantanea senza aspettare il server |
| Middleware errore (4 param) | `backend/server.js` | Centralizza la gestione degli errori Express |
| `concurrently` | `devnotes/package.json` | Avvia più processi con un comando solo |
| Stato `submitting` | `NoteForm.jsx` | Impedisce doppio invio durante la richiesta |

---

## ➡️ Prossimo laboratorio

[Lab 09 — Autenticazione base con JWT](lab-09-auth-jwt.md)

Aggiungeremo autenticazione JWT al backend Express, proteggendo le route con un middleware, e implementeremo il flusso login/logout/token nel frontend React con `localStorage` e route protette.
