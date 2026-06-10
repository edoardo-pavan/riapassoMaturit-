# GIORNO 3 — SICUREZZA + TECNOLOGIE AVANZATE + BUSINESS CONTINUITY

---

## 1. VPN — VIRTUAL PRIVATE NETWORK

### Cos'è

Una VPN crea un **tunnel crittografato** attraverso una rete pubblica (Internet), permettendo a due reti private o a un client remoto di comunicare come se fossero sulla stessa rete locale.

### Tipi di VPN

| Tipo | Descrizione | Protocolli | Uso tipico |
|---|---|---|---|
| **Site-to-Site** | Collega due reti intere (es. sede A ↔ sede B) | IPsec, MPLS | Interconnessione sedi aziendali |
| **Client-to-Site** (Remote Access) | Collega un singolo dispositivo alla rete aziendale | IPsec, SSL/TLS, WireGuard | Smart working, dipendenti in viaggio |
| **SSL VPN** | Accesso via browser, senza client dedicato | TLS/SSL | Accesso rapido a portali aziendali |

### IPsec (il più usato nelle prove d'esame)

IPsec opera a **livello 3 (rete)**. Due modalità:

| Modalità | Cosa cifra | Uso |
|---|---|---|
| **Transport** | Solo il payload (dati) del pacchetto IP | Host-to-host |
| **Tunnel** | L'intero pacchetto IP originale (incapsulato in un nuovo pacchetto) | Site-to-Site, Client-to-Site |

**Protocolli IPsec:**
- **AH** (Authentication Header): garantisce integrità e autenticazione, NON cifra
- **ESP** (Encapsulating Security Payload): garantisce cifratura + integrità + autenticazione
- **IKE** (Internet Key Exchange): negozia le chiavi di sessione

### VPN nella seconda prova

Quando descrivi la VPN tra due sedi:
1. **Tipo**: VPN Site-to-Site IPsec in modalità tunnel
2. **Apparati**: firewall NGFW con funzionalità VPN integrati ai bordi di ciascuna rete
3. **Cifratura**: AES-256 con ESP
4. **Autenticazione**: pre-shared key o certificati digitali
5. **Rete**: ogni sede mantiene il proprio piano di indirizzamento privato, le VPN incapsulano il traffico

---

## 2. FIREWALL E ACL

### Firewall

Un firewall è un dispositivo che **filtra il traffico** di rete in base a regole. Può essere:

| Tipo | Descrizione | Esempi |
|---|---|---|
| **Packet filter** | Filtra per IP, porta, protocollo (livello 3-4) | ACL su router |
| **Stateful** | Tiene traccia dello stato delle connessioni, permette risposte a richieste autorizzate | Windows Firewall, iptables |
| **NGFW** (Next-Gen) | Aggiunge ispezione applicativa (livello 7), IPS, antivirus, filtro URL | Fortinet, Palo Alto, Cisco Firepower |

### ACL — Access Control List

Le ACL sono liste di regole applicate alle interfacce di un router/switch/firewall per permettere o negare traffico.

**Regole fondamentali:**
1. Le regole vengono valutate **dall'alto verso il basso**
2. La prima regola che fa match **blocca la valutazione**
3. Alla fine c'è un **deny implicito** (se nessuna regola matcha → il traffico è bloccato)

**ACL standard vs estese:**

| | Standard | Estese |
|---|---|---|
| **Filtra per** | Solo IP sorgente | IP sorgente, IP destinazione, protocollo, porta |
| **Range numerico** | 1-99, 1300-1999 | 100-199, 2000-2699 |
| **Esempio** | `permit 192.168.10.0 0.0.0.255` | `permit tcp 192.168.10.0 0.0.0.255 host 10.0.0.10 eq 80` |

### Posizionamento ACL

- **ACL in ingresso (in)**: il traffico viene filtrato PRIMA di essere processato dal router → più efficiente
- **ACL in uscita (out)**: il traffico viene filtrato DOPO essere stato processato

### Esempio ACL per la seconda prova

Scenario: azienda con VLAN Uffici (10), Server (20), DMZ (30), Ospiti (99).

