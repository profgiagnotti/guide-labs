# 🖥️ Assembly 8086 — Cheatsheet Completo

> **Prof. Giagnotti** · [profgiagnotti.it](https://profgiagnotti.it)
> Livello: ⭐ Base → ⭐⭐⭐ Avanzato · Aggiornato: 2026

---

## 📑 Indice

- [Registri](#-registri)
- [Struttura del Programma](#-struttura-del-programma)
- [Dati e Variabili](#-dati-e-variabili)
- [Trasferimento Dati](#-trasferimento-dati)
- [Istruzioni Aritmetiche](#-istruzioni-aritmetiche)
- [Logica, Shift e Rotazione](#-logica-shift-e-rotazione)
- [Flag](#-flag-e-registro-flags)
- [Controllo del Flusso](#-controllo-del-flusso)
- [Cicli](#-cicli)
- [Stack](#-stack)
- [Subroutine](#-subroutine)
- [Interrupt e I/O](#-interrupt-e-io)
- [Modi di Indirizzamento](#-modi-di-indirizzamento)
- [Operazioni su Stringhe](#-operazioni-su-stringhe)
- [Tips e Idiomi](#-tips-e-idiomi)

---

## 🗃️ Registri

### Registri General Purpose (16 bit)

| Registro | Parte alta | Parte bassa | Nome | Uso convenzionale |
|----------|-----------|-------------|------|-------------------|
| `AX` | `AH` (bit 15–8) | `AL` (bit 7–0) | Accumulator | Aritmetica, I/O, valore di ritorno |
| `BX` | `BH` | `BL` | Base | Indirizzamento base, puntatori strutture |
| `CX` | `CH` | `CL` | Counter | Contatore cicli LOOP, shift con `CL` |
| `DX` | `DH` | `DL` | Data | I/O porte, parte alta MUL/DIV (`DX:AX`) |

### Registri Indice e Puntatori

| Registro | Nome | Uso |
|----------|------|-----|
| `SI` | Source Index | Indice sorgente — indirizzamento indiretto, stringhe |
| `DI` | Destination Index | Indice destinazione — indirizzamento indiretto, stringhe |
| `BP` | Base Pointer | Accesso parametri nello stack frame (SS per default) |
| `SP` | Stack Pointer | Punta alla cima dello stack — gestito da PUSH/POP |
| `IP` | Instruction Pointer | Indirizzo prossima istruzione — **non modificabile direttamente** |

### Registri di Segmento

| Registro | Nome | Punta a |
|----------|------|---------|
| `CS` | Code Segment | Segmento codice — istruzioni in esecuzione |
| `DS` | Data Segment | Segmento dati — variabili del programma |
| `SS` | Stack Segment | Segmento stack — PUSH/POP/CALL/RET |
| `ES` | Extra Segment | Segmento extra — operazioni su stringhe (DI) |

> ⚠️ **Indirizzo fisico 8086** → l'8086 ha bus indirizzi a 20 bit (1 MB max).
> Formula: `indirizzo_fisico = segmento × 10h + offset`
> Esempio: DS=1000h, offset=50h → `10050h`

---

## 📄 Struttura del Programma

### Template .EXE — Direttive Standard ⭐

```asm
; ── TEMPLATE COMPLETO .EXE ──────────────────────────────────────────────
data SEGMENT
  variabile  DB  42d           ; dichiarazione variabile
  COSTANTE   EQU 10h           ; costante simbolica
data ENDS

stack SEGMENT STACK
  DB 100h DUP(?)               ; riserva 256 byte per lo stack
stack ENDS

code SEGMENT
ASSUME CS:code, DS:data, SS:stack

start:
  MOV AX, data                 ; carica indirizzo del Data Segment
  MOV DS, AX                   ; inizializza DS — OBBLIGATORIO!

  ; ── codice qui ──────────────────────────────────────────────────────

  MOV AH, 4Ch                  ; INT 21h servizio 4Ch: termina
  INT 21h

code ENDS
END start                      ; fine file — label di inizio
```

### Template .COM — ORG 100h ⭐

```asm
; ── TEMPLATE .COM ───────────────────────────────────────────────────────
ORG 100h                       ; codice parte da 100h (PSP nei primi 256 byte)

start:
  ; DS già puntato al segmento corrente — non serve inizializzare
  MOV AH, 4Ch
  INT 21h
```

### Direttive Semplificate — `.model`

```asm
.model small        ; definisce il modello di memoria
.stack 100h         ; riserva stack
.data               ; inizio data segment
  ; variabili
.code               ; inizio code segment
start:
  MOV AX, @data     ; @data = indirizzo automatico del data segment
  MOV DS, AX
  ; codice
  MOV AH, 4Ch
  INT 21h
END start
```

| Modello | Segmenti codice | Segmenti dati | Uso tipico |
|---------|----------------|---------------|------------|
| `tiny`  | 1 (condiviso)  | 1 (condiviso) | File .COM — tutto in un segmento |
| `small` | 1              | 1             | Programmi piccoli — **il più usato in corso** |
| `compact` | 1            | multipli      | Dati grandi, codice piccolo |
| `medium` | multipli      | 1             | Codice grande, dati piccoli |
| `large` | multipli       | multipli      | Applicazioni grandi e complesse |

> 🚫 **Vincolo DS** → `MOV DS, 1000h` è **ILLEGALE**.
> I registri di segmento non accettano valori immediati.
> Obbligatorio passare per un registro: `MOV AX, data` → `MOV DS, AX`

---

## 💾 Dati e Variabili

### Direttive di Definizione

| Direttiva | Dimensione | Bit | Range (senza segno) |
|-----------|-----------|-----|---------------------|
| `DB` | Define Byte | 8 bit | 0 – 255 |
| `DW` | Define Word | 16 bit | 0 – 65.535 |
| `DD` | Define Double-Word | 32 bit | 0 – 4.294.967.295 |
| `DQ` | Define Quad-Word | 64 bit | molto grande |
| `DT` | Define Ten-Byte | 80 bit | BCD packed |

```asm
; ── ESEMPI DICHIARAZIONE ────────────────────────────────────────────────

; variabili inizializzate — diverse basi numeriche
a         DB  67d              ; decimale
b         DB  43h              ; esadecimale
c         DB  53o              ; ottale
d         DB  1001b            ; binario
n         DW  1000d            ; word (2 byte)

; variabili non inizializzate
risultato DB  ?                ; 1 byte — valore indefinito a runtime
somma     DW  ?                ; 2 byte — valore indefinito a runtime

; costanti — non occupano memoria (sostituzione a compile-time)
MAX       EQU 100d
MASK      EQU 0Fh

; array e buffer
voti      DB  28d, 30d, 25d   ; array di 3 byte
buffer    DB  16 DUP(?)        ; 16 byte non inizializzati
zeros     DB  8  DUP(0)        ; 8 byte azzerati

; stringhe — terminare con $ per INT 21h servizio 09h
msg       DB  'Ciao!', '$'
newline   DB  13, 10, '$'      ; CR+LF — a capo
```

### Basi Numeriche in EMU8086

| Base | Suffisso | Esempio | Valore decimale |
|------|----------|---------|----------------|
| Decimale | `d` (o niente) | `67d` | 67 |
| Esadecimale | `h` | `43h` | 67 |
| Ottale | `o` | `103o` | 67 |
| Binario | `b` | `1000011b` | 67 |

> 💡 **EQU vs DB** → `EQU` è una costante simbolica sostituita a compile-time e **non occupa memoria**.
> `DB/DW` è una variabile e occupa locazioni reali nel Data Segment.

---

## 📦 Trasferimento Dati

```asm
; MOV — copia valore (non modifica i flag)
MOV AL, 34d               ; immediato → registro
MOV AX, BX                ; registro → registro
MOV AX, [6Ah]             ; memoria → registro (indirizzo diretto)
MOV [0200h], BX           ; registro → memoria
MOV [BX], 0Fh             ; immediato → memoria (via puntatore)
MOV AX, variabile         ; variabile del data segment → AX

; XCHG — scambio senza registro temporaneo
XCHG AX, BX               ; AX ↔ BX in un'unica istruzione
XCHG AX, [BX]             ; scambio con memoria

; LEA — carica indirizzo effettivo (non il valore)
LEA  SI, array            ; SI ← &array  (come & in C)
LEA  DI, 4h[BX]           ; DI ← BX + 4  (indirizzo calcolato)
LEA  DX, messaggio        ; usato per preparare DS:DX per INT 21h/09h

; PUSH / POP — stack (SP decrementato di 2 ad ogni PUSH)
PUSH AX                   ; SP ← SP-2 · Mem[SS:SP] ← AX
PUSH 1234h                ; immediato nello stack
POP  AX                   ; AX ← Mem[SS:SP] · SP ← SP+2
PUSHF                     ; salva FLAGS nello stack
POPF                      ; ripristina FLAGS dallo stack
```

> 🚫 **Vincolo MOV** → entrambi gli operandi **non possono essere in memoria**.
> `MOV [dest], [src]` è illegale.
> Usa due istruzioni: `MOV AL, [src]` poi `MOV [dest], AL`

---

## 🔢 Istruzioni Aritmetiche

```asm
; ADD / SUB — aggiornano tutti i flag (ZF, SF, CF, OF, PF, AF)
ADD AL, 5d                ; AL = AL + 5
ADD AX, BX                ; AX = AX + BX
ADD AX, [BX]              ; AX = AX + Mem[BX]
SUB AL, 10d               ; AL = AL - 10 (se neg → complemento a 2)
SUB AX, variabile         ; AX = AX - variabile

; ADC / SBB — con carry/borrow (somme a 32 bit su CPU 16 bit)
ADD AX, BX                ; somma parte bassa — può generare CF
ADC DX, CX                ; somma parte alta + CF  →  (DX:AX) = (DX:AX) + (CX:BX)
SUB AX, BX                ; sottrae parte bassa
SBB DX, CX                ; sottrae parte alta con eventuale borrow

; INC / DEC — non modificano CF (utile nei cicli)
INC AL                    ; AL = AL + 1
INC WORD PTR [BX]         ; incrementa word in memoria
DEC CX                    ; CX = CX - 1

; NEG — negazione (complemento a 2): NEG x = NOT(x) + 1
NEG AL                    ; AL = -AL

; MUL — moltiplicazione senza segno
MUL BL                    ; AX ← AL × BL         (op 8 bit)
MUL BX                    ; DX:AX ← AX × BX      (op 16 bit)

; DIV — divisione senza segno
DIV BL                    ; AL=quoziente · AH=resto   (dividendo in AX)
DIV BX                    ; AX=quoziente · DX=resto   (dividendo in DX:AX)

; CMP — confronto: calcola op1-op2, aggiorna solo i flag, NON salva risultato
CMP AL, BL                ; AL - BL → imposta ZF, SF, CF, OF
CMP AX, 100d              ; confronto con immediato
CMP AX, [BX]              ; confronto con memoria
```

### MUL e DIV — Schema Operandi

| Istruzione | Tipo operando | Operazione | Risultato |
|------------|--------------|------------|-----------|
| `MUL op8`  | 8 bit  | AL × op    | AX (16 bit) |
| `MUL op16` | 16 bit | AX × op    | DX:AX (32 bit) |
| `DIV op8`  | 8 bit  | AX ÷ op    | AL=quoziente · AH=resto |
| `DIV op16` | 16 bit | DX:AX ÷ op | AX=quoziente · DX=resto |

> 💡 **Complemento a 2** → se SUB produce risultato negativo, AL = NOT(n)+1.
> Esempio: -21 → `00010101` → NOT = `11101010` → +1 = `11101011` = **EBh**. SF=1 indica risultato negativo.

---

## ⚙️ Logica, Shift e Rotazione

```asm
; AND — azzera bit selezionati (masking)
AND AL, 0Fh               ; azzera nibble alto:  AL &= 00001111b
AND AL, 11111110b         ; azzera solo bit 0
TEST AL, 01h              ; AND senza salvare — aggiorna solo ZF (bit0 = 0?)

; OR — forza bit a 1
OR  AL, 0F0h              ; forza nibble alto a 1: AL |= 11110000b
OR  AL, 00000001b         ; forza solo bit 0 a 1

; XOR — inverte bit selezionati / azzera registro
XOR AX, AX                ; AX = 0  ← IDIOMA CLASSICO (più veloce di MOV AX,0)
XOR AL, 0FFh              ; complemento a 1 di AL
XOR AL, 00000001b         ; inverte solo bit 0

; NOT — complemento a 1 (inverte tutti i bit)
NOT AL                    ; AL = ~AL

; SHL / SHR — shift logico (bit uscente va nel CF)
SHL AL, 1                 ; AL <<= 1  →  AL × 2
SHL AL, CL                ; AL <<= CL →  AL × 2^CL  (CL contiene lo shift count)
SHR AL, 1                 ; AL >>= 1  →  AL ÷ 2
SHR AX, CL                ; AX >>= CL →  AX ÷ 2^CL

; SAR — shift aritmetico destra (preserva il segno, bit 7 invariato)
SAR AL, 1                 ; divide per 2 con segno

; ROL / ROR — rotazione circolare
ROL AL, 1                 ; bit7 → CF e → bit0
ROR AL, 1                 ; bit0 → CF e → bit7
ROL AL, CL                ; ruota di CL posizioni
ROR AX, CL                ; ruota AX di CL posizioni
```

### Operazioni Bit a Bit — Riepilogo

| Operazione | Uso | Equivalente C |
|------------|-----|---------------|
| `AND AL, mask` | Isolare / azzerare bit | `AL &= mask` |
| `OR AL, mask` | Impostare bit a 1 | `AL \|= mask` |
| `XOR AL, mask` | Invertire bit selezionati | `AL ^= mask` |
| `TEST AL, mask` | Verificare bit (solo ZF) | `if (AL & mask)` |
| `NOT AL` | Complemento a 1 | `AL = ~AL` |
| `SHL AL, n` | Moltiplicare per 2ⁿ (veloce) | `AL <<= n` |
| `SHR AL, n` | Dividere per 2ⁿ (veloce, senza segno) | `AL >>= n` |
| `SAR AL, n` | Dividere per 2ⁿ (con segno) | `AL >>= n` (signed) |

---

## 🚩 Flag e Registro FLAGS

| Flag | Nome | Attivo (=1) quando | Usato da |
|------|------|--------------------|----------|
| `ZF` | Zero Flag | Risultato = 0 · op1 = op2 in CMP | `JE/JZ` `JNE/JNZ` `LOOP` |
| `SF` | Sign Flag | Bit più significativo = 1 (negativo) | `JG` `JL` `JGE` `JLE` |
| `CF` | Carry Flag | Riporto/prestito dal MSB (overflow senza segno) | `JA` `JB` `ADC` `SBB` |
| `OF` | Overflow Flag | Overflow con segno (risultato non rappresentabile) | `JO` `JNO` |
| `PF` | Parity Flag | Numero di bit a 1 nel risultato è pari | Comunicazioni seriali |
| `AF` | Auxiliary Carry | Riporto dal bit 3 al bit 4 | `DAA` `DAS` (BCD) |
| `DF` | Direction Flag | Direzione operazioni su stringhe | `CLD` `STD` |
| `IF` | Interrupt Flag | Interrupt hardware abilitati | `STI` `CLI` |

```asm
; operazioni dirette sui flag
CLC     ; CF = 0  (clear carry)
STC     ; CF = 1  (set carry)
CMC     ; CF = ~CF (complement carry)
CLD     ; DF = 0  (stringhe scorrono in avanti — SI/DI incrementati)
STD     ; DF = 1  (stringhe scorrono all'indietro — SI/DI decrementati)
CLI     ; IF = 0  (disabilita interrupt hardware)
STI     ; IF = 1  (abilita interrupt hardware)
```

---

## 🔀 Controllo del Flusso

### JMP — Salto Incondizionato

```asm
JMP etichetta             ; salta sempre — IP ← indirizzo etichetta
JMP AX                    ; salto indiretto — IP ← AX
JMP [BX]                  ; salto indiretto — IP ← Mem[BX]
```

### Salti Condizionati — usati dopo `CMP op1, op2` ⭐

| Istruzione | Condizione | Flag | Tipo |
|------------|-----------|------|------|
| `JE` / `JZ` | op1 = op2 · Zero | ZF=1 | entrambi |
| `JNE` / `JNZ` | op1 ≠ op2 · Non zero | ZF=0 | entrambi |
| `JG` / `JNLE` | op1 > op2 | ZF=0 e SF=OF | ⭐ **con segno** |
| `JGE` / `JNL` | op1 ≥ op2 | SF=OF | ⭐ **con segno** |
| `JL` / `JNGE` | op1 < op2 | SF≠OF | ⭐ **con segno** |
| `JLE` / `JNG` | op1 ≤ op2 | ZF=1 o SF≠OF | ⭐ **con segno** |
| `JA` / `JNBE` | op1 > op2 | CF=0 e ZF=0 | ⭐⭐ **senza segno** |
| `JAE` / `JNB` | op1 ≥ op2 | CF=0 | ⭐⭐ **senza segno** |
| `JB` / `JNAE` | op1 < op2 | CF=1 | ⭐⭐ **senza segno** |
| `JBE` / `JNA` | op1 ≤ op2 | CF=1 o ZF=1 | ⭐⭐ **senza segno** |
| `JS` | Negativo | SF=1 | — |
| `JNS` | Non negativo | SF=0 | — |
| `JO` / `JNO` | Overflow / No overflow | OF=1/0 | — |
| `JC` / `JNC` | Carry / No carry | CF=1/0 | — |

> ⚠️ **Con segno (JG/JL) vs Senza segno (JA/JB)**
> `MOV BL, -30d` → BL = `EBh` = 235 se interpretato senza segno.
> Dopo `CMP AL, BL` con AL=10: **JG** vede 10 > -30 (salta) · **JA** vede 10 < 235 (non salta).
> Scegli l'istruzione in base al tipo dei tuoi dati!

### Pattern IF / IF-ELSE in Assembly

```asm
; IF (AL > BL) { ... } ELSE { ... }
  CMP AL, BL
  JG  ramo_vero           ; se AL > BL salta
  ; === ELSE ===
  MOV CX, 0               ; codice ramo falso
  JMP fine_if             ; OBBLIGATORIO: salta oltre il ramo if
ramo_vero:
  MOV CX, 1               ; codice ramo vero
fine_if:

; IF senza ELSE (esecuzione condizionale)
  CMP AX, 100d
  JL  fine_if             ; salta se AX < 100 (condizione negata)
  ; codice eseguito solo se AX >= 100
fine_if:
```

---

## ↺ Cicli

```asm
; LOOP — DEC CX poi salta se CX ≠ 0  (equivale a do-while con CX iterazioni)
  MOV CX, 5               ; CX = numero di iterazioni
ciclo:
  ; corpo del ciclo
  LOOP ciclo              ; CX-- · se CX≠0 → ripete · eseguito min. 1 volta

; LOOPE — salta se CX≠0 E ZF=1  (loop while equal)
  MOV CX, 10
scansione_eq:
  CMP [SI], AL            ; confronta elemento corrente con target
  INC SI
  LOOPE scansione_eq      ; continua se CX≠0 E l'ultimo CMP era uguale

; LOOPNE — salta se CX≠0 E ZF=0  (loop while not equal)
  MOV CX, 100
cerca_zero:
  CMP BYTE PTR [SI], 0    ; cerca il byte terminatore 0
  INC SI
  LOOPNE cerca_zero       ; continua finché CX≠0 E elemento≠0
```

### Pattern dei Tre Cicli in Assembly

```asm
; ── FOR (post-test con LOOP) ────────────────────────────────────────────
; for(i = N; i >= 1; i--)  corpo eseguito sempre almeno una volta
  MOV CX, N
for_body:
  ; corpo
  LOOP for_body

; ── WHILE (pre-test) ────────────────────────────────────────────────────
; while(CX > 1)  può non eseguire mai se CX ≤ 1 all'inizio
while_test:
  CMP CX, 1
  JLE while_end           ; esci se condizione falsa
  ; corpo
  DEC CX
  JMP while_test
while_end:

; ── DO-WHILE (post-test manuale) ────────────────────────────────────────
; do { corpo } while (AX < 100)  eseguito sempre almeno una volta
do_body:
  ; corpo
  INC AX
  CMP AX, 100d
  JL  do_body             ; continua se AX < 100
```

---

## 📚 Stack

```asm
; lo stack cresce verso il basso — SP decrementato di 2 ad ogni PUSH

; salvataggio/ripristino registri — ordine LIFO!
PUSH AX                   ; salva AX
PUSH BX                   ; salva BX  (ora in cima)
PUSH CX                   ; salva CX  (ora in cima)
  ; ... usa liberamente AX, BX, CX ...
POP  CX                   ; ripristina CX (ultimo salvato → primo estratto)
POP  BX                   ; ripristina BX
POP  AX                   ; ripristina AX

; PUSHF / POPF — salva e ripristina l'intero registro FLAGS
PUSHF                     ; salva FLAGS nello stack
POPF                      ; ripristina FLAGS dallo stack

; accesso diretto allo stack senza POP (via BP)
MOV BP, SP                ; BP punta alla cima dello stack
MOV AX, [BP]              ; legge TOS senza rimuoverlo
MOV BX, [BP+2]            ; legge il secondo elemento
MOV CX, [BP+4]            ; legge il terzo elemento
```

> 🚫 **Regola fondamentale** → ogni `PUSH` deve avere il suo `POP`, in ordine inverso (LIFO).
> Uno stack non bilanciato corrompe l'indirizzo di ritorno di `RET` → crash silenzioso.

---

## 📞 Subroutine — CALL e RET

```asm
; CALL salva IP nello stack e salta alla subroutine
  MOV AX, 10              ; passa parametri in registri (convenzione)
  MOV BX, 20
  CALL somma_numeri       ; IP ← stack · IP ← indirizzo somma_numeri
  ; ← esecuzione riprende qui dopo RET
  ; risultato in AX per convenzione

somma_numeri PROC
  PUSH BX                 ; salva registri modificati (buona pratica)
  PUSH CX
  ; ── corpo ────────────────────────────────────────────────────────
  ADD AX, BX              ; AX = AX + BX
  ; ─────────────────────────────────────────────────────────────────
  POP  CX                 ; ripristina nell'ordine inverso
  POP  BX
  RET                     ; IP ← dallo stack → torna al chiamante
somma_numeri ENDP

; passaggio parametri via stack
  PUSH param1             ; parametri prima di CALL
  PUSH param2
  CALL funzione
  ADD SP, 4               ; pulisce parametri dallo stack (2 word = 4 byte)

funzione PROC
  MOV BP, SP              ; accede ai parametri via BP
  MOV AX, [BP+2]          ; primo parametro  (+2 perché IP è in cima)
  MOV BX, [BP+4]          ; secondo parametro
  RET
funzione ENDP
```

| Convenzione | Cosa fare |
|-------------|-----------|
| Valore di ritorno | Metti il risultato in `AX` prima di `RET` |
| Preservare registri | `PUSH` all'inizio, `POP` alla fine nell'ordine inverso |
| Parametri semplici | Passa in registri (AX, BX, CX...) prima di `CALL` |
| Parametri via stack | `PUSH` prima di `CALL`, `ADD SP, N` dopo per pulire |

---

## ⚡ Interrupt e I/O

```asm
; INT N — invoca l'interrupt N (vettore in IVT a 0000:N×4)
; Prima di INT: caricare AH con il numero del servizio

; ── INT 21h — Servizi DOS ────────────────────────────────────────────────

; AH=01h: leggi carattere da tastiera (con echo) → AL = codice ASCII
  MOV AH, 01h
  INT 21h                 ; AL ← codice ASCII del tasto premuto

; AH=08h: leggi carattere da tastiera (senza echo) → AL = codice ASCII
  MOV AH, 08h
  INT 21h

; AH=02h: stampa carattere a video → DL = codice ASCII
  MOV DL, 'A'
  MOV AH, 02h
  INT 21h                 ; stampa il carattere 'A'

; AH=09h: stampa stringa (terminata da '$') → DS:DX = indirizzo stringa
  LEA DX, messaggio       ; DX ← indirizzo della stringa
  MOV AH, 09h
  INT 21h                 ; stampa finché non trova '$'

; AH=0Ah: leggi stringa da tastiera con buffer → DS:DX = indirizzo buffer
; struttura buffer: [0]=dimensione_max · [1]=byte_letti · [2..]=stringa
  LEA DX, input_buf
  MOV AH, 0Ah
  INT 21h

; AH=4Ch: termina programma e restituisce controllo al SO
  MOV AH, 4Ch
  MOV AL, 0               ; codice di uscita (0 = successo)
  INT 21h                 ; SEMPRE l'ultima istruzione del programma
```

### INT 10h — Servizi Video BIOS ⭐⭐

```asm
; AH=02h: posiziona cursore
  MOV AH, 02h
  MOV BH, 0               ; pagina video 0
  MOV DH, 5               ; riga 5
  MOV DL, 10              ; colonna 10
  INT 10h

; AH=06h: pulisci schermo (scroll up)
  MOV AH, 06h
  MOV AL, 0               ; AL=0 → pulisci tutto
  MOV BH, 07h             ; attributo: testo bianco su sfondo nero
  XOR CX, CX              ; angolo superiore sinistro (riga 0, col 0)
  MOV DH, 24              ; angolo inferiore destro (riga 24)
  MOV DL, 79              ; colonna 79
  INT 10h
```

### Tabella Servizi Interrupt

| Interrupt | AH | Servizio | Input | Output |
|-----------|-----|---------|-------|--------|
| `INT 21h` | `01h` | Leggi char (con echo) | — | AL=ASCII |
| `INT 21h` | `08h` | Leggi char (senza echo) | — | AL=ASCII |
| `INT 21h` | `02h` | Scrivi char | DL=ASCII | — |
| `INT 21h` | `09h` | Scrivi stringa | DS:DX=str$ | — |
| `INT 21h` | `0Ah` | Leggi stringa buffer | DS:DX=buf | Buffer riempito |
| `INT 21h` | `4Ch` | Termina programma | AL=exit code | — |
| `INT 10h` | `02h` | Posiziona cursore | BH=pagina · DH=riga · DL=col | — |
| `INT 10h` | `06h` | Pulisci schermo | AL=0 · BH=attr · CX/DX=angoli | — |

---

## 📍 Modi di Indirizzamento

| Modo | Sintassi | Formula EA | Passi mem. | Uso tipico |
|------|---------|------------|-----------|------------|
| Immediato | `MOV AL, 4Dh` | — (dato nell'istruzione) | 1 | Costanti, inizializzazione |
| Registro | `MOV AX, BX` | — (solo registri) | 1 | Copia rapida tra registri |
| Diretto | `MOV AX, [6Ah]` | DS×10h + offset | 2 | Variabili a indirizzo fisso |
| Indiretto registro | `MOV AX, [SI]` | DS×10h + SI | 2 | Puntatori, scorrimento array |
| Registro + displacement | `MOV AX, 3h[SI]` | DS×10h + SI + disp | 3 | Accesso a campi strutture |
| Base + indice | `MOV AX, [BX][DI]` | DS×10h + BX + DI | 2 | Matrici, array 2D |
| Base + indice + disp | `MOV AX, 2h[BX][SI]` | DS×10h + BX + SI + disp | 3 | Array di strutture |

```asm
; WORD PTR / BYTE PTR — specifica esplicitamente la dimensione dell'accesso
MOV BYTE PTR [BX], 5      ; scrivi 1 byte in memoria
MOV WORD PTR [BX], 5      ; scrivi 2 byte (word) in memoria
INC WORD PTR [SI]         ; incrementa word in memoria
CMP BYTE PTR [BX], 0      ; confronta byte in memoria con 0
```

---

## 🔤 Operazioni su Stringhe ⭐⭐⭐

```asm
; Prerequisiti:
; SI → punta alla sorgente (DS:SI)
; DI → punta alla destinazione (ES:DI)
; CX → numero di elementi da elaborare
; DF=0 (CLD) → SI/DI incrementati · DF=1 (STD) → SI/DI decrementati

CLD                       ; DF=0 — scorri in avanti (quasi sempre)

; MOVSB / MOVSW — copia byte/word da DS:SI a ES:DI
MOVSB                     ; copia 1 byte · SI++ · DI++
MOVSW                     ; copia 1 word · SI+=2 · DI+=2
REP MOVSB                 ; copia CX byte  (equivale a memcpy in C)
REP MOVSW                 ; copia CX word

; LODSB / LODSW — carica DS:SI in AL/AX
LODSB                     ; AL ← DS:SI · SI++
LODSW                     ; AX ← DS:SI · SI+=2

; STOSB / STOSW — scrivi AL/AX in ES:DI
STOSB                     ; ES:DI ← AL · DI++
REP STOSB                 ; riempie CX byte con AL (equivale a memset in C)

; CMPSB / CMPSW — confronta DS:SI con ES:DI (aggiorna flag)
CMPSB                     ; DS:SI - ES:DI → flag · SI++ · DI++
REPE  CMPSB               ; confronta CX byte, ferma al primo diverso
REPNE CMPSB               ; confronta CX byte, ferma al primo uguale

; SCASB / SCASW — cerca AL/AX in ES:DI
SCASB                     ; AL - ES:DI → flag · DI++
REPNE SCASB               ; cerca AL nel buffer di CX byte (equivale a strchr in C)
REPE  SCASB               ; scorre finché trova un byte diverso da AL
```

### Prefissi REP

| Prefisso | Condizione di continuazione |
|----------|----------------------------|
| `REP` | CX ≠ 0 |
| `REPE` / `REPZ` | CX ≠ 0 **e** ZF = 1 (elementi uguali) |
| `REPNE` / `REPNZ` | CX ≠ 0 **e** ZF = 0 (elementi diversi) |

---

## 💡 Tips e Idiomi

```asm
; ── IDIOMI CLASSICI ──────────────────────────────────────────────────────

; azzera registro — più veloce di MOV AX, 0 (1 byte in meno)
XOR AX, AX                ; AX = 0

; moltiplica / dividi per potenze di 2 — più veloce di MUL/DIV
SHL AX, 1                 ; AX × 2
SHL AX, 2                 ; AX × 4
SHL AX, 3                 ; AX × 8
SHR AX, 1                 ; AX ÷ 2
SHR AX, 2                 ; AX ÷ 4

; scambia due registri senza temporaneo
XCHG AX, BX               ; AX ↔ BX

; incrementa puntatore di 1 / 2
INC SI                    ; SI++ (byte per byte)
ADD SI, 2                 ; SI += 2 (word per word)

; converti cifra BCD → ASCII (aggiungi 30h)
ADD AL, 30h               ; '0'=30h, '1'=31h, ..., '9'=39h

; converti ASCII → cifra numerica
SUB AL, 30h               ; '5' (35h) → 5d

; verifica se AL è zero senza usare CMP
OR  AL, AL                ; ZF=1 se AL=0  (non modifica AL)
AND AL, AL                ; equivalente — ZF=1 se AL=0

; ── PATTERN SUBROUTINE ───────────────────────────────────────────────────

; prologo standard (salva contesto)
funzione PROC
  PUSH AX
  PUSH BX
  PUSH CX
  ; ... corpo ...
  POP  CX
  POP  BX
  POP  AX
  RET
funzione ENDP

; ── PATTERN I/O COMPLETO ─────────────────────────────────────────────────

; leggi un carattere e stampalo maiuscolo
  MOV AH, 01h
  INT 21h                 ; AL ← char letto
  AND AL, 11011111b       ; forza bit 5 a 0 → maiuscolo (A-Z)
  MOV DL, AL
  MOV AH, 02h
  INT 21h                 ; stampa il carattere

; stampa un numero singola cifra (0–9)
  MOV AL, numero          ; AL = 0..9
  ADD AL, 30h             ; converti in ASCII
  MOV DL, AL
  MOV AH, 02h
  INT 21h

; ── SHORTCUTS TASTIERA EMU8086 ───────────────────────────────────────────
; F5     → Run (esegui tutto)
; F8     → Single Step (passo singolo)
; F9     → esegui fino al cursore
; Ctrl+F2 → reset / riavvia simulazione
; F2     → assembla il sorgente
```

### Errori Frequenti — Diagnostica Rapida

| Sintomo | Causa probabile | Soluzione |
|---------|----------------|-----------|
| Crash all'avvio | DS non inizializzato | Aggiungi `MOV AX, data` / `MOV DS, AX` dopo `start:` |
| Risultato errato dopo DIV | Dividendo in AX non azzerato | `XOR AH, AH` prima di `DIV BL` (op 8 bit) |
| Stack overflow | PUSH senza POP corrispondente | Verifica bilanciamento PUSH/POP in ogni subroutine |
| Stringa non stampata | Manca `$` terminatore | Aggiungi `'$'` alla fine di ogni stringa per INT 21h/09h |
| Errore `MOV DS, valore` | DS non accetta immediati | Usa `MOV AX, data` → `MOV DS, AX` |
| MUL risultato troncato | Dimentica la parte alta | Per `MUL BL` il risultato è in **AX**, non AL |
| Loop infinito | CX=0 con LOOP | LOOP con CX=0 esegue 65536 volte (underflow) — controlla CX prima |

---

## 📖 Risorse Correlate

- 📝 Lezioni complete: [profgiagnotti.it](https://profgiagnotti.it)
- 📦 Esercizi .asm: [profgiagnotti.it/risorse](https://profgiagnotti.it/download/14004/?tmstv=1773162807)
- 🔗 x86 Assembly Guide (UVA): [cs.virginia.edu/~evans/cs216/guides/x86.html](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)
- 🔗 Intel x86 Reference (Wikibooks): [en.wikibooks.org/wiki/X86_Assembly](https://en.wikibooks.org/wiki/X86_Assembly)
- 🛠️ Godbolt — Compiler Explorer: [godbolt.org](https://godbolt.org)
- 💬 Domande: [Discord Prof. Giagnotti](https://discord.gg/profgiagnotti)

---

*⬅️ [Torna all'indice dei Cheatsheet](../README.md)*
