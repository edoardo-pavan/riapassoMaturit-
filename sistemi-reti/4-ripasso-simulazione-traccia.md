# GIORNO 4 — RIPASSO + SIMULAZIONE TRACCIA + ED. CIVICA

---

## 1. SCHEMA RIPASSO RAPIDO (30 min)

### Mappa mentale della seconda prova

```
SECONDA PROVA — STRUTTURA IN 9 PUNTI
│
├── 1. ANALISI SCENARIO
│   └── Riscrivi la traccia, elenca requisiti e vincoli
│
├── 2. IPOTESI AGGIUNTIVE
│   └── Colma i buchi della traccia (host per reparto, banda, ecc.)
│
├── 3. TOPOLOGIA LOGICA
│   └── Disegno delle sedi, reti, apparati, collegamenti
│
├── 4. PIANO DI INDIRIZZAMENTO (VLSM) ★
│   └── Subnetting + VLAN + tabella indirizzi completa
│
├── 5. SICUREZZA (AAA, DMZ, ACL) ★
│   └── RADIUS, DMZ, firewall, ACL, MFA, Zero Trust
│
├── 6. SERVIZI DI RETE ★
│   └── DNS, DHCP, AD/LDAP, file sharing, web, mail
│
├── 7. INTERCONNESSIONE E CLOUD
│   └── VPN, SD-WAN, smart working, Cloud/Edge
│
├── 8. BUSINESS CONTINUITY
│   └── RTO, RPO, backup, ridondanza, disaster recovery
│
└── 9. QUESITI (SECONDA PARTE)
    └── DB (schema E-R + logico) o API REST o quesito tecnico
```

### Tabella riepilogativa tecnologie

| Categoria | Tecnologie chiave | Quando usarle |
|---|---|---|
| **Indirizzamento** | IPv4 privati, VLSM, NAT | Sempre, per ogni rete interna |
| **Segmentazione** | VLAN 802.1Q, trunk, router-on-a-stick | Separare reparti, sicurezza, broadcast |
| **Sicurezza accesso** | 802.1X, RADIUS, AAA, MFA | Reti con dati sensibili, autenticazione centralizzata |
| **Sicurezza perimetrale** | NGFW, ACL, DMZ, IDS/IPS, DPI | Confine tra Internet e rete interna |
| **Interconnessione** | VPN IPsec Site-to-Site | Collegare sedi remote |
| **Smart working** | VPN SSL Client-to-Site, ZTNA | Dipendenti da casa |
| **Wi-Fi** | WPA3-Enterprise, 802.1X | Rete aziendale wireless |
| **Wi-Fi ospiti** | WPA2-Personal, VLAN isolata, captive portal | Separare visitatori |
| **Servizi core** | DNS, DHCP, AD/LDAP | Base di ogni rete aziendale |
| **Server** | Virtualizzazione, container | Server farm, alta disponibilità |
| **WAN intelligente** | SD-WAN | Più sedi con link diversificati |
| **WAN tradizionale** | MPLS | Sedi con esigenze di QoS garantito |
| **IoT** | MQTT, 5G, Edge | Sensori, automazione, industria 4.0 |
| **Cloud** | IaaS/PaaS/SaaS, Hybrid Cloud | Scalabilità, backup, DR |
| **Resilienza** | RAID, UPS, doppio link, hot/warm site | Business continuity |

---

## 2. SIMULAZIONE TRACCIA COMPLETA

### SCENARIO: Azienda TechSolutions S.p.A.

TechSolutions è un'azienda di sviluppo software con **2 sedi** (Milano HQ e Torino) e circa **150 dipendenti** totali. Vuole rinnovare la propria infrastruttura di rete.

**Requisiti dalla traccia:**

**Sede di Milano (HQ):**
- 80 dipendenti negli uffici (PC connessi via cavo)
- 20 sviluppatori in un'area separata (rete isolata per motivi di sicurezza del codice)
- 5 server interni: Domain Controller (AD/DNS), file server, database server, server CI/CD, web server staging
- Wi-Fi aziendale per portatili aziendali (max 30 dispositivi)
- Wi-Fi ospiti separato (max 50 dispositivi)

**Sede di Torino:**
- 40 dipendenti (ufficio unico)
- Wi-Fi ospiti (max 30 dispositivi)
- 1 server locale per backup e DNS secondario

