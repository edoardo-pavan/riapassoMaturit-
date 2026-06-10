# GIORNO 3 — CRITTOGRAFIA, PEC, FIRMA DIGITALE, SSL/TLS, BLOCKCHAIN

> Commissario esterno: la crittografia è un suo cavallo di battaglia. Qui va saputa bene.

---

## 1. SICUREZZA INFORMATICA: I 3 PILASTRI (CIA TRIAD)

| Pilastro | Inglese | Significato | Esempio di violazione |
|---|---|---|---|
| **Riservatezza** | Confidentiality | Solo chi è autorizzato può leggere i dati | Un hacker ruba un database di password |
| **Integrità** | Integrity | I dati non vengono modificati senza autorizzazione | Qualcuno altera un bonifico bancario |
| **Disponibilità** | Availability | I dati/servizi sono accessibili quando servono | Attacco DDoS che butta giù un sito |

---

## 2. CRITTOGRAFIA: CONCETTI BASE

### Definizioni

- **Testo in chiaro (plaintext)**: il messaggio originale leggibile
- **Testo cifrato (ciphertext)**: il messaggio dopo la cifratura, illeggibile
- **Cifratura**: trasformare plaintext → ciphertext usando una chiave
- **Decifratura**: trasformare ciphertext → plaintext usando una chiave
- **Chiave**: parametro segreto che controlla la trasformazione

### Tecniche classiche

| Tecnica | Come funziona | Esempio |
|---|---|---|
| **Sostituzione** | Ogni lettera viene sostituita con un'altra secondo una regola fissa | **Cifrario di Cesare**: A→D, B→E, C→F (shift di 3). "CIAO" → "FLDR" |
| **Trasposizione** | Le lettere vengono mescolate (stesse lettere, ordine diverso) | **Scitala spartana**: scrivi su un cilindro, srotoli → lettere in disordine. "ATTACCO" → "ATCTAKO" |

**Limiti della crittografia classica**: vulnerabile all'analisi delle frequenze (in italiano la 'E' è la lettera più frequente).

### Principio di Kerckhoffs

> La sicurezza di un sistema crittografico deve dipendere **solo dalla segretezza della chiave**, non dalla segretezza dell'algoritmo.

L'algoritmo può (e deve) essere pubblico. La chiave è l'unico segreto.

---

## 3. CRITTOGRAFIA SIMMETRICA

**Stessa chiave** per cifrare e decifrare.

```
Mittente:  plaintext ──[Chiave K]──→ ciphertext
Ricevente: ciphertext ──[Chiave K]──→ plaintext
```

**Problema**: come scambiarsi la chiave in modo sicuro? Se l'attaccante intercetta la chiave, può decifrare tutto.

### DES (Data Encryption Standard)

- Sviluppato da IBM, adottato come standard USA nel 1977
- **Chiave**: 56 bit (troppo corta per gli standard moderni)
- **Blocco**: 64 bit
- **Struttura**: rete di Feistel con 16 round
- **Rotazione** (oggi): facilmente forzabile con brute force in ore/giorni

### Triple DES (3DES)

Applica DES **tre volte** con 3 chiavi diverse (o 2 chiavi).

```
ciphertext = Encrypt(K3, Decrypt(K2, Encrypt(K1, plaintext)))
```

- Chiave effettiva: 168 bit (3 × 56) o 112 bit (2 chiavi)
- **Più sicuro** di DES ma **lento** (3 operazioni anziché 1)
- Oggi sostituito da AES

### AES (Advanced Encryption Standard)

- Standard dal 2001 (vincitore del concorso NIST: algoritmo Rijndael)
- Chiavi: **128, 192, 256 bit**
- Blocco: 128 bit
- Struttura: rete a sostituzione-permutazione (non Feistel)
- **Usato ovunque**: Wi-Fi (WPA2/WPA3), HTTPS, SSH, VPN, BitLocker, file ZIP

**Velocità**: AES in hardware (CPU moderne hanno istruzioni AES-NI) è estremamente veloce.

---

## 4. CRITTOGRAFIA ASIMMETRICA

**Due chiavi distinte**: una **pubblica** (condivisibile con tutti) e una **privata** (segreta).

```
Mittente:  plaintext ──[Chiave PUBBLICA del ricevente]──→ ciphertext
Ricevente: ciphertext ──[Chiave PRIVATA del ricevente]──→ plaintext
```

**Vantaggio**: non serve scambiarsi una chiave segreta! La chiave pubblica può essere distribuita apertamente. Solo chi ha la privata può decifrare.

### RSA (Rivest, Shamir, Adleman, 1977)

Basato sulla difficoltà di **fattorizzare il prodotto di due grandi numeri primi**.