```
! Blocca tutto il traffico dagli Ospiti verso le reti interne
access-list 101 deny ip 192.168.99.0 0.0.0.255 192.168.0.0 0.0.255.255

! Permetti agli Uffici di raggiungere i Server solo su porte specifiche
access-list 101 permit tcp 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255 eq 443
access-list 101 permit tcp 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255 eq 80

! Permetti a Internet di raggiungere solo la DMZ
access-list 101 permit ip any 192.168.30.0 0.0.0.255

! Nega tutto il resto (implicito, ma meglio esplicitarlo)
access-list 101 deny ip any any
```

---

## 3. DMZ — DEMILITARIZED ZONE

### Cos'è

La DMZ è una **zona di rete isolata** che ospita i servizi accessibili dall'esterno (web server, mail server, VPN gateway), separandoli dalla rete interna. Se un server nella DMZ viene compromesso, l'attaccante **non raggiunge la rete interna**.

```
Internet ──→ Firewall ──┬──→ DMZ (web, mail, VPN)
                         │
                         └──→ Rete Interna (server DB, PC, AD)
                              ↓
                         Accesso da DMZ a Interna: BLOCCATO (o limitato)
```

### Due modelli di DMZ

| Modello | Descrizione |
|---|---|
| **Single firewall** | Un solo firewall con 3 interfacce: Internet, DMZ, Interna |
| **Dual firewall** | Due firewall in cascata: Internet → FW1 → DMZ → FW2 → Interna (più sicuro) |

---

## 4. AAA, RADIUS e 802.1X

### AAA — Authentication, Authorization, Accounting

| A | Significato | Domanda a cui risponde |
|---|---|---|
| **Authentication** | Verifica l'identità dell'utente (username/password, certificato, MFA) | "Chi sei?" |
| **Authorization** | Determina cosa l'utente può fare (VLAN assegnata, ACL, permessi) | "Cosa puoi fare?" |
| **Accounting** | Traccia le attività dell'utente (log accessi, dati consumati) | "Cosa hai fatto?" |

### RADIUS (Remote Authentication Dial-In User Service)

RADIUS è un protocollo **client-server** che centralizza AAA. Un NAS (Network Access Server, es. switch, AP, firewall) inoltra le richieste di autenticazione al server RADIUS.

```
Client ──→ Switch/AP (NAS) ──→ Server RADIUS
  │                                  │
  │  "Username: mario"               │ Verifica credenziali
  │  "Password: *****"               │ su Active Directory / DB
  │                                  │
  │←── Access-Accept (o Reject) ────┘
  │←── VLAN dinamica assegnata ─────┘
```

**Vantaggi di RADIUS**:
- Autenticazione centralizzata (un solo DB utenti)
- Assegnazione dinamica VLAN (utente X → VLAN 10, utente Y → VLAN 20)
- Accounting (chi ha fatto login, quando, da dove)

### 802.1X — Autenticazione a livello di porta

802.1X blocca l'accesso alla rete **a livello fisico** finché il dispositivo non si è autenticato. Prima dell'autenticazione, solo il traffico EAP (Extensible Authentication Protocol) è permesso.

```
Dispositivo ──→ Switch (authenticator) ──→ Server RADIUS
(Supplicant)     porta bloccata              (Authentication Server)
                 fino ad autenticazione
```

**Flusso 802.1X**:
1. Il PC si connette alla porta dello switch → la porta è bloccata
2. Lo switch chiede le credenziali al PC (via EAP)
3. Lo switch inoltra al server RADIUS
4. Il server RADIUS verifica → Access-Accept
5. Lo switch sblocca la porta e assegna la VLAN configurata

---

## 5. MFA E ZERO TRUST

### Multi-Factor Authentication (MFA)

L'MFA richiede **almeno 2 fattori** di autenticazione tra categorie diverse:

| Fattore | Esempio |
|---|---|
| **Qualcosa che sai** | Password, PIN |
| **Qualcosa che hai** | Token, smartphone (OTP), smart card |
| **Qualcosa che sei** | Impronta digitale, volto, retina |

Esempio: password + codice OTP dall'app authenticator = 2 fattori.

