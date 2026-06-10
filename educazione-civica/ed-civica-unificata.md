# ED. CIVICA UNIFICATA — Tutte le materie

> Un giorno dedicato (4-5 ore). Copre i moduli di Italiano, Sistemi e Reti, e Informatica.

---

## PARTE 1 — ITALIANO / SISTEMI E RETI: CONFINI REALI E VIRTUALI

### Confine reale

Il confine **fisico** di uno stato: territorio, frontiere, dogane. È definito da leggi, trattati, confini geografici. Dentro quel confine valgono le leggi di quello stato. La giurisdizione è chiara.

### Confine virtuale

Il confine nel **cyberspazio**: non è geografico. I dati attraversano server in paesi diversi in millisecondi. Un attacco informatico può partire dalla Russia e colpire un'azienda italiana senza mai varcare fisicamente una frontiera. La rete Internet non ha confini.

### Sfide del confine virtuale

| Problema | Descrizione |
|---|---|
| **Giurisdizione** | Se un server in USA contiene dati di cittadini EU, quale legge si applica? |
| **Sovranità digitale** | Chi controlla i dati? Dove risiedono fisicamente? |
| **Data localization** | Alcuni paesi impongono che i dati dei cittadini restino su server nazionali |
| **Attacchi cross-border** | L'attacco può venire da qualsiasi parte del mondo |
| **Cyberwarfare** | Stati usano attacchi informatici come strumento di conflitto |

### GDPR e confini

Il **GDPR** (Regolamento Europeo 2016/679, in vigore dal 2018) protegge i dati dei cittadini EU **ovunque vengano trattati**. Anche se un'azienda USA tratta dati di europei, deve rispettare il GDPR.

Principi chiave del GDPR:
- **Consenso esplicito**: non puoi raccogliere dati senza permesso chiaro
- **Diritto all'oblio**: l'utente può chiedere la cancellazione dei propri dati
- **Portabilità**: l'utente può esportare i propri dati
- **Data breach notification**: se i dati vengono violati, devi avvisare entro 72 ore
- **Privacy by design**: la privacy va progettata fin dall'inizio, non aggiunta dopo

### Collegamento con Sistemi e Reti e Informatica

- **Dove metti i server?** Se i dati sono in cloud, devi sapere in che paese risiedono fisicamente
- **VPN e crittografia**: strumenti per difendere i confini virtuali
- **DMZ e Firewall**: le "dogane" del mondo digitale
- **Database e GDPR**: devi implementare meccanismi per cancellare/esportare dati utente
- **Cookie e consenso**: i banner che vedi sui siti ("accetta tutti") sono richiesti dal GDPR

---

## PARTE 2 — INFORMATICA: INTELLIGENZA ARTIFICIALE

### Definizioni fondamentali

**Intelligenza Artificiale (IA)**: ramo dell'informatica che crea sistemi capaci di svolgere compiti che normalmente richiedono intelligenza umana (ragionamento, apprendimento, percezione, linguaggio).

### Tipi di IA

| Tipo | Descrizione | Esiste? |
|---|---|---|
| **IA Ristretta (ANI)** | Specializzata in un compito specifico (riconoscimento facciale, traduzione, scacchi) | ✅ Sì, è quella attuale |
| **IA Generale (AGI)** | Capace di svolgere QUALSIASI compito intellettuale umano | ❌ Non ancora |
| **Superintelligenza (ASI)** | Supera l'intelligenza umana in ogni campo | ❌ Solo teorica |

### Machine Learning e Deep Learning

```
INTELLIGENZA ARTIFICIALE
└── MACHINE LEARNING (apprendimento automatico)
    └── DEEP LEARNING (reti neurali profonde)
```

**Machine Learning**: il sistema **impara dai dati** invece di essere programmato esplicitamente. Invece di scrivere regole, dai esempi e il sistema trova pattern.

### Tipi di apprendimento automatico

| Tipo | Come funziona | Esempio |
|---|---|---|
| **Supervisionato** | Addestrato su dati etichettati (input → output corretto) | Classificare email come spam/non spam |
| **Non supervisionato** | Trova pattern in dati NON etichettati | Segmentare clienti in gruppi (clustering) |
| **Per rinforzo** | L'agente impara per tentativi, con premi/penalità | AlphaGo, robot che imparano a camminare |
| **Online** | Impara continuamente, un dato alla volta | Raccomandazioni in tempo reale |

### Reti neurali

Struttura ispirata al cervello umano: **nodi (neuroni)** organizzati in **livelli (layer)**, collegati da **pesi** che si modificano durante l'addestramento.

```
Input → [Layer 1] → [Layer 2] → ... → [Layer N] → Output
```

- **Input layer**: riceve i dati grezzi (es. pixel di un'immagine)
- **Hidden layers**: elaborano i dati, estraendo caratteristiche sempre più astratte
- **Output layer**: produce il risultato (es. "gatto" vs "cane")

**Deep Learning** = reti neurali con **molti** hidden layers. Più layer = più capacità di astrazione.

### IA Generativa

Sistemi capaci di **creare contenuti nuovi**: testo (ChatGPT), immagini (DALL-E, Midjourney), audio, video, codice.

**Come funziona un modello linguistico (LLM):**
1. Addestrato su **enormi quantità di testo** (internet, libri, articoli)
2. Impara **miliardi di parametri** (pesi) che catturano pattern del linguaggio
3. Data una sequenza di parole, **predice la parola successiva**
4. Genera testo una parola alla volta

**Limiti dei modelli generativi**:
- **Allucinazioni**: inventano fatti con sicurezza (il modello non "sa", predice)
- **Bias**: se i dati di training contengono pregiudizi, il modello li riproduce
- **Dipendenza dai dati**: la qualità dipende da cosa ha "letto"
- **Non ragionano**: non capiscono veramente, predicono pattern statistici

### AI Act — Regolamento Europeo sull'IA

L'**AI Act** (approvato dall'UE nel 2024) è il primo quadro normativo organico al mondo per l'intelligenza artificiale.