**Generazione chiavi**:
1. Scegli due numeri primi molto grandi: p, q
2. Calcola n = p × q (il modulo, pubblico)
3. Calcola φ(n) = (p-1)(q-1)
4. Scegli e (esponente pubblico) coprimo con φ(n) — spesso 65537
5. Calcola d (esponente privato) tale che e × d ≡ 1 (mod φ(n))

**Chiave pubblica**: (n, e) — può essere condivisa con tutti
**Chiave privata**: (n, d) — deve rimanere segreta

**Cifratura**: c = m^e mod n
**Decifratura**: m = c^d mod n

**Sicurezza di RSA**: si basa sul fatto che è computazionalmente molto difficile fattorizzare n nei suoi fattori primi p e q quando sono abbastanza grandi (2048 o 4096 bit).

**Lunghezze tipiche chiavi RSA oggi**: 2048 bit (minimo), 4096 bit (raccomandato).

### Confronto simmetrica vs asimmetrica

| | Simmetrica (AES, DES) | Asimmetrica (RSA, ECC) |
|---|---|---|
| **Chiavi** | Una sola, condivisa | Due (pubblica + privata) |
| **Velocità** | Molto veloce | Lenta (100-1000× più lenta) |
| **Uso tipico** | Cifrare grandi quantità di dati | Scambiare chiavi simmetriche, firma digitale |
| **Lunghezza chiave** | 128-256 bit | 2048-4096 bit |
| **Problema** | Come scambiarsi la chiave? | Come garantire che la chiave pubblica sia autentica? |

**Nella pratica**: si usa la crittografia **ibrida**. Si genera una chiave simmetrica temporanea (chiave di sessione), la si scambia con crittografia asimmetrica, poi si usa la simmetrica per i dati. Esempio: **HTTPS** (RSA o ECDHE per lo scambio chiavi → AES per i dati).

---

## 5. FIRMA DIGITALE

Garantisce **autenticità** (chi ha firmato) e **integrità** (il documento non è stato modificato).

### Come funziona

```
Mittente:
1. Calcola l'HASH del documento (es. SHA-256)
2. Cifra l'hash con la propria CHIAVE PRIVATA → questa è la FIRMA
3. Invia documento + firma

Ricevente:
1. Calcola l'HASH del documento ricevuto
2. Decifra la firma con la CHIAVE PUBBLICA del mittente → ottiene l'hash originale
3. Se i due hash coincidono → il documento è autentico e integro
```

**Proprietà della firma digitale**:
- **Autenticità**: solo il possessore della chiave privata poteva firmare
- **Integrità**: se il documento viene modificato, l'hash non corrisponde più
- **Non ripudio**: il firmatario non può negare di aver firmato

### PEC (Posta Elettronica Certificata)

Sistema italiano che certifica l'invio e la ricezione delle email, equiparandole a una raccomandata con ricevuta di ritorno.

**Come funziona**:
1. Il mittente invia una mail PEC al suo gestore
2. Il gestore PEC del mittente la inoltra al gestore PEC del destinatario
3. Il mittente riceve una **ricevuta di accettazione**
4. Il destinatario riceve la mail nella sua casella PEC
5. Il mittente riceve una **ricevuta di consegna** (prova che è arrivata)

**Differenze con la mail normale**:
- Le ricevute hanno **valore legale** (opponibili a terzi)
- Il gestore certifica data e ora esatte di invio e consegna
- Il contenuto non è cifrato di default (la PEC certifica il trasporto, non la segretezza)

### Funzione Hash

Trasforma un input di qualsiasi lunghezza in un output di lunghezza fissa (digest).

```
"Hello World" ──→ SHA-256 ──→ a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e
```

**Proprietà**:
- **Deterministica**: stesso input → sempre stesso hash
- **Unidirezionale**: dall'hash non puoi risalire all'input
- **Resistente alle collisioni**: è computazionalmente impossibile trovare due input diversi con lo stesso hash
- **Effetto valanga**: un piccolo cambio nell'input produce un hash completamente diverso

**Algoritmi hash**:
| Algoritmo | Bit output | Stato |
|---|---|---|
| MD5 | 128 | ❌ Rotto (collisioni trovate) |
| SHA-1 | 160 | ❌ Rotto, non più usato |
| SHA-2 (SHA-256, SHA-512) | 256, 512 | ✅ Sicuro, standard attuale |
| SHA-3 | Variabile | ✅ Alternativa recente |

---

## 6. SICUREZZA A LIVELLO TRASPORTO: SSL/TLS

**TLS** (Transport Layer Security, evoluzione di SSL) protegge le comunicazioni su Internet. È la "S" di HTTPS.

### Cosa garantisce TLS

1. **Riservatezza**: i dati sono cifrati (con AES)
2. **Integrità**: nessuno può modificare i dati in transito (MAC)
3. **Autenticazione**: il server (e opzionalmente il client) è chi dice di essere (certificati)

### Handshake TLS (semplificato)

