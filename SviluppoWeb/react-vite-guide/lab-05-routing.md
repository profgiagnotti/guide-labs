# Lab 05 — Routing di DevNotes con React Router v6

> **Serie:** React + Vite + Node.js — Guida completa  
> **Repo:** `profgiagnotti/react-vite-guide`  
> **File:** `lab-05-routing.md`

---

## Obiettivi

- [ ] Installare `react-router-dom` e configurare `BrowserRouter` in `main.jsx`
- [ ] Ristrutturare il progetto con cartella `pages/`
- [ ] Creare un `Layout` condiviso con navbar e `<Outlet />`
- [ ] Implementare la pagina di dettaglio nota con URL parametrico `/note/:id`
- [ ] Usare `NavLink` per evidenziare la voce di menu attiva
- [ ] Creare una pagina 404 con redirect automatico
- [ ] Implementare una route protetta simulata
- [ ] Navigare programmaticamente con `useNavigate`

---

## Prerequisiti

- Lab 04 completato (Tailwind installato e configurato, componenti styled)
- Node.js ≥ 18 installato
- `npm run dev` parte senza errori

---

## Indice

1. [Installazione react-router-dom](#1-installazione-react-router-dom)
2. [Ristruttura cartelle — pages/](#2-ristruttura-cartelle--pages)
3. [Configura BrowserRouter in main.jsx](#3-configura-browserrouter-in-mainjsx)
4. [Crea il Layout con navbar e Outlet](#4-crea-il-layout-con-navbar-e-outlet)
5. [Pagina Home](#5-pagina-home)
6. [Context globale per le note](#6-context-globale-per-le-note)
7. [Pagina NoteDetail con useParams](#7-pagina-notedetail-con-useparams)
8. [Pagina Impostazioni](#8-pagina-impostazioni)
9. [Pagina 404 Not Found](#9-pagina-404-not-found)
10. [Route protetta simulata](#10-route-protetta-simulata)
11. [Naviga da NoteCard con useNavigate](#11-naviga-da-notecard-con-usenavigate)
12. [Aggiorna App.jsx con le Routes](#12-aggiorna-appjsx-con-le-routes)
13. [Esercizi di consolidamento](#13-esercizi-di-consolidamento)
14. [Checklist finale](#14-checklist-finale)

---

## 1. Installazione react-router-dom

```bash
# dalla root del progetto
npm install react-router-dom
```

Verifica in `package.json` che sia presente in `dependencies` (non `devDependencies`):

```json
"dependencies": {
  "react": "^18.x.x",
  "react-dom": "^18.x.x",
  "react-router-dom": "^6.x.x"
}
```

---

## 2. Ristruttura cartelle — pages/

Crea la cartella `src/pages/` e sposta/crea i file:

```
src/
├── components/
│   ├── Header.jsx          ← ora dentro Layout, puoi rimuoverlo
│   ├── Layout.jsx          ← NUOVO: navbar + Outlet
│   ├── NoteCard.jsx
│   ├── NoteForm.jsx
│   ├── NoteList.jsx
│   └── ProtectedRoute.jsx  ← NUOVO: wrapper route protette
├── context/
│   └── NoteContext.jsx     ← NUOVO: stato globale note
├── pages/
│   ├── Home.jsx            ← NUOVO: pagina principale
│   ├── NoteDetail.jsx      ← NUOVO: dettaglio nota
│   ├── Impostazioni.jsx    ← NUOVO: pagina impostazioni
│   └── NotFound.jsx        ← NUOVO: pagina 404
├── App.jsx                 ← da aggiornare
├── index.css
└── main.jsx                ← da aggiornare
```

### Windows

```powershell
mkdir src\pages
mkdir src\context
New-Item src\pages\Home.jsx -ItemType File
New-Item src\pages\NoteDetail.jsx -ItemType File
New-Item src\pages\Impostazioni.jsx -ItemType File
New-Item src\pages\NotFound.jsx -ItemType File
New-Item src\context\NoteContext.jsx -ItemType File
New-Item src\components\Layout.jsx -ItemType File
New-Item src\components\ProtectedRoute.jsx -ItemType File
```

### macOS / Linux

```bash
mkdir -p src/pages src/context
touch src/pages/{Home,NoteDetail,Impostazioni,NotFound}.jsx
touch src/context/NoteContext.jsx
touch src/components/Layout.jsx
touch src/components/ProtectedRoute.jsx
```

---

## 3. Configura BrowserRouter in main.jsx

```jsx
// src/main.jsx
import React    from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'  // ← aggiungi
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>   {/* ← avvolgi App */}
      <App />
    </BrowserRouter>
  </React.StrictMode>,
)
```

> **Perché in `main.jsx` e non in `App.jsx`?** `BrowserRouter` deve essere il componente più esterno. Metterlo in `main.jsx` garantisce che anche il root sia coperto dal contesto router.

---

## 4. Crea il Layout con navbar e Outlet

```jsx
// src/components/Layout.jsx
import { Outlet, NavLink } from 'react-router-dom'

function Layout() {

  // NavLink riceve { isActive } come argomento di className
  const navClasse = ({ isActive }) =>
    isActive
      ? 'font-mono text-[12px] text-cyan-400 border-b border-cyan-400 pb-0.5'
      : 'font-mono text-[12px] text-[#6B7A99] hover:text-white transition-colors'

  return (
    <div className="max-w-5xl mx-auto px-4 py-8">

      {/* ── Navbar condivisa */}
      <nav className="
        flex items-center justify-between
        border-b border-border pb-5 mb-10
      ">
        {/* Logo — NavLink con end per non essere sempre "attivo" */}
        <NavLink
          to="/"
          className="font-serif text-xl font-black text-white hover:text-cyan-400 transition-colors"
        >
          📝 DevNotes
        </NavLink>

        {/* Voci di menu */}
        <div className="flex items-center gap-6">
          <NavLink to="/" end className={navClasse}>
            note
          </NavLink>
          <NavLink to="/impostazioni" className={navClasse}>
            impostazioni
          </NavLink>
        </div>
      </nav>

      {/* ── Outlet: React Router monta qui la pagina corrente */}
      <main>
        <Outlet />
      </main>

    </div>
  )
}

export default Layout
```

---

## 5. Pagina Home

```jsx
// src/pages/Home.jsx
import { useContext } from 'react'
import { NoteContext } from '../context/NoteContext'
import NoteForm from '../components/NoteForm'
import NoteList from '../components/NoteList'

function Home() {
  const { note, aggiungiNota, eliminaNota } = useContext(NoteContext)

  return (
    <>
      {/* ── intestazione pagina */}
      <div className="mb-6">
        <h1 className="font-serif text-2xl font-black text-white">
          Le tue note
        </h1>
        <p className="font-mono text-[11px] text-[#6B7A99] mt-1">
          {note.length} {note.length === 1 ? 'nota salvata' : 'note salvate'}
        </p>
      </div>

      {/* ── form creazione */}
      <NoteForm onAggiungi={aggiungiNota} />

      {/* ── lista note */}
      <NoteList note={note} onElimina={eliminaNota} />
    </>
  )
}

export default Home
```

---

## 6. Context globale per le note

Invece di tenere lo stato in `App.jsx` e passarlo via props, usiamo il Context API di React per renderlo accessibile a qualsiasi componente.

```jsx
// src/context/NoteContext.jsx
import { createContext, useState } from 'react'

// ① crea il context
export const NoteContext = createContext(null)

// ② crea il Provider — avvolgerà l'app in App.jsx
export function NoteProvider({ children }) {

  const [note, setNote] = useState([
    {
      id: 1,
      titolo: 'useState — anatomia',
      testo:  'const [state, setState] = useState(initialValue)',
      tag:    'react',
      data:   new Date().toISOString(),
    },
    {
      id: 2,
      titolo: 'Arrow function',
      testo:  'Sintassi concisa: const fn = (x) => x * 2',
      tag:    'js',
      data:   new Date().toISOString(),
    },
    {
      id: 3,
      titolo: 'npm scripts utili',
      testo:  'npm run dev | npm run build | npm run preview',
      tag:    'node',
      data:   new Date().toISOString(),
    },
  ])

  function aggiungiNota(dati) {
    const nuova = { ...dati, id: Date.now(), data: new Date().toISOString() }
    setNote(prev => [nuova, ...prev])
    return nuova  // ← utile per navigate dopo submit
  }

  function eliminaNota(id) {
    setNote(prev => prev.filter(n => n.id !== id))
  }

  // ③ valore esposto a tutti i componenti dentro il Provider
  const valore = { note, aggiungiNota, eliminaNota }

  return (
    <NoteContext.Provider value={valore}>
      {children}
    </NoteContext.Provider>
  )
}
```

---

## 7. Pagina NoteDetail con useParams

```jsx
// src/pages/NoteDetail.jsx
import { useParams, Link } from 'react-router-dom'
import { useContext }      from 'react'
import { NoteContext }     from '../context/NoteContext'

function NoteDetail() {

  // ① leggi il parametro :id dall'URL — è SEMPRE una stringa
  const { id } = useParams()

  // ② cerca la nota con conversione esplicita a numero
  const { note, eliminaNota } = useContext(NoteContext)
  const nota = note.find(n => n.id === Number(id))

  // ③ nota non trovata → messaggio con link home
  if (!nota) {
    return (
      <div className="flex flex-col items-center justify-center py-24 gap-4">
        <span className="text-5xl opacity-20">🗒️</span>
        <p className="font-mono text-sm text-[#6B7A99]">
          Nota #{id} non trovata.
        </p>
        <Link
          to="/"
          className="font-mono text-[12px] text-cyan-400 hover:underline"
        >
          ← Torna alle note
        </Link>
      </div>
    )
  }

  // ④ formatta la data
  const dataFormattata = new Date(nota.data).toLocaleDateString('it-IT', {
    weekday: 'long', day: 'numeric', month: 'long', year: 'numeric',
  })

  return (
    <article className="max-w-2xl">

      {/* breadcrumb */}
      <Link
        to="/"
        className="font-mono text-[11px] text-[#6B7A99] hover:text-white transition-colors mb-6 inline-block"
      >
        ← tutte le note
      </Link>

      {/* header */}
      <h1 className="font-serif text-3xl font-black text-white leading-tight mb-3">
        {nota.titolo}
      </h1>

      <div className="flex items-center gap-3 mb-8">
        <span className="font-mono text-[11px] text-[#6B7A99]">
          {dataFormattata}
        </span>
        <span className="font-mono text-[11px] px-2 py-0.5 rounded bg-cyan-900 text-cyan-400 border border-cyan-400/30">
          {nota.tag}
        </span>
      </div>

      {/* contenuto */}
      <div className="bg-surface border border-border rounded-card p-6 mb-8">
        <p className="text-[#A8B8D0] leading-relaxed whitespace-pre-wrap">
          {nota.testo || 'Nessun contenuto aggiuntivo.'}
        </p>
      </div>

      {/* azioni */}
      <div className="flex items-center gap-4">
        <Link
          to="/"
          className="font-mono text-[12px] text-[#6B7A99] hover:text-white transition-colors"
        >
          ← Indietro
        </Link>
        <button
          onClick={() => eliminaNota(nota.id)}
          className="font-mono text-[12px] text-red-400 hover:text-white transition-colors"
        >
          Elimina nota
        </button>
      </div>

    </article>
  )
}

export default NoteDetail
```

---

## 8. Pagina Impostazioni

```jsx
// src/pages/Impostazioni.jsx
function Impostazioni() {
  return (
    <div className="max-w-lg">
      <h1 className="font-serif text-2xl font-black text-white mb-2">
        Impostazioni
      </h1>
      <p className="font-mono text-[11px] text-[#6B7A99] mb-8">
        Configura DevNotes secondo le tue preferenze.
      </p>

      {/* sezione 1 */}
      <section className="bg-surface border border-border rounded-card p-5 mb-4">
        <h2 className="font-mono text-[12px] text-cyan-400 uppercase tracking-wider mb-4">
          // Interfaccia
        </h2>
        <div className="flex items-center justify-between">
          <div>
            <p className="text-sm text-white font-semibold">Tema</p>
            <p className="font-mono text-[11px] text-[#6B7A99]">Modalità scura attiva</p>
          </div>
          <span className="font-mono text-[11px] bg-green-950 text-green-400 border border-green-400/30 px-2 py-0.5 rounded">
            dark ✓
          </span>
        </div>
      </section>

      {/* sezione 2 */}
      <section className="bg-surface border border-border rounded-card p-5">
        <h2 className="font-mono text-[12px] text-cyan-400 uppercase tracking-wider mb-4">
          // Dati
        </h2>
        <p className="text-sm text-[#A8B8D0] mb-4">
          Le note sono salvate in memoria. Chiudendo il browser i dati vengono persi.
          Nel Lab 07 aggiungeremo un backend Node.js con persistenza.
        </p>
        <div className="font-mono text-[11px] bg-[#0A0D14] border border-border rounded px-4 py-3 text-[#6B7A99]">
          storage: in-memory (React state) · backend: coming soon in L07
        </div>
      </section>
    </div>
  )
}

export default Impostazioni
```

---

## 9. Pagina 404 Not Found

```jsx
// src/pages/NotFound.jsx
import { Link, useLocation } from 'react-router-dom'

function NotFound() {
  // useLocation restituisce l'oggetto location corrente
  const location = useLocation()

  return (
    <div className="flex flex-col items-center justify-center py-24 text-center gap-4">

      <div className="font-mono text-7xl font-black text-[#1E2535] mb-2">
        404
      </div>

      <h1 className="font-serif text-2xl font-black text-white">
        Pagina non trovata
      </h1>

      <p className="font-mono text-[12px] text-[#6B7A99]">
        La route <span className="text-red-400">{location.pathname}</span> non esiste.
      </p>

      <Link
        to="/"
        className="
          mt-4 font-mono text-sm text-cyan-400
          border border-cyan-400/40 rounded px-5 py-2
          hover:bg-cyan-900 transition-colors
        "
      >
        ← Torna alla Home
      </Link>

    </div>
  )
}

export default NotFound
```

---

## 10. Route protetta simulata

```jsx
// src/components/ProtectedRoute.jsx
// In questo lab simuliamo l'auth con una variabile.
// Nel Lab 09 userai JWT reali.

import { Navigate, Outlet } from 'react-router-dom'

// simula: true = loggato, false = non loggato
const IS_LOGGED_IN = true

function ProtectedRoute() {
  if (!IS_LOGGED_IN) {
    // redirect alla home se non autenticato
    // replace: true → non aggiunge entry nella history
    return <Navigate to="/" replace />
  }

  // autenticato: renderizza la route figlia
  return <Outlet />
}

export default ProtectedRoute
```

**Per testare il redirect:** cambia `IS_LOGGED_IN` in `false`, poi naviga su `/impostazioni`. Dovresti essere reindirizzato automaticamente a `/`.

---

## 11. Naviga da NoteCard con useNavigate

Aggiorna `NoteCard.jsx` per navigare alla pagina di dettaglio quando l'utente clicca sul titolo:

```jsx
// src/components/NoteCard.jsx
// aggiungi l'import di useNavigate
import { useNavigate } from 'react-router-dom'

function NoteCard({ note, onElimina }) {
  const navigate = useNavigate()

  // ...resto del codice invariato...

  return (
    <div className="bg-surface border border-border rounded-card p-5 flex flex-col gap-3 hover:border-cyan-400/50 transition-colors duration-200">

      <div className="flex items-start justify-between gap-2">
        {/* ← il titolo ora è cliccabile */}
        <h3
          onClick={() => navigate(`/note/${note.id}`)}
          className="
            text-base font-bold text-white leading-snug line-clamp-2 flex-1
            cursor-pointer hover:text-cyan-400 transition-colors
          "
        >
          {note.titolo}
        </h3>
        <span className={`font-mono text-[11px] px-2 py-0.5 rounded whitespace-nowrap ${tagClasse}`}>
          {note.tag}
        </span>
      </div>

      {/* ...resto invariato */}
    </div>
  )
}
```

---

## 12. Aggiorna App.jsx con le Routes

```jsx
// src/App.jsx — versione con routing completo
import { Routes, Route }  from 'react-router-dom'
import { NoteProvider }   from './context/NoteContext'
import Layout             from './components/Layout'
import ProtectedRoute     from './components/ProtectedRoute'
import Home               from './pages/Home'
import NoteDetail         from './pages/NoteDetail'
import Impostazioni       from './pages/Impostazioni'
import NotFound           from './pages/NotFound'

function App() {
  return (
    // NoteProvider avvolge tutto: le note sono accessibili ovunque
    <NoteProvider>
      <Routes>

        {/* Layout condiviso con navbar */}
        <Route path="/" element={<Layout />}>

          {/* index → route "/" esatta */}
          <Route index element={<Home />} />

          {/* route protette */}
          <Route element={<ProtectedRoute />}>
            <Route path="note/:id"    element={<NoteDetail />} />
            <Route path="impostazioni" element={<Impostazioni />} />
          </Route>

        </Route>

        {/* 404 — matcha qualsiasi URL non gestito sopra */}
        <Route path="*" element={<NotFound />} />

      </Routes>
    </NoteProvider>
  )
}

export default App
```

---

## 13. Esercizi di consolidamento

### A — Query string con useSearchParams (medio)

In `Home.jsx`, aggiungi un campo di ricerca che sincronizza il valore con il query string dell'URL (`?q=react`). Quando ricarichi la pagina, la ricerca deve essere già attiva.

```jsx
// hint — useSearchParams
import { useSearchParams } from 'react-router-dom'

function Home() {
  const [searchParams, setSearchParams] = useSearchParams()
  const query = searchParams.get('q') ?? ''

  // nel campo di ricerca
  const handleSearch = (e) => {
    const valore = e.target.value
    if (valore) {
      setSearchParams({ q: valore })
    } else {
      setSearchParams({})  // rimuove il query string
    }
  }

  const noteFiltrate = note.filter(n =>
    n.titolo.toLowerCase().includes(query.toLowerCase())
  )
  // ...
}
```

### B — Redirect dopo eliminazione (medio)

In `NoteDetail.jsx`: quando l'utente clicca "Elimina nota", dopo l'eliminazione deve essere reindirizzato alla Home. Usa `useNavigate` con `navigate('/', { replace: true })` (non aggiunge la pagina di dettaglio nella history).

### C — Stato passato con navigate (avanzato)

React Router permette di passare dati con la navigazione via `state`. In `Home.jsx`, dopo aver aggiunto una nota con `aggiungiNota`, naviga al dettaglio e passa un messaggio:

```jsx
// invio
navigate(`/note/${nuovaNota.id}`, { state: { nuova: true } })

// ricezione — in NoteDetail.jsx
import { useLocation } from 'react-router-dom'
const location = useLocation()
const { nuova } = location.state ?? {}

// mostra un banner se è appena stata creata
{nuova && (
  <div className="bg-green-950 border border-green-400/30 rounded p-3 mb-4 font-mono text-[12px] text-green-400">
    ✓ Nota creata con successo!
  </div>
)}
```

---

## 14. Checklist finale

- [ ] `npm run dev` funziona senza errori
- [ ] La navbar mostra "note" evidenziato in ciano quando sei su `/`
- [ ] La navbar mostra "impostazioni" evidenziato su `/impostazioni`
- [ ] Cliccare il titolo di una NoteCard naviga su `/note/:id` (senza reload)
- [ ] La pagina di dettaglio mostra il titolo, il testo e il tag della nota corretta
- [ ] Il link "← tutte le note" in NoteDetail torna alla Home
- [ ] Navigando su un URL inesistente (es. `/ciao`) appare la pagina 404 con il pathname rosso
- [ ] Cambiando `IS_LOGGED_IN = false` in `ProtectedRoute.jsx`, navigare su `/impostazioni` redirige a `/`
- [ ] Il pulsante "Elimina nota" nella pagina di dettaglio rimuove la nota

### Tabella riepilogo file modificati/creati

| File | Azione | Ruolo |
|------|--------|-------|
| `src/main.jsx` | Modificato | Aggiunto `<BrowserRouter>` |
| `src/App.jsx` | Sostituito | `<Routes>`, `<Route>`, `<NoteProvider>` |
| `src/context/NoteContext.jsx` | Creato | Stato globale note con Context API |
| `src/components/Layout.jsx` | Creato | Navbar con `NavLink` + `<Outlet />` |
| `src/components/ProtectedRoute.jsx` | Creato | Wrapper route protette |
| `src/components/NoteCard.jsx` | Modificato | `useNavigate` per click titolo |
| `src/pages/Home.jsx` | Creato | Pagina principale |
| `src/pages/NoteDetail.jsx` | Creato | Dettaglio con `useParams` |
| `src/pages/Impostazioni.jsx` | Creato | Pagina impostazioni |
| `src/pages/NotFound.jsx` | Creato | Pagina 404 con `useLocation` |

---

➡️ **Prossimo lab:** [Lab 06 — Lavorare con le API REST](lab-06-api-rest.md)
