# GIORNO 1 — SUBNETTING, VLSM, INDIRIZZAMENTO, CABLAGGIO

> ⚠️ Giorno più tosto. Qui costruisci le fondamenta per tutta la seconda prova.
> Se non sai creare un piano di indirizzamento, la traccia non parte.

---

## 1. INDIRIZZI IPv4 — RIPASSO LAMPO

### Struttura di un indirizzo IPv4

Un indirizzo IPv4 è formato da **32 bit**, scritti come 4 numeri decimali separati da punti (notazione dotted decimal).

```
192 . 168 . 1 . 10
11000000.10101000.00000001.00001010  (binario)
```

### Classi di indirizzi (storico, ma serve capirle)

| Classe | Primo byte (binario) | Range decimale | Subnet mask default | CIDR |
|---|---|---|---|---|
| **A** | 0xxxxxxx | 0.0.0.0 – 127.255.255.255 | 255.0.0.0 | /8 |
| **B** | 10xxxxxx | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 | /16 |
| **C** | 110xxxxx | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 | /24 |
| **D** | 1110xxxx | 224.0.0.0 – 239.255.255.255 | Multicast | – |
| **E** | 1111xxxx | 240.0.0.0 – 255.255.255.255 | Sperimentale | – |

### Indirizzi privati (RFC 1918)

Questi **non sono instradabili su Internet**, vanno usati nelle reti locali + NAT:

| Classe | Range | CIDR |
|---|---|---|
| A | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| B | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| C | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

**Per la seconda prova**: usa sempre indirizzi privati per le reti interne. L'indirizzo pubblico lo assegna l'ISP.

---

## 2. SUBNET MASK E NOTAZIONE CIDR

### Cos'è la subnet mask

La subnet mask separa la **parte rete** dalla **parte host** di un indirizzo IP.

```
IP:      192.168.1.10
Mask:    255.255.255.0
         └───┬───┘ └┬┘
         rete     host
```

In binario: la subnet mask ha `1` dove c'è la parte rete, `0` dove c'è la parte host.

```
255.255.255.0 = 11111111.11111111.11111111.00000000
                 └────── 24 bit di rete ──────┘└─ 8 bit host ─┘
```

### Notazione CIDR (Classless Inter-Domain Routing)

Invece di scrivere `192.168.1.0 255.255.255.0`, si scrive `192.168.1.0/24`.

Il numero dopo lo `/` è il **prefisso**: quanti bit sono riservati alla rete.

| CIDR | Subnet Mask | Host disponibili |
|---|---|---|
| /8 | 255.0.0.0 | 16.777.214 |
| /16 | 255.255.0.0 | 65.534 |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /29 | 255.255.255.248 | 6 |
| /30 | 255.255.255.252 | 2 |

### Come calcolare gli host disponibili

```
Host = 2^(32 - prefisso) - 2
```

Il `-2` è perché il primo indirizzo è l'indirizzo di **rete** e l'ultimo è l'indirizzo di **broadcast**.

```
Esempio: /26 → 2^(32-26) - 2 = 2^6 - 2 = 64 - 2 = 62 host
```

### Indirizzo di rete e broadcast

Data una rete `192.168.1.0/26`:
- **Indirizzo di rete**: 192.168.1.0 (tutti i bit host a 0)
- **Primo host**: 192.168.1.1
- **Ultimo host**: 192.168.1.62
- **Indirizzo di broadcast**: 192.168.1.63 (tutti i bit host a 1)

---

## 3. SUBNETTING — CREARE SOTTORETI

### Il concetto

Subnetting significa "prendere in prestito" bit dalla parte host per creare più reti più piccole.

**Regola base**: se prendo `n` bit, creo `2^n` sottoreti.

### Esercizio GUIDATO: Subnetting di una /24

**Scenario**: hai la rete `192.168.1.0/24`. Devi creare 4 sottoreti.

**Step 1**: Quanti bit devo prendere in prestito?
- `2^n >= 4` → `n = 2` bit
- Nuovo prefisso: `24 + 2 = /26`