```
Client                                    Server
  │─── ClientHello (versioni TLS, cipher) ──→│
  │←── ServerHello + Certificato ────────────│  (il server manda il suo certificato)
  │──── Scambio chiavi (RSA/ECDHE) ────────→│  (client e server generano una chiave di sessione)
  │←── Finished (cifrato) ──────────────────│
  │──── Finished (cifrato) ────────────────→│
  │                                          │
  │←── Dati cifrati con AES (sessione) ────→│
```

Il certificato del server contiene la sua **chiave pubblica** ed è firmato da una **Certificate Authority (CA)** fidata.

---

## 7. FIREWALL

Dispositivo (hardware o software) che filtra il traffico di rete in base a regole.

| Tipo di firewall | Cosa controlla | Esempio |
|---|---|---|
| **Packet filter** | IP sorgente/destinazione, porta, protocollo | ACL su router |
| **Stateful** | Tiene traccia delle connessioni attive. Blocca pacchetti fuori contesto. | iptables, Windows Firewall |
| **Application (proxy)** | Analizza il contenuto a livello applicativo (HTTP, FTP) | Proxy firewall, WAF (Web Application Firewall) |
| **NGFW** (Next-Gen) | Stateful + IPS + filtraggio applicazioni + intelligence | Palo Alto, Fortinet |

**Regola d'oro del firewall**: "quello che non è esplicitamente permesso è negato" (default deny).

---

## 8. BLOCKCHAIN

### Cos'è

Un **registro digitale decentralizzato e immutabile** dove le transazioni sono raggruppate in blocchi, collegati tra loro da funzioni hash crittografiche.

```
Blocco N-1              Blocco N                Blocco N+1
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│ Hash blocco  │←──────│ Hash blocco  │←──────│ Hash blocco  │
│   precedente │       │   precedente │       │   precedente │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ Timestamp    │       │ Timestamp    │       │ Timestamp    │
│ Dati (tx)    │       │ Dati (tx)    │       │ Dati (tx)    │
│ Nonce        │       │ Nonce        │       │ Nonce        │
│ Hash proprio │       │ Hash proprio │       │ Hash proprio │
└──────────────┘       └──────────────┘       └──────────────┘
```

**Perché è immutabile**: ogni blocco contiene l'hash del blocco precedente. Se modifichi un blocco, il suo hash cambia, rompendo il collegamento col blocco successivo. Per falsificare devi rifare TUTTI i blocchi successivi.

### Mining e Consenso

Il **mining** è il processo di validazione di un nuovo blocco. I miner competono per risolvere un problema crittografico (trovare un nonce tale che l'hash del blocco inizi con un certo numero di zeri).

**Proof of Work (PoW)**: il miner deve dimostrare di aver speso potenza di calcolo. Usato da Bitcoin. Pro: sicurissimo. Contro: consuma tantissima energia.

**Proof of Stake (PoS)**: il validatore viene scelto in base a quante monete possiede e "mette in gioco" (stake). Usato da Ethereum 2.0. Pro: efficiente energeticamente. Contro: favorisce chi ha più monete.

### Funzione hash nella blockchain

La blockchain usa funzioni hash crittografiche (SHA-256 per Bitcoin) per:
1. Collegare i blocchi (hash del blocco precedente)
2. Identificare univocamente ogni blocco
3. Il puzzle del mining (trovare un hash che inizia con N zeri)
4. Verificare l'integrità dei dati (Merkle tree)

### Applicazioni oltre le criptovalute

- **Smart contract** (Ethereum): accordi auto-eseguibili
- **Supply chain**: tracciamento prodotti dalla fabbrica al negozio
- **NFT**: certificati di proprietà digitale
- **Identità digitale**: documenti verificabili senza autorità centrale
- **Sanità**: cartelle cliniche condivise e immutabili
- **Voto elettronico**: trasparente e inalterabile

---

## DOMANDE PER L'AUTOVERIFICA

1. Spiega i 3 pilastri della sicurezza informatica (CIA triad).
2. Differenza tra crittografia simmetrica e asimmetrica.
3. Cos'è DES? Perché non si usa più? Cos'è il Triple DES?
4. Come funziona RSA (a parole, non serve la matematica). Su cosa si basa la sua sicurezza?
5. Nella pratica, perché si usa la crittografia ibrida (asimmetrica + simmetrica)?
6. Cos'è la firma digitale? Quali proprietà garantisce?
7. Cos'è la PEC? In cosa si differenzia da una mail normale?
8. Cos'è una funzione hash? Proprietà fondamentali. Esempi: SHA-256 vs MD5.
9. SSL/TLS: cosa garantisce? Quali sono le fasi dell'handshake?
10. Tipi di firewall: packet filter vs stateful vs application.
11. Cos'è una blockchain? Perché si dice che è immutabile?
12. Differenza tra Proof of Work e Proof of Stake.
13. Quali applicazioni ha la blockchain oltre alle criptovalute? (almeno 3)
