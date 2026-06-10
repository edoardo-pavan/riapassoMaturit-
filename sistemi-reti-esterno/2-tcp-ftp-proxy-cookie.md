# GIORNO 2 — TCP/UDP APPROFONDITO, FTP, PROXY, CACHE, COOKIE

> Cose che il commissario esterno potrebbe chiedere e che non abbiamo visto nel nostro programma.

---

## 1. COMMUTAZIONE DI CIRCUITO vs COMMUTAZIONE DI PACCHETTO

| | Commutazione di circuito | Commutazione di pacchetto |
|---|---|---|
| **Come funziona** | Stabilisce un percorso dedicato prima di trasmettere, lo mantiene per tutta la comunicazione | I dati vengono divisi in pacchetti, instradati indipendentemente |
| **Esempio classico** | Rete telefonica tradizionale (PSTN) | Internet (IP) |
| **Vantaggi** | Latenza costante, banda garantita | Uso efficiente della rete, robusto ai guasti |
| **Svantaggi** | Spreco di risorse (il circuito è riservato anche se non trasmetti) | Latenza variabile, congestione possibile |
| **Adatto per** | Chiamate vocali tradizionali | Dati, web, email |

---

## 2. PROTOCOLLI LIVELLO RETE (INTERNET LAYER)

### IP (Internet Protocol)

Compiti principali:
- **Indirizzamento**: ogni host ha un indirizzo IP univoco
- **Instradamento**: decidere il percorso migliore per ogni pacchetto
- **Frammentazione**: se il pacchetto è troppo grande per il link, lo spezza

**IPv4 vs IPv6**:

| | IPv4 | IPv6 |
|---|---|---|
| Bit | 32 bit (4 miliardi di indirizzi) | 128 bit (3.4×10³⁸) |
| Formato | 4 numeri decimali (192.168.1.1) | 8 gruppi esadecimali (2001:db8::1) |
| NAT | Necessario (pochi indirizzi) | Non serve (indirizzi per tutti) |
| Sicurezza | IPSec opzionale | IPSec obbligatorio |
| Configurazione | Manuale o DHCP | Auto-configurazione (SLAAC) |
| Broadcast | Sì | No (sostituito da multicast) |

**Tipi di indirizzi IPv4**:
- **Pubblici**: assegnati da IANA/ISP, univoci su Internet
- **Privati** (RFC 1918): 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 — usati nelle LAN con NAT

### NAT (Network Address Translation)

Traduce indirizzi privati in un indirizzo pubblico per uscire su Internet.

```
LAN (192.168.1.0/24)                 Internet
  PC1: 192.168.1.10 ──┐
  PC2: 192.168.1.11 ──┤── Router NAT ── 203.0.113.5 (IP pubblico)
  PC3: 192.168.1.12 ──┘
```

Il router tiene una tabella NAT: per ogni connessione, ricorda chi ha fatto la richiesta e su quale porta, per instradare correttamente la risposta.

### ARP (Address Resolution Protocol)

Traduce indirizzi IP in indirizzi MAC (livello 3 → livello 2).

```
Chi ha l'IP 192.168.1.5? Dimmelo!
        ↓ (broadcast ARP request)
  "Io! Il mio MAC è AA:BB:CC:DD:EE:FF"
        ↑ (ARP reply)
```

Il risultato viene salvato nella **tabella ARP** (cache).

### ICMP (Internet Control Message Protocol)

Messaggi di controllo e diagnostica.
- **Ping** (echo request/reply): verifica se un host è raggiungibile
- **Traceroute**: scopre il percorso dei pacchetti
- **Destination unreachable**: host o porta non raggiungibili

---

## 3. PROTOCOLLI LIVELLO TRASPORTO: TCP E UDP

### TCP (Transmission Control Protocol)

**Protocollo affidabile**, orientato alla connessione.

**Caratteristiche**:
- **Connessione**: 3-way handshake prima di trasmettere dati
- **Affidabilità**: ogni segmento viene confermato (ACK). Se non arriva ACK → ritrasmissione
- **Ordine**: i segmenti sono numerati, il ricevente li riordina
- **Controllo di flusso (windowing)**: il ricevente comunica quanto spazio ha nel buffer (window). Il mittente non manda più dati di quanti il ricevente possa gestire.
- **Controllo di congestione**: il mittente riduce la velocità se rileva congestione (slow start, AIMD)

**3-way handshake (apertura connessione)**:

```
Client                              Server
  │─── SYN (seq=x) ────────────────→│  "Posso connettermi?"
  │←── SYN+ACK (seq=y, ack=x+1) ───│  "Sì, puoi. Posso connettermi anch'io?"
  │─── ACK (ack=y+1) ──────────────→│  "Sì, connesso."
  │                                  │
  │←──── dati (connessione attiva) ─→│
```

### Socket

Un **socket** è l'endpoint di una connessione di rete: combinazione di **IP + porta**.

```
192.168.1.10:49152  →  93.184.216.34:80
└─────┬─────┘└┬┘      └──────┬──────┘└┬┘
    IP       porta         IP       porta (HTTP)
```

Un server web ascolta su `0.0.0.0:80`. Ogni client che si connette ottiene un socket diverso (stesso IP server, stessa porta server, ma IP client e porta client diversi).

### UDP (User Datagram Protocol)

**Protocollo non affidabile**, senza connessione.

**Caratteristiche**:
- **Nessuna connessione**: si manda e basta (fire and forget)
- **Nessuna garanzia**: i pacchetti possono perdersi, arrivare fuori ordine, duplicati
- **Veloce**: nessun overhead di handshake, ACK, ritrasmissioni
- **Header minimo**: 8 byte (TCP ne ha 20)