**Requisiti trasversali:**
- Le due sedi devono essere collegate in modo sicuro e permanente
- I dipendenti devono poter lavorare da casa (smart working) in sicurezza
- L'accesso alla rete (cabata e Wi-Fi) deve essere autenticato con credenziali aziendali
- I visitatori non devono poter accedere a nessuna risorsa interna
- Il database clienti è il dato più critico: richiede backup ogni ora e RPO di 15 minuti
- Un portale web pubblico (per i clienti) deve essere accessibile da Internet
- L'azienda vuole usare il cloud per il disaster recovery
- Si richiede ridondanza su tutti i collegamenti critici
- La rete deve essere pronta per future espansioni (almeno 30% di spazio IP libero)

---

### SVOLGIMENTO GUIDATO

#### 1. ANALISI DELLO SCENARIO

**Contesto organizzativo**: azienda IT con 2 sedi, 150 dipendenti totali.
**Requisiti espliciti**: segmentazione sviluppatori, Wi-Fi aziendale e ospiti, VPN Site-to-Site, smart working, autenticazione centralizzata, portale pubblico, backup orario, cloud DR, ridondanza, espandibilità.

#### 2. IPOTESI AGGIUNTIVE

- Totale 150 dipendenti → 190 host cablati (con margine 30%)
- Assegno 1 IP per PC + 1 IP per telefono VoIP + 1 IP per stampante = 3 IP per dipendente cablato
- Collego le due sedi con VPN IPsec Site-to-Site su Internet + link di backup 4G (SD-WAN)
- Uso indirizzi privati classe A: `10.0.0.0/8`

#### 3. TOPOLOGIA LOGICA

```
                    INTERNET
                       │
                  ┌────┴────┐
                  │ ISP 1   │ (fibra)
                  │ ISP 2   │ (backup 4G)
                  └────┬────┘
                       │
              ┌────────┴────────┐
              │  NGFW Milano    │
              │  (Firewall+VPN) │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              │  Core Switch L3 │
              └────────┬────────┘
     ┌─────────┬───────┼───────┬─────────┐
 ┌───┴───┐ ┌───┴───┐ ┌──┴──┐ ┌──┴──┐ ┌───┴───┐
 │Uffici │ │Svilup.│ │Server│ │WiFi │ │ Ospiti│
 │VLAN 10│ │VLAN 20│ │VLAN30│ │VLAN40│ │VLAN 99│
 └───────┘ └───────┘ └──────┘ └─────┘ └───────┘
                                       │
                              ┌────────┴────────┐
                              │     DMZ         │
                              │  (Web server)   │
                              │  VLAN 50        │
                              └─────────────────┘

                 VPN IPsec Site-to-Site
                       │
              ┌────────┴────────┐
              │  NGFW Torino    │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              │  Switch L2      │
              └────────┬────────┘
         ┌─────────────┼─────────────┐
     ┌───┴───┐     ┌───┴───┐     ┌───┴───┐
     │Uffici │     │Server │     │Ospiti │
     │VLAN 10│     │VLAN 30│     │VLAN 99│
     └───────┘     └───────┘     └───────┘
```

#### 4. PIANO DI INDIRIZZAMENTO E SEGMENTAZIONE (VLSM)

**Rete di partenza**: `10.0.0.0/8` (usiamo solo `10.100.0.0/16` per Milano, `10.200.0.0/16` per Torino)

**Sede Milano** — calcolo host per VLAN:

| VLAN | ID | Host richiesti | +30% espansione | Host necessari | Prefisso | Rete | Gateway | Broadcast |
|---|---|---|---|---|---|---|---|---|
| Uffici MI | 10 | 80 PC × 3 = 240 | 312 | 510 | /23 | 10.100.0.0/23 | .1 | 10.100.1.255 |
| Sviluppatori | 20 | 20 PC = 20 | 26 | 30 | /27 | 10.100.2.0/27 | .1 | 10.100.2.31 |
| Server MI | 30 | 5 server | 7 | 14 | /28 | 10.100.2.32/28 | .33 | 10.100.2.47 |
| Wi-Fi Aziendale | 40 | 30 disp. | 39 | 62 | /26 | 10.100.2.64/26 | .65 | 10.100.2.127 |
| DMZ | 50 | 2 server | 3 | 6 | /29 | 10.100.2.128/29 | .129 | 10.100.2.135 |
| Ospiti MI | 99 | 50 disp. | 65 | 126 | /25 | 10.100.2.136/25 | .137 | 10.100.2.255 |
| Management | 255 | 10 disp. | 13 | 14 | /28 | 10.100.3.0/28 | .1 | 10.100.3.15 |