### Approccio basato sul rischio

| Livello di rischio | Cosa include | Conseguenze |
|---|---|---|
| **Inaccettabile** | Manipolazione comportamentale, social scoring, riconoscimento biometrico in tempo reale | **Vietato** |
| **Alto** | IA in sanità, giustizia, educazione, recruitment, infrastrutture critiche | Obblighi severi: valutazione rischi, qualità dati, trasparenza, supervisione umana, robustezza |
| **Limitato** | Chatbot, deepfake (se etichettati) | Obblighi di trasparenza (devi dire che è IA) |
| **Minimo/nullo** | Filtri spam, videogiochi | Nessun obbligo particolare |

### Obblighi per i sistemi ad alto rischio

- Valutazione dei rischi
- Qualità dei dati di addestramento
- Documentazione tecnica
- Tracciabilità e trasparenza
- **Supervisione umana** (l'uomo deve poter intervenire)
- Robustezza, sicurezza e accuratezza
- Registrazione in un database UE

### Impatto economico dell'IA

- **Costi enormi**: addestrare modelli come GPT-4 costa centinaia di milioni di dollari
- **GPU e hardware**: NVIDIA domina il mercato dei chip per IA (GPU H100)
- **Concentrazione di potere**: poche aziende (OpenAI, Google, Meta, Microsoft) controllano l'IA avanzata
- **Automazione**: mansioni ripetitive sempre più automatizzate → trasformazione del lavoro

### Impatto ambientale

| Fase | Impatto |
|---|---|
| **Training** | Enorme consumo energetico e idrico. Addestrare un grande modello può emettere quanto 5 auto nella loro intera vita utile. |
| **Inferenza** | Ogni domanda a ChatGPT consuma energia. Moltiplica per milioni di utenti. |
| **Data center** | Consumano ~1-2% dell'elettricità mondiale. Necessitano raffreddamento ad acqua. |

**Mitigazioni**: energie rinnovabili, modelli più efficienti, data center ottimizzati.

### Impatto sociale

- **Automazione del lavoro**: alcuni lavori ripetitivi spariranno, altri si trasformeranno
- **Nuove competenze**: cresce la domanda di chi sa lavorare CON l'IA
- **Disuguaglianze**: chi ha accesso all'IA ha un vantaggio competitivo
- **Disinformazione**: deepfake e contenuti generati rendono difficile distinguere vero da falso
- **Opportunità**: supporto a diagnosi mediche, traduzione, accessibilità per disabili

### Questioni etiche

| Problema | Descrizione |
|---|---|
| **Trasparenza** | Come decide l'IA? Le reti neurali sono "scatole nere" |
| **Spiegabilità** | Posso capire PERCHÉ l'IA ha preso una decisione? |
| **Bias e discriminazione** | L'IA può amplificare pregiudizi presenti nei dati |
| **Privacy** | I modelli vengono addestrati su dati che potrebbero contenere informazioni personali |
| **Responsabilità** | Se un'IA sbaglia (es. diagnosi medica), di chi è la colpa? |
| **Autonomia umana** | L'IA non deve sostituire il giudizio umano nelle decisioni critiche |

### Ruolo dello sviluppatore

Chi sviluppa IA ha la responsabilità di:
- Garantire che il sistema sia **sicuro, trasparente e verificabile**
- Rispettare il **quadro normativo** (AI Act, GDPR)
- **Testare** per individuare bias e comportamenti indesiderati
- Progettare in modo **inclusivo**, considerando l'impatto su tutti gli utenti
- Documentare limiti e rischi del sistema

---

## DOMANDE PER L'AUTOVERIFICA

### Confini reali e virtuali / GDPR
1. Differenza tra confine reale e confine virtuale. Perché il cyberspazio sfida i confini tradizionali?
2. Cos'è il GDPR? Quali sono i diritti principali che garantisce ai cittadini EU?
3. In che modo il GDPR "supera" i confini geografici?
4. Quali implicazioni ha il GDPR per chi sviluppa un sito web?
5. Come si collegano VPN, firewall e DMZ al concetto di difesa dei confini virtuali?

### Intelligenza Artificiale
6. Differenza tra IA ristretta (ANI), IA generale (AGI) e superintelligenza (ASI).
7. Cos'è il Machine Learning? In cosa si differenzia dalla programmazione tradizionale?
8. Spiega i 4 tipi di apprendimento automatico (supervisionato, non supervisionato, rinforzo, online).
9. Cos'è una rete neurale? Cosa sono input layer, hidden layer e output layer?
10. Cos'è l'IA generativa? Come funziona un modello linguistico (LLM)?
11. Cosa sono le "allucinazioni" di un LLM? Perché accadono?
12. Cos'è l'AI Act europeo? Qual è l'approccio basato sul rischio?
13. Quali sono i 4 livelli di rischio previsti dall'AI Act? Un esempio per ciascuno.
14. Elenca 4 obblighi per i sistemi IA ad alto rischio.
15. Qual è l'impatto ambientale dell'IA? Differenza tra fase di training e inferenza.
16. Quali sono le principali questioni etiche legate all'IA? (almeno 4)
17. Qual è la responsabilità di uno sviluppatore che crea sistemi basati su IA?