**Step 2**: Qual è la nuova subnet mask?
- `/26` = `255.255.255.192` (128+64)

**Step 3**: Quanti host per sottorete?
- `2^(32-26) - 2 = 64 - 2 = 62 host`

**Step 4**: Quali sono gli indirizzi delle 4 sottoreti?

| Sottorete | Indirizzo di rete | Range host | Broadcast |
|---|---|---|---|
| 1 | 192.168.1.0/26 | .1 – .62 | .63 |
| 2 | 192.168.1.64/26 | .65 – .126 | .127 |
| 3 | 192.168.1.128/26 | .129 – .190 | .191 |
| 4 | 192.168.1.192/26 | .193 – .254 | .255 |

### Esercizio GUIDATO: Subnetting in base agli host

**Scenario**: hai `10.0.0.0/16`. Devi creare sottoreti per:
- Reparto A: 1000 host
- Reparto B: 500 host
- Reparto C: 200 host
- Reparto D: 50 host

**ATTENZIONE**: qui serve **VLSM** (Variable Length Subnet Mask), non subnetting a maschera fissa! Le sottoreti avranno maschere DIVERSE.

---

## 4. VLSM — Variable Length Subnet Mask

VLSM permette di assegnare a ogni sottorete **esattamente** il numero di host che servono, senza sprechi. Si ordinano le richieste **dalla più grande alla più piccola**.

### Esercizio GUIDATO VLSM

**Rete di partenza**: `10.0.0.0/16`

**Step 1**: Ordina per host decrescenti

| Reparto | Host richiesti |
|---|---|
| A | 1000 |
| B | 500 |
| C | 200 |
| D | 50 |

**Step 2**: Per ogni reparto, trova il prefisso minimo

| Reparto | Host | 2^n - 2 >= host | n bit host | Prefisso |
|---|---|---|---|---|
| A | 1000 | 2^10 - 2 = 1022 ≥ 1000 ✅ | 10 | /22 |
| B | 500 | 2^9 - 2 = 510 ≥ 500 ✅ | 9 | /23 |
| C | 200 | 2^8 - 2 = 254 ≥ 200 ✅ | 8 | /24 |
| D | 50 | 2^6 - 2 = 62 ≥ 50 ✅ | 6 | /26 |

**Step 3**: Assegna le sottoreti (partendo dalla più grande)

| Reparto | Rete | Subnet Mask | Host disp. | Broadcast |
|---|---|---|---|---|
| A | 10.0.0.0/22 | 255.255.252.0 | 1022 | 10.0.3.255 |
| B | 10.0.4.0/23 | 255.255.254.0 | 510 | 10.0.5.255 |
| C | 10.0.6.0/24 | 255.255.255.0 | 254 | 10.0.6.255 |
| D | 10.0.7.0/26 | 255.255.255.192 | 62 | 10.0.7.63 |

> **Trucco per l'esame**: VLSM si fa sempre ordinando dal reparto più grande al più piccolo. Ogni nuova sottorete inizia subito dopo il broadcast della precedente.

---

## 5. ESERCIZI PRATICI — COSTRUISCI RETI DA ZERO

### Esercizio 1 — Piccola azienda (entry level)

**Scenario**: azienda con 1 sede, 3 reparti:
- Amministrazione: 25 PC
- Produzione: 50 PC  
- Ospiti Wi-Fi: 30 dispositivi

Hai la rete `192.168.10.0/24`. Applica VLSM.

<details>
<summary>🔍 Soluzione</summary>

Ordine: Produzione (50) → Ospiti (30) → Amministrazione (25)

| Reparto | Host | n bit | Prefisso | Maschera | Rete | Broadcast |
|---|---|---|---|---|---|---|
| Produzione | 50 | 6 | /26 | .192 | 192.168.10.0/26 | .63 |
| Ospiti | 30 | 5 | /27 | .224 | 192.168.10.64/27 | .95 |
| Amministr. | 25 | 5 | /27 | .224 | 192.168.10.96/27 | .127 |

