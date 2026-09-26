# 🔍 Nmap — Cheatsheet Completo

> **Prof. Giagnotti** · [profgiagnotti.it](https://profgiagnotti.it)
> Livello: ⭐ Base → ⭐⭐⭐ Avanzato · Versione Nmap: 7.x · Aggiornato: 2025

> ⚠️ **Avviso legale:** usare Nmap solo su reti e sistemi di cui si è proprietari o per cui si ha esplicita autorizzazione scritta. La scansione non autorizzata è illegale.

---

## 📑 Indice

- [Sintassi Base](#-sintassi-base)
- [Selezione Target](#-selezione-target)
- [Tipi di Scansione](#-tipi-di-scansione)
- [Selezione Porte](#-selezione-porte)
- [Rilevamento Servizi e OS](#-rilevamento-servizi-e-os)
- [Timing e Performance](#-timing-e-performance)
- [Firewall Evasion e Spoofing](#-firewall-evasion-e-spoofing)
- [Nmap Scripting Engine (NSE)](#-nmap-scripting-engine-nse)
- [Output e Formati](#-output-e-formati)
- [Combinazioni Pratiche](#-combinazioni-pratiche)
- [Lettura Output](#-lettura-output)

---

## ⌨️ Sintassi Base

```bash
nmap [opzioni] [target]

# Esempi base
nmap 192.168.1.1                  # scansione singolo host
nmap scanme.nmap.org              # scansione per hostname
nmap -v 192.168.1.1               # verbose (più dettagli)
nmap -vv 192.168.1.1              # very verbose
nmap --help                       # mostra aiuto
nmap --version                    # versione installata
```

---

## 🎯 Selezione Target

```bash
# Singolo host
nmap 192.168.1.1

# Range di IP
nmap 192.168.1.1-254              # da .1 a .254
nmap 192.168.1.*                  # tutti i .x (wildcard)

# Subnet CIDR
nmap 192.168.1.0/24               # intera subnet /24 (256 host)
nmap 10.0.0.0/8                   # intera classe A ⚠️ lento

# Più target
nmap 192.168.1.1 192.168.1.2 10.0.0.1
nmap 192.168.1.1,2,3              # .1, .2, .3

# Da file di testo
nmap -iL targets.txt              # un IP/hostname per riga

# Escludere host
nmap 192.168.1.0/24 --exclude 192.168.1.1
nmap 192.168.1.0/24 --excludefile esclusi.txt

# Host discovery — trova host attivi senza scansionare porte
nmap -sn 192.168.1.0/24           # ping scan (solo discovery)
nmap -sn 192.168.1.0/24 --send-ip # forza IP invece di ARP
```

---

## 🔬 Tipi di Scansione

```bash
# ── TCP ──────────────────────────────────────────────────

# SYN Scan — "half-open" (default, richiede root)
# Invia SYN → aspetta SYN/ACK → manda RST (non completa handshake)
# Veloce, meno rumoroso nei log
sudo nmap -sS 192.168.1.1

# TCP Connect Scan (non richiede root)
# Completa il three-way handshake — più lento e più tracciabile
nmap -sT 192.168.1.1

# ACK Scan — mappa regole firewall (non rileva porte aperte)
# Risposta RST = porta non filtrata / nessuna risposta = filtrata
sudo nmap -sA 192.168.1.1

# Window Scan — variante ACK, sfrutta dimensione TCP window
sudo nmap -sW 192.168.1.1

# Maimon Scan — FIN/ACK, elude alcuni firewall
sudo nmap -sM 192.168.1.1

# ── STEALTH / EVASION ────────────────────────────────────

# FIN Scan — invia solo flag FIN (elude firewall stateless)
sudo nmap -sF 192.168.1.1

# NULL Scan — nessun flag TCP impostato
sudo nmap -sN 192.168.1.1

# Xmas Scan — FIN + PSH + URG impostati (come albero di Natale)
sudo nmap -sX 192.168.1.1

# ── UDP ──────────────────────────────────────────────────

# UDP Scan — molto più lento del TCP
sudo nmap -sU 192.168.1.1
sudo nmap -sU -p 53,67,68,69,123,161 192.168.1.1  # porte UDP comuni

# ── COMBINATA ────────────────────────────────────────────

# TCP SYN + UDP insieme
sudo nmap -sS -sU 192.168.1.1

# ── ALTRI PROTOCOLLI ─────────────────────────────────────

# SCTP INIT Scan
sudo nmap -sY 192.168.1.1

# IP Protocol Scan — quali protocolli IP sono supportati
sudo nmap -sO 192.168.1.1

# ICMP / Ping types
sudo nmap -PE 192.168.1.1        # ICMP echo
sudo nmap -PP 192.168.1.1        # ICMP timestamp
sudo nmap -PM 192.168.1.1        # ICMP netmask
```

---

## 🚪 Selezione Porte

```bash
# Porte specifiche
nmap -p 80 192.168.1.1            # solo porta 80
nmap -p 80,443,8080 192.168.1.1   # porte multiple
nmap -p 1-1024 192.168.1.1        # range di porte
nmap -p- 192.168.1.1              # tutte le 65535 porte
nmap -p U:53,T:80 192.168.1.1     # UDP 53 e TCP 80

# Porte per nome servizio
nmap -p http,https 192.168.1.1
nmap -p ssh 192.168.1.1

# Top porte più comuni
nmap --top-ports 100 192.168.1.1  # le 100 più comuni
nmap --top-ports 1000 192.168.1.1 # le 1000 più comuni (default)

# Porte in ordine casuale (evasione)
nmap --randomize-hosts -p- 192.168.1.0/24

# Stato porte nell'output
# open        = porta aperta, accetta connessioni
# closed      = porta chiusa, risponde con RST
# filtered    = firewall blocca, nessuna risposta
# unfiltered  = raggiungibile ma stato incerto (ACK scan)
# open|filtered = aperta o filtrata (UDP, FIN, NULL, Xmas)
```

---

## 🧬 Rilevamento Servizi e OS

```bash
# Version Detection — rileva versione dei servizi
nmap -sV 192.168.1.1
nmap -sV --version-intensity 9 192.168.1.1  # intensità max (0-9)
nmap -sV --version-light 192.168.1.1        # intensità bassa (più veloce)

# OS Detection — rileva sistema operativo (richiede root)
sudo nmap -O 192.168.1.1
sudo nmap -O --osscan-guess 192.168.1.1     # forza un'ipotesi anche se incerta
sudo nmap -O --osscan-limit 192.168.1.1     # solo su host promettenti

# Traceroute
nmap --traceroute 192.168.1.1

# Scansione aggressiva (combina -sV -O --traceroute -sC)
# ⚠️ molto rumorosa — usare solo in lab o con autorizzazione
sudo nmap -A 192.168.1.1

# Rilevamento hostname
nmap -sV --resolve-all 192.168.1.0/24
```

---

## ⏱️ Timing e Performance

```bash
# Template di timing (-T0 più lento, -T5 più veloce)
nmap -T0 192.168.1.1    # Paranoid    — 5 min tra probe, IDS evasion
nmap -T1 192.168.1.1    # Sneaky      — 15 sec tra probe
nmap -T2 192.168.1.1    # Polite      — non sovraccarica la rete
nmap -T3 192.168.1.1    # Normal      # default
nmap -T4 192.168.1.1    # Aggressive  — reti veloci e affidabili
nmap -T5 192.168.1.1    # Insane      — può perdere pacchetti ⚠️

# Controllo granulare
nmap --min-rate 1000 192.168.1.0/24       # minimo 1000 pacchetti/sec
nmap --max-rate 500 192.168.1.0/24        # massimo 500 pacchetti/sec
nmap --min-parallelism 10 192.168.1.0/24  # probe paralleli minimi
nmap --max-parallelism 100 192.168.1.0/24 # probe paralleli massimi
nmap --host-timeout 30s 192.168.1.0/24   # timeout per host
nmap --scan-delay 1s 192.168.1.1         # pausa tra probe
```

---

## 🕵️ Firewall Evasion e Spoofing

```bash
# Fragmentazione pacchetti (elude IDS/firewall)
sudo nmap -f 192.168.1.1           # frammenta in 8 byte
sudo nmap -f -f 192.168.1.1        # frammenta in 16 byte
sudo nmap --mtu 24 192.168.1.1     # dimensione personalizzata (multiplo di 8)

# Decoy — maschera il tuo IP con IP falsi
sudo nmap -D RND:10 192.168.1.1               # 10 IP casuali come decoy
sudo nmap -D 10.0.0.1,10.0.0.2,ME 192.168.1.1 # decoy specifici + tuo IP

# Spoofing IP sorgente
sudo nmap -S 10.0.0.5 -e eth0 192.168.1.1    # ⚠️ non ricevi risposte

# Spoofing MAC address
sudo nmap --spoof-mac 0 192.168.1.1           # MAC casuale
sudo nmap --spoof-mac Apple 192.168.1.1       # MAC da vendor specifico
sudo nmap --spoof-mac 00:11:22:33:44:55 192.168.1.1

# Porta sorgente personalizzata (bypassa firewall che fidano porta 53/80)
sudo nmap --source-port 53 192.168.1.1
sudo nmap -g 80 192.168.1.1                   # equivalente

# Bad checksum (alcuni firewall non controllano)
sudo nmap --badsum 192.168.1.1

# Dati casuali nei pacchetti
sudo nmap --data-length 25 192.168.1.1        # aggiunge 25 byte casuali

# Idle/Zombie Scan — scansione completamente anonima
# Richiede uno "zombie host" con IP ID incrementale prevedibile
sudo nmap -sI zombie_host 192.168.1.1
```

---

## 📜 Nmap Scripting Engine (NSE)

```bash
# Eseguire script
nmap --script=default 192.168.1.1             # script di default (-sC)
nmap -sC 192.168.1.1                          # equivalente a --script=default
nmap --script=vuln 192.168.1.1                # script vulnerabilità
nmap --script=safe 192.168.1.1                # solo script sicuri

# Script specifici
nmap --script=http-title 192.168.1.1          # titolo pagine web
nmap --script=http-headers 192.168.1.1        # headers HTTP
nmap --script=banner 192.168.1.1              # banner dei servizi
nmap --script=ftp-anon 192.168.1.1            # FTP anonimo
nmap --script=smb-vuln-ms17-010 192.168.1.1   # EternalBlue (WannaCry)
nmap --script=ssh-brute 192.168.1.1           # brute force SSH ⚠️
nmap --script=http-sql-injection 192.168.1.1  # SQL injection base

# Categorie NSE disponibili
# auth      → autenticazione e bypass
# broadcast → discovery senza target
# brute     → brute force credenziali
# default   → script sicuri e informativi (-sC)
# discovery → raccolta informazioni
# dos       → denial of service ⚠️
# exploit   → sfruttamento vulnerabilità ⚠️
# external  → usa risorse esterne (shodan ecc)
# fuzzer    → fuzzing input
# intrusive → potenzialmente dannosi ⚠️
# malware   → rileva backdoor e malware
# safe      → non danneggiano il target
# version   → rilevamento versioni
# vuln      → vulnerabilità note

# Passare argomenti a uno script
nmap --script=http-brute --script-args \
  http-brute.path=/admin,userdb=users.txt,passdb=pass.txt \
  192.168.1.1

# Trovare script per servizio
ls /usr/share/nmap/scripts/ | grep ssh
ls /usr/share/nmap/scripts/ | grep http

# Aggiornare database script
sudo nmap --script-updatedb
```

---

## 💾 Output e Formati

```bash
# Normale (testo leggibile)
nmap 192.168.1.1 -oN output.txt

# XML (per parsing con altri tool)
nmap 192.168.1.1 -oX output.xml

# Grepable (una riga per host)
nmap 192.168.1.1 -oG output.gnmap

# Tutti i formati contemporaneamente
nmap 192.168.1.1 -oA scan_completo
# → crea scan_completo.nmap / .xml / .gnmap

# Aumentare verbosità nell'output
nmap -v 192.168.1.1                # verbose
nmap -vv 192.168.1.1               # very verbose
nmap -d 192.168.1.1                # debug
nmap -d2 192.168.1.1               # debug level 2

# Mostrare solo host aperti
nmap 192.168.1.0/24 --open

# Riprendere scansione interrotta
nmap --resume output.gnmap

# Parsing output XML con Python
python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('output.xml')
for host in tree.findall('.//host'):
    addr = host.find('address').get('addr')
    for port in host.findall('.//port'):
        portid = port.get('portid')
        state  = port.find('state').get('state')
        if state == 'open':
            print(f'{addr}:{portid}')
"
```

---

## 🛠️ Combinazioni Pratiche

```bash
# ── RICOGNIZIONE BASE ──────────────────────────────────────

# Discovery rapido della rete locale
sudo nmap -sn 192.168.1.0/24

# Scansione veloce porte più comuni
nmap -T4 --top-ports 1000 192.168.1.1

# Scansione completa con versioni (bilanciata)
sudo nmap -sS -sV -T4 -p- 192.168.1.1

# ── WEB SERVER ─────────────────────────────────────────────

# Analisi completa server web
sudo nmap -sS -sV -p 80,443,8080,8443 \
  --script=http-title,http-headers,http-methods \
  192.168.1.1

# Ricerca vulnerabilità web
sudo nmap -sV -p 80,443 \
  --script=http-vuln* \
  192.168.1.1

# ── RETE AZIENDALE ─────────────────────────────────────────

# Scansione silenziosa con evasione IDS
sudo nmap -sS -T2 -f --randomize-hosts \
  --data-length 15 -p 22,80,443 \
  192.168.1.0/24

# Mappa completa della rete (lenta ma precisa)
sudo nmap -sS -sV -O -T3 \
  --script=default -p- \
  -oA scan_rete \
  192.168.1.0/24

# ── VULNERABILITY SCANNING ─────────────────────────────────

# Scansione vulnerabilità su host specifico
sudo nmap -sV --script=vuln 192.168.1.1

# Check EternalBlue (MS17-010)
sudo nmap -p 445 --script=smb-vuln-ms17-010 192.168.1.0/24

# Check Heartbleed (OpenSSL)
sudo nmap -p 443 --script=ssl-heartbleed 192.168.1.1

# Check servizi con credenziali default
sudo nmap --script=http-default-accounts 192.168.1.1

# ── LAB E CTF ──────────────────────────────────────────────

# Prima scansione su macchina CTF
sudo nmap -sS -sV -sC -O -T4 -p- TARGET_IP

# Scansione UDP su CTF (spesso ignorata)
sudo nmap -sU -T4 --top-ports 100 TARGET_IP
```

---

## 📖 Lettura Output

```
Starting Nmap 7.94 at 2025-01-01 12:00
Nmap scan report for 192.168.1.1          ← host scansionato
Host is up (0.0012s latency).             ← latenza RTT
Not shown: 994 closed ports              ← porte non mostrate

PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 8.4 (protocol 2.0)
80/tcp   open     http       Apache httpd 2.4.51
443/tcp  open     ssl/https  Apache httpd 2.4.51
3306/tcp filtered mysql                   ← firewall blocca
8080/tcp open     http-proxy
9200/tcp open     http       Elasticsearch REST API

MAC Address: 00:0C:29:XX:XX:XX (VMware)
OS details: Linux 4.15 - 5.6
Uptime guess: 2.134 days

# Legenda stati
# open        → servizio in ascolto, accetta connessioni
# closed      → porta raggiungibile ma nessun servizio
# filtered    → firewall / ACL blocca il traffico
# unfiltered  → raggiungibile, stato non determinabile
# open|filtered → impossibile distinguere (tipico UDP)
```

---

## 📖 Risorse Correlate

- 📝 Articolo completo: [profgiagnotti.it/blog/cybersecurity/nmap-guida-completa](https://profgiagnotti.it)
- ▶️ Video tutorial Lab #01: [youtube.com/@profgiagnotti](https://youtube.com/@profgiagnotti)
- 💬 Domande: [Discord Prof. Giagnotti](https://discord.gg/profgiagnotti)
- 🔗 Documentazione ufficiale: [nmap.org/docs](https://nmap.org/docs.html)
- 🔗 NSE Script Database: [nmap.org/nsedoc](https://nmap.org/nsedoc/)

---

*⬅️ [Torna all'indice dei Cheatsheet](../README.md)*
