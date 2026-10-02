# Lab 03 — Stato e interattività con Hooks

> **Serie:** React + Vite + Node.js — Guida completa  
> **Repo:** `https://github.com/profgiagnotti/sviluppoWeb-labs/tree/main/react-vite-guide`  
> **Articolo di riferimento:** [Lezione 3 — Stato e interattività](https://profgiagnotti.it/lezione-3-stato-e-interattivita-con-hooks/)  
> **Difficoltà:** ⭐⭐☆☆☆ Principiante+  
> **Tempo stimato:** 90–120 minuti  

---

## ✅ Obiettivi

Al termine di questo laboratorio avrai:

- [ ] Aggiunto il form `NoteForm.jsx` con input controllati e validazione base
- [ ] Sollevato lo stato (`lift state up`) in `App.jsx` per gestire la lista note
- [ ] Implementato l'aggiunta e l'eliminazione di note con `useState`
- [ ] Aggiunto un campo di ricerca in tempo reale con `useState` + `useMemo`
- [ ] Usato `useEffect` per il titolo della pagina dinamico e un contatore live
- [ ] Verificato il cleanup di `useEffect` con un timer (prevenzione memory leak)

---

## 📋 Prerequisiti

- Lab 01 e Lab 02 completati ✅  
- Il progetto `devnotes` avviato con `npm run dev`  
- I componenti `NoteCard`, `NoteList`, `Header` già presenti da L02

---

## 📑 Indice

1. [Struttura cartelle aggiornata](#1-struttura-cartelle-aggiornata)
2. [NoteForm.jsx — form controllato](#2-noteformjsx--form-controllato)
3. [App.jsx — lift state up](#3-appjsx--lift-state-up)
4. [Eliminazione note — collegare NoteCard](#4-eliminazione-note--collegare-notecard)
5. [Ricerca in tempo reale](#5-ricerca-in-tempo-reale)
6. [useEffect — titolo pagina dinamico](#6-useeffect--titolo-pagina-dinamico)
7. [useEffect con cleanup — contatore live](#7-useeffect-con-cleanup--contatore-live)
8. [Esercizi di consolidamento](#8-esercizi-di-consolidamento)
9. [Checklist finale](#9-checklist-finale)

---

## 1. Struttura cartelle aggiornata

Alla fine di L02 la struttura era:

```
src/
  components/
    Header/
    NoteCard/
    NoteList/
  App.jsx
  main.jsx
  index.css
```

In questo lab aggiungi la cartella per il form:

```
src/
  components/
    Header/
      Header.jsx
      Header.module.css
    NoteCard/
      NoteCard.jsx
      NoteCard.module.css
    NoteForm/          ← NUOVO
      NoteForm.jsx
      NoteForm.module.css
    NoteList/
      NoteList.jsx
      NoteList.module.css
  App.jsx
  main.jsx
  index.css
```

**Crea la cartella:**

```bash
# Windows (PowerShell)
mkdir src\components\NoteForm

# macOS / Linux
mkdir -p src/components/NoteForm
```

---

## 2. NoteForm.jsx — form controllato

### 2.1 Crea il componente

Crea il file `src/components/NoteForm/NoteForm.jsx`:

```jsx
// NoteForm.jsx
// Form controllato per aggiungere nuove note a DevNotes
// Concetti: useState, onChange, onSubmit, validazione, reset post-invio

import { useState } from 'react';
import stili from './NoteForm.module.css';

// I tag disponibili per categorizzare la nota
const TAG_DISPONIBILI = ['idea', 'todo', 'studio', 'progetto', 'altro'];

function NoteForm({ onAggiungi }) {
  // Stato dei campi del form — ogni campo ha il proprio state slice
  const [titolo, setTitolo]   = useState('');
  const [testo,  setTesto]    = useState('');
  const [tag,    setTag]      = useState('idea');

  // Stato UI: errore di validazione
  const [errore, setErrore] = useState('');

  function handleSubmit(e) {
    e.preventDefault();  // blocca il comportamento default del browser (reload)

    // Validazione: il titolo non può essere vuoto
    if (!titolo.trim()) {
      setErrore('Il titolo è obbligatorio.');
      return;
    }

    // Crea l'oggetto nota e lo passa al genitore tramite la prop onAggiungi
    onAggiungi({
      titolo: titolo.trim(),
      testo:  testo.trim(),
      tag,
    });

    // Reset del form dopo l'invio
    setTitolo('');
    setTesto('');
    setTag('idea');
    setErrore('');
  }

  return (
    <form className={stili.form} onSubmit={handleSubmit} noValidate>
      <div className={stili.header}>
        <span className={stili.label}>// nuova nota</span>
      </div>

      {/* Campo titolo — input controllato */}
      <input
        className={stili.input}
        type="text"
        value={titolo}                            // ← valore dallo stato
        onChange={e => {
          setTitolo(e.target.value);
          if (errore) setErrore('');              // pulisce l'errore mentre si scrive
        }}
        placeholder="Titolo della nota"
        maxLength={80}
      />

      {/* Messaggio di errore — rendering condizionale */}
      {errore && <p className={stili.errore}>{errore}</p>}

      {/* Campo testo — textarea controllata */}
      <textarea
        className={stili.textarea}
        value={testo}
        onChange={e => setTesto(e.target.value)}
        placeholder="Scrivi qui il contenuto della nota..."
        rows={4}
      />

      {/* Contatore caratteri */}
      <div className={stili.contatore}>
        {testo.length} / 500 caratteri
      </div>

      {/* Selettore tag */}
      <div className={stili.tagSelector}>
        {TAG_DISPONIBILI.map(t => (
          <button
            key={t}
            type="button"                         // type="button" evita il submit accidentale
            className={`${stili.tagBtn} ${tag === t ? stili.tagAttivo : ''}`}
            onClick={() => setTag(t)}
          >
            {t}
          </button>
        ))}
      </div>

      {/* Pulsante submit — disabilitato se il titolo è vuoto */}
      <button
        type="submit"
        className={stili.btnSubmit}
        disabled={!titolo.trim()}
      >
        + Aggiungi nota
      </button>
    </form>
  );
}

export default NoteForm;
```

### 2.2 Crea il CSS Module

Crea `src/components/NoteForm/NoteForm.module.css`:

```css
/* NoteForm.module.css */
/* Form per aggiungere note — design system DevNotes */

.form {
  background: #0E1118;
  border: 1px solid #1E2535;
  border-radius: 10px;
  padding: 20px 22px;
  margin-bottom: 28px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.header {
  margin-bottom: 4px;
}

.label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  color: #00D4FF;
  text-transform: uppercase;
  letter-spacing: 0.14em;
}

.input,
.textarea {
  background: #07090F;
  border: 1px solid #1E2535;
  border-radius: 6px;
  padding: 10px 14px;
  color: #DCE6F5;
  font-family: 'Syne', sans-serif;
  font-size: 14px;
  transition: border-color 0.2s;
  outline: none;
  width: 100%;
  box-sizing: border-box;
}

.input:focus,
.textarea:focus {
  border-color: #00D4FF;
}

.input::placeholder,
.textarea::placeholder {
  color: #3A4A62;
}

.textarea {
  resize: vertical;
  min-height: 90px;
  line-height: 1.6;
}

.contatore {
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  color: #3A4A62;
  text-align: right;
  margin-top: -4px;
}

.tagSelector {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.tagBtn {
  background: transparent;
  border: 1px solid #1E2535;
  border-radius: 20px;
  padding: 4px 12px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: #6B7A99;
  cursor: pointer;
  transition: all 0.2s;
}

.tagBtn:hover {
  border-color: #00D4FF;
  color: #00D4FF;
}

.tagAttivo {
  background: #001E2A;
  border-color: #00D4FF;
  color: #00D4FF;
}

.btnSubmit {
  background: #00D4FF;
  border: none;
  border-radius: 6px;
  padding: 10px 18px;
  font-family: 'Orbitron', sans-serif;
  font-size: 11px;
  font-weight: 700;
  color: #001218;
  cursor: pointer;
  letter-spacing: 0.06em;
  transition: opacity 0.2s;
  align-self: flex-end;
}

.btnSubmit:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.btnSubmit:not(:disabled):hover {
  opacity: 0.85;
}

.errore {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: #FF5C5C;
  margin: 0;
  padding: 6px 10px;
  background: #2A0A0A;
  border-radius: 4px;
}
```

---

## 3. App.jsx — lift state up

Aggiorna `src/App.jsx` per sollevare lo stato e collegare i componenti:

```jsx
// App.jsx
// Stato centralizzato: App è l'unica fonte di verità per la lista note.
// NoteForm aggiunge note → App aggiorna lo stato → NoteList ri-renderizza.

import { useState } from 'react';
import Header    from './components/Header/Header';
import NoteForm  from './components/NoteForm/NoteForm';
import NoteList  from './components/NoteList/NoteList';
import stili     from './App.module.css';  // crea questo file nella sezione 3.1

// Dati iniziali di esempio
const NOTE_INIZIALI = [
  {
    id: 1,
    titolo: 'useState — concetti chiave',
    testo:  'Restituisce [valore, setter]. Il setter notifica React del cambiamento e pianifica un re-render.',
    tag:    'studio',
    data:   new Date('2026-04-10'),
  },
  {
    id: 2,
    titolo: 'Immutabilità degli array',
    testo:  'Usa filter() per rimuovere, map() per modificare, spread [...arr] per aggiungere.',
    tag:    'todo',
    data:   new Date('2026-04-11'),
  },
  {
    id: 3,
    titolo: 'Lift state up',
    testo:  'Se due componenti fratelli condividono dati, lo stato sale al genitore comune.',
    tag:    'idea',
    data:   new Date('2026-04-12'),
  },
];

function App() {
  // ─── Stato principale ───────────────────────────────────────
  const [note, setNote] = useState(NOTE_INIZIALI);

  // ─── Handler: aggiungi una nuova nota ────────────────────────
  function aggiungiNota({ titolo, testo, tag }) {
    const nuova = {
      id:    Date.now(),   // id univoco temporaneo (in L07 useremo il DB)
      titolo,
      testo,
      tag,
      data:  new Date(),
    };
    // Aggiornamento funzionale + nuova nota in cima alla lista
    setNote(prev => [nuova, ...prev]);
  }

  // ─── Handler: elimina una nota per id ────────────────────────
  function eliminaNota(id) {
    setNote(prev => prev.filter(n => n.id !== id));
  }

  return (
    <div className={stili.app}>
      <Header contatoreNote={note.length} />

      <main className={stili.main}>
        <NoteForm onAggiungi={aggiungiNota} />
        <NoteList note={note} onElimina={eliminaNota} />
      </main>
    </div>
  );
}

export default App;
```

### 3.1 Crea App.module.css

Crea `src/App.module.css`:

```css
/* App.module.css */
.app {
  min-height: 100vh;
  background: #07090F;
}

.main {
  max-width: 860px;
  margin: 0 auto;
  padding: 24px 20px 60px;
}
```

### 3.2 Aggiorna Header per accettare il contatore

In `Header.jsx`, aggiungi la prop `contatoreNote` per mostrare il numero di note:

```jsx
// Header.jsx — aggiunta prop contatoreNote
function Header({ contatoreNote = 0 }) {
  return (
    <header className={styles.header}>
      <div className={styles.logo}>DevNotes</div>
      {/* badge con il numero di note */}
      <div className={styles.badge}>{contatoreNote} note</div>
    </header>
  );
}
```

Aggiungi in `Header.module.css`:

```css
.badge {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: #00D4FF;
  background: #001E2A;
  border: 1px solid #00D4FF;
  border-radius: 20px;
  padding: 3px 12px;
}
```

---

## 4. Eliminazione note — collegare NoteCard

Verifica che `NoteCard.jsx` accetti e usi la prop `onElimina` (già definita in L02).
Se non è presente, aggiungi il pulsante elimina:

```jsx
// In NoteCard.jsx — aggiunta pulsante elimina
// La prop onElimina viene chiamata con l'id della nota

function NoteCard({ nota, onElimina }) {
  const { id, titolo, testo, tag, data } = nota;

  return (
    <article className={stili.card}>
      {/* intestazione con tag e pulsante elimina */}
      <div className={stili.header}>
        <span className={`${stili.tag} ${stili[tag] || ''}`}>{tag}</span>
        <button
          className={stili.btnElimina}
          onClick={() => onElimina(id)}   // ← passa l'id al genitore
          aria-label="Elimina nota"
          title="Elimina nota"
        >
          ×
        </button>
      </div>

      <h3 className={stili.titolo}>{titolo}</h3>
      {testo && <p className={stili.testo}>{testo}</p>}
      <div className={stili.data}>{data ? new Date(data).toLocaleDateString('it-IT') : ''}</div>
    </article>
  );
}
```

Aggiungi in `NoteCard.module.css`:

```css
.btnElimina {
  background: transparent;
  border: 1px solid #1E2535;
  border-radius: 4px;
  color: #3A4A62;
  font-size: 16px;
  line-height: 1;
  padding: 2px 7px;
  cursor: pointer;
  transition: all 0.2s;
}

.btnElimina:hover {
  border-color: #FF5C5C;
  color: #FF5C5C;
}
```

Verifica in `NoteList.jsx` che la prop `onElimina` venga passata a ogni `NoteCard`:

```jsx
// NoteList.jsx — assicurati che onElimina sia trasmessa
function NoteList({ note, onElimina }) {
  if (note.length === 0) {
    return <p className={stili.vuota}>Nessuna nota. Aggiungine una!</p>;
  }

  return (
    <div className={stili.griglia}>
      {note.map(nota => (
        <NoteCard
          key={nota.id}
          nota={nota}
          onElimina={onElimina}   // ← prop forwarding
        />
      ))}
    </div>
  );
}
```

---

## 5. Ricerca in tempo reale

Aggiungi la funzionalità di ricerca full-text nelle note.

### 5.1 Campo ricerca in App.jsx

```jsx
// Aggiungi questi import e stato in App.jsx
import { useState, useMemo } from 'react';

function App() {
  const [note, setNote]       = useState(NOTE_INIZIALI);
  const [ricerca, setRicerca] = useState('');   // ← NUOVO

  // useMemo ricalcola le note filtrate SOLO quando note o ricerca cambiano
  // Evita il filtraggio ad ogni rendering non correlato
  const noteFiltrate = useMemo(() => {
    const q = ricerca.trim().toLowerCase();
    if (!q) return note;
    return note.filter(
      n =>
        n.titolo.toLowerCase().includes(q) ||
        n.testo.toLowerCase().includes(q)
    );
  }, [note, ricerca]);

  // ... aggiungiNota, eliminaNota invariati

  return (
    <div className={stili.app}>
      <Header contatoreNote={noteFiltrate.length} />

      <main className={stili.main}>
        {/* Campo di ricerca */}
        <input
          className={stili.ricerca}
          type="search"
          value={ricerca}
          onChange={e => setRicerca(e.target.value)}
          placeholder="🔍  Cerca nelle note..."
        />

        <NoteForm onAggiungi={aggiungiNota} />
        <NoteList note={noteFiltrate} onElimina={eliminaNota} />
      </main>
    </div>
  );
}
```

### 5.2 Stile campo ricerca in App.module.css

```css
.ricerca {
  width: 100%;
  background: #0E1118;
  border: 1px solid #1E2535;
  border-radius: 8px;
  padding: 11px 16px;
  font-family: 'Syne', sans-serif;
  font-size: 14px;
  color: #DCE6F5;
  outline: none;
  margin-bottom: 20px;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.ricerca:focus {
  border-color: #00D4FF;
}

.ricerca::placeholder {
  color: #3A4A62;
}
```

**Testa la ricerca:** avvia il dev server e digita nel campo — le note si filtrano in tempo reale senza bisogno di premere Invio.

---

## 6. useEffect — titolo pagina dinamico

Usa `useEffect` per aggiornare il `<title>` del browser in base al numero di note.

Aggiungi in `App.jsx`:

```jsx
import { useState, useMemo, useEffect } from 'react';

function App() {
  // ... stato e handler già presenti

  // useEffect per il titolo del documento
  // Si aggiorna ogni volta che il numero di note cambia
  useEffect(() => {
    const n = noteFiltrate.length;
    document.title = n > 0
      ? `DevNotes — ${n} nota${n !== 1 ? 'e' : ''}`
      : 'DevNotes';
  }, [noteFiltrate.length]);  // ← dipendenza: numero di note filtrate

  // ... resto del componente
}
```

> 💡 **Verifica:** apri il browser — guarda la tab. Aggiungi e rimuovi note: il titolo si aggiorna. Cancella note: il contatore scende. Cerca: il titolo mostra le note filtrate.

---

## 7. useEffect con cleanup — contatore live

Crea un piccolo componente `ContatoreLive` che mostra da quanti secondi l'app è aperta. Serve a capire concretamente il cleanup.

Crea `src/components/ContatoreLive/ContatoreLive.jsx`:

```jsx
// ContatoreLive.jsx
// Dimostra useEffect con cleanup: il timer viene cancellato all'unmount.
// Senza il cleanup, clearInterval non verrebbe mai chiamato → memory leak.

import { useState, useEffect } from 'react';
import stili from './ContatoreLive.module.css';

function ContatoreLive() {
  const [secondi, setSecondi] = useState(0);

  useEffect(() => {
    // Avvia il timer al mount del componente
    const id = setInterval(() => {
      setSecondi(prev => prev + 1);   // forma funzionale ← dipende dal valore precedente
    }, 1000);

    // ← Cleanup: React chiama questa funzione PRIMA di smontare il componente
    // oppure prima di rieseguire l'effetto. Senza return, il timer girerebbe
    // in background anche se il componente non esiste più nel DOM.
    return () => {
      clearInterval(id);
      console.log('ContatoreLive smontato — timer cancellato');
    };
  }, []);  // [] → l'effetto gira una sola volta al mount

  return (
    <div className={stili.wrap}>
      <span className={stili.label}>sessione attiva</span>
      <span className={stili.valore}>{secondi}s</span>
    </div>
  );
}

export default ContatoreLive;
```

Crea `src/components/ContatoreLive/ContatoreLive.module.css`:

```css
.wrap {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: #07090F;
  border: 1px solid #1E2535;
  border-radius: 20px;
  padding: 5px 14px;
}

.label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  color: #3A4A62;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.valore {
  font-family: 'JetBrains Mono', monospace;
  font-size: 12px;
  color: #3DE8A0;
  font-weight: 700;
}
```

Importa e aggiungi `<ContatoreLive />` dentro `Header.jsx` per vederlo in azione:

```jsx
import ContatoreLive from '../ContatoreLive/ContatoreLive';

function Header({ contatoreNote = 0 }) {
  return (
    <header className={stili.header}>
      <div className={stili.logo}>DevNotes</div>
      <div className={stili.destra}>
        <ContatoreLive />
        <div className={stili.badge}>{contatoreNote} note</div>
      </div>
    </header>
  );
}
```

### 7.1 Verifica il cleanup con React DevTools

1. Apri il browser con il dev server attivo  
2. Apri **React DevTools** (tab Components)  
3. Trova `ContatoreLive` nell'albero e osserva il contatore che sale  
4. Nella console, aggiungi temporaneamente in App.jsx:

```jsx
const [mostraContatore, setMostraContatore] = useState(true);

// Nel JSX:
<button onClick={() => setMostraContatore(m => !m)}>Toggle contatore</button>
{mostraContatore && <ContatoreLive />}
```

5. Clicca il pulsante: il componente viene smontato → controlla la console → vedrai il messaggio di cleanup. Riclicca: il componente viene rimontato e il contatore riparte da zero.

---

## 8. Esercizi di consolidamento

### Esercizio A — Modifica nota inline ⭐

**Obiettivo:** permettere la modifica del titolo di una nota con un doppio click.

1. In `NoteCard.jsx` aggiungi uno stato locale `const [editing, setEditing] = useState(false)`.
2. Al doppio click sul titolo (`onDoubleClick`), imposta `editing = true`.
3. Quando `editing` è `true`, mostra un `<input>` controllato al posto del titolo.
4. Alla pressione di `Enter` (handler `onKeyDown`) o `onBlur`, chiama una prop `onModifica(id, nuovoTitolo)` passata dal genitore.
5. In `App.jsx`, implementa `modificaNota(id, nuovoTitolo)` che usa `setNote` con `.map()` per aggiornare immutabilmente.

**Firma attesa:**
```jsx
// App.jsx
function modificaNota(id, nuovoTitolo) {
  setNote(prev =>
    prev.map(n => n.id === id ? { ...n, titolo: nuovoTitolo } : n)
  );
}

// NoteCard.jsx
<NoteCard nota={nota} onElimina={onElimina} onModifica={onModifica} />
```

---

### Esercizio B — Filtro per tag ⭐⭐

**Obiettivo:** aggiungere una barra di filtri per tag sopra la lista note.

1. In `App.jsx` aggiungi `const [tagAttivo, setTagAttivo] = useState(null)`.
2. Crea una lista di tag univoci dalle note con `useMemo`:
   ```jsx
   const tagUnici = useMemo(
     () => [...new Set(note.map(n => n.tag))],
     [note]
   );
   ```
3. Renderizza un pulsante per ogni tag e uno "Tutti".
4. Aggiorna `noteFiltrate` per filtrare anche per `tagAttivo`.
5. Evidenzia il tag attivo con una classe CSS dedicata.

---

### Esercizio C — Persistenza con localStorage ⭐⭐⭐

**Obiettivo:** salvare le note nel `localStorage` del browser, così sopravvivono al refresh.

1. Sostituisci l'inizializzazione dello stato con una funzione lazy:
   ```jsx
   const [note, setNote] = useState(() => {
     const salvate = localStorage.getItem('devnotes');
     return salvate ? JSON.parse(salvate) : NOTE_INIZIALI;
   });
   ```
2. Aggiungi un `useEffect` che salva le note ogni volta che cambiano:
   ```jsx
   useEffect(() => {
     localStorage.setItem('devnotes', JSON.stringify(note));
   }, [note]);
   ```
3. Testa: aggiungi una nota, ricarica la pagina → la nota deve essere ancora presente.
4. **Bonus:** le date vengono serializzate come stringhe da JSON. Aggiungi una funzione `reidrata` che converte `data` da stringa a `Date` dopo il parse.

---

## 9. Checklist finale

Verifica che tutto funzioni correttamente:

- [ ] Il form si invia con Enter e con il pulsante
- [ ] Il pulsante "Aggiungi nota" è disabilitato se il titolo è vuoto
- [ ] Dopo l'invio il form si azzera (titolo, testo, tag)
- [ ] Il messaggio di errore appare se si tenta di inviare senza titolo
- [ ] Nuove note appaiono in cima alla lista
- [ ] Il pulsante × elimina la nota corretta (non un'altra)
- [ ] La ricerca filtra in tempo reale per titolo e testo
- [ ] Il titolo del browser mostra il numero aggiornato di note
- [ ] Il contatore live sale ogni secondo e si azzera se smontato
- [ ] Nessun warning nella console (inclusi warning React)
- [ ] Il progetto compila senza errori TypeScript/ESLint

---

## 📊 Tabella riepilogo — concetti del lab

| Concetto | File | Hook/API usata | Note |
|---|---|---|---|
| Input controllato | `NoteForm.jsx` | `useState` | `value` + `onChange` |
| Reset post-submit | `NoteForm.jsx` | `useState` | setter a stringa vuota |
| Lift state up | `App.jsx` | `useState` | note[ ] nel genitore |
| Aggiunta immutabile | `App.jsx` | `useState` | spread `[nuova, ...prev]` |
| Rimozione immutabile | `App.jsx` | `useState` | `filter()` |
| Ricerca filtrata | `App.jsx` | `useMemo` | ricalcolo su `[note, ricerca]` |
| Titolo pagina dinamico | `App.jsx` | `useEffect` | dipendenza numero note |
| Timer con cleanup | `ContatoreLive.jsx` | `useEffect` | `clearInterval` nel return |

---

## 🔗 Lab successivo

**Lab 04 — Styling con CSS Modules e Tailwind CSS**  
`lab-04-styling.md` — installa Tailwind in Vite, sostituisci parti del CSS Module con classi utility, gestisci il responsive e crea un sistema di tema coerente con il design system DevNotes.