**Quando usare UDP**: streaming video/audio (qualche pacchetto perso non è drammatico), giochi online (latenza bassa > affidabilità), DNS, VoIP, DHCP.

### TCP vs UDP

| | TCP | UDP |
|---|---|---|
| **Connessione** | Sì (3-way handshake) | No |
| **Affidabilità** | ACK, ritrasmissioni | Nessuna garanzia |
| **Ordine** | Garantito | Non garantito |
| **Velocità** | Più lento | Più veloce |
| **Header** | 20-60 byte | 8 byte |
| **Uso** | Web, email, file transfer | Streaming, gaming, DNS, VoIP |

---

## 4. PROTOCOLLI LIVELLO APPLICAZIONE

### HTTP (HyperText Transfer Protocol)

Protocollo per il web. Il browser (client) richiede pagine al web server.

**Struttura URL**:
```
https://www.example.com:443/path/page?query=value#fragment
└┬┘   └──┬───┘└──────┬──────┘ └─┬─┘ └───┬───┘ └──┬──┘
schema  host     dominio       porta  path   query   fragment
```

**Metodi HTTP**:
- `GET`: richiedi una risorsa
- `POST`: invia dati al server (form)
- `PUT`: aggiorna una risorsa
- `DELETE`: cancella una risorsa
- `HEAD`: come GET ma solo gli header (senza body)

**Connessioni**:
- **Non persistenti (HTTP/1.0)**: per ogni oggetto (HTML, immagine, CSS) viene aperta una nuova connessione TCP → inefficiente
- **Persistenti (HTTP/1.1)**: la connessione TCP rimane aperta per più richieste → più veloce
- **HTTP/2**: multiplexing: più richieste sulla stessa connessione contemporaneamente
- **HTTP/3**: usa UDP (QUIC) invece di TCP

### Proxy Server

Un server intermedio tra client e server web.

```
Client ──→ Proxy ──→ Web server
```

**Funzioni**:
- **Cache**: salva copie di pagine visitate di frequente → risposta più veloce
- **Filtraggio**: blocca siti vietati (aziende, scuole)
- **Anonimato**: il server web vede l'IP del proxy, non quello del client
- **Logging**: monitora cosa fanno gli utenti

### Cache Web

Memorizza temporaneamente risorse web per riutilizzarle senza richiederle di nuovo.
- **Browser cache**: immagini, CSS, JS salvati sul disco locale del browser
- **Proxy cache**: contenuti condivisi salvati sul proxy server
- **CDN** (Content Delivery Network): server sparsi nel mondo che servono contenuti dal nodo più vicino

**Header HTTP per la cache**: `Cache-Control: max-age=3600` (valido per 1 ora), `ETag` (identificatore versione), `Expires` (data di scadenza).

### Cookie

Piccoli file di testo salvati dal browser su richiesta del server.

**A cosa servono**:
- **Gestione sessione**: login, carrello acquisti
- **Personalizzazione**: lingua, tema, preferenze
- **Tracciamento**: pubblicità mirata, analytics

**Tipi di cookie**:
- **Di sessione**: cancellati quando chiudi il browser
- **Persistenti**: rimangono fino alla scadenza impostata (giorni, mesi)
- **First-party**: dal sito che stai visitando
- **Third-party**: da altri domini (pubblicità, analytics) — sempre più bloccati dai browser (privacy)

### FTP (File Transfer Protocol)

Protocollo per trasferire file tra client e server. Usa **DUE** connessioni TCP separate:
- **Connessione di controllo** (porta 21): invio comandi (login, lista directory, cambia cartella)
- **Connessione dati** (porta 20 o dinamica): trasferimento effettivo dei file

**Modalità FTP**:

| | Attiva | Passiva |
|---|---|---|
| **Chi apre la connessione dati** | Il server si connette al client | Il client si connette al server |
| **Problema** | Il firewall del client blocca connessioni in ingresso | Più compatibile con firewall/NAT |
| **Comando** | `PORT` (il client dice al server dove connettersi) | `PASV` (il server dice al client dove connettersi) |

**Oggi si usa quasi sempre la modalità passiva** per compatibilità con firewall e NAT.

### Posta Elettronica (SMTP, POP3, IMAP)

| Protocollo | Ruolo | Porta |
|---|---|---|
| **SMTP** | Invio della mail (client → server, server → server) | 25, 587 (con TLS) |
| **POP3** | Scarica la mail dal server al client. Dopo il download, la mail viene (di solito) cancellata dal server. | 110, 995 (TLS) |
| **IMAP** | Gestisce le mail sul server. Il client vede tutto ciò che è sul server. Puoi leggere da più dispositivi. | 143, 993 (TLS) |

**Differenza chiave**: POP3 = scarica e cancella. IMAP = mantiene tutto sul server, accesso da più device.

---

## DOMANDE PER L'AUTOVERIFICA

1. Commutazione di circuito vs commutazione di pacchetto: 2 differenze.
2. IPv4 vs IPv6: 3 differenze chiave.
3. Cos'è il NAT? Perché è necessario con IPv4?
4. A cosa servono ARP e ICMP?
5. TCP: spiega il 3-way handshake e il windowing.
6. TCP vs UDP: quando usi l'uno e quando l'altro?
7. Cos'è un socket?
8. Struttura di un URL. Cosa sono schema, host, path, query?
9. Connessioni persistenti vs non persistenti in HTTP.
10. Cos'è un proxy server? 3 funzioni.
11. Cosa sono i cookie? Differenza tra first-party e third-party.
12. FTP: perché usa due connessioni? Differenza tra modalità attiva e passiva.
13. Differenza tra POP3 e IMAP.
