# Lab 04 — Styling di DevNotes: CSS Modules e Tailwind CSS

> **Serie:** React + Vite + Node.js — Guida completa  
> **Repo:** `profgiagnotti/react-vite-guide`  
> **File:** `lab-04-styling.md`

---

## Obiettivi

- [ ] Rifattorizzare i CSS globali del progetto con variabili custom in `index.css`
- [ ] Installare e configurare Tailwind CSS v3 in Vite
- [ ] Ridisegnare `NoteCard` con classi Tailwind utility
- [ ] Costruire un layout responsive a colonne con breakpoint `md:` e `lg:`
- [ ] Aggiungere un toggle dark/light mode con stato React
- [ ] Gestire classi condizionali con template literal e la libreria `clsx`

---

## Prerequisiti

- Lab 03 completato (il progetto DevNotes con `useState`, `useEffect`, form controllato e lift state)
- Node.js ≥ 18 installato
- Progetto avviato con `npm run dev`

---

## Indice

1. [Pulizia e reset del CSS globale](#1-pulizia-e-reset-del-css-globale)
2. [Installazione Tailwind CSS](#2-installazione-tailwind-css)
3. [Configurazione del tema DevNotes](#3-configurazione-del-tema-devnotes)
4. [Refactoring NoteCard con Tailwind](#4-refactoring-notecard-con-tailwind)
5. [Layout NoteList responsive](#5-layout-notelist-responsive)
6. [Header con Tailwind](#6-header-con-tailwind)
7. [NoteForm con Tailwind](#7-noteform-con-tailwind)
8. [Toggle dark/light mode](#8-toggle-darklight-mode)
9. [Esercizi di consolidamento](#9-esercizi-di-consolidamento)
10. [Checklist finale](#10-checklist-finale)

---

## 1. Pulizia e reset del CSS globale

Prima di installare Tailwind, sistemiamo il file `src/index.css` rendendolo il punto unico di verità per le variabili del design system. Apri `src/index.css` e **sostituisci tutto il contenuto** con:

```css
/* src/index.css
   Questo file viene importato una sola volta in main.jsx.
   Non usare classi qui — solo variabili, reset e stili body. */

@tailwind base;
@tailwind components;
@tailwind utilities;

/* ── Design System DevNotes ─────────────────────────────── */
:root {
  --color-void:    #07090F;
  --color-surface: #0E1118;
  --color-surface2:#151A24;
  --color-border:  #1E2535;
  --color-primary: #00D4FF;
  --color-green:   #3DE8A0;
  --color-amber:   #FFAA3D;
  --color-red:     #FF5C5C;
  --color-purple:  #BC8CFF;
  --color-text:    #A8B8D0;
  --color-heading: #DCE6F5;
  --color-muted:   #6B7A99;

  --radius-card:   8px;
  --font-mono:     'JetBrains Mono', monospace;
  --font-sans:     'Syne', sans-serif;
}

body {
  background-color: var(--color-void);
  color: var(--color-text);
  font-family: var(--font-sans);
  font-size: 15px;
  line-height: 1.75;
}

* {
  box-sizing: border-box;
}
```

> **Nota:** Le tre righe `@tailwind` devono stare **in cima** al file. Verranno processate da PostCSS quando esegui `npm run dev`.

---

## 2. Installazione Tailwind CSS

### Windows / macOS / Linux (stesso comando)

Dalla root del progetto DevNotes (dove c'è `package.json`):

```bash
# installa Tailwind, PostCSS e Autoprefixer come dev dependencies
npm install -D tailwindcss postcss autoprefixer

# genera tailwind.config.js e postcss.config.js
npx tailwindcss init -p
```

Verifica che siano stati creati:

```
devnotes/
├── tailwind.config.js    ← nuovo
├── postcss.config.js     ← nuovo
├── package.json
└── src/
```

---

## 3. Configurazione del tema DevNotes

Apri `tailwind.config.js` e sostituisci il contenuto con:

```js
// tailwind.config.js
export default {
  // ❶ Content: Tailwind legge questi file per trovare le classi usate
  //    e rimuove tutto il resto nel build di produzione (tree-shaking).
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],

  // ❷ Dark mode: 'class' significa che Tailwind attiva dark: quando
  //    l'elemento <html> ha la classe "dark". Lo gestiremo con React.
  darkMode: 'class',

  theme: {
    extend: {
      // ❸ Colori custom del design system
      colors: {
        void:    '#07090F',
        surface: '#0E1118',
        surface2:'#151A24',
        border:  '#1E2535',
        cyan: {
          400: '#00D4FF',
          900: '#001E2A',
        },
        green: {
          400: '#3DE8A0',
          950: '#0a1e14',
        },
        amber: {
          400: '#FFAA3D',
          950: '#1a1000',
        },
        red: {
          400: '#FF5C5C',
        },
        purple: {
          400: '#BC8CFF',
        },
        // Alias semantici
        'text-primary': '#DCE6F5',
        'text-body':    '#A8B8D0',
        'text-muted':   '#6B7A99',
      },

      // ❹ Font custom
      fontFamily: {
        mono:  ['JetBrains Mono', 'monospace'],
        sans:  ['Syne', 'sans-serif'],
        serif: ['Fraunces', 'serif'],
      },

      // ❺ Border radius custom
      borderRadius: {
        card: '8px',
      },
    },
  },
  plugins: [],
};
```

Salva e assicurati che `npm run dev` non mostri errori in console.

---

## 4. Refactoring NoteCard con Tailwind

Crea (o sostituisci) `src/components/NoteCard.jsx`:

```jsx
// src/components/NoteCard.jsx
// NoteCard — versione Tailwind CSS

function NoteCard({ note, onElimina }) {

  // ── mappa tag → classi Tailwind per il badge colorato
  const TAG_STILI = {
    react: 'bg-cyan-900  text-cyan-400  border border-cyan-400/30',
    js:    'bg-amber-950 text-amber-400 border border-amber-400/30',
    node:  'bg-green-950 text-green-400 border border-green-400/30',
    css:   'bg-purple-950 text-purple-400 border border-purple-400/30',
    altro: 'bg-surface   text-[#6B7A99] border border-border',
  };

  const tagClasse = TAG_STILI[note.tag] ?? TAG_STILI.altro;

  // ── formatta la data in italiano
  const dataFormattata = new Date(note.data).toLocaleDateString('it-IT', {
    day:   '2-digit',
    month: 'short',
    year:  'numeric',
  });

  return (
    <div className="
      bg-surface border border-border rounded-card
      p-5 flex flex-col gap-3
      hover:border-cyan-400/50 transition-colors duration-200
    ">
      {/* ── riga top: titolo + badge tag */}
      <div className="flex items-start justify-between gap-2">
        <h3 className="text-base font-bold text-white leading-snug line-clamp-2 flex-1">
          {note.titolo}
        </h3>
        <span className={`font-mono text-[11px] px-2 py-0.5 rounded whitespace-nowrap ${tagClasse}`}>
          {note.tag}
        </span>
      </div>

      {/* ── corpo: testo troncato a 3 righe */}
      <p className="text-sm text-[#A8B8D0] leading-relaxed line-clamp-3 flex-1">
        {note.testo}
      </p>

      {/* ── footer: data + pulsante elimina */}
      <div className="flex items-center justify-between pt-3 border-t border-border">
        <span className="font-mono text-[11px] text-[#6B7A99]">
          {dataFormattata}
        </span>
        <button
          onClick={() => onElimina(note.id)}
          className="font-mono text-[11px] text-[#6B7A99]
                     hover:text-red-400 transition-colors cursor-pointer"
        >
          elimina ✕
        </button>
      </div>
    </div>
  );
}

export default NoteCard;
```

> **Hai rimosso** `NoteCard.module.css`? Puoi eliminarlo — non serve più. Verifica che non ci siano import rimasti nel file.

---

## 5. Layout NoteList responsive

Sostituisci `src/components/NoteList.jsx`:

```jsx
// src/components/NoteList.jsx
import NoteCard from './NoteCard';

function NoteList({ note, onElimina }) {

  // ── lista vuota
  if (note.length === 0) {
    return (
      <div className="flex flex-col items-center justify-center py-20 gap-3">
        <span className="text-4xl opacity-30">📭</span>
        <p className="font-mono text-sm text-[#6B7A99]">
          Nessuna nota. Creane una!
        </p>
      </div>
    );
  }

  return (
    <div className="
      grid
      grid-cols-1
      md:grid-cols-2
      lg:grid-cols-3
      gap-4
    ">
      {note.map(n => (
        <NoteCard
          key={n.id}
          note={n}
          onElimina={onElimina}
        />
      ))}
    </div>
  );
}

export default NoteList;
```

Cosa succede con i breakpoint:
- **< 768px** (mobile): 1 colonna
- **≥ 768px** (tablet): 2 colonne
- **≥ 1024px** (desktop): 3 colonne

Testa ridimensionando la finestra con DevTools (F12 → icona telefono).

---

## 6. Header con Tailwind

Sostituisci `src/components/Header.jsx`:

```jsx
// src/components/Header.jsx
function Header({ totaleNote, onToggleDark, isDark }) {
  return (
    <header className="
      border-b border-border mb-8 pb-6
      flex items-center justify-between gap-4
    ">
      {/* ── logo / titolo */}
      <div className="flex items-center gap-3">
        <span className="text-2xl">📝</span>
        <div>
          <h1 className="font-serif text-2xl font-black text-white leading-none">
            DevNotes
          </h1>
          <p className="font-mono text-[11px] text-[#6B7A99] mt-0.5">
            {totaleNote} {totaleNote === 1 ? 'nota' : 'note'} salvate
          </p>
        </div>
      </div>

      {/* ── controlli destra */}
      <div className="flex items-center gap-2">
        <span className="font-mono text-[10px] text-[#6B7A99] uppercase tracking-wider hidden sm:block">
          v1.0.0
        </span>
        <button
          onClick={onToggleDark}
          className="
            font-mono text-[11px] px-3 py-1.5
            border border-border rounded
            text-[#6B7A99] hover:border-cyan-400/50
            hover:text-cyan-400 transition-colors
          "
        >
          {isDark ? '☀ light' : '🌙 dark'}
        </button>
      </div>
    </header>
  );
}

export default Header;
```

---

## 7. NoteForm con Tailwind

Sostituisci `src/components/NoteForm.jsx`:

```jsx
// src/components/NoteForm.jsx
import { useState } from 'react';

const TAG_OPTIONS = ['react', 'js', 'node', 'css', 'altro'];

function NoteForm({ onAggiungi }) {
  const [titolo, setTitolo] = useState('');
  const [testo,  setTesto]  = useState('');
  const [tag,    setTag]    = useState('react');
  const [errore, setErrore] = useState('');

  // classi condivise per gli input
  const inputClasse = `
    w-full bg-surface2 border border-border rounded px-3 py-2
    font-mono text-sm text-white placeholder-[#2E3A52]
    focus:outline-none focus:border-cyan-400/60 transition-colors
  `;

  function handleSubmit(e) {
    e.preventDefault();
    if (!titolo.trim()) {
      setErrore('Il titolo è obbligatorio.');
      return;
    }
    onAggiungi({ titolo: titolo.trim(), testo: testo.trim(), tag });
    setTitolo('');
    setTesto('');
    setTag('react');
    setErrore('');
  }

  return (
    <form
      onSubmit={handleSubmit}
      className="
        bg-surface border border-border rounded-card
        p-5 mb-8 flex flex-col gap-4
      "
    >
      {/* ── label gruppo titolo */}
      <div className="flex flex-col gap-1.5">
        <label className="font-mono text-[10px] text-[#6B7A99] uppercase tracking-wider">
          Titolo *
        </label>
        <input
          type="text"
          value={titolo}
          onChange={e => { setTitolo(e.target.value); setErrore(''); }}
          placeholder="Es: Hook useMemo — quando e perché"
          className={inputClasse}
        />
        {errore && (
          <span className="font-mono text-[11px] text-red-400">{errore}</span>
        )}
      </div>

      {/* ── testo + tag affiancati su md: */}
      <div className="flex flex-col md:flex-row gap-4">
        <div className="flex flex-col gap-1.5 flex-1">
          <label className="font-mono text-[10px] text-[#6B7A99] uppercase tracking-wider">
            Contenuto
          </label>
          <textarea
            value={testo}
            onChange={e => setTesto(e.target.value)}
            placeholder="Appunti, snippet, link utili..."
            rows={3}
            className={`${inputClasse} resize-none`}
          />
        </div>

        <div className="flex flex-col gap-1.5 md:w-36">
          <label className="font-mono text-[10px] text-[#6B7A99] uppercase tracking-wider">
            Tag
          </label>
          <select
            value={tag}
            onChange={e => setTag(e.target.value)}
            className={inputClasse}
          >
            {TAG_OPTIONS.map(t => (
              <option key={t} value={t}>{t}</option>
            ))}
          </select>
        </div>
      </div>

      {/* ── submit */}
      <button
        type="submit"
        className="
          self-start font-mono text-sm font-bold
          bg-cyan-900 text-cyan-400 border border-cyan-400/40
          px-5 py-2 rounded hover:bg-cyan-400 hover:text-void
          transition-colors duration-200
        "
      >
        + Aggiungi nota
      </button>
    </form>
  );
}

export default NoteForm;
```

---

## 8. Toggle dark/light mode

Tailwind supporta la dark mode con la strategia `class`: quando `<html>` ha la classe `dark`, le classi con prefisso `dark:` si attivano.

Aggiorna `src/App.jsx`:

```jsx
// src/App.jsx
import { useState, useEffect } from 'react';
import Header   from './components/Header';
import NoteForm from './components/NoteForm';
import NoteList from './components/NoteList';

function App() {
  const [note,   setNote]   = useState([
    { id: 1, titolo: 'useState — anatomia', testo: 'const [state, setState] = useState(initialValue)', tag: 'react', data: new Date().toISOString() },
    { id: 2, titolo: 'Arrow function', testo: 'Sintassi concisa: const fn = (x) => x * 2', tag: 'js', data: new Date().toISOString() },
    { id: 3, titolo: 'npm scripts', testo: 'npm run dev | npm run build | npm run preview', tag: 'node', data: new Date().toISOString() },
  ]);

  // ── dark mode: gestisce la classe sull'elemento <html>
  const [isDark, setIsDark] = useState(true);

  useEffect(() => {
    if (isDark) {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
  }, [isDark]);

  function aggiungiNota(nuovaNota) {
    setNote(prev => [
      { ...nuovaNota, id: Date.now(), data: new Date().toISOString() },
      ...prev,
    ]);
  }

  function eliminaNota(id) {
    setNote(prev => prev.filter(n => n.id !== id));
  }

  return (
    // ── wrapper con max-width centrato e padding laterale
    <div className="max-w-5xl mx-auto px-4 py-10">
      <Header
        totaleNote={note.length}
        onToggleDark={() => setIsDark(p => !p)}
        isDark={isDark}
      />
      <NoteForm onAggiungi={aggiungiNota} />
      <NoteList note={note} onElimina={eliminaNota} />
    </div>
  );
}

export default App;
```

Per usare classi `dark:` in un componente, aggiungi semplicemente il prefisso:

```jsx
// esempio dark mode su un elemento
<div className="bg-white dark:bg-surface text-black dark:text-white">
  ...
</div>
```

---

## 9. Esercizi di consolidamento

### A — Badge tag con `clsx` (medio)

Installa la libreria `clsx`:

```bash
npm install clsx
```

Refactoring: sostituisci la logica dei `TAG_STILI` in `NoteCard.jsx` usando `clsx` al posto dell'oggetto di stringhe. Il risultato visivo deve essere identico.

```jsx
// hint — struttura con clsx
import clsx from 'clsx';

const tagClasse = clsx(
  'font-mono text-[11px] px-2 py-0.5 rounded',
  {
    'bg-cyan-900 text-cyan-400':   note.tag === 'react',
    'bg-amber-950 text-amber-400': note.tag === 'js',
    // ... aggiungi gli altri
  }
);
```

### B — Ricerca con highlight (avanzato)

Aggiungi un campo di ricerca in `App.jsx` che filtra le note per titolo (come nel Lab 03). Aggiungi queste classi Tailwind all'input di ricerca:

```jsx
<input
  className="
    w-full bg-surface border border-border rounded-lg
    px-4 py-3 font-mono text-sm text-white
    placeholder-[#2E3A52]
    focus:outline-none focus:ring-1 focus:ring-cyan-400/50
    mb-6
  "
  placeholder="🔍  Cerca nelle note..."
  value={query}
  onChange={e => setQuery(e.target.value)}
/>
```

### C — Animazione di entrata (sfida)

Aggiungi un'animazione fade-in alle NoteCard. Crea `src/components/NoteCard.module.css` (sì, insieme a Tailwind!):

```css
/* NoteCard.module.css */
.fadeIn {
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

In `NoteCard.jsx`, importa il modulo e combina la classe CSS Module con le classi Tailwind:

```jsx
import styles from './NoteCard.module.css';

// nella JSX
<div className={`${styles.fadeIn} bg-surface border border-border ...`}>
```

Questo dimostra che CSS Modules e Tailwind convivono perfettamente.

---

## 10. Checklist finale

Prima di proseguire con il Lab 05, verifica:

- [ ] `npm run dev` parte senza errori in console
- [ ] L'interfaccia mostra le note in griglia responsive (1/2/3 colonne)
- [ ] Il badge colorato del tag funziona per tutti i valori (`react`, `js`, `node`, `css`, `altro`)
- [ ] Il form si invia, aggiunge la nota in cima e resetta i campi
- [ ] Il pulsante elimina rimuove la nota corretta
- [ ] Il toggle dark/light aggiunge/rimuove la classe `dark` sull'elemento `<html>` (verificabile con DevTools → Elements)
- [ ] `npm run build` produce output senza errori

### Tabella riepilogo file modificati

| File | Azione | Note |
|------|--------|------|
| `src/index.css` | Sostituito | Aggiunto `@tailwind`, variabili CSS |
| `tailwind.config.js` | Creato + configurato | Content, tema custom, darkMode |
| `postcss.config.js` | Creato (automatico) | Non toccare |
| `src/components/NoteCard.jsx` | Refactoring Tailwind | Rimosso `.module.css` associato |
| `src/components/NoteList.jsx` | Refactoring Tailwind | Grid responsive |
| `src/components/Header.jsx` | Refactoring Tailwind | Toggle dark mode |
| `src/components/NoteForm.jsx` | Refactoring Tailwind | Layout responsive form |
| `src/App.jsx` | Aggiornato | Stato isDark + useEffect |

---

➡️ **Prossimo lab:** [Lab 05 — Routing con React Router v6](lab-05-routing.md)