### Zero Trust

Modello di sicurezza che presuppone **"non fidarti mai, verifica sempre"**. Nessun dispositivo o utente è considerato affidabile solo perché è dentro il perimetro di rete.

**Principi Zero Trust**:
1. **Verifica esplicita**: autentica e autorizza ogni accesso, ogni volta
2. **Accesso minimo**: dai solo i permessi necessari (least privilege)
3. **Assume breach**: comportati come se la rete fosse già compromessa
4. **Micro-segmentazione**: isola carichi di lavoro e dati sensibili

**Zero Trust Network Access (ZTNA)**: alternativa alle VPN tradizionali — l'utente accede solo all'applicazione specifica, non all'intera rete.

---

## 6. IDS, IPS e DPI

| Sistema | Cosa fa | Posizione |
|---|---|---|
| **IDS** (Intrusion Detection) | **Rileva** attività sospette e genera allarmi. NON blocca. | Fuori banda (copia del traffico) |
| **IPS** (Intrusion Prevention) | **Rileva E blocca** il traffico malevolo in tempo reale. | In linea (tutto il traffico lo attraversa) |
| **DPI** (Deep Packet Inspection) | Ispeziona il **contenuto** dei pacchetti, non solo gli header. Livello 7. | Integrato in NGFW, IPS, sistemi di filtraggio |

**Differenza chiave**: IDS = allarme (passivo), IPS = blocca (attivo), DPI = guarda dentro i dati.

---

## 7. ATTACCHI E SOLUZIONI

| Attacco | Descrizione | Soluzione |
|---|---|---|
| **DDoS** | Sovraccarica un server con traffico falso | Rate limiting, CDN, scrubbing center, IPS |
| **MITM** (Man in the Middle) | Intercetta la comunicazione tra due parti | Crittografia (TLS/SSL), certificate pinning |
| **Phishing** | Inganna l'utente per rubare credenziali | Filtri email, MFA, formazione utenti |
| **ARP Spoofing** | Falsifica tabelle ARP per intercettare traffico | DHCP Snooping, Dynamic ARP Inspection |
| **SQL Injection** | Inietta codice SQL dannoso in campi input | Prepared statement, input validation, WAF |
| **XSS** (Cross-Site Scripting) | Inietta JavaScript malevolo in pagine web | Output encoding, CSP, input sanitization |
| **Brute Force** | Prova tutte le combinazioni di password | Rate limiting, account lockout, MFA |
| **Ransomware** | Cifra i dati e chiede un riscatto | Backup offline, segmentazione rete, EDR |

---

## 8. WI-FI SECURITY

### Evoluzione protocolli

| Protocollo | Anno | Cifratura | Sicurezza |
|---|---|---|---|
| **WEP** | 1999 | RC4 (64/128 bit) | ❌ Facilmente crackabile |
| **WPA** | 2003 | TKIP + RC4 | ⚠️ Superato |
| **WPA2** | 2004 | AES-CCMP | ✅ Sicuro, standard attuale |
| **WPA3** | 2018 | AES-GCMP, SAE | ✅✅ Più sicuro, protegge da brute force |

### Autenticazione Wi-Fi

| Modalità | Descrizione | Uso |
|---|---|---|
| **WPA2-Personal** | Password condivisa (PSK) | Casa, piccolo ufficio |
| **WPA2-Enterprise** | Autenticazione via RADIUS (802.1X), ogni utente ha le sue credenziali | Aziende, scuole, ospedali |

**Per la seconda prova**: la rete aziendale USA SEMPRE WPA2-Enterprise / WPA3-Enterprise con autenticazione 802.1X via RADIUS. Il PSK va bene solo per la rete Ospiti (e va isolata in VLAN separata).

### BYOD (Bring Your Own Device)

I dispositivi personali dei dipendenti vanno messi in una **VLAN separata** con accesso limitato (solo Internet, no rete interna). Autenticazione via **captive portal** o 802.1X.

---

## 9. MPLS — MULTIPROTOCOL LABEL SWITCHING

### Cos'è