**Sede Torino**:

| VLAN | ID | Host richiesti | Host necessari | Prefisso | Rete | Gateway |
|---|---|---|---|---|---|---|
| Uffici TO | 10 | 40 PC × 3 = 120 | 254 | /24 | 10.200.0.0/24 | .1 |
| Server TO | 30 | 1 server | 14 | /28 | 10.200.1.0/28 | .1 |
| Ospiti TO | 99 | 30 disp. | 62 | /26 | 10.200.1.64/26 | .65 |

**Link VPN**: `/30` tra MI e TO: `10.255.255.0/30`

#### 5. ARCHITETTURA DI SICUREZZA

**AAA/RADIUS**:
- Server RADIUS su VLAN Server MI (integrato con AD)
- Switch e AP configurati come NAS (Network Access Server)
- 802.1X su tutte le porte cablate e Wi-Fi aziendale
- Assegnazione dinamica VLAN via RADIUS (dipendente → VLAN 10/40, sviluppatore → VLAN 20)

**DMZ**:
- VLAN 50 isolata dal resto della rete
- Contiene: web server pubblico (portale clienti), reverse proxy
- ACL: Internet → DMZ: consentito su porte 80/443. DMZ → Interna: BLOCCATO. DMZ → DB Server: solo porta 3306.

**Firewall/ACL**:
- NGFW con ispezione applicativa (livello 7)
- Regola principale: traffico da Ospiti verso reti interne = DENY
- Traffico tra VLAN filtrato da ACL su core switch L3
- IDS/IPS integrato nel NGFW

**MFA**: per accesso VPN smart working (password + OTP via app)

**Zero Trust**: accesso al server DB consentito solo da applicazioni autorizzate, non da utenti finali direttamente

#### 6. DEFINIZIONE DEI SERVIZI DI RETE

| Servizio | Dove | Note |
|---|---|---|
| **AD/LDAP** | Server 1, VLAN 30 MI | Domain Controller, autenticazione centralizzata |
| **DNS primario** | Server 1, VLAN 30 MI | Record interni aziendali |
| **DNS secondario** | Server TO, VLAN 30 TO | Ridondanza |
| **DNS pubblico** | Cloud (provider) | Record pubblici: www, mail |
| **DHCP** | Server 1, VLAN 30 MI | Scope diversi per ogni VLAN |
| **DHCP Relay** | Core switch L3 | Inoltra richieste dalle VLAN al server DHCP |
| **File server** | Server 2, VLAN 30 MI | Cartelle condivise, permessi AD |
| **DB Server** | Server 3, VLAN 30 MI | Dati clienti, backup orario |
| **Web staging** | Server 4, VLAN 30 MI | Test interni, CI/CD |
| **Web pubblico** | DMZ, VLAN 50 MI | Portale clienti accessibile da Internet |
| **VPN gateway** | NGFW MI | Endpoint VPN Site-to-Site e Client-to-Site |

#### 7. INTERCONNESSIONE, MOBILITÀ E CLOUD

- **Milano ↔ Torino**: VPN IPsec Site-to-Site in modalità tunnel (AES-256, ESP). Backup via SD-WAN su 4G.
- Ogni sede ha NGFW con doppio link WAN (fibra primaria, 4G backup)
- **Smart working**: VPN SSL Client-to-Site (o ZTNA per accesso granulare) + MFA
- **Cloud**: backup giornaliero del DB su cloud (AWS S3). In caso di disaster, failover su VM cloud (warm DR). RPO: 15 minuti, RTO: 4 ore.

#### 8. BUSINESS CONTINUITY E DISASTER RECOVERY

| Parametro | Valore |
|---|---|
| **RPO** | 15 minuti (log shipping DB ogni 15 min) |
| **RTO** | 4 ore (tempo per attivare VM cloud e redirect DNS) |
| **Backup DB** | Ogni ora su NAS locale + ogni ora su cloud |
| **Backup file server** | Incrementale ogni 6 ore, completo settimanale |
| **Ridondanza switch** | Due core switch in stack |
| **Ridondanza alimentazione** | UPS + gruppo elettrogeno (MI), UPS (TO) |
| **Ridondanza link** | Doppio ISP (fibra + 4G) con failover automatico SD-WAN |
| **Sito secondario** | Cloud DR (warm): VM pre-configurate, dati da ripristinare. In futuro: possibile hot site nella sede TO per i servizi critici |

