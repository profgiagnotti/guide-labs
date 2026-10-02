# Lab 01 — Setup dell'ambiente di sviluppo

> **Serie:** Costruisci app web moderne con React, Vite e Node.js  
> **Repo:** `https://github.com/profgiagnotti/sviluppoWeb-labs/tree/main/react-vite-guide`  
> **Articolo di riferimento:** [Lezione 1 — Introduzione allo stack e setup ambiente](https://profgiagnotti.it/blog-react-vite-nodejs-l01-introduzione/)  
> **Difficoltà:** ⭐☆☆☆☆ Principiante  
> **Tempo stimato:** 30–45 minuti  

---

## Obiettivi di questo lab

Al termine avrai:

- [ ] Node.js installato e funzionante sul tuo sistema
- [ ] npm verificato e aggiornato
- [ ] Visual Studio Code configurato con le estensioni essenziali per React
- [ ] Un progetto Vite + React creato e in esecuzione su `localhost:5173`
- [ ] Comprensione della struttura di ogni file generato da Vite

---

## Indice

1. [Installare Node.js](#1-installare-nodejs)
2. [Verificare npm e aggiornarlo](#2-verificare-npm-e-aggiornarlo)
3. [Installare Visual Studio Code](#3-installare-visual-studio-code)
4. [Configurare VSCode — estensioni essenziali](#4-configurare-vscode--estensioni-essenziali)
5. [Creare il primo progetto Vite + React](#5-creare-il-primo-progetto-vite--react)
6. [Struttura del progetto — file per file](#6-struttura-del-progetto--file-per-file)
7. [Avviare il dev server](#7-avviare-il-dev-server)
8. [Prima modifica — HMR in azione](#8-prima-modifica--hmr-in-azione)
9. [Capire package.json](#9-capire-packagejson)
10. [Checklist finale](#10-checklist-finale)

---

## 1. Installare Node.js

Node.js è il runtime che esegue JavaScript fuori dal browser. Includerà automaticamente **npm** (Node Package Manager), lo strumento per installare e gestire le dipendenze del progetto.

### Quale versione installare?

Usa sempre la versione **LTS** (Long Term Support): è la più stabile e supportata per almeno 2 anni. Evita le versioni "Current" per i progetti seri — sono più recenti ma meno testate.

Al momento della scrittura di questa guida, la versione LTS consigliata è **Node.js 22.x**.

---

### Installazione su Windows

**Metodo consigliato: installer ufficiale**

1. Vai su [nodejs.org](https://nodejs.org/en/download)
2. Clicca su **"LTS"** (il bottone verde a sinistra)
3. Scarica il file `.msi` per Windows
4. Esegui l'installer — accetta le impostazioni di default
5. Alla schermata "Tools for Native Modules" **spunta la casella** per installare automaticamente Chocolatey e gli strumenti di build (utile in futuro)
6. Riavvia il terminale (PowerShell o CMD) dopo l'installazione

**Metodo alternativo: winget** (solo Windows 11 o Windows 10 aggiornato)

Apri PowerShell come amministratore e lancia:

```powershell
winget install OpenJS.NodeJS.LTS
```

---

### Installazione su macOS

**Metodo consigliato: installer ufficiale**

1. Vai su [nodejs.org](https://nodejs.org/en/download)
2. Scarica il pacchetto `.pkg` per macOS
3. Esegui l'installer con le impostazioni di default

**Metodo alternativo: Homebrew** (se hai già Homebrew installato)

```bash
brew install node@22
brew link node@22
```

**Metodo per chi usa più versioni di Node.js: nvm**

Se lavori su più progetti che richiedono versioni diverse di Node, usa **nvm** (Node Version Manager):

```bash
# Installa nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Riapri il terminale, poi installa Node.js LTS
nvm install --lts
nvm use --lts
```

---

### Installazione su Linux (Ubuntu/Debian)

Il pacchetto Node.js nei repository ufficiali di Ubuntu è spesso troppo vecchio. Usa il repository ufficiale NodeSource:

```bash
# Aggiungi il repository NodeSource per Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Installa Node.js e npm
sudo apt-get install -y nodejs

# Verifica
node --version
npm --version
```

**Metodo alternativo: nvm su Linux**

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc   # o ~/.zshrc se usi Zsh
nvm install --lts
nvm use --lts
```

---

## 2. Verificare npm e aggiornarlo

Dopo l'installazione di Node.js, apri un **nuovo** terminale (importante: il terminale aperto prima dell'installazione non vede le nuove variabili d'ambiente).

Verifica che tutto funzioni:

```bash
# Versione Node.js — deve mostrare v22.x.x o superiore
node --version

# Versione npm — deve mostrare 10.x.x o superiore
npm --version
```

Aggiorna npm all'ultima versione (npm si aggiorna separatamente da Node.js):

```bash
npm install -g npm@latest
```

> **Cosa fa `-g`?** L'opzione `-g` installa il pacchetto in modo **globale**, rendendolo disponibile da qualunque cartella nel terminale — non solo nel progetto corrente. Usalo solo per strumenti da linea di comando (come `npm` stesso, o Vite CLI).

---

## 3. Installare Visual Studio Code

1. Vai su [code.visualstudio.com](https://code.visualstudio.com)
2. Scarica la versione per il tuo sistema operativo
3. Installa con le impostazioni di default

**Su Linux (Ubuntu/Debian)**, puoi installarlo via terminal:

```bash
# Scarica e installa il pacchetto .deb
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/
sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update
sudo apt install code
```

### Abilitare il comando `code` nel terminale (macOS)

Su macOS, dopo aver installato VSCode:
1. Apri VSCode
2. Premi `Cmd+Shift+P` per aprire la Command Palette
3. Digita `Shell Command: Install 'code' command in PATH`
4. Premi Invio

Da ora puoi aprire qualunque cartella in VSCode direttamente dal terminale:

```bash
code nome-cartella
# oppure, per aprire la cartella corrente:
code .
```

---

## 4. Configurare VSCode — estensioni essenziali

Apri VSCode e installa le seguenti estensioni. Per ogni estensione:
- Premi `Ctrl+P` (o `Cmd+P` su macOS)
- Digita `ext install [nome-estensione]`
- Premi Invio

Oppure vai nella barra laterale sull'icona Extensions (quadrati) e cerca per nome.

### Estensioni obbligatorie

| Estensione | ID | Funzione |
|---|---|---|
| **ES7+ React/Redux/React-Native snippets** | `dsznajder.es7-react-js-snippets` | Shortcut per creare componenti React (digita `rfce` + Tab) |
| **ESLint** | `dbaeumer.vscode-eslint` | Evidenzia errori e warning nel codice JavaScript/React in tempo reale |
| **Prettier - Code formatter** | `esbenp.prettier-vscode` | Formatta automaticamente il codice al salvataggio |
| **Auto Rename Tag** | `formulahendry.auto-rename-tag` | Rinomina automaticamente il tag di chiusura quando modifichi quello di apertura in JSX |
| **GitLens** | `eamodio.gitlens` | Mostra chi ha scritto ogni riga, confronta versioni, integrazione Git avanzata |

### Estensioni consigliate

| Estensione | ID | Funzione |
|---|---|---|
| **Bracket Pair Color DLW** | `BracketPairColorDlw.bracket-pair-color-dlw` | Colora le parentesi aperte/chiuse dello stesso colore — salva la vista |
| **Path Intellisense** | `christian-kohler.path-intellisense` | Autocompletamento per i percorsi dei file negli import |
| **Color Highlight** | `naumovs.color-highlight` | Mostra un quadratino del colore accanto ai codici hex nel CSS |
| **Thunder Client** | `rangav.vscode-thunder-client` | Client HTTP integrato in VSCode per testare le API REST (alternativa a Postman) |

### Configurare il formato automatico al salvataggio

Apri le impostazioni di VSCode (`Ctrl+,` o `Cmd+,`), clicca sull'icona JSON in alto a destra, e aggiungi:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.wordWrap": "on",
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

---

## 5. Creare il primo progetto Vite + React

Apri il terminale e naviga nella cartella dove vuoi creare i tuoi progetti (es. `Documenti/progetti`):

```bash
# Su Windows
cd C:\Users\TuoNome\Documenti\progetti

# Su macOS/Linux
cd ~/progetti
# Se la cartella non esiste: mkdir ~/progetti && cd ~/progetti
```

Crea il progetto con Vite:

```bash
npm create vite@latest devnotes -- --template react
```

> **Cosa fa questo comando?**  
> - `npm create vite@latest` — scarica ed esegue il generatore di progetti Vite (non installa nulla permanentemente)  
> - `devnotes` — il nome della cartella che verrà creata  
> - `-- --template react` — specifica il template da usare: React con JavaScript (non TypeScript)  

Dopo qualche secondo vedrai questo output:

```
Scaffolding project in /percorso/devnotes...

Done. Now run:

  cd devnotes
  npm install
  npm run dev
```

Segui le istruzioni:

```bash
# Entra nella cartella del progetto
cd devnotes

# Installa le dipendenze (legge package.json e scarica tutto in node_modules/)
npm install

# Avvia il dev server
npm run dev
```

---

## 6. Struttura del progetto — file per file

Prima di avviare il server, capiamo cosa ha generato Vite. Apri la cartella in VSCode:

```bash
code .
```

La struttura che vedrai è:

```
devnotes/
├── node_modules/          ← dipendenze installate (NON modificare, NON committare su Git)
├── public/
│   └── vite.svg           ← file statici serviti senza trasformazione (favicon, robots.txt, ecc.)
├── src/
│   ├── assets/
│   │   └── react.svg      ← immagini, font e altri asset usati nei componenti
│   ├── App.css            ← stili specifici del componente App
│   ├── App.jsx            ← il componente principale dell'applicazione
│   ├── index.css          ← stili globali dell'applicazione
│   └── main.jsx           ← punto di ingresso — monta React nel DOM
├── .gitignore             ← file e cartelle da non tracciare in Git
├── eslint.config.js       ← configurazione ESLint (regole di linting)
├── index.html             ← l'unica pagina HTML — il punto di partenza del browser
├── package.json           ← metadati del progetto + lista dipendenze + script npm
├── package-lock.json      ← versioni esatte delle dipendenze (generato automaticamente)
└── vite.config.js         ← configurazione di Vite (plugin, proxy, alias, ecc.)
```

### Analisi dei file chiave

**`index.html`** — L'unica pagina HTML dell'applicazione

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React</title>
  </head>
  <body>
    <div id="root"></div>                        <!-- ← React monta qui -->
    <script type="module" src="/src/main.jsx"></script>  <!-- ← punto di ingresso JS -->
  </body>
</html>
```

Il `<div id="root">` è la porta d'ingresso: React inserisce qui tutta l'interfaccia generata dinamicamente.

---

**`src/main.jsx`** — Il punto di ingresso JavaScript

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

Questo file fa tre cose:
1. Cerca nel DOM il `<div id="root">`
2. Crea una "root" React su quell'elemento
3. Renderizza il componente `<App />` dentro quella root

`StrictMode` è un wrapper che attiva avvisi extra durante lo sviluppo — non influisce sulla produzione.

---

**`src/App.jsx`** — Il componente principale

```jsx
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)   // ← il tuo primo useState!

  return (
    <>
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>Edit <code>src/App.jsx</code> and save to test HMR</p>
      </div>
    </>
  )
}

export default App
```

Nota l'estensione `.jsx`: indica che il file contiene JSX — sintassi che mescola JavaScript e HTML-like. Vite (tramite il plugin `@vitejs/plugin-react`) trasforma il JSX in JavaScript standard prima di servirlo al browser.

---

**`vite.config.js`** — La configurazione di Vite

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

Per ora è minimale. In futuro aggiungeremo la configurazione del proxy per comunicare con il backend e alias per gli import.

---

## 7. Avviare il dev server

Se non l'hai già fatto, avvia il dev server:

```bash
npm run dev
```

Dovresti vedere:

```
  VITE v6.x.x  ready in 312 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

Apri il browser su [http://localhost:5173](http://localhost:5173). Vedrai la pagina di default di Vite con i loghi React e Vite e un contatore cliccabile.

> **Porta già in uso?** Se la 5173 è occupata, Vite usa automaticamente la successiva libera (5174, 5175…). Se vuoi forzare una porta specifica, aggiungi in `vite.config.js`:
> ```javascript
> export default defineConfig({
>   plugins: [react()],
>   server: { port: 3000 }
> })
> ```

---

## 8. Prima modifica — HMR in azione

**HMR** (Hot Module Replacement) è la caratteristica di Vite che aggiorna il browser istantaneamente quando modifichi un file, senza ricaricare la pagina e senza perdere lo stato dei componenti.

Prova subito:

1. Tieni aperto il browser su `localhost:5173`
2. In VSCode apri `src/App.jsx`
3. Trova la riga con `<h1>Get started</h1>`
4. Modificala in `<h1>DevNotes — La mia prima app</h1>`
5. Salva il file (`Ctrl+S` o `Cmd+S`)

Il browser si aggiorna istantaneamente — senza F5, senza perdere il valore del contatore. Questa è la differenza rispetto all'approccio tradizionale con Webpack.

### Modifica più significativa — cambia l'intestazione e il colore

Apri `src/App.css` e aggiungi al contenuto con:

```css
#root {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #00D4FFaa);
}

.logo.react:hover {
  filter: drop-shadow(0 0 2em #61dafbaa);
}

h1 {
  color: #00D4FF;
  font-size: 2.4em;
  margin: 1rem 0;
}

.card {
  padding: 2em;
}

button {
  background-color: #001E2A;
  border: 1px solid #00D4FF;
  color: #00D4FF;
  padding: 0.75em 1.5em;
  border-radius: 6px;
  font-size: 1em;
  cursor: pointer;
  transition: background-color 200ms;
}

button:hover {
  background-color: #00D4FF22;
}
```

Salva — l'interfaccia cambia istantaneamente nel browser. Questo è il flusso di lavoro che userai per tutta la serie.

---

## 9. Capire package.json

`package.json` è il cuore del progetto: contiene tutte le informazioni su dipendenze, script e metadati.

```json
{
  "name": "devnotes",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@eslint/js": "^9.13.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.3.2",
    "eslint": "^9.13.0",
    "eslint-plugin-react-hooks": "^5.0.0",
    "eslint-plugin-react-refresh": "^0.4.14",
    "globals": "^15.11.0",
    "vite": "^6.0.1"
  }
}
```

**Cosa significa ogni sezione:**

| Campo | Spiegazione |
|---|---|
| `"type": "module"` | Abilita la sintassi ES Modules (`import/export`) in tutti i file `.js` del progetto |
| `scripts.dev` | Eseguito da `npm run dev` — avvia il dev server Vite |
| `scripts.build` | Eseguito da `npm run build` — compila il progetto per la produzione in `dist/` |
| `scripts.preview` | Eseguito da `npm run preview` — serve localmente la build di produzione per testarla |
| `scripts.lint` | Eseguito da `npm run lint` — analizza il codice con ESLint e mostra i problemi |
| `dependencies` | Librerie necessarie in **produzione** — incluse nel bundle finale |
| `devDependencies` | Librerie necessarie solo in **sviluppo** — non incluse nel bundle finale |

**`dependencies` vs `devDependencies`:**

- `react` e `react-dom` vanno in `dependencies` — servono nell'app in produzione
- `vite` e `eslint` vanno in `devDependencies` — servono solo durante lo sviluppo, non nell'app finale

Quando installi un nuovo pacchetto, usa:
```bash
npm install nome-pacchetto          # → va in dependencies
npm install nome-pacchetto --save-dev  # → va in devDependencies
# abbreviazione: npm install nome-pacchetto -D
```

---

## 10. Checklist finale

Verifica che tutto sia a posto prima di passare alla Lezione 2:

```bash
# Nel terminale, dalla cartella devnotes:

# ✅ Node.js installato
node --version   # v22.x.x o superiore

# ✅ npm aggiornato
npm --version    # 10.x.x o superiore

# ✅ Dev server funzionante
npm run dev      # deve mostrare "Local: http://localhost:5173/"

# ✅ Build funzionante (test rapido)
npm run build    # deve completare senza errori e creare la cartella dist/
```

In VSCode verifica che:
- [ ] L'estensione ESLint mostra un'icona verde nella barra inferiore
- [ ] Prettier formatta il file quando salvi (prova ad aggiungere spazi extra e salva)
- [ ] La sintassi JSX in `App.jsx` è colorata correttamente (se è tutto grigio manca il plugin JSX)

---

## Riepilogo

Hai configurato un ambiente di sviluppo completo per React con Vite. Ricapitolando:

| Strumento | Ruolo | Comando principale |
|---|---|---|
| **Node.js** | Runtime JS + gestore dipendenze | `node --version` |
| **npm** | Installa e gestisce pacchetti | `npm install`, `npm run dev` |
| **Vite** | Dev server + build tool | `npm run dev` (avvio), `npm run build` (produzione) |
| **VSCode** | IDE | `code .` (apri cartella) |

La struttura essenziale del progetto è:
- `index.html` → unica pagina HTML, punto d'ingresso
- `src/main.jsx` → monta React nel DOM
- `src/App.jsx` → il componente principale che modificherai
- `src/index.css` → stili globali

---

## Prossimo lab

**[Lab 02 — Componenti e JSX](./lab-02-componenti-jsx.md)**  
Esploriamo in dettaglio la sintassi JSX, creiamo i primi componenti personalizzati, capiamo come passare dati tra componenti con le props e iniziamo a costruire la struttura di DevNotes.

---

## Risorse

- [Node.js — Documentazione ufficiale](https://nodejs.org/en/docs)
- [Vite — Getting Started](https://vitejs.dev/guide/)
- [React — Tutorial ufficiale](https://react.dev/learn)
- [VSCode — Keyboard Shortcuts Reference](https://code.visualstudio.com/docs/getstarted/keybindings)
- [npm — Documentazione CLI](https://docs.npmjs.com/cli/v10)

---

*Guida a cura di [Prof. Giagnotti](https://profgiagnotti.it) · Serie: Sviluppo Web Moderno · Licenza [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)*
