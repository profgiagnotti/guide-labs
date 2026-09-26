# 🐧 Comandi Linux — Cheatsheet Completo

> **Prof. Giagnotti** · [profgiagnotti.it](https://profgiagnotti.it)
> Livello: ⭐ Base → ⭐⭐⭐ Avanzato · Aggiornato: 2026

---

## 📑 Indice

- [Navigazione Filesystem](#-navigazione-filesystem)
- [Gestione File e Directory](#-gestione-file-e-directory)
- [Permessi](#-permessi)
- [Processi](#-processi)
- [Rete](#-rete)
- [Utenti e Gruppi](#-utenti-e-gruppi)
- [Ricerca](#-ricerca)
- [Archivi e Compressione](#-archivi-e-compressione)
- [Testo e Stream](#-testo-e-stream)
- [Disk e Memoria](#-disk-e-memoria)
- [Systemd e Servizi](#-systemd-e-servizi)
- [SSH](#-ssh)
- [Tips Avanzati](#-tips-avanzati)

---

## 📁 Navigazione Filesystem

```bash
pwd                    # mostra directory corrente
ls                     # elenca file e cartelle
ls -la                 # lista dettagliata con file nascosti
ls -lh                 # dimensioni in formato leggibile (KB, MB)
cd /path/to/dir        # cambia directory
cd ..                  # vai su di un livello
cd ~                   # vai alla home
cd -                   # torna alla directory precedente
tree                   # mostra struttura ad albero (installa con apt)
tree -L 2              # albero con profondità massima 2
```

---

## 📂 Gestione File e Directory

```bash
# Creare
touch file.txt              # crea file vuoto (o aggiorna timestamp)
mkdir cartella              # crea directory
mkdir -p a/b/c              # crea directory e tutte le parent

# Copiare
cp file.txt copia.txt       # copia file
cp -r dir/ dir_copia/       # copia directory ricorsivamente
cp -p file.txt dest/        # copia preservando permessi e timestamp

# Spostare / Rinominare
mv file.txt nuovo.txt       # rinomina file
mv file.txt /path/dest/     # sposta file

# Eliminare
rm file.txt                 # elimina file
rm -r cartella/             # elimina directory ricorsivamente
rm -rf cartella/            # forza eliminazione senza conferma ⚠️
rmdir cartella/             # elimina directory SOLO se vuota

# Link
ln -s /path/originale link  # crea symlink (collegamento simbolico)
ls -la                      # i symlink mostrano -> destinazione
```

---

## 🔐 Permessi

```bash
# Lettura permessi
ls -l file.txt
# Output: -rwxr-xr-- 1 user group 1234 Jan 1 12:00 file.txt
#          ↑ tipo  ↑ owner ↑ group ↑ others

# chmod — modifica permessi (notazione numerica)
chmod 755 file.sh           # rwx r-x r-x  (standard per script)
chmod 644 file.txt          # rw- r-- r--  (standard per file)
chmod 600 ~/.ssh/id_rsa     # rw- --- ---  (chiave privata SSH)
chmod 777 file              # rwx rwx rwx  ⚠️ mai in produzione

# chmod — notazione simbolica
chmod +x script.sh          # aggiunge permesso esecuzione
chmod -w file.txt           # rimuove permesso scrittura
chmod u+x,g-w file          # owner +exec, group -write

# Tabella numerica
# 4 = read (r)
# 2 = write (w)
# 1 = execute (x)
# Somma: 7=rwx  6=rw-  5=r-x  4=r--

# chown — cambia proprietario
chown user file.txt         # cambia owner
chown user:group file.txt   # cambia owner e group
chown -R user:group dir/    # ricorsivo su directory

# SUID, SGID, Sticky bit
chmod u+s file              # SUID — esegue come owner
chmod g+s dir               # SGID — file ereditano gruppo dir
chmod +t /tmp               # Sticky bit — solo owner può eliminare
```

---

## ⚙️ Processi

```bash
# Visualizzazione
ps aux                      # tutti i processi con dettagli
ps aux | grep nginx         # filtra per nome processo
top                         # monitor processi in tempo reale
htop                        # versione migliorata di top (installa separato)
pgrep nginx                 # mostra PID di un processo per nome

# Gestione
kill PID                    # invia SIGTERM (terminazione gentile)
kill -9 PID                 # invia SIGKILL (terminazione forzata)
killall nginx               # termina tutti i processi per nome
pkill nginx                 # come killall ma accetta pattern

# Background / Foreground
comando &                   # avvia processo in background
jobs                        # elenca processi in background
fg %1                       # porta job 1 in foreground
bg %1                       # riprende job 1 in background
nohup comando &             # esegue anche dopo logout
Ctrl+C                      # interrompe processo in foreground
Ctrl+Z                      # sospende processo (poi usa bg/fg)

# Priorità
nice -n 10 comando          # avvia con priorità bassa (10)
renice -n 5 -p PID          # cambia priorità a processo esistente
# range: -20 (alta) → +19 (bassa)
```

---

## 🌐 Rete

```bash
# Informazioni
ip a                        # mostra interfacce e indirizzi IP
ip r                        # mostra tabella di routing
ss -tulnp                   # porte in ascolto con processi
netstat -tulnp              # alternativa a ss (legacy)
hostname -I                 # IP locale della macchina

# Connettività
ping 8.8.8.8                # testa connettività verso host
ping -c 4 google.com        # invia solo 4 pacchetti
traceroute google.com       # percorso verso destinazione
mtr google.com              # traceroute continuo interattivo

# DNS
nslookup google.com         # risoluzione DNS base
dig google.com              # risoluzione DNS dettagliata
dig google.com MX           # record MX (mail)
dig +short google.com       # solo l'IP

# Download
curl https://example.com    # scarica contenuto URL
curl -O https://example.com/file.zip  # scarica file
wget https://example.com/file.zip     # alternativa a curl
curl -I https://example.com           # solo headers HTTP

# Firewall (ufw)
ufw status                  # stato firewall
ufw allow 22/tcp            # apri porta SSH
ufw allow 80/tcp            # apri porta HTTP
ufw deny 23                 # blocca porta Telnet
ufw enable                  # attiva firewall
```

---

## 👤 Utenti e Gruppi

```bash
# Informazioni
whoami                      # nome utente corrente
id                          # UID, GID e gruppi
who                         # utenti connessi
w                           # utenti connessi con attività
last                        # storico login

# Gestione utenti
useradd -m -s /bin/bash username    # crea utente con home e shell
usermod -aG sudo username           # aggiunge utente al gruppo sudo
userdel -r username                 # elimina utente e home
passwd username                     # cambia password
passwd -l username                  # blocca account

# Sudo
sudo comando                # esegui come root
sudo -i                     # apri shell root
sudo -u altro_user comando  # esegui come altro utente
visudo                      # modifica /etc/sudoers in sicurezza

# Gruppi
groupadd nome_gruppo        # crea gruppo
groups username             # mostra gruppi di un utente
gpasswd -a user gruppo      # aggiunge user a gruppo
```

---

## 🔍 Ricerca

```bash
# find — ricerca file nel filesystem
find /path -name "*.log"           # per nome (case sensitive)
find /path -iname "*.log"          # per nome (case insensitive)
find /path -type f                 # solo file
find /path -type d                 # solo directory
find /path -size +100M             # file più grandi di 100MB
find /path -mtime -7               # modificati negli ultimi 7 giorni
find /path -perm 777               # con permessi specifici
find /path -name "*.log" -delete   # trova e cancella

# grep — ricerca nel contenuto
grep "pattern" file.txt            # cerca stringa in file
grep -r "pattern" /path/           # ricerca ricorsiva
grep -i "pattern" file.txt         # case insensitive
grep -n "pattern" file.txt         # mostra numero di riga
grep -v "pattern" file.txt         # escludi righe con pattern
grep -E "pat1|pat2" file.txt       # regex estesa (OR)
grep -c "pattern" file.txt         # conta le occorrenze

# Combinazioni utili
find /var/log -name "*.log" | xargs grep "ERROR"
grep -r "TODO" . --include="*.py"
```

---

## 📦 Archivi e Compressione

```bash
# tar
tar -czf archivio.tar.gz dir/      # comprimi in .tar.gz
tar -xzf archivio.tar.gz           # estrai .tar.gz
tar -czf archivio.tar.gz -C /path dir/  # specifica percorso base
tar -tzf archivio.tar.gz           # lista contenuto senza estrarre

# zip / unzip
zip -r archivio.zip dir/           # comprimi in .zip
unzip archivio.zip                 # estrai .zip
unzip -l archivio.zip              # lista contenuto

# Cheat: decomprimere qualsiasi formato
# .tar.gz  → tar -xzf
# .tar.bz2 → tar -xjf
# .tar.xz  → tar -xJf
# .zip     → unzip
# .gz      → gunzip
```

---

## 📝 Testo e Stream

```bash
# Visualizzazione
cat file.txt                # stampa intero file
less file.txt               # visualizzazione paginata (q per uscire)
head -n 20 file.txt         # prime 20 righe
tail -n 20 file.txt         # ultime 20 righe
tail -f /var/log/syslog     # segue file in tempo reale (log live)

# Manipolazione
sort file.txt               # ordina righe
sort -r file.txt            # ordina in ordine inverso
sort -n numeri.txt          # ordina numericamente
uniq file.txt               # rimuove righe duplicate consecutive
sort file.txt | uniq        # rimuove tutti i duplicati
wc -l file.txt              # conta righe
wc -w file.txt              # conta parole

# sed — stream editor
sed 's/vecchio/nuovo/' file.txt      # sostituisce prima occorrenza
sed 's/vecchio/nuovo/g' file.txt     # sostituisce tutte
sed -i 's/vecchio/nuovo/g' file.txt  # modifica file direttamente
sed -n '5,10p' file.txt              # stampa righe 5-10

# awk — elaborazione testi strutturati
awk '{print $1}' file.txt            # stampa prima colonna
awk -F: '{print $1}' /etc/passwd     # usa : come separatore
awk '{sum+=$1} END {print sum}'      # somma colonna

# Redirect e pipe
comando > file.txt          # redirect stdout (sovrascrive)
comando >> file.txt         # redirect stdout (appende)
comando 2> errori.txt       # redirect stderr
comando &> tutto.txt        # redirect stdout e stderr
cmd1 | cmd2                 # pipe: output di cmd1 → input di cmd2
```

---

## 💾 Disk e Memoria

```bash
# Disco
df -h                       # spazio dischi in formato leggibile
du -sh /path/               # dimensione directory
du -sh * | sort -h          # dimensione di ogni elemento, ordinata
lsblk                       # elenca dispositivi a blocchi
fdisk -l                    # partizioni (richiede root)
mount                       # mostra filesystem montati

# Memoria
free -h                     # RAM e swap in formato leggibile
vmstat                      # statistiche memoria e CPU
cat /proc/meminfo           # info dettagliate memoria

# I/O
iostat                      # statistiche I/O disco
iotop                       # processi con più I/O (come top)
```

---

## 🔧 Systemd e Servizi

```bash
# Gestione servizi
systemctl start nginx          # avvia servizio
systemctl stop nginx           # ferma servizio
systemctl restart nginx        # riavvia servizio
systemctl reload nginx         # ricarica config senza riavvio
systemctl enable nginx         # avvio automatico al boot
systemctl disable nginx        # disabilita avvio automatico
systemctl status nginx         # stato del servizio

# Informazioni
systemctl list-units           # elenca tutti i servizi attivi
systemctl list-units --failed  # servizi con errori
journalctl -u nginx            # log di un servizio
journalctl -u nginx -f         # log in tempo reale
journalctl --since "1 hour ago"  # log ultima ora
```

---

## 🔑 SSH

```bash
# Connessione
ssh user@host                  # connessione base
ssh -p 2222 user@host          # porta custom
ssh -i ~/.ssh/chiave user@host # usa chiave specifica

# Generazione chiavi
ssh-keygen -t ed25519          # genera coppia chiavi (algoritmo moderno)
ssh-keygen -t rsa -b 4096      # genera RSA 4096 bit
cat ~/.ssh/id_ed25519.pub      # mostra chiave pubblica

# Copia chiave pubblica
ssh-copy-id user@host          # copia chiave pubblica sull'host
# oppure manualmente:
cat ~/.ssh/id_ed25519.pub | ssh user@host "cat >> ~/.ssh/authorized_keys"

# SCP — copia file via SSH
scp file.txt user@host:/path/  # copia file verso host
scp user@host:/path/file.txt . # copia file da host
scp -r dir/ user@host:/path/   # copia directory

# Tunnel SSH
ssh -L 8080:localhost:80 user@host   # tunnel locale
ssh -D 1080 user@host                # SOCKS proxy
```

---

## ⚡ Tips Avanzati

```bash
# History
history                        # storico comandi
!!                             # ripete ultimo comando
!n                             # ripete comando numero n
Ctrl+R                         # ricerca nella history
history | grep ssh             # filtra history

# Shortcuts tastiera
Ctrl+A                         # vai a inizio riga
Ctrl+E                         # vai a fine riga
Ctrl+U                         # cancella da cursore a inizio
Ctrl+K                         # cancella da cursore a fine
Ctrl+W                         # cancella parola precedente
Alt+F / Alt+B                  # sposta parola avanti/indietro

# Alias utili (aggiungi a ~/.bashrc)
alias ll='ls -la'
alias gs='git status'
alias ports='ss -tulnp'
alias myip='curl ifconfig.me'
alias update='sudo apt update && sudo apt upgrade -y'

# Variabili d'ambiente
export VARIABILE=valore        # imposta variabile
echo $VARIABILE                # leggi variabile
env                            # mostra tutte le variabili
printenv PATH                  # mostra PATH

# Cron — task pianificati
crontab -e                     # modifica crontab utente
crontab -l                     # lista crontab
# Formato: minuto ora giorno mese giorno_settimana comando
# * * * * * /path/script.sh   → ogni minuto
# 0 2 * * * /path/backup.sh   → ogni giorno alle 2:00
# 0 * * * 1 /path/script.sh   → ogni lunedì all'ora
```

---

## 📖 Risorse Correlate

- 📝 Articolo completo: [profgiagnotti.it/blog/networking-os/comandi-linux-guida](https://profgiagnotti.it)
- ▶️ Video tutorial: [youtube.com/@profgiagnotti](https://youtube.com/@profgiagnotti)
- 💬 Domande: [Discord Prof. Giagnotti](https://discord.gg/profgiagnotti)

---

*⬅️ [Torna all'indice dei Cheatsheet](../README.md)*
