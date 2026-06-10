# GIORNO 2 — VLAN, ROUTING, DHCP, DNS, CLOUD, VIRTUALIZZAZIONE

---

## 1. VLAN — VIRTUAL LOCAL AREA NETWORK

### Cos'è una VLAN

Una VLAN è una **rete locale virtuale** che permette di suddividere uno switch fisico in più reti logiche isolate. Dispositivi su VLAN diverse non comunicano tra loro senza un router.

```
┌──────────────────────────────────────────┐
│              SWITCH FISICO                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │ VLAN 10 │  │ VLAN 20 │  │ VLAN 30 │  │
│  │ Uffici  │  │ Server  │  │ Ospiti  │  │
│  └─────────┘  └─────────┘  └─────────┘  │
└──────────────────────────────────────────┘
```

### Perché usare le VLAN

1. **Sicurezza**: isola il traffico sensibile (es. reparto IT separato dagli ospiti)
2. **Riduzione broadcast**: ogni VLAN è un dominio di broadcast separato
3. **Flessibilità**: sposti un PC fisicamente ma resta nella sua VLAN logica
4. **Performance**: meno traffico broadcast per ogni segmento

### Come funziona: 802.1Q Tagging

Il **tagging 802.1Q** aggiunge 4 byte all'intestazione del frame Ethernet, contenenti l'ID della VLAN (VID, 12 bit → max 4094 VLAN).

```
Frame Ethernet senza tag:
┌──────────────┬───────────┬──────┬──────┐
│ MAC Dest     │ MAC Src   │ Tipo │ Dati │
└──────────────┴───────────┴──────┴──────┘

Frame Ethernet con tag 802.1Q:
┌──────────────┬───────────┬──────────────┬──────┬──────┐
│ MAC Dest     │ MAC Src   │ 802.1Q (VID) │ Tipo │ Dati │
└──────────────┴───────────┴──────────────┴──────┴──────┘
```

### Tipi di porte VLAN

| Tipo porta | Descrizione |
|---|---|
| **Access** | Appartiene a UNA sola VLAN. Connette dispositivi finali (PC, stampante). Il frame esce senza tag. |
| **Trunk** | Trasporta PIÙ VLAN sullo stesso cavo (switch-switch o switch-router). Il frame ha il tag 802.1Q. |

### VLAN nativa (Native VLAN)

Sul trunk, la VLAN nativa è quella i cui frame **non vengono taggati**. Di default è la VLAN 1. Per sicurezza, cambiala sempre (es. VLAN 999).

### Router-on-a-Stick

Per far comunicare VLAN diverse serve un router. Con "router-on-a-stick" si usa una sola interfaccia fisica del router, configurata come trunk, e si creano sottointerfacce logiche, una per VLAN.

```
Router
  │
  │ (trunk 802.1Q)
  │
Switch
  ├── VLAN 10 (192.168.10.0/24)
  ├── VLAN 20 (192.168.20.0/24)
  └── VLAN 30 (192.168.30.0/24)
```

Configurazione concettuale sul router:
- Interfaccia `g0/0.10` → VLAN 10, gateway 192.168.10.1
- Interfaccia `g0/0.20` → VLAN 20, gateway 192.168.20.1
- Interfaccia `g0/0.30` → VLAN 30, gateway 192.168.30.1

### Esempio pratico: VLAN per la seconda prova

**Scenario**: azienda con 1 sede.

| VLAN ID | Nome | Rete | Gateway | Scopo |
|---|---|---|---|---|
| 10 | Uffici | 192.168.10.0/24 | .1 | Dipendenti |
| 20 | Server | 192.168.20.0/26 | .1 | Server interni |
| 30 | DMZ | 192.168.30.0/28 | .1 | Servizi pubblici |
| 40 | Management | 192.168.40.0/28 | .1 | Gestione apparati |
| 99 | Ospiti | 192.168.99.0/24 | .1 | Wi-Fi pubblico |
| 999 | Native | – | – | VLAN nativa (modificata) |

---

## 2. ROUTING

