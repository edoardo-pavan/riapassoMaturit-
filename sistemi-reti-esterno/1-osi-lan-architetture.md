# GIORNO 1 — MODELLO OSI, LIVELLO FISICO/DATA LINK, ARCHITETTURE

> Commissario esterno: Prof. Piazzi. Qui copriamo ciò che il NOSTRO programma non approfondisce.

---

## 1. MODELLO ISO OSI (7 LIVELLI)

Il modello **ISO/OSI** (Open Systems Interconnection) è uno standard teorico che divide la comunicazione di rete in 7 livelli. Non è implementato veramente (quello implementato è TCP/IP a 4/5 livelli), ma serve come modello di riferimento.

### I 7 livelli

```
┌─────────────────────────┐
│  7. Applicazione        │  ← Interfaccia con l'utente (HTTP, FTP, SMTP, DNS)
├─────────────────────────┤
│  6. Presentazione       │  ← Formattazione, cifratura, compressione (SSL/TLS, JPEG, ASCII)
├─────────────────────────┤
│  5. Sessione            │  ← Gestisce il dialogo tra applicazioni (apertura/chiusura connessione)
├─────────────────────────┤
│  4. Trasporto           │  ← Consegna end-to-end affidabile (TCP, UDP)
├─────────────────────────┤
│  3. Rete                │  ← Instradamento e indirizzamento logico (IP, router)
├─────────────────────────┤
│  2. Data Link (Linea)   │  ← Frame, MAC address, rilevazione errori (Ethernet, switch)
├─────────────────────────┤
│  1. Fisico              │  ← Bit sul mezzo trasmissivo (cavi, tensioni, frequenze)
└─────────────────────────┘
```

### Confronto OSI vs TCP/IP

```
OSI (7 livelli)              TCP/IP (4 livelli)
─────────────                ─────────────────
7. Applicazione ─┐
6. Presentazione ├──→        4. Applicazione
5. Sessione     ─┘
────────────────                ────────────────
4. Trasporto    ────→        3. Trasporto (TCP/UDP)
────────────────                ────────────────
3. Rete          ────→        2. Internet (IP)
────────────────                ────────────────
2. Data Link    ─┐
1. Fisico       ─┴──→        1. Accesso alla rete
```

### Incapsulamento (imbustamento)

Ogni livello aggiunge la propria **intestazione (header)** ai dati. Il livello destinatario fa l'operazione inversa (estrazione).

```
Mittente:                        Destinatario:
Dati                              Dati
+ header Applicazione            - header Applicazione
+ header Trasporto               - header Trasporto
+ header Rete                    - header Rete
+ header Link + trailer          - header Link - trailer
→ bit sul cavo                   ← bit dal cavo
```

**PDU (Protocol Data Unit)** a ogni livello:
- Applicazione → **dati/messaggio**
- Trasporto → **segmento** (TCP) o **datagram** (UDP)
- Rete → **pacchetto**
- Data Link → **frame**
- Fisico → **bit**

---

## 2. RETI LAN — LIVELLO FISICO

### Mezzi trasmissivi

| Mezzo | Tipo | Velocità | Distanza | Note |
|---|---|---|---|---|
| **UTP Cat5e/6** | Rame, doppino | 1-10 Gbps | 100m | Più usato in ufficio |
| **Fibra ottica** | Vetro, luce | Fino a 100+ Gbps | km | Lunghe distanze, immune EMI |
| **Wi-Fi** | Onde radio (2.4/5/6 GHz) | Fino a 9.6 Gbps (Wi-Fi 6E) | 30-100m | Mobilità |
| **Coassiale** | Rame | 10 Mbps (obsoleto) | 500m | Reti via cavo TV |

### Codifica del segnale

Trasforma i bit in segnali fisici.