MPLS è una tecnologia che instrada il traffico usando **etichette (label)** invece degli indirizzi IP. Opera a un livello intermedio tra il livello 2 e il livello 3 (livello 2.5).

### Come funziona

1. Il primo router MPLS (ingress) esamina il pacchetto IP e gli appiccica un'etichetta
2. I router intermedi (LSR) instradano basandosi solo sull'etichetta → più veloce
3. L'ultimo router (egress) rimuove l'etichetta e consegna il pacchetto IP normalmente

### Vantaggi per le aziende

- **VPN MPLS**: il provider crea canali separati per ogni cliente sulla stessa infrastruttura
- **QoS garantito**: puoi prioritizzare il traffico voce/video
- **Scalabilità**: gestisce reti grandi con molte sedi
- **Ingegneria del traffico**: puoi decidere il percorso esatto del traffico

### MPLS vs Internet VPN

| | MPLS | VPN su Internet |
|---|---|---|
| **Affidabilità** | Alta (SLA garantito) | Dipendente da Internet |
| **Sicurezza** | Isolamento a livello di label | Crittografia IPsec |
| **Costo** | Alto (circuito dedicato) | Basso (usa Internet) |
| **Latenza** | Bassa e prevedibile | Variabile |

---

## 10. SD-WAN — Software-Defined WAN

### Cos'è

SD-WAN è un approccio che usa il **software per gestire la WAN**, separando il piano di controllo dal piano dati. Permette di usare più collegamenti (MPLS, Internet, 4G/5G) in modo intelligente.

### Come funziona

```
                 ┌──────────┐
                 │ SD-WAN   │ (orchestratore centrale)
                 │ Controller│
                 └────┬─────┘
        ┌─────────────┼─────────────┐
   ┌────┴────┐  ┌─────┴─────┐  ┌────┴────┐
   │ Sede A  │  │ Sede B    │  │ Cloud   │
   │ MPLS+LTE│  │ Internet  │  │ Gateway │
   └─────────┘  └───────────┘  └─────────┘
```

### Vantaggi SD-WAN

1. **Zero-touch provisioning**: nuovi siti si configurano automaticamente
2. **Application-aware routing**: il traffico critico va su MPLS, il resto su Internet
3. **Failover automatico**: se un link cade, il traffico passa sull'altro in secondi
4. **Costi ridotti**: puoi usare Internet economico invece di MPLS costoso per tutto
5. **Visibilità centralizzata**: monitoraggio da un unico pannello

### SD-WAN vs MPLS tradizionale

| | MPLS | SD-WAN |
|---|---|---|
| **Gestione** | Manuale, sede per sede | Centralizzata, via software |
| **Failover** | Lento (minuti) | Rapido (secondi) |
| **Trasporto** | Un solo tipo di link | MPLS + Internet + 4G/5G insieme |
| **Costo** | Alto | Medio-basso |
| **Complessità** | Bassa (il provider gestisce) | Media (gestisci tu le policy) |

---

## 11. IoT e MQTT

### IoT — Internet of Things

Dispositivi connessi a Internet che raccolgono e scambiano dati (sensori, attuatori, wearable, elettrodomestici smart).

**Sfide**: sicurezza, gestione identità, aggiornamenti, interoperabilità, consumo energetico.

### MQTT (Message Queuing Telemetry Transport)

Protocollo di **messaggistica leggero** per IoT, basato sul modello **publish-subscribe**.

```
┌─────────┐                  ┌──────────┐                  ┌─────────┐
│ Sensore │ ──pubblica──→   │  Broker  │  ←──sottoscrivi─ │ App     │
│ (temp.) │   "cucina/25°"  │  MQTT    │   topic          │ mobile  │
└─────────┘                  └──────────┘   "cucina/#"    └─────────┘
```

**Caratteristiche**:
- **Leggero**: header minimo (2 byte), ideale per dispositivi a bassa potenza
- **Asincrono**: il sensore pubblica e chiude, non aspetta risposta
- **QoS a 3 livelli**: 0 (al massimo una volta), 1 (almeno una volta), 2 (esattamente una volta)
- **Topic**: organizzazione gerarchica (`edificio/piano/stanza/sensore`)