### Cos'è il routing

Il routing è il processo di **inoltrare pacchetti** da una rete a un'altra, scegliendo il percorso migliore.

### Router: come funziona

Un router ha più interfacce, ognuna su una rete diversa. Quando riceve un pacchetto:
1. Guarda l'IP di destinazione
2. Consulta la **tabella di routing**
3. Inoltra il pacchetto sull'interfaccia corretta
4. Se non trova una rotta → scarta il pacchetto (a meno che non ci sia una **default route**)

### Tabella di routing

Contiene entry con:
- **Rete di destinazione** (es. 192.168.20.0/24)
- **Next hop** (a chi passo il pacchetto)
- **Interfaccia di uscita** (da quale porta esce)
- **Metrica** (quanto "costa" quel percorso)

### Routing statico vs dinamico

| | Statico | Dinamico |
|---|---|---|
| **Configurazione** | Manuale, rotta per rotta | Automatica, i router si scambiano informazioni |
| **Adatto per** | Reti piccole, stabili | Reti grandi, che cambiano |
| **Protocolli** | – | RIP, OSPF, EIGRP, BGP |
| **Vantaggi** | Semplice, sicuro, nessun overhead | Adattivo, scala bene |
| **Svantaggi** | Non scala, non reagisce ai guasti | Overhead di rete, più complesso |

### Default route

Rotta "di default": tutto il traffico verso destinazioni sconosciute viene inoltrato qui.

```
0.0.0.0/0 via 192.168.1.254
```