| Codifica | Come funziona | Vantaggi |
|---|---|---|
| **NRZ** (Non Return to Zero) | 1 = tensione alta, 0 = tensione bassa. Il segnale NON torna a zero tra un bit e l'altro. | Semplice |
| **NRZI** (NRZ Inverted) | 1 = transizione (cambio), 0 = nessuna transizione | Risolve sequenze di 1 |
| **Manchester** | 1 = transizione alto→basso, 0 = transizione basso→alto. Metà bit. | Auto-sincronizzante, usato in Ethernet 10 Mbps |
| **PAM5** (Pulse Amplitude Modulation) | 5 livelli di tensione diversi = più bit per simbolo | Gigabit Ethernet (1000BASE-T) |

**Problema di NRZ**: lunghe sequenze di 0 o 1 → difficile sincronizzare il clock.

### Modulazione (ASK, FSK, PSK)

Tecniche per trasmettere dati su onde portanti.

| Modulazione | Cosa cambia | Esempio |
|---|---|---|
| **ASK** (Amplitude Shift Keying) | Ampiezza dell'onda | 1 = ampiezza alta, 0 = ampiezza bassa (o zero) |
| **FSK** (Frequency Shift Keying) | Frequenza dell'onda | 1 = frequenza alta, 0 = frequenza bassa |
| **PSK** (Phase Shift Keying) | Fase dell'onda | 1 = fase 0°, 0 = fase 180° |

**Wi-Fi**: usa modulazioni complesse come QAM (Quadrature Amplitude Modulation) che combina ampiezza e fase.

---

## 3. RETI LAN — LIVELLO DATA LINK

### Funzioni principali

1. **Framing**: impacchettare i bit in frame con delimitatori
2. **Indirizzamento fisico**: MAC address (48 bit)
3. **Rilevazione errori**: CRC (Cyclic Redundancy Check), bit di parità
4. **Controllo di flusso**: evitare che il mittente sovraccarichi il ricevente
5. **Accesso al mezzo**: chi trasmette e quando?

### Rilevazione e correzione errori

| Tecnica | Come funziona | Rileva? | Corregge? |
|---|---|---|---|
| **Bit di parità** | Aggiunge un bit per rendere pari/dispari il numero di 1 | Sì (errori singoli) | No |
| **CRC** (Cyclic Redundancy Check) | Divide i dati per un polinomio, il resto è il codice CRC | Sì (burst di errori) | No |
| **Codici di Hamming** | Aggiunge bit di parità incrociati | Sì | Sì (errori singoli) |
| **Ritrasmissione** | Se il frame ha errore → chiedi la ritrasmissione | Sì | Sì (via software) |

### Metodi di accesso al mezzo

| Metodo | Tipo | Come funziona |
|---|---|---|
| **CSMA/CD** | Casuale | Carrier Sense Multiple Access / Collision Detection. Ascolto il canale: se libero trasmetto. Se c'è collisione, mi fermo e riprovo dopo un tempo casuale. Usato in Ethernet classico. |
| **CSMA/CA** | Casuale | Collision Avoidance. Usa RTS/CTS per prenotare il canale. Usato in Wi-Fi (non può rilevare collisioni). |
| **Polling** | Controllato | Un nodo master interroga a turno ciascun nodo ("puoi trasmettere?"). Nessuna collisione. |
| **Token passing** | Controllato | Un "gettone" (token) circola tra i nodi. Solo chi ha il token può trasmettere. Usato in Token Ring (storico). |

### Ethernet

Standard IEEE 802.3. Usa CSMA/CD (nelle versioni half-duplex, ormai obsolete) o full-duplex con switch.
- **Frame Ethernet**: DA (6 byte) + SA (6 byte) + Tipo/Lunghezza (2 byte) + Dati (46-1500 byte) + CRC (4 byte)
- **MTU (Maximum Transmission Unit)**: 1500 byte standard

### Wi-Fi (IEEE 802.11)

Usa CSMA/CA perché non può rilevare collisioni (il segnale del proprio trasmettitore coprirebbe il segnale in arrivo).