</details>

### Esercizio 2 — Tre sedi (intermedio)

**Scenario**: azienda con 3 sedi (Milano, Roma, Napoli), collegate in VPN. Hai la rete `172.16.0.0/16`.

**Sede Milano (HQ)**:
- Uffici: 200 host
- Server farm: 50 host  
- DMZ: 10 host

**Sede Roma**:
- Uffici: 100 host

**Sede Napoli**:
- Uffici: 50 host

Collegamenti VPN: ogni link punto-punto ha bisogno di una /30 (2 host).

<details>
<summary>🔍 Soluzione</summary>

Ordine: Milano Uffici (200) → Roma (100) → Milano Server (50) → Napoli (50) → Milano DMZ (10) → Link VPN (2)

| Sede | Rete | CIDR | Maschera | Host |
|---|---|---|---|---|
| Milano Uffici | 172.16.0.0/24 | /24 | .0 | 254 |
| Roma Uffici | 172.16.1.0/25 | /25 | .128 | 126 |
| Milano Server | 172.16.1.128/26 | /26 | .192 | 62 |
| Napoli Uffici | 172.16.1.192/26 | /26 | .192 | 62 |
| Milano DMZ | 172.16.2.0/28 | /28 | .240 | 14 |
| VPN MI-RO | 172.16.2.16/30 | /30 | .252 | 2 |
| VPN MI-NA | 172.16.2.20/30 | /30 | .252 | 2 |

</details>

### Esercizio 3 — Scenario da seconda prova (avanzato)

**Scenario**: rete ospedaliera con 2 sedi.

**Sede Centrale**:
- Reparto Medici: 120 host (rete isolata, dati sensibili)
- Reparto Infermieri: 80 host
- Amministrazione: 45 host
- Server farm (cartelle cliniche, DB): 25 host
- DMZ: 8 host (portale pazienti, web server)
- Wi-Fi Ospiti: 200 dispositivi

**Sede Distaccata**:
- Medici: 40 host
- Ambulatori: 30 host
- Wi-Fi Ospiti: 60 dispositivi

Collegamento VPN Site-to-Site tra le due sedi (/30).

**Hai la rete `10.0.0.0/16`. Applica VLSM e compila la tabella completa.**

---

## 6. CABLAGGIO STRUTTURATO

### Cos'è

Il cablaggio strutturato è l'infrastruttura fisica passiva che trasporta i segnali in un edificio o campus. È standardizzato dalle norme **ANSI/TIA/EIA-568** e **ISO/IEC 11801**.

### Componenti principali

```
┌─────────────────────────────────────────────────┐
│            CABLAGGIO STRUTTURATO                 │
├─────────────────────────────────────────────────┤
│  1. Area di lavoro (WA)                         │
│     Presa RJ45 a muro → patch cord → dispositivo│
├─────────────────────────────────────────────────┤
│  2. Cablaggio orizzontale                       │
│     Cavo UTP/STP tra presa e patch panel        │
│     Max 90 metri (Cat5e, Cat6, Cat6a, Cat7)     │
├─────────────────────────────────────────────────┤
│  3. Armadio rack (TR - Telecom Room)            │
│     Patch panel, switch, router, server         │
├─────────────────────────────────────────────────┤
│  4. Cablaggio verticale (backbone)              │
│     Collega i rack di piani diversi             │
│     Fibra ottica o UTP (distanze > 90m)         │
├─────────────────────────────────────────────────┤
│  5. Ingresso di edificio (EF)                   │
│     Punto di accesso del provider (ISP)         │
│     Connessione rete esterna                    │
└─────────────────────────────────────────────────┘
```

### Categorie cavi Ethernet