Tipicamente punta al **gateway verso Internet** (il firewall o il router dell'ISP).

### Routing tra VLAN

Per far parlare VLAN diverse, il router (o uno switch Layer 3) deve avere una rotta (diretta o statica) verso ogni rete VLAN. Esempio:

```
Rete 192.168.10.0/24 (VLAN 10) ─┐
                                 ├── Router ── Internet
Rete 192.168.20.0/24 (VLAN 20) ─┘
```

Il router conosce entrambe le reti perché ha un'interfaccia su ciascuna (o sottointerfacce su trunk).

---

## 3. DHCP — DYNAMIC HOST CONFIGURATION PROTOCOL

### A cosa serve

DHCP assegna **automaticamente** indirizzi IP, subnet mask, gateway, DNS e altri parametri ai dispositivi di rete. Senza DHCP, dovresti configurare ogni PC a mano.

### Come funziona (scambio DORA)

```
Client                              Server
  │                                    │
  │────── DHCP DISCOVER ──────────────→│  (broadcast: "c'è un server DHCP?")
  │                                    │
  │←───── DHCP OFFER ──────────────────│  (il server offre un IP)
  │                                    │
  │────── DHCP REQUEST ───────────────→│  (il client accetta l'offerta)
  │                                    │
  │←───── DHCP ACK ────────────────────│  (il server conferma l'assegnazione)
  │                                    │
```

**DORA**: **D**iscover → **O**ffer → **R**equest → **A**cknowledge

### Parametri assegnati da DHCP

| Parametro | Descrizione | Esempio |
|---|---|---|
| IP Address | Indirizzo del client | 192.168.10.50 |
| Subnet Mask | Maschera di sottorete | 255.255.255.0 |
| Default Gateway | Router per uscire dalla rete | 192.168.10.1 |
| DNS Server | Server per risoluzione nomi | 8.8.8.8, 192.168.10.5 |
| Lease Time | Durata del "prestito" dell'IP | 86400 secondi (24h) |
| Domain Name | Dominio di appartenenza | azienda.local |

### DHCP Relay

In reti con più VLAN, il server DHCP è su una VLAN e i client su altre. Il **DHCP Relay** (configurato sul router o switch L3) inoltra le richieste DHCP dai client al server.

```
Client (VLAN 10) ──→ Router (DHCP Relay) ──→ Server DHCP (VLAN 20)
```

Senza relay, il broadcast DHCP non uscirebbe dalla VLAN 10 e non raggiungerebbe mai il server.

### DHCP Snooping (sicurezza)

Previene attacchi di **DHCP spoofing** (qualcuno mette un server DHCP rogue in rete). Lo switch permette risposte DHCP solo dalle porte "trusted" (dove c'è il vero server).

### Esempio per la seconda prova

Un'azienda con 4 VLAN configurerà tipicamente:
- **1 server DHCP** nella VLAN Server
- **DHCP Relay** su router/switch L3 per le altre VLAN
- **Scope diversi** per ogni VLAN (range IP diversi)
- **DHCP Snooping** abilitato per sicurezza

---

## 4. DNS — DOMAIN NAME SYSTEM

### A cosa serve

Traduce **nomi di dominio** (www.google.com) in **indirizzi IP** (142.250.184.14).

### Gerarchia DNS

```
                              . (root)
              ┌────────────────┼────────────────┐
             .com             .it              .org
          ┌────┴────┐     ┌────┴────┐
      google.com  amazon  governo.it  unimi.it
```

### Tipi di server DNS

| Tipo | Ruolo |
|---|---|
| **Root server** | 13 cluster globali, punto di partenza di ogni risoluzione |
| **TLD server** | Gestiscono i domini di primo livello (.com, .it, .org) |
| **Authoritative** | Ha i record definitivi per un dominio specifico |
| **Resolver (ricorsivo)** | Fa il lavoro sporco per conto del client, interrogando la gerarchia |

### Record DNS principali

| Record | Significato | Esempio |
|---|---|---|
| **A** | Associa nome a IPv4 | `www → 192.168.1.10` |
| **AAAA** | Associa nome a IPv6 | `www → 2001:db8::1` |
| **CNAME** | Alias (nome alternativo) | `mail → www.azienda.it` |
| **MX** | Mail server del dominio | `azienda.it → mail.azienda.it` (priorità 10) |
| **NS** | Name server autoritativo | `azienda.it → ns1.azienda.it` |
| **PTR** | Risoluzione inversa (IP → nome) | `1.168.192.in-addr.arpa → www` |
| **TXT** | Testo libero (es. SPF per email) | `v=spf1 mx -all` |

### Risoluzione DNS passo passo

1. Il client chiede al resolver: "qual è l'IP di www.azienda.it?"
2. Il resolver chiede a un root server: "chi gestisce .it?"
3. Il root risponde con il TLD server di .it
4. Il resolver chiede al TLD: "chi gestisce azienda.it?"
5. Il TLD risponde con l'authoritative server di azienda.it
6. Il resolver chiede all'authoritative: "qual è l'IP di www?"
7. L'authoritative risponde con il record A
8. Il resolver restituisce l'IP al client

### DNS nella seconda prova

Nella traccia devi dire:
1. Dove metti i server DNS (primario nella Server Farm, secondario in sede distaccata per ridondanza)
2. Quali record configuri (A, MX, CNAME, NS)
3. DNS interno vs esterno: i client interni usano il DNS aziendale, il DNS pubblico gestisce il dominio verso Internet
4. Forwarding: il DNS interno inoltra le richieste esterne a un DNS pubblico (es. 8.8.8.8)

---

## 5. CLOUD COMPUTING

### Modelli di servizio

| Modello | Cosa fornisce | Esempi | Tu gestisci |
|---|---|---|---|
| **IaaS** | Infrastruttura (VM, storage, rete) | AWS EC2, Azure VM | OS, middleware, app, dati |
| **PaaS** | Piattaforma di sviluppo | Google App Engine, Heroku | App e dati |
| **SaaS** | Applicazione pronta all'uso | Office 365, Gmail, Salesforce | Niente (solo dati utente) |

```
On-Premise      IaaS            PaaS            SaaS
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ App     │    │ App     │    │ App     │    │ App     │
│ Dati    │    │ Dati    │    │ Dati    │    │ Dati    │
│ Runtime │    │ Runtime │    │ Runtime │    │ Runtime │
│ Middlew.│    │ Middlew.│    │ Middlew.│    │ Middlew.│
│ OS      │    │ OS      │    │ OS      │    │ OS      │
│ Virt.   │    │ Virt.   │    │ Virt.   │    │ Virt.   │
│ Server  │    │ Server  │    │ Server  │    │ Server  │
│ Storage │    │ Storage │    │ Storage │    │ Storage │
│ Rete    │    │ Rete    │    │ Rete    │    │ Rete    │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
  Tu gestisci    Tu: OS+      Tu: App+Dati    Provider
  TUTTO          Provider:    Provider:       gestisce
                 infra        piattaforma     TUTTO
```

### Modelli di deployment

| Modello | Descrizione |
|---|---|
| **Public Cloud** | Infrastruttura condivisa, accessibile via Internet (AWS, Azure, GCP) |
| **Private Cloud** | Infrastruttura dedicata a una sola organizzazione (on-premise o hosted) |
| **Hybrid Cloud** | Mix di public e private, con orchestrazione tra i due |
| **Community Cloud** | Condivisa tra più organizzazioni con interessi comuni |

### Edge Computing vs Cloud Computing

| | Cloud Computing | Edge Computing |
|---|---|---|
| **Dove elabora** | Data center centralizzato | Vicino alla fonte dei dati |
| **Latenza** | Alta (decine/centinaia ms) | Bassissima (< 10 ms) |
| **Banda** | Alta richiesta verso il cloud | Bassa (elabora localmente) |
| **Casi d'uso** | Web app, storage, ML training | IoT, veicoli autonomi, fabbrica 4.0 |
| **Esempi** | AWS, Azure | Gateway IoT, edge server locali |

---

## 6. VIRTUALIZZAZIONE E CONTAINER

### Virtualizzazione — Hypervisor

Un **hypervisor** è un software che crea ed esegue macchine virtuali (VM), ognuna con il proprio OS.

| Tipo | Descrizione | Esempi |
|---|---|---|
| **Type 1 (bare metal)** | Installato direttamente sull'hardware | VMware ESXi, Hyper-V, KVM, Proxmox |
| **Type 2 (hosted)** | Installato su un OS esistente | VirtualBox, VMware Workstation |

**Vantaggi**: consolidamento server, isolamento, snapshot, migrazione a caldo, disaster recovery.

### Container (Docker)

I **container** sono ambienti isolati che condividono il kernel del sistema operativo host. A differenza delle VM, non hanno bisogno di un OS guest completo.

| | VM | Container |
|---|---|---|
| **OS** | Ogni VM ha il suo OS guest | Condividono il kernel host |
| **Avvio** | Secondi/minuti | Millisecondi |
| **Peso** | GB | MB |
| **Isolamento** | Completo | A livello di processo |
| **Orchestrazione** | vCenter, Proxmox | Kubernetes, Docker Swarm |

**Quando usare cosa**:
- **VM**: quando serve isolamento forte, OS diversi, carichi di lavoro legacy
- **Container**: microservizi, CI/CD, scalabilità rapida, sviluppo

---

## DOMANDE PER L'AUTOVERIFICA

1. Cos'è una VLAN e perché si usa? Spiega la differenza tra porta access e trunk.
2. Cos'è il tagging 802.1Q? Cosa contiene il tag?
3. Come si fa a far comunicare due VLAN diverse? Spiega il router-on-a-stick.
4. Qual è la differenza tra routing statico e dinamico? Quando usi l'uno o l'altro?
5. Spiega lo scambio DORA del DHCP.
6. Cos'è il DHCP Relay e a cosa serve in una rete con VLAN?
7. Quali sono i principali record DNS? Cosa fa un record MX e un record A?
8. Descrivi il processo di risoluzione DNS passo passo.
9. Differenza tra IaaS, PaaS e SaaS. Fai un esempio per ciascuno.
10. Differenza tra virtualizzazione (VM) e container. Quando usi una VM e quando un container?
11. Cos'è l'Edge Computing? In cosa si differenzia dal Cloud?
12. Come configureresti DHCP e DNS in una rete aziendale con 3 VLAN?
