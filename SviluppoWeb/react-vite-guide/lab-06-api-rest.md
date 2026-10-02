# Lab 06 — Lavorare con le API REST

> **Serie:** React + Vite + Node.js — Guida completa  
> **Repository:** `profgiagnotti/react-vite-guide`  
> **File:** `lab-06-api-rest.md`

---

## ✅ Obiettivi

- [ ] Creare la cartella `src/hooks/` con il custom hook `useFetch`
- [ ] Creare la cartella `src/services/` con il file `notesApi.js`
- [ ] Aggiungere i componenti `LoadingSpinner` e `ErrorMessage`
- [ ] Collegare la pagina `Home` all'API JSONPlaceholder tramite `useFetch`
- [ ] Aggiornare `NoteDetail` per caricare la nota tramite ID dall'API
- [ ] Gestire correttamente i quattro stati: loading / data / error / empty

---

## 📋 Prerequisiti

- Lab 05 completato: routing funzionante con `NoteContext`, pagine `Home`, `NoteDetail`, `Impostazioni`, `NotFound`
- Node.js ≥ 18 installato
- DevNotes in esecuzione con `npm run dev`
- Connessione internet (per le chiamate a JSONPlaceholder)

---

## 📑 Indice

1. [Struttura cartelle aggiornata](#1-struttura-cartelle-aggiornata)
2. [Custom hook useFetch](#2-custom-hook-usefetch)
3. [Servizio API notesApi.js](#3-servizio-api-notesapijs)
4. [Componenti UI per loading ed errore](#4-componenti-ui-per-loading-ed-errore)
5. [Aggiornare Home con useFetch](#5-aggiornare-home-con-usefetch)
6. [Aggiornare NoteDetail](#6-aggiornare-notedetail)
7. [Variabile d'ambiente per la BASE_URL](#7-variabile-dambiente-per-la-base_url)
8. [Esercizi di consolidamento](#8-esercizi-di-consolidamento)
9. [Checklist finale](#9-checklist-finale)

---

## 1. Struttura cartelle aggiornata

Dopo questo lab la struttura di `src/` sarà:

```
src/
├── components/
│   ├── Header/
│   ├── NoteCard/
│   ├── NoteList/
│   ├── NoteForm/
│   ├── LoadingSpinner/       ← NUOVO
│   │   ├── LoadingSpinner.jsx
│   │   └── LoadingSpinner.module.css
│   └── ErrorMessage/         ← NUOVO
│       ├── ErrorMessage.jsx
│       └── ErrorMessage.module.css
├── hooks/                    ← NUOVO
│   └── useFetch.js
├── services/                 ← NUOVO
│   └── notesApi.js
├── pages/
│   ├── Home.jsx
│   ├── NoteDetail.jsx
│   ├── Impostazioni.jsx
│   └── NotFound.jsx
├── context/
│   └── NoteContext.jsx
├── App.jsx
├── main.jsx
└── index.css
```

Crea le nuove cartelle:

**Windows (PowerShell):**
```powershell
mkdir src\hooks
mkdir src\services
mkdir src\components\LoadingSpinner
mkdir src\components\ErrorMessage
```

**macOS / Linux:**
```bash
mkdir -p src/hooks src/services src/components/LoadingSpinner src/components/ErrorMessage
```

---

## 2. Custom hook useFetch

Crea `src/hooks/useFetch.js`:

```javascript
// src/hooks/useFetch.js
import { useState, useEffect } from 'react';

/**
 * Hook per eseguire chiamate HTTP GET con gestione automatica
 * di loading, data ed error.
 *
 * @param {string|null} url - URL da cui caricare i dati. Se null, non esegue la chiamata.
 * @returns {{ data: any, loading: boolean, error: string|null, refetch: () => void }}
 */
export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  // refetchIndex permette di rilanciare la chiamata manualmente
  const [refetchIndex, setRefetchIndex] = useState(0);

  const refetch = () => setRefetchIndex((i) => i + 1);

  useEffect(() => {
    if (!url) return;

    // AbortController: cancella la richiesta se il componente si smonta
    // o se url/refetchIndex cambiano prima della risposta
    const controller = new AbortController();

    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url, { signal: controller.signal });

        // fetch NON rigetta per errori HTTP (4xx, 5xx): controlla response.ok
        if (!response.ok) {
          throw new Error(`Errore HTTP ${response.status}: ${response.statusText}`);
        }

        const json = await response.json();
        setData(json);
      } catch (err) {
        // Ignora AbortError: non è un vero errore, è il cleanup normale
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        // finally garantisce che loading venga azzerato anche in caso di errore
        setLoading(false);
      }
    };

    fetchData();

    // Cleanup: eseguito quando url, refetchIndex cambiano o il componente si smonta
    return () => controller.abort();
  }, [url, refetchIndex]);

  return { data, loading, error, refetch };
}
```

**Verifica:** nel file non ci sono importazioni inutilizzate e le dipendenze di `useEffect` includono tutti i valori esterni usati al suo interno (`url` e `refetchIndex`).

---

## 3. Servizio API notesApi.js

Crea `src/services/notesApi.js`:

```javascript
// src/services/notesApi.js
// Centralizza tutte le chiamate verso l'API delle note.
// In questo lab usiamo JSONPlaceholder; in L08 sostituiremo BASE_URL
// con il nostro server Express locale.

const BASE_URL = import.meta.env.VITE_API_URL ?? 'https://jsonplaceholder.typicode.com';

/**
 * Funzione helper: verifica response.ok e lancia un errore in caso contrario.
 */
async function handleResponse(response) {
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }
  return response.json();
}

export const notesApi = {
  /** Recupera tutte le note (usiamo /posts come proxy di /notes) */
  getAll: () =>
    fetch(`${BASE_URL}/posts`).then(handleResponse),

  /** Recupera una nota per ID */
  getById: (id) =>
    fetch(`${BASE_URL}/posts/${id}`).then(handleResponse),

  /** Crea una nuova nota */
  create: (data) =>
    fetch(`${BASE_URL}/posts`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    }).then(handleResponse),

  /** Aggiorna una nota (sostituzione completa) */
  update: (id, data) =>
    fetch(`${BASE_URL}/posts/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    }).then(handleResponse),

  /** Elimina una nota */
  remove: (id) =>
    fetch(`${BASE_URL}/posts/${id}`, { method: 'DELETE' }),
};
```

---

## 4. Componenti UI per loading ed errore

### 4.1 LoadingSpinner

Crea `src/components/LoadingSpinner/LoadingSpinner.jsx`:

```jsx
// src/components/LoadingSpinner/LoadingSpinner.jsx
import styles from './LoadingSpinner.module.css';

export default function LoadingSpinner({ message = 'Caricamento…' }) {
  return (
    <div className={styles.wrapper}>
      <div className={styles.ring} aria-hidden="true" />
      <p className={styles.text}>{message}</p>
    </div>
  );
}
```

Crea `src/components/LoadingSpinner/LoadingSpinner.module.css`:

```css
/* LoadingSpinner.module.css */
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  padding: 60px 20px;
}

.ring {
  width: 40px;
  height: 40px;
  border: 3px solid var(--border);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

.text {
  font-family: 'JetBrains Mono', monospace;
  font-size: 13px;
  color: var(--text-muted);
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

Assicurati che `index.css` contenga la variabile `--color-primary: #00D4FF;` (già presente dal lab 02).

---

### 4.2 ErrorMessage

Crea `src/components/ErrorMessage/ErrorMessage.jsx`:

```jsx
// src/components/ErrorMessage/ErrorMessage.jsx
import styles from './ErrorMessage.module.css';

export default function ErrorMessage({ message, onRetry }) {
  return (
    <div className={styles.wrapper} role="alert">
      <div className={styles.icon}>⚠️</div>
      <p className={styles.message}>{message}</p>
      {onRetry && (
        <button className={styles.retryBtn} onClick={onRetry}>
          Riprova
        </button>
      )}
    </div>
  );
}
```

Crea `src/components/ErrorMessage/ErrorMessage.module.css`:

```css
/* ErrorMessage.module.css */
.wrapper {
  background: #2A0A0A;
  border: 1px solid #FF5C5C;
  border-radius: 8px;
  padding: 24px;
  text-align: center;
  max-width: 480px;
  margin: 40px auto;
}

.icon {
  font-size: 2rem;
  margin-bottom: 12px;
}

.message {
  font-family: 'JetBrains Mono', monospace;
  font-size: 13px;
  color: #FF5C5C;
  margin-bottom: 16px;
}

.retryBtn {
  background: transparent;
  border: 1px solid #FF5C5C;
  color: #FF5C5C;
  font-family: 'JetBrains Mono', monospace;
  font-size: 12px;
  padding: 8px 20px;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.2s;
}

.retryBtn:hover {
  background: rgba(255, 92, 92, 0.12);
}
```

---

## 5. Aggiornare Home con useFetch

Modifica `src/pages/Home.jsx` per caricare le note dall'API anziché dal context locale:

```jsx
// src/pages/Home.jsx
import { useFetch } from '../hooks/useFetch';
import NoteList from '../components/NoteList/NoteList';
import LoadingSpinner from '../components/LoadingSpinner/LoadingSpinner';
import ErrorMessage from '../components/ErrorMessage/ErrorMessage';

const API_URL = 'https://jsonplaceholder.typicode.com/posts?_limit=12';

export default function Home() {
  const { data: notes, loading, error, refetch } = useFetch(API_URL);

  // Stato 1: loading
  if (loading) return <LoadingSpinner message="Caricamento note…" />;

  // Stato 2: errore
  if (error) return <ErrorMessage message={error} onRetry={refetch} />;

  // Stato 3: lista vuota
  if (!notes?.length) {
    return (
      <p style={{ color: 'var(--text-muted)', textAlign: 'center', marginTop: '60px' }}>
        Nessuna nota trovata.
      </p>
    );
  }

  // Stato 4: dati presenti
  // Mappiamo i campi di JSONPlaceholder ai campi che si aspetta NoteCard
  const noteMappate = notes.map((post) => ({
    id: post.id,
    titolo: post.title,
    contenuto: post.body,
    tag: 'lavoro',
    data: new Date().toISOString(),
  }));

  return <NoteList note={noteMappate} />;
}
```

**Nota:** JSONPlaceholder restituisce 100 post; l'URL usa `?_limit=12` per limitare i risultati a 12 durante lo sviluppo.

---

## 6. Aggiornare NoteDetail

Modifica `src/pages/NoteDetail.jsx` per caricare la nota dall'API tramite il parametro URL `id`:

```jsx
// src/pages/NoteDetail.jsx
import { useParams, useNavigate } from 'react-router-dom';
import { useFetch } from '../hooks/useFetch';
import LoadingSpinner from '../components/LoadingSpinner/LoadingSpinner';
import ErrorMessage from '../components/ErrorMessage/ErrorMessage';

export default function NoteDetail() {
  const { id } = useParams();
  const navigate = useNavigate();

  // L'URL dipende dall'id: useFetch riesegue la chiamata ogni volta che id cambia
  const { data: post, loading, error, refetch } = useFetch(
    `https://jsonplaceholder.typicode.com/posts/${id}`
  );

  if (loading) return <LoadingSpinner message={`Caricamento nota #${id}…`} />;
  if (error) return <ErrorMessage message={error} onRetry={refetch} />;
  if (!post) return null;

  return (
    <div style={{ maxWidth: '720px', margin: '0 auto', padding: '32px 16px' }}>
      <button
        onClick={() => navigate(-1)}
        style={{
          background: 'transparent',
          border: '1px solid var(--border)',
          color: 'var(--text-muted)',
          fontFamily: "'JetBrains Mono', monospace",
          fontSize: '12px',
          padding: '6px 14px',
          borderRadius: '4px',
          cursor: 'pointer',
          marginBottom: '24px',
        }}
      >
        ← Torna indietro
      </button>

      <div style={{
        background: 'var(--surface-1)',
        border: '1px solid var(--border)',
        borderRadius: '8px',
        padding: '28px 32px',
      }}>
        <div style={{
          fontFamily: "'JetBrains Mono', monospace",
          fontSize: '10px',
          color: 'var(--text-muted)',
          marginBottom: '8px',
          textTransform: 'uppercase',
          letterSpacing: '0.1em',
        }}>
          Nota #{post.id}
        </div>
        <h1 style={{
          fontSize: '1.4rem',
          fontWeight: '700',
          color: 'var(--text-primary)',
          marginBottom: '20px',
          lineHeight: '1.4',
        }}>
          {post.title}
        </h1>
        <p style={{ color: 'var(--text-secondary)', lineHeight: '1.8' }}>
          {post.body}
        </p>
      </div>
    </div>
  );
}
```

---

## 7. Variabile d'ambiente per la BASE_URL

Per non scrivere l'URL dell'API nel codice (hardcoded), usa le variabili d'ambiente di Vite. Crea il file `.env` nella root del progetto:

```
# .env
# Questa variabile verrà sostituita con il nostro server Express in L08
VITE_API_URL=https://jsonplaceholder.typicode.com
```

> **Importante:** tutte le variabili d'ambiente di Vite devono iniziare con `VITE_`. Sono incorporate nel bundle a build time — non usarle per dati segreti.

Aggiungi `.env.local` al `.gitignore` (già presente di default in Vite):

```
# .gitignore (già presente, verifica)
.env.local
.env.*.local
```

Aggiorna `notesApi.js` per usarla:

```javascript
// src/services/notesApi.js — riga 1
const BASE_URL = import.meta.env.VITE_API_URL ?? 'https://jsonplaceholder.typicode.com';
```

**Dopo aver modificato `.env`, riavvia il dev server** con `npm run dev`.

---

## 8. Esercizi di consolidamento

### Esercizio A — Refetch manuale con pulsante

Nel componente `Home`, aggiungi un pulsante "Aggiorna" che richiama `refetch` del hook:

```jsx
// In Home.jsx, dopo la gestione degli stati, prima del return con NoteList
<div style={{ display: 'flex', justifyContent: 'flex-end', marginBottom: '16px' }}>
  <button onClick={refetch} style={{ /* stile a tua scelta */ }}>
    🔄 Aggiorna
  </button>
</div>
```

Verifica nella scheda Network del DevTools che al click parta una nuova richiesta HTTP.

---

### Esercizio B — useReducer per lo stato fetch

`useFetch` gestisce tre stati separati con tre `useState`. Un approccio più robusto usa `useReducer` con un unico oggetto stato. Riscrivi l'hook usando questo schema:

```javascript
// Stato iniziale
const initialState = { data: null, loading: false, error: null };

// Reducer
function fetchReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { data: action.payload, loading: false, error: null };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}
```

Beneficio: impossibile avere uno stato inconsistente come `{ loading: true, error: 'qualcosa' }`.

---

### Esercizio C — Wrapper NoteFetcher

Crea un componente `NoteFetcher.jsx` che usa `useFetch` e `notesApi.getAll()`. Passa il risultato come props a `NoteList`. Lo scopo è separare la logica di fetching dalla UI di presentazione (*container/presentational pattern*):

```jsx
// src/components/NoteFetcher/NoteFetcher.jsx
import { useFetch } from '../../hooks/useFetch';
import NoteList from '../NoteList/NoteList';
import LoadingSpinner from '../LoadingSpinner/LoadingSpinner';
import ErrorMessage from '../ErrorMessage/ErrorMessage';

const API_URL = 'https://jsonplaceholder.typicode.com/posts?_limit=12';

export default function NoteFetcher() {
  const { data, loading, error, refetch } = useFetch(API_URL);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} onRetry={refetch} />;
  if (!data?.length) return <p>Nessuna nota.</p>;

  const note = data.map(/* mapping come in Home */);
  return <NoteList note={note} />;
}
```

---

## 9. Checklist finale

| Controllo | OK? |
|-----------|-----|
| `src/hooks/useFetch.js` creato con AbortController nel cleanup | ☐ |
| `src/services/notesApi.js` creato con getAll/getById/create/update/remove | ☐ |
| `LoadingSpinner` mostra l'anello animato CSS | ☐ |
| `ErrorMessage` mostra il messaggio e il pulsante Riprova | ☐ |
| `Home` mostra spinner durante il caricamento | ☐ |
| `Home` mostra `ErrorMessage` se la chiamata fallisce | ☐ |
| `NoteDetail` carica la nota dall'API tramite `useParams` | ☐ |
| File `.env` creato con `VITE_API_URL` | ☐ |
| Nessun warning in console (AbortError ignorato correttamente) | ☐ |
| Network DevTools: si vede la richiesta GET a JSONPlaceholder | ☐ |

---

## 📊 Riepilogo file prodotti

| File | Tipo | Descrizione |
|------|------|-------------|
| `src/hooks/useFetch.js` | Hook | fetch + loading/data/error + AbortController |
| `src/services/notesApi.js` | Servizio | Wrapper fetch con metodi CRUD |
| `src/components/LoadingSpinner/LoadingSpinner.jsx` | Componente | Spinner CSS animato |
| `src/components/LoadingSpinner/LoadingSpinner.module.css` | Stile | Animazione ring |
| `src/components/ErrorMessage/ErrorMessage.jsx` | Componente | Messaggio errore + Riprova |
| `src/components/ErrorMessage/ErrorMessage.module.css` | Stile | Stile errore rosso |
| `src/pages/Home.jsx` | Pagina | Aggiornata con useFetch |
| `src/pages/NoteDetail.jsx` | Pagina | Caricamento nota per ID |
| `.env` | Config | VITE_API_URL |

---

## ➡️ Prossimo lab

**[Lab 07 — Backend con Node.js ed Express](https://github.com/profgiagnotti/react-vite-guide/blob/main/lab-07-express-backend.md)**  
Costruiremo il server Express che sostituirà JSONPlaceholder: route CRUD `/api/notes`, middleware, CORS e dati in-memory.