| Categoria | Banda | Velocità max | Distanza | Note |
|---|---|---|---|---|
| **Cat5e** | 100 MHz | 1 Gbps | 100m | Minimo accettabile |
| **Cat6** | 250 MHz | 10 Gbps (fino a 55m) | 100m | Standard moderno |
| **Cat6a** | 500 MHz | 10 Gbps | 100m | Per data center |
| **Cat7** | 600 MHz | 10 Gbps | 100m | Schermato (S/FTP) |
| **Cat8** | 2000 MHz | 40 Gbps | 30m | Solo data center |

### Fibra ottica

- **Monomodale (SMF)**: nucleo stretto (9µm), lunghe distanze (km), più costosa. Laser.
- **Multimodale (MMF)**: nucleo largo (50/62.5µm), brevi distanze (500m-2km), più economica. LED.

### Differenza UTP vs STP

| | UTP (Unshielded) | STP/FTP (Shielded) |
|---|---|---|
| **Schermatura** | Nessuna | Lamina o treccia metallica |
| **Interferenze** | Più sensibile | Protetto da EMI |
| **Costo** | Basso | Medio-alto |
| **Uso** | Uffici standard | Ambienti industriali, medicali |

### Cosa scrivere nella seconda prova

Quando descrivi il cablaggio nella traccia, devi dire:
1. **Tipo di cavo** per ogni tratta (UTP Cat6 per orizzontale, fibra per backbone)
2. **Distanze massime** rispettate (90m orizzontale, 100m totale)
3. **Armadi rack** per piano, con switch e patch panel
4. **Collegamenti tra piani** (backbone in fibra)
5. **Ingresso ISP** e punto di demarcazione

---

## 7. RIEPILOGO: COSA DEVI SAPER FARE DOPO IL GIORNO 1

1. ✅ Dato un range IP, calcolare subnet mask, CIDR, host disponibili
2. ✅ Dato un numero di host, trovare il prefisso minimo necessario
3. ✅ Subnetting a maschera fissa (tutte le sottoreti uguali)
4. ✅ **VLSM**: assegnare sottoreti di dimensioni diverse, ordinando per host decrescenti
5. ✅ Compilare una tabella di indirizzamento completa (rete, maschera, gateway, broadcast, range)
6. ✅ Descrivere il cablaggio strutturato di un edificio
7. ✅ Scegliere la categoria di cavo e il tipo (rame vs fibra) in base allo scenario

---

## PAROLE CHIAVE

| Termine | Significato |
|---|---|
| **CIDR** | Notazione compatta: IP/prefisso (es. 192.168.1.0/24) |
| **VLSM** | Subnetting a maschera variabile, per non sprecare IP |
| **Subnet mask** | Separa parte rete da parte host |
| **Broadcast** | Ultimo IP della sottorete (tutti i bit host a 1) |
| **Gateway** | Primo (o ultimo) IP della sottorete, usato per uscire dalla rete |
| **NAT** | Traduzione indirizzi privati → pubblico per uscire su Internet |
| **DMZ** | Zona demilitarizzata: rete isolata per servizi esposti all'esterno |
| **UTP** | Cavo a doppino non schermato |
| **Backbone** | Dorsale di collegamento tra armadi/piani/edifici |

---

## DOMANDE PER L'AUTOVERIFICA

1. Quanti host può contenere una rete /27? E una /22?
2. Hai `10.0.0.0/8`. Devi creare sottoreti per 5000, 2000, 1000, 500, 100 host. Applica VLSM.
3. Qual è la differenza tra UTP Cat5e e Cat6a?
4. Quando usi la fibra monomodale invece del rame?
5. Cosa significa `/30` e quando si usa?
6. Perché nel cablaggio strutturato il cavo orizzontale non può superare i 90 metri?
7. Disegna la struttura di un cablaggio strutturato: area di lavoro → rack → backbone → EF.
8. Quali sono i range di indirizzi privati (RFC 1918)?
9. Se hai `192.168.0.0/23`, qual è la subnet mask in notazione decimale? Quanti host?
10. VLSM: perché si ordinano le sottoreti dalla più grande alla più piccola?