#### 9. QUESITI — SECONDA PARTE (schema sintetico)

**Scenario DB**: l'azienda gestisce un database di progetti e clienti. Entità principali:
- **CLIENTE** (ID, nome, email, telefono, settore)
- **PROGETTO** (ID, nome, descrizione, data_inizio, data_fine, budget)
- **SVILUPPATORE** (ID, nome, cognome, email, specializzazione)

Relazioni:
- Ogni cliente può commissionare molti progetti (1:N)
- Ogni progetto è assegnato a un team di sviluppatori (N:N → necessita tabella ponte PROGETTO_SVILUPPATORE)

**Schema E-R**: CLIENTE ──1:N── PROGETTO ──N:N── SVILUPPATORE

**Schema logico** (3 tabelle + 1 ponte):
```
CLIENTE (id_cliente PK, nome, email, telefono, settore)
PROGETTO (id_progetto PK, nome, descrizione, data_inizio, data_fine, budget, id_cliente FK)
SVILUPPATORE (id_sviluppatore PK, nome, cognome, email, specializzazione)
PROGETTO_SVILUPPATORE (id_progetto FK, id_sviluppatore FK, ruolo, ore_previste) PK composta
```

---

## 3. ED. CIVICA — CONFINI REALI E CONFINI VIRTUALI

### Confine reale

Il confine **fisico** di uno stato: territorio, frontiere, dogane. È definito da leggi, trattati, confini geografici. Ha una giurisdizione chiara: dentro quel confine valgono le leggi di quello stato.

### Confine virtuale

Il confine nel **cyberspazio**: non è geografico. I dati attraversano server in paesi diversi in millisecondi. Un attacco informatico può partire dalla Russia e colpire un'azienda italiana senza mai varcare fisicamente la frontiera. La rete Internet non ha confini.

### Sfide del confine virtuale

| Problema | Descrizione |
|---|---|
| **Giurisdizione** | Se un server in USA contiene dati di cittadini EU, quale legge si applica? |
| **Sovranità digitale** | Chi controlla i dati? Dove risiedono fisicamente? (→ GDPR) |
| **Data localization** | Alcuni paesi impongono che i dati dei cittadini restino su server nazionali |
| **Attacchi cross-border** | Un attacco può venire da qualsiasi parte del mondo, rendendo difficile l'attribuzione e la risposta legale |
| **Cyberwarfare** | Stati usano attacchi informatici come strumento di conflitto (es. attacchi a infrastrutture critiche) |

### GDPR e confini

Il **GDPR** (Regolamento Europeo 2016/679) protegge i dati dei cittadini EU **ovunque vengano trattati**. Anche se un'azienda USA tratta dati di europei, deve rispettare il GDPR. È un esempio di come il "confine virtuale" richieda nuove forme di regolamentazione che superano i confini fisici.

### Collegamenti con Sistemi e Reti

- **Dove metti i server?** Se i dati sono in cloud, devi sapere in che paese risiedono fisicamente (→ data sovereignty)
- **VPN e crittografia**: strumenti per difendere i confini virtuali
- **DMZ**: il confine tra "fuori" e "dentro" la rete è un confine virtuale da difendere
- **Firewall e ACL**: le "dogane" del mondo digitale

---

## DOMANDE PER L'AUTOVERIFICA

1. Elenca i 9 punti della struttura di una seconda prova di Sistemi e Reti.
2. Qual è la differenza tra requisiti espliciti e ipotesi aggiuntive?
3. Quando usi VLSM invece del subnetting a maschera fissa?
4. Quali sono le VLAN tipiche di una rete aziendale e perché le separi?
5. Disegna uno schema di DMZ con singolo firewall e spiega le ACL minime.
6. Come configuri RADIUS + 802.1X per autenticare i dipendenti su rete cablata e Wi-Fi?
7. Quale protocollo VPN usi per collegare due sedi? E per lo smart working?
8. RTO e RPO: spiegali con un esempio concreto.
9. Schema E-R: entità CLIENTE, ORDINE, PRODOTTO. Definisci le relazioni e lo schema logico.
10. Cosa si intende per "confine virtuale"? Fai un esempio concreto di conflitto tra confini reali e virtuali.
11. In che modo il GDPR supera i confini geografici tradizionali?
