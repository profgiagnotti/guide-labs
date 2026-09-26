# 🌐 Comandi Cisco IOS — Cheatsheet Completo

> **Prof. Giagnotti** · [profgiagnotti.it](https://profgiagnotti.it)
> Livello: ⭐⭐ Medio · Testato su: IOS 15.x / IOS-XE · Aggiornato: 2026

---

## 📑 Indice

- [Modalità IOS](#-modalità-ios)
- [Comandi Base e Navigazione](#-comandi-base-e-navigazione)
- [Configurazione Hostname e Password](#-configurazione-hostname-e-password)
- [Interfacce](#-interfacce)
- [VLAN e Trunk](#-vlan-e-trunk)
- [Routing Statico](#-routing-statico)
- [OSPF](#-ospf)
- [EIGRP](#-eigrp)
- [ACL — Access Control List](#-acl--access-control-list)
- [NAT](#-nat)
- [DHCP](#-dhcp)
- [SSH e Accesso Remoto](#-ssh-e-accesso-remoto)
- [Salvataggio e Ripristino](#-salvataggio-e-ripristino)
- [Troubleshooting](#-troubleshooting)
- [Comandi Show Essenziali](#-comandi-show-essenziali)

---

## 🔄 Modalità IOS

```
Router>                    # User EXEC — comandi di sola lettura
Router# enable             # entra in Privileged EXEC
Router#                    # Privileged EXEC — tutti i comandi show/debug
Router# configure terminal # entra in Global Configuration
Router(config)#            # Global Configuration — modifiche globali
Router(config)# interface fa0/0    # entra in Interface Configuration
Router(config-if)#         # Interface Configuration
Router(config)# router ospf 1      # entra in Router Configuration
Router(config-router)#     # Router Configuration

# Uscire dalle modalità
Router(config-if)# exit    # torna al livello superiore
Router(config-if)# end     # torna direttamente a Privileged EXEC
Ctrl+Z                     # equivalente a end
```

---

## ⌨️ Comandi Base e Navigazione

```
# Shortcuts tastiera
?                          # mostra comandi disponibili
co?                        # mostra comandi che iniziano con "co"
copy ?                     # mostra opzioni del comando copy
Tab                        # autocompletamento comando
Ctrl+C                     # interrompe comando o debug
Ctrl+Shift+6               # interrompe ping/traceroute in corso
Up Arrow                   # comando precedente nella history
no comando                 # annulla/rimuove una configurazione

# Disabilita DNS lookup (evita attese su typo)
Router(config)# no ip domain-lookup

# Disabilita messaggi che interrompono la digitazione
Router(config)# line console 0
Router(config-line)# logging synchronous

# Paginazione output
Router# terminal length 0  # disabilita paginazione (mostra tutto)
Router# terminal length 24 # ripristina paginazione standard
```

---

## 🏷️ Configurazione Hostname e Password

```
# Hostname
Router(config)# hostname R1

# Password Enable (accesso a Privileged EXEC)
R1(config)# enable secret CiscoPass123   # ← SEMPRE usare secret (MD5)
R1(config)# enable password cisco        # ← evitare, testo in chiaro ⚠️

# Password Console
R1(config)# line console 0
R1(config-line)# password cisco123
R1(config-line)# login
R1(config-line)# exec-timeout 10 0      # timeout 10 minuti

# Password VTY (Telnet/SSH)
R1(config)# line vty 0 4
R1(config-line)# password cisco123
R1(config-line)# login
R1(config-line)# transport input ssh    # solo SSH (sicuro)

# Cifratura di tutte le password in chiaro
R1(config)# service password-encryption

# Banner MOTD
R1(config)# banner motd #
Accesso autorizzato esclusivamente al personale autorizzato.
#
```

---

## 🔌 Interfacce

```
# Configurazione IP su interfaccia router
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description LAN principale
R1(config-if)# no shutdown              # attiva interfaccia ← obbligatorio!
R1(config-if)# exit

# Interfaccia seriale
R1(config)# interface Serial0/0/0
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# clock rate 64000         # solo sul DCE (lato clock)
R1(config-if)# no shutdown

# Interfaccia Loopback (virtuale, sempre UP)
R1(config)# interface Loopback0
R1(config-if)# ip address 1.1.1.1 255.255.255.255

# Spegnere un'interfaccia
R1(config-if)# shutdown

# Sub-interfacce (Router on a Stick)
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# encapsulation dot1Q 10        # VLAN 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0

R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# encapsulation dot1Q 20        # VLAN 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
```

---

## 🏗️ VLAN e Trunk

```
# Creazione VLAN (su switch)
SW1(config)# vlan 10
SW1(config-vlan)# name UFFICIO
SW1(config)# vlan 20
SW1(config-vlan)# name SERVER
SW1(config)# vlan 99
SW1(config-vlan)# name MANAGEMENT

# Assegnare porta a VLAN (Access port)
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10

# Trunk port
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk encapsulation dot1q  # su alcuni switch
SW1(config-if)# switchport trunk allowed vlan 10,20,99
SW1(config-if)# switchport trunk native vlan 99       # VLAN nativa

# IP di management su SVI (Switch Virtual Interface)
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config)# ip default-gateway 192.168.99.1

# Verifica
SW1# show vlan brief
SW1# show interfaces trunk
SW1# show interfaces fa0/1 switchport
```

---

## 🗺️ Routing Statico

```
# Route statica
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
#                    ↑ rete dest  ↑ subnet mask  ↑ next-hop

# Route statica con interfaccia uscita
R1(config)# ip route 192.168.2.0 255.255.255.0 Serial0/0/0

# Route di default (gateway of last resort)
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2

# Route con distanza amministrativa (floating static route)
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2 150
# AD 150 = usata solo se la route principale cade

# Verifica
R1# show ip route
R1# show ip route static
R1# ping 192.168.2.1 source GigabitEthernet0/0
```

---

## 🔄 OSPF

```
# Configurazione OSPF base (single area)
R1(config)# router ospf 1                    # process ID (locale)
R1(config-router)# router-id 1.1.1.1         # consigliato: impostarlo manualmente
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.0.0 0.0.0.3 area 0
R1(config-router)# passive-interface GigabitEthernet0/0  # non invia hello su LAN

# Propagare default route
R1(config-router)# default-information originate

# Costo interfaccia (modifica metrica)
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip ospf cost 10

# Autenticazione OSPF
R1(config-if)# ip ospf authentication message-digest
R1(config-if)# ip ospf message-digest-key 1 md5 Password123

# Verifica
R1# show ip ospf neighbor
R1# show ip ospf database
R1# show ip route ospf
R1# debug ip ospf events   # ⚠️ disabilita dopo uso: undebug all
```

---

## 🚀 EIGRP

```
# Configurazione EIGRP base
R1(config)# router eigrp 100               # AS number (deve essere uguale su tutti)
R1(config-router)# network 192.168.1.0 0.0.0.255
R1(config-router)# network 10.0.0.0 0.0.0.3
R1(config-router)# no auto-summary          # disabilita auto-summarization
R1(config-router)# passive-interface GigabitEthernet0/0

# Bandwidth interfaccia (influenza metrica EIGRP)
R1(config)# interface Serial0/0/0
R1(config-if)# bandwidth 1544              # in Kbps

# Verifica
R1# show ip eigrp neighbors
R1# show ip eigrp topology
R1# show ip route eigrp
```

---

## 🛡️ ACL — Access Control List

```
# ACL Standard (filtra solo IP sorgente) — numerata 1-99
R1(config)# access-list 10 permit 192.168.1.0 0.0.0.255
R1(config)# access-list 10 deny any
# → Applica il più vicino possibile alla DESTINAZIONE

# ACL Estesa (filtra src, dst, protocollo, porta) — numerata 100-199
R1(config)# access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80
R1(config)# access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 443
R1(config)# access-list 100 deny ip any any
# → Applica il più vicino possibile alla SORGENTE

# ACL Named (consigliata — più leggibile e modificabile)
R1(config)# ip access-list extended BLOCCA-TELNET
R1(config-ext-nacl)# deny tcp any any eq 23
R1(config-ext-nacl)# permit ip any any

# Applicare ACL a interfaccia
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip access-group 100 in     # traffico IN entrata
R1(config-if)# ip access-group 100 out    # traffico IN uscita

# Applicare ACL a VTY (filtra accesso SSH/Telnet)
R1(config)# line vty 0 4
R1(config-line)# access-class 10 in

# Verifica
R1# show access-lists
R1# show ip interface GigabitEthernet0/0   # mostra ACL applicate
```

---

## 🔁 NAT

```
# NAT Statico (1:1 — server accessibile da internet)
R1(config)# ip nat inside source static 192.168.1.10 203.0.113.10

# NAT Dinamico con Pool
R1(config)# ip nat pool POOL_PUBBLICO 203.0.113.1 203.0.113.10 netmask 255.255.255.0
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)# ip nat inside source list 1 pool POOL_PUBBLICO

# PAT — Port Address Translation (più comune, 1 IP pubblico per tutti)
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload

# Definire interfacce inside/outside
R1(config)# interface GigabitEthernet0/0   # LAN
R1(config-if)# ip nat inside

R1(config)# interface GigabitEthernet0/1   # WAN
R1(config-if)# ip nat outside

# Verifica
R1# show ip nat translations
R1# show ip nat statistics
R1# clear ip nat translation *             # pulisce tabella NAT
```

---

## 📡 DHCP

```
# Configurazione DHCP server su router
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10   # esclude IP riservati

R1(config)# ip dhcp pool LAN_POOL
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
R1(dhcp-config)# lease 7                  # lease di 7 giorni
R1(dhcp-config)# domain-name profgiagnotti.it

# DHCP Relay (forward richieste verso server DHCP remoto)
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip helper-address 10.0.0.10  # IP del server DHCP

# Verifica
R1# show ip dhcp binding              # client con IP assegnato
R1# show ip dhcp pool
R1# show ip dhcp conflict
```

---

## 🔑 SSH e Accesso Remoto

```
# Prerequisiti: hostname e dominio configurati
R1(config)# hostname R1
R1(config)# ip domain-name profgiagnotti.it

# Generare chiavi RSA
R1(config)# crypto key generate rsa modulus 2048

# Configurare SSH v2
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 3

# Configurare VTY per SSH
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local

# Creare utente locale
R1(config)# username admin privilege 15 secret Password123

# Verifica
R1# show ip ssh
R1# show ssh

# Da client Linux per connettersi
# ssh admin@192.168.1.1
```

---

## 💾 Salvataggio e Ripristino

```
# Salvare configurazione (RAM → NVRAM)
R1# write memory                           # metodo classico
R1# copy running-config startup-config     # metodo esteso

# Vedere configurazioni
R1# show running-config                    # config attuale (RAM)
R1# show startup-config                    # config al boot (NVRAM)
R1# show running-config | include ospf     # filtra per keyword

# Backup su TFTP
R1# copy running-config tftp:
# → inserisci IP server TFTP e nome file

# Ripristino da TFTP
R1# copy tftp: running-config

# Cancellare configurazione (factory reset)
R1# write erase                            # cancella startup-config
R1# reload                                 # riavvia senza config

# Backup locale
R1# copy running-config flash:backup.cfg
R1# dir flash:                             # lista file in flash
```

---

## 🔍 Troubleshooting

```
# Ping esteso (più opzioni)
R1# ping 192.168.2.1 source GigabitEthernet0/0 repeat 100 size 1500

# Traceroute
R1# traceroute 8.8.8.8
R1# traceroute 8.8.8.8 source GigabitEthernet0/0

# Debug (⚠️ disabilitare sempre dopo l'uso su router in produzione)
R1# debug ip packet                        # tutti i pacchetti IP
R1# debug ip ospf events                   # eventi OSPF
R1# debug ip routing                       # cambiamenti routing table
R1# undebug all                            # DISABILITA tutti i debug

# Verifica interfacce
R1# show interfaces GigabitEthernet0/0
# Cerca: "line protocol is up" = OK
#        "line protocol is down" = problema fisico o keepalive
#        input/output errors = problemi fisici

# Verifica CDP (Cisco Discovery Protocol)
R1# show cdp neighbors                     # dispositivi vicini Cisco
R1# show cdp neighbors detail              # con IP e versione IOS
```

---

## 📊 Comandi Show Essenziali

```
# Stato generale
show version                   # versione IOS, uptime, memoria
show running-config            # configurazione corrente completa
show startup-config            # configurazione salvata
show flash:                    # contenuto memoria flash

# Interfacce
show interfaces                # tutte le interfacce con statistiche
show interfaces GigabitEthernet0/0  # singola interfaccia
show interfaces status         # stato sintetico (su switch)
show ip interface brief        # stato IP di tutte le interfacce ← molto usato

# Routing
show ip route                  # tabella di routing completa
show ip route ospf             # solo route OSPF
show ip route eigrp            # solo route EIGRP
show ip route static           # solo route statiche
show ip protocols              # protocolli di routing attivi

# VLAN e Switching
show vlan brief                # tabella VLAN sintetica
show interfaces trunk          # porte in modalità trunk
show mac address-table         # tabella MAC (su switch)
show spanning-tree             # stato STP

# Sicurezza
show access-lists              # tutte le ACL
show ip nat translations       # tabella NAT
show ssh                       # sessioni SSH attive
show users                     # utenti connessi
```

---

## 📖 Risorse Correlate

- 📝 Articolo completo: [profgiagnotti.it/blog/networking-os/cisco-ios-guida](https://profgiagnotti.it)
- ▶️ Video tutorial: [youtube.com/@profgiagnotti](https://youtube.com/@profgiagnotti)
- 💬 Domande: [Discord Prof. Giagnotti](https://discord.gg/profgiagnotti)
- 🔗 Documentazione ufficiale: [cisco.com/c/en/us/support](https://cisco.com/c/en/us/support)

---

*⬅️ [Torna all'indice dei Cheatsheet](../README.md)*