---

## 12. RETI 5G E SATELLITARI

### 5G

Quinta generazione della telefonia mobile. Tre categorie di servizio:

| Categoria | Descrizione | Prestazioni |
|---|---|---|
| **eMBB** (enhanced Mobile Broadband) | Banda larga potenziata (streaming 4K/8K, VR) | Fino a 10 Gbps |
| **URLLC** (Ultra-Reliable Low Latency) | Comunicazioni critiche a bassissima latenza | < 1 ms |
| **mMTC** (massive Machine Type Communications) | Milioni di dispositivi IoT per km² | Bassa velocità, alta densità |

### Satellite

Nuove costellazioni **LEO** (Low Earth Orbit, ~500 km) come Starlink offrono Internet a bassa latenza (20-50 ms) anche in aree remote.

**Nella seconda prova**: 5G come backup per le sedi (failover SD-WAN) o per connettere sedi temporanee/cantieri. Satellite per sedi in zone non coperte da fibra.

---

## 13. BUSINESS CONTINUITY E DISASTER RECOVERY

### Definizioni

| Termine | Significato |
|---|---|
| **RTO** (Recovery Time Objective) | Tempo massimo accettabile di fermo (es. 4 ore) |
| **RPO** (Recovery Point Objective) | Quantità massima di dati che puoi permetterti di perdere (es. 15 minuti di transazioni) |
| **Business Continuity** | Mantiene i processi critici attivi durante un disastro |
| **Disaster Recovery** | Ripristina i sistemi dopo un evento catastrofico |

### Strategie

| Strategia | Descrizione | RTO tipico |
|---|---|---|
| **Backup** | Copia periodica dei dati su nastro/cloud | Ore/giorni |
| **Cold site** | Locazione vuota, attivi solo dopo disastro | Giorni |
| **Warm site** | Locazione con HW pronto, dati da ripristinare | Ore |
| **Hot site** | Replica in tempo reale, failover istantaneo | Minuti/secondi |
| **Cloud DR** | Replica su cloud (es. AWS, Azure) | Minuti |

### Ridondanza

- **RAID**: dischi ridondanti nel server
- **Alimentazione**: doppia PSU, UPS, gruppo elettrogeno
- **Rete**: doppio link Internet (provider diversi), doppio switch/core
- **Server**: cluster con failover automatico
- **DNS**: server DNS primario + secondario (su sede diversa)
- **DHCP**: split scope o failover DHCP

### Cosa scrivere nella seconda prova

1. Indica **RTO e RPO** target
2. Descrivi la strategia di **backup** (frequenza, conservazione, dove)
3. Indica la **ridondanza** degli apparati critici
4. Specifica se usi un sito secondario (warm/hot) o cloud DR
5. Menziona il **piano di test** periodico

---

## DOMANDE PER L'AUTOVERIFICA

1. Differenza tra VPN Site-to-Site e Client-to-Site. Quali protocolli?
2. IPsec: modalità tunnel vs transport. Differenza?
3. Cos'è un ACL? Dove si posiziona (in/out) e perché?
4. Differenza tra IDS e IPS. Cosa fa il DPI?
5. Descrivi un'architettura AAA con RADIUS in una rete con più VLAN.
6. Cos'è 802.1X? Come funziona l'autenticazione a livello di porta?
7. MFA: quali sono i 3 fattori di autenticazione?
8. Cos'è lo Zero Trust? Perché supera il modello a "castello con fossato"?
9. Differenza WPA2-Personal vs WPA2-Enterprise. Quale usi in azienda?
10. MPLS: come funziona? Vantaggi per un'azienda con più sedi?
11. SD-WAN: cos'è e perché sta sostituendo l'MPLS tradizionale?
12. MQTT: modello publish-subscribe. A cosa servono i topic?
13. IoT: principali sfide di sicurezza.
14. 5G: cosa sono eMBB, URLLC, mMTC?
15. RTO e RPO: cosa misurano e che differenza c'è?
16. Differenza tra hot site, warm site, cold site.
17. Come implementeresti il disaster recovery per una PMI con 2 sedi?
