# Lab 02 — Struttura del progetto e primi componenti

> **Serie:** Costruisci app web moderne con React, Vite e Node.js  
> **Repo:** `https://github.com/profgiagnotti/sviluppoWeb-labs/tree/main/react-vite-guide`  
> **Articolo di riferimento:** [Lezione 2 — Struttura del progetto e primi componenti](https://profgiagnotti.it/lezione-2-web-app-con-react-vite-nodejs-componenti-jsx/)  
> **Difficoltà:** ⭐⭐☆☆☆ Principiante+  
> **Tempo stimato:** 45–60 minuti  

---

## Obiettivi di questo lab

Al termine avrai:

- [ ] Progetto DevNotes riorganizzato con struttura cartelle professionale
- [ ] Componente `Header` creato con logo e titolo dell'app
- [ ] Componente `NoteCard` creato con props: titolo, contenuto, tag, data
- [ ] Componente `NoteList` che renderizza un array di note con `.map()`
- [ ] `App.jsx` aggiornato che compone tutti i componenti insieme
- [ ] Comprensione pratica di JSX, props e export

---

## Indice

1. [Pulizia del progetto di default](#1-pulizia-del-progetto-di-default)
2. [Creare la struttura delle cartelle](#2-creare-la-struttura-delle-cartelle)
3. [Componente Header](#3-componente-header)
4. [Componente NoteCard](#4-componente-notecard)
5. [Componente NoteList](#5-componente-notelist)
6. [Aggiornare App.jsx](#6-aggiornare-appjsx)
7. [Aggiornare index.css con gli stili globali](#7-aggiornare-indexcss-con-gli-stili-globali)
8. [Rendering condizionale e liste con .map()](#8-rendering-condizionale-e-liste-con-map)
9. [Esercizi di consolidamento](#9-esercizi-di-consolidamento)
10. [Checklist finale](#10-checklist-finale)

---

## Prerequisiti

- Lab 01 completato: Node.js installato, progetto `devnotes` creato, dev server funzionante
- VS Code aperto sulla cartella `devnotes`
- Dev server in esecuzione (`npm run dev` nel terminale)

---

## 1. Pulizia del progetto di default

Vite genera del codice di esempio che non ci serve. Partiamo da una base pulita.

**Elimina i file che non useremo:**

```bash
# Dalla cartella devnotes — terminale integrato di VSCode (Ctrl+` o Cmd+`)

# Elimina i file di esempio
rm src/assets/react.svg
rm src/assets/vite.svg
```

Su Windows (PowerShell):
```powershell
Remove-Item src\assets\react.svg
Remove-Item public\vite.svg
```

**Svuota `src/App.css`** — seleziona tutto il contenuto e cancella. Il file deve essere vuoto (lo riempiremo dopo).

**Sostituisci `src/App.jsx`** con questo contenuto minimale:

```jsx
function App() {
  return (
    <div className="app">
      <h1>DevNotes</h1>
    </div>
  )
}

export default App
```

Salva e verifica nel browser: deve comparire la scritta "DevNotes" su sfondo bianco (o il tema di default del tuo sistema). Nessun errore in console.

---

## 2. Creare la struttura delle cartelle

Crea le cartelle con questi comandi nel terminale:

```bash
# Dalla cartella devnotes
mkdir -p src/components/Header
mkdir -p src/components/NoteCard
mkdir -p src/components/NoteList
mkdir -p src/pages
mkdir -p src/utils
```

Su Windows (PowerShell):
```powershell
New-Item -ItemType Directory -Force -Path src\components\Header
New-Item -ItemType Directory -Force -Path src\components\NoteCard
New-Item -ItemType Directory -Force -Path src\components\NoteList
New-Item -ItemType Directory -Force -Path src\pages
New-Item -ItemType Directory -Force -Path src\utils
```

La struttura risultante sarà:

```
src/
├── components/
│   ├── Header/
│   ├── NoteCard/
│   └── NoteList/
├── pages/
├── utils/
├── App.jsx
├── App.css
├── index.css
└── main.jsx
```

> **Tip VSCode:** puoi creare cartelle e file anche dal pannello Explorer (barra laterale sinistra): click destro → "New Folder" o "New File". È spesso più comodo del terminale per piccole operazioni.

---

## 3. Componente Header

Crea i due file nella cartella `src/components/Header/`:

### `src/components/Header/Header.jsx`

```jsx
import styles from './Header.module.css'

function Header() {
  return (
    <header className={styles.header}>
      <div className={styles.brand}>
        <span className={styles.logo}>📝</span>
        <span className={styles.nome}>DevNotes</span>
      </div>
      <nav className={styles.nav}>
        <a href="/" className={styles.link}>Le mie note</a>
        <a href="/nuova" className={styles.link}>+ Nuova nota</a>
      </nav>
    </header>
  )
}

export default Header
```

**Cosa sta succedendo:**
- `import styles from './Header.module.css'` importa il file CSS Modules (lo creiamo subito)
- `className={styles.header}` applica la classe CSS `.header` definita nel modulo — il nome è scoped a questo componente, non può scontrarsi con classi di altri componenti
- Il componente non riceve props: il suo contenuto è fisso (per ora — più avanti potremmo passare il titolo o il link attivo come prop)

### `src/components/Header/Header.module.css`

```css
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  height: 60px;
  background-color: #001E2A;
  border-bottom: 1px solid #0A3A52;
  position: sticky;
  top: 0;
  z-index: 100;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  text-decoration: none;
}

.logo {
  font-size: 1.4rem;
}

.nome {
  font-family: 'Syne', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
  color: #00D4FF;
  letter-spacing: 0.04em;
}

.nav {
  display: flex;
  align-items: center;
  gap: 20px;
}

.link {
  font-family: 'JetBrains Mono', monospace;
  font-size: 13px;
  color: #6B7A99;
  text-decoration: none;
  padding: 6px 12px;
  border-radius: 4px;
  transition: color 200ms, background-color 200ms;
}

.link:hover {
  color: #00D4FF;
  background-color: rgba(0, 212, 255, 0.08);
}
```

> **Cosa sono i CSS Modules?** Un file `.module.css` è un CSS normale con un superpotere: i nomi delle classi vengono rinominati automaticamente con un hash univoco durante la build (es. `.header` diventa `Header_header__xK3aP`). Questo garantisce che due componenti diversi possano usare lo stesso nome di classe (`.card`, `.btn`, ecc.) senza conflitti. Si importano come oggetto JavaScript: `styles.header` restituisce il nome della classe generata.

---

## 4. Componente NoteCard

Crea i due file in `src/components/NoteCard/`:

### `src/components/NoteCard/NoteCard.jsx`

```jsx
import styles from './NoteCard.module.css'

// Funzione utility per formattare la data in italiano
function formattaData(dataStringa) {
  const data = new Date(dataStringa)
  return data.toLocaleDateString('it-IT', {
    day: '2-digit',
    month: 'short',
    year: 'numeric',
  })
}

// Mappa di colori per i tag — aggiungine altri se vuoi
const TAG_COLORI = {
  React: '#00D4FF',
  'Node.js': '#3DE8A0',
  CSS: '#BC8CFF',
  JavaScript: '#FFAA3D',
  default: '#6B7A99',
}

function NoteCard({ titolo, contenuto, tag, data, onElimina }) {
  // Determina il colore del tag (usa 'default' se il tag non è mappato)
  const coloreTag = TAG_COLORI[tag] || TAG_COLORI.default

  return (
    <article className={styles.card}>

      {/* Header della card: tag + bottone elimina */}
      <div className={styles.cardHeader}>
        <span
          className={styles.tag}
          style={{ color: coloreTag, borderColor: coloreTag }}
        >
          {tag}
        </span>
        {/* onElimina è una prop funzione — la chiama quando il bottone viene cliccato */}
        {onElimina && (
          <button
            className={styles.btnElimina}
            onClick={onElimina}
            aria-label={`Elimina nota: ${titolo}`}
          >
            ✕
          </button>
        )}
      </div>

      {/* Corpo della card */}
      <h3 className={styles.titolo}>{titolo}</h3>
      <p className={styles.contenuto}>{contenuto}</p>

      {/* Footer: data di creazione */}
      <footer className={styles.cardFooter}>
        <time className={styles.data} dateTime={data}>
          {formattaData(data)}
        </time>
      </footer>

    </article>
  )
}

export default NoteCard
```

**Osservazioni importanti:**

1. **`formattaData`** è una funzione helper definita *fuori* dal componente — è logica pura, non dipende dallo stato o dalle props, quindi non c'è bisogno di metterla dentro.
2. **`TAG_COLORI`** è una costante definita fuori dal componente — viene creata una sola volta all'avvio del modulo, non ad ogni render.
3. **`onElimina && (...)`** è il pattern per il rendering condizionale: se `onElimina` non viene passato come prop, il bottone non viene renderizzato. Questo rende il componente flessibile — funziona con o senza la funzione di eliminazione.
4. **`style={{ color: coloreTag }}`** — le doppie graffe: quella esterna è la "finestra su JS" di JSX, quella interna è l'oggetto JavaScript dello stile.

### `src/components/NoteCard/NoteCard.module.css`

```css
.card {
  background-color: #0E1118;
  border: 1px solid #1E2535;
  border-radius: 10px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  transition: border-color 200ms, transform 200ms;
  cursor: default;
}

.card:hover {
  border-color: #00D4FF;
  transform: translateY(-2px);
}

.cardHeader {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.tag {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  border: 1px solid;
  border-radius: 4px;
  padding: 3px 8px;
  background-color: transparent;
}

.btnElimina {
  background: none;
  border: none;
  color: #2E3A52;
  font-size: 14px;
  cursor: pointer;
  padding: 4px 6px;
  border-radius: 4px;
  line-height: 1;
  transition: color 200ms, background-color 200ms;
}

.btnElimina:hover {
  color: #FF5C5C;
  background-color: rgba(255, 92, 92, 0.1);
}

.titolo {
  font-family: 'Syne', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  color: #DCE6F5;
  line-height: 1.4;
  margin: 0;
}

.contenuto {
  font-size: 14px;
  color: #6B7A99;
  line-height: 1.7;
  margin: 0;
  /* Limita a 3 righe con ellissi */
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.cardFooter {
  margin-top: auto; /* spinge il footer in fondo alla card */
  padding-top: 12px;
  border-top: 1px solid #1E2535;
}

.data {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: #2E3A52;
}
```

---

## 5. Componente NoteList

Crea i due file in `src/components/NoteList/`:

### `src/components/NoteList/NoteList.jsx`

```jsx
import NoteCard from '../NoteCard/NoteCard'
import styles from './NoteList.module.css'

function NoteList({ note, onEliminaNota }) {
  // Rendering condizionale: lista vuota
  if (note.length === 0) {
    return (
      <div className={styles.vuoto}>
        <span className={styles.vuotoIcon}>📭</span>
        <p className={styles.vuotoTesto}>Nessuna nota trovata.</p>
        <p className={styles.vuotoSub}>Crea la tua prima nota cliccando su &ldquo;+ Nuova nota&rdquo;.</p>
      </div>
    )
  }

  return (
    <section className={styles.lista}>
      {/* .map() trasforma ogni oggetto nota in un componente NoteCard */}
      {note.map((nota) => (
        <NoteCard
          key={nota.id}           // ← prop KEY: obbligatoria nelle liste, mai usare l'indice
          titolo={nota.titolo}
          contenuto={nota.contenuto}
          tag={nota.tag}
          data={nota.data}
          onElimina={() => onEliminaNota(nota.id)}  // ← callback con id della nota da eliminare
        />
      ))}
    </section>
  )
}

export default NoteList
```

**La prop `key` — perché è obbligatoria:**

Quando renderizzi una lista con `.map()`, React ha bisogno di un modo per identificare ogni elemento in modo univoco. La prop `key` serve a questo: permette a React di capire quali elementi sono stati aggiunti, rimossi o riordinati, ottimizzando gli aggiornamenti al DOM. Le regole:
- Deve essere **unica** tra i fratelli della lista (non globalmente)
- Deve essere **stabile** nel tempo (non cambiare tra i render)
- **Mai usare l'indice del `.map()`** come key se la lista può cambiare ordine o avere elementi aggiunti/rimossi in mezzo — causa bug di rendering difficili da diagnosticare
- La soluzione corretta: usa un ID univoco dal dato (come `nota.id`)

### `src/components/NoteList/NoteList.module.css`

```css
.lista {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  padding: 24px;
}

/* Responsive: su schermi piccoli una sola colonna */
@media (max-width: 640px) {
  .lista {
    grid-template-columns: 1fr;
    padding: 16px;
  }
}

.vuoto {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 24px;
  text-align: center;
}

.vuotoIcon {
  font-size: 3rem;
  margin-bottom: 16px;
}

.vuotoTesto {
  font-family: 'Syne', sans-serif;
  font-size: 1.1rem;
  font-weight: 600;
  color: #DCE6F5;
  margin: 0 0 8px;
}

.vuotoSub {
  font-size: 14px;
  color: #6B7A99;
  margin: 0;
}
```

---

## 6. Aggiornare App.jsx

Ora assembla tutto in `src/App.jsx`. Definiremo un array di note statiche — dati "hardcodati" che useremo fino alla lezione 3, dove li sposteremo nello stato.

```jsx
import Header from './components/Header/Header'
import NoteList from './components/NoteList/NoteList'
import './App.css'

// Dati statici di esempio — nella lezione 3 questi diventeranno useState
const NOTE_ESEMPIO = [
  {
    id: 1,
    titolo: 'Setup ambiente di sviluppo',
    contenuto: 'Node.js installato, VSCode configurato con ESLint e Prettier. Progetto Vite + React creato con npm create vite@latest.',
    tag: 'React',
    data: '2026-04-11',
  },
  {
    id: 2,
    titolo: 'Componenti e JSX',
    contenuto: 'JSX non è HTML: className invece di class, attributi camelCase, un solo elemento radice. Le graffe {} aprono espressioni JavaScript dentro il markup.',
    tag: 'React',
    data: '2026-04-11',
  },
  {
    id: 3,
    titolo: 'Installare Node.js su macOS',
    contenuto: 'Usare Homebrew: brew install node@22 && brew link node@22. Oppure nvm per gestire più versioni: nvm install --lts.',
    tag: 'Node.js',
    data: '2026-04-10',
  },
  {
    id: 4,
    titolo: 'Grid CSS con auto-fill',
    contenuto: 'grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)) crea una griglia responsive senza media query: le colonne si adattano automaticamente.',
    tag: 'CSS',
    data: '2026-04-09',
  },
]

function App() {
  // Funzione placeholder — nella lezione 3 useremo setState
  function handleEliminaNota(id) {
    console.log('Elimina nota con id:', id)
    // TODO: nella lezione 3 rimuoveremo la nota dallo stato
  }

  return (
    <div className="app">
      <Header />
      <main className="main-content">
        <NoteList
          note={NOTE_ESEMPIO}
          onEliminaNota={handleEliminaNota}
        />
      </main>
    </div>
  )
}

export default App
```

---

## 7. Aggiornare index.css con gli stili globali

Sostituisci tutto il contenuto di `src/index.css`:

```css
/* Reset e box-model */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* Font dalle Google Fonts — assicurati di averle nel index.html o importale qui */
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;600;700&family=Syne:wght@400;600;700;800&display=swap');

/* Variabili CSS — il design system dell'app */
:root {
  --bg-void: #07090F;
  --bg-surface: #0E1118;
  --bg-surface2: #151A24;
  --border: #1E2535;
  --color-primary: #00D4FF;
  --color-green: #3DE8A0;
  --color-amber: #FFAA3D;
  --color-red: #FF5C5C;
  --color-purple: #BC8CFF;
  --text-primary: #DCE6F5;
  --text-secondary: #A8B8D0;
  --text-muted: #6B7A99;
  --text-dim: #2E3A52;
}

/* Base */
html {
  font-size: 16px;
  scroll-behavior: smooth;
}

body {
  font-family: 'Syne', sans-serif;
  background-color: var(--bg-void);
  color: var(--text-secondary);
  line-height: 1.6;
  min-height: 100vh;
}

/* Layout principale dell'app */
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
}

/* Link di default */
a {
  color: var(--color-primary);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}
```

Salva tutti i file e controlla il browser. Dovresti vedere:
- Header blu scuro con logo 📝 e navigazione
- Griglia di 4 card con note di esempio
- Hover sulle card con bordo azzurro e leggero sollevamento
- Bottone ✕ su ogni card (che per ora logga in console)

---

## 8. Rendering condizionale e liste con .map()

Ora che il progetto funziona, proviamo a vedere i due pattern di rendering condizionale più usati in React.

### Pattern 1 — Operatore ternario

Modifica temporaneamente `App.jsx` per testare la lista vuota: sostituisci `NOTE_ESEMPIO` con un array vuoto nel prop `note`:

```jsx
<NoteList
  note={[]}           {/* ← array vuoto per testare */}
  onEliminaNota={handleEliminaNota}
/>
```

Salva: dovresti vedere il messaggio "Nessuna nota trovata". Ripristina `note={NOTE_ESEMPIO}`.

### Pattern 2 — Operatore `&&` (rendering condizionale breve)

Aggiungi un contatore in cima alla lista in `NoteList.jsx`. Prima del `return` principale, aggiungi:

```jsx
// Aggiunta dentro NoteList, subito prima del return con la griglia
return (
  <div>
    {/* Mostra il contatore solo se ci sono note */}
    {note.length > 0 && (
      <p style={{
        fontFamily: "'JetBrains Mono', monospace",
        fontSize: '12px',
        color: '#6B7A99',
        padding: '12px 24px 0',
      }}>
        {note.length} {note.length === 1 ? 'nota' : 'note'}
      </p>
    )}
    <section className={styles.lista}>
      {note.map((nota) => (
        <NoteCard
          key={nota.id}
          titolo={nota.titolo}
          contenuto={nota.contenuto}
          tag={nota.tag}
          data={nota.data}
          onElimina={() => onEliminaNota(nota.id)}
        />
      ))}
    </section>
  </div>
)
```

Salva e verifica: sopra le card compare "4 note". Prova con `note={[]}` — il contatore scompare, appare il messaggio vuoto.

---

## 9. Esercizi di consolidamento

Completa almeno i primi due prima di passare alla Lezione 3.

### Esercizio A — Prop aggiuntiva: priorità ⭐

Aggiungi una prop `priorità` a `NoteCard` con valori `'alta'`, `'media'`, `'bassa'`. Mostrala come un indicatore visivo nell'header della card (es. un pallino colorato: rosso/arancio/verde). Aggiorna i dati di esempio in `App.jsx` aggiungendo il campo `priorita` a ogni nota.

<details>
<summary>Suggerimento</summary>

```jsx
const PRIORITA_COLORI = {
  alta: '#FF5C5C',
  media: '#FFAA3D',
  bassa: '#3DE8A0',
}

// Nel JSX di NoteCard, nell'header:
<span
  style={{
    width: '8px',
    height: '8px',
    borderRadius: '50%',
    backgroundColor: PRIORITA_COLORI[priorita] || '#2E3A52',
    display: 'inline-block',
  }}
  title={`Priorità: ${priorita}`}
/>
```

</details>

---

### Esercizio B — Componente Badge riutilizzabile

Estrai il tag colorato di `NoteCard` in un componente separato chiamato `Badge` in `src/components/Badge/Badge.jsx`. Il componente riceve `testo` e `colore` come props. Usa `Badge` dentro `NoteCard` al posto del codice inline.

Questo esercizio mostra come si "scompone" un componente esistente in parti più piccole e riutilizzabili — un'abilità fondamentale in React.

---

### Esercizio C — Filtraggio per tag (sfida)

In `App.jsx`, aggiungi un array di bottoni sopra la `NoteList`, uno per ogni tag unico presente nelle note (`'React'`, `'Node.js'`, `'CSS'`, `'JavaScript'`, più un bottone "Tutte"). Quando si clicca un bottone, filtra le note mostrate. 

Per ora **non usare ancora useState** (lo vedremo nella lezione 3): usa una variabile `let filtroAttivo = 'Tutte'` e usa `.filter()` su `NOTE_ESEMPIO`. Nota come il filtro non funziona ancora in modo interattivo — questo è il punto che ti farà capire *perché* abbiamo bisogno di useState nella prossima lezione.

---

## 10. Checklist finale

Verifica nel browser:

- [ ] Header visibile con logo e navigazione
- [ ] 4 card renderizzate con titolo, contenuto, tag colorato e data
- [ ] Hover su card: bordo azzurro + leggero sollevamento
- [ ] Bottone ✕ visibile, click logga in console `Elimina nota con id: X`
- [ ] Testa `note={[]}` in App.jsx: compare il messaggio "Nessuna nota trovata"
- [ ] Nessun warning in console (tranne eventuali errori di Google Fonts se offline)

Verifica nel codice:

- [ ] Ogni componente è in una sotto-cartella separata con il suo CSS Module
- [ ] Tutti i file JSX usano `export default`
- [ ] La prop `key` nelle liste usa `nota.id`, non l'indice del `.map()`
- [ ] Nessuna logica JavaScript dentro le graffe JSX (solo espressioni)

---

## Struttura finale del progetto

Alla fine di questo lab la struttura di `src/` è:

```
src/
├── components/
│   ├── Header/
│   │   ├── Header.jsx
│   │   └── Header.module.css
│   ├── NoteCard/
│   │   ├── NoteCard.jsx
│   │   └── NoteCard.module.css
│   └── NoteList/
│       ├── NoteList.jsx
│       └── NoteList.module.css
├── pages/           (vuota per ora, usata dalla lezione 5)
├── utils/           (vuota per ora, usata dalla lezione 6)
├── App.jsx
├── App.css          (vuoto, stili specifici di App se necessari)
├── index.css        (stili globali + variabili CSS)
└── main.jsx         (invariato da Vite)
```

---

## Riepilogo

| Concetto | Dove l'hai visto | Da ricordare |
|---|---|---|
| **JSX** | Tutti i file `.jsx` | Viene compilato in `React.createElement()` — non è HTML |
| **className** | Tutti i CSS Modules | Usa sempre `className`, mai `class` in JSX |
| **CSS Modules** | `*.module.css` | Le classi sono scoped al componente — niente conflitti |
| **Props** | `NoteCard`, `NoteList` | Flusso unidirezionale genitore → figlio, si destrutturano nel parametro |
| **children** | Pattern generico | Tutto dentro i tag del componente diventa la prop `children` |
| **key** | `.map()` in `NoteList` | Obbligatoria nelle liste, usa ID univoco dal dato — mai l'indice |
| **Rendering condizionale** | `NoteList` vuota, contatore | Usa `&&` per on/off, ternario `?:` per on/altro |
| **export default** | Fine di ogni file | Un componente principale per file |

---

## Prossimo lab

**[Lab 03 — Stato e interattività con hooks](./lab-03-state-hooks.md)**  
Rendiamo DevNotes interattiva: le note si aggiungono e si eliminano. Impariamo `useState` per gestire i dati dell'interfaccia e `useEffect` per sincronizzare lo stato con fonti esterne. Alla fine della lezione 3, il bottone ✕ eliminerà davvero le note, e ci sarà un form per aggiungerne di nuove.

---

## Risorse

- [React Docs — Your First Component](https://react.dev/learn/your-first-component)
- [React Docs — Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [React Docs — Rendering Lists](https://react.dev/learn/rendering-lists)
- [React Docs — Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [MDN — CSS Modules (via Vite)](https://vitejs.dev/guide/features#css-modules)

---

*Guida a cura di [Prof. Giagnotti](https://profgiagnotti.it) · Serie: Sviluppo Web Moderno · Licenza [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)*