**Bluetooth** (IEEE 802.15): corto raggio (10m), bassa potenza, usa frequency hopping (salta tra 79 frequenze, 1600 volte al secondo).

---

## 4. ARCHITETTURE DI RETE

### Classificazione per dimensione

| Rete | Estensione | Esempio |
|---|---|---|
| **PAN** (Personal) | Pochi metri | Bluetooth, cavo USB |
| **LAN** (Local) | Edificio/campus | Rete aziendale o scolastica |
| **MAN** (Metropolitan) | Città | Rete universitaria cittadina, WiMAX |
| **WAN** (Wide) | Nazione/continente | Internet |
| **GAN** (Global) | Globale | Internet stessa, reti satellitari |

### Topologie

| Topologia | Come sono collegati | Pro | Contro |
|---|---|---|---|
| **Stella** | Tutti a un nodo centrale (switch) | Guasto di un nodo non blocca gli altri | Se lo switch muore, muore tutto |
| **Bus** | Tutti su un unico cavo | Semplice, economica | Una rottura blocca tutto |
| **Anello** | Ognuno collegato al successivo | Nessuna collisione | Guasto di un nodo rompe l'anello |
| **Mesh** | Ognuno collegato a tutti (o molti) | Altissima ridondanza | Costosissima (n² collegamenti) |
| **Albero** | Gerarchica (stella di stelle) | Scalabile | Il guasto di un ramo isola i nodi sotto |

### Architetture logiche

| Architettura | Chi comanda? | Esempio |
|---|---|---|
| **Master-Slave** | Un nodo master controlla tutti gli slave. Comunicazione solo master↔slave (no slave↔slave). | Bluetooth classico, bus I2C/SPI, database replication |
| **Peer-to-Peer (P2P)** | Tutti i nodi sono uguali. Ognuno può fare sia da client che da server. | BitTorrent, blockchain, Skype (vecchio) |
| **Client-Server** | Il server offre servizi, i client li richiedono. Il server è sempre in ascolto. | Web (browser → server), email, database |

| | Master-Slave | P2P | Client-Server |
|---|---|---|---|
| **Controllo** | Centralizzato | Distribuito | Centralizzato |
| **Scalabilità** | Limitata dal master | Molto scalabile | Scalabile orizzontalmente |
| **Single point of failure** | Sì (il master) | No | Sì (il server) |
| **Comunicazione** | Solo master↔slave | Qualsiasi nodo | Client↔server (no client↔client diretto) |

### Intranet, Extranet, VPN

| Rete | Chi può accedere | Esempio |
|---|---|---|
| **Intranet** | Solo dipendenti interni | Portale HR aziendale, wiki interno |
| **Extranet** | Partner, fornitori, clienti autorizzati | Portale fornitori, area clienti B2B |
| **VPN** | Chiunque abbia credenziali + tunnel cifrato | Smart working, accesso remoto sicuro |

---

## DOMANDE PER L'AUTOVERIFICA

1. Elenca i 7 livelli del modello ISO OSI e la funzione di ciascuno.
2. Qual è il nome della PDU al livello Trasporto? E al livello Rete? E al livello Data Link?
3. Cos'è l'incapsulamento (imbustamento)? Cosa aggiunge ogni livello?
4. Differenza tra OSI (7 livelli) e TCP/IP (4 livelli).
5. Spiega la codifica Manchester e perché è migliore di NRZ.
6. Differenza tra CSMA/CD e CSMA/CA. Quale si usa in Wi-Fi e perché?
7. Cos'è il CRC? A che livello opera?
8. Differenza tra Master-Slave, Peer-to-Peer e Client-Server. Un esempio per ciascuno.
9. Intranet vs Extranet vs VPN: differenze.
10. Topologia a stella vs a mesh: pro e contro.
11. ASK vs FSK: cosa modulano?
