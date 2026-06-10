# GIORNO 4 — ANALISI NUMERICA + INTELLIGENZA ARTIFICIALE

> Argomenti del commissario esterno non presenti nel nostro programma principale.

---

## 1. ANALISI NUMERICA

### Complessità computazionale

Misura quanto tempo/memoria un algoritmo richiede al crescere dell'input.

### Notazione O-grande (Big-O)

Descrive il **comportamento asintotico**: come cresce il tempo di esecuzione quando l'input diventa molto grande.

```
O(1)       costante          accesso a un elemento di un array
O(log n)   logaritmica       ricerca binaria
O(n)       lineare           scorrere un array
O(n log n) quasi-lineare     mergesort, quicksort (caso medio)
O(n^2)     quadratica        bubblesort, due cicli annidati
O(2^n)     esponenziale      Fibonacci ricorsivo ingenuo
O(n!)      fattoriale        commesso viaggiatore (brute force)
```

### Problemi di classe P e NP

- **P** (Polynomial): risolvibili in tempo polinomiale. "Facili".
- **NP** (Nondeterministic Polynomial): la soluzione può essere **verificata** in tempo polinomiale. Non si sa se sono anche risolvibili in tempo polinomiale (P = NP? e un problema aperto).
- **NP-Completi**: i piu difficili in NP. Risolverne uno in tempo polinomiale risolverebbe TUTTI gli NP.

Esempi:
- Ordinare un array: P (O(n log n))
- Commesso viaggiatore: NP-Completo
- Fattorizzare un numero: NP (non si sa se e in P)

### Numeri macchina e floating-point

I computer rappresentano i reali con un numero **finito di bit** -> errori di approssimazione.

**Standard IEEE 754**:

```
float (32 bit):  [S] [EEEEEEEE] [MMMMMMMMMMMMMMMMMMMMMMM]
                   1      8                23
                 segno esponente         mantissa

double (64 bit): 1 bit segno + 11 bit esponente + 52 bit mantissa

Valore = (-1)^s x 1.m x 2^(e - bias)
```

### Errori

**Overflow**: numero troppo grande -> infinito
**Underflow**: numero troppo vicino a zero -> 0

**Precisione di macchina (epsilon)**: la piu piccola differenza tra 1 e il numero successivo. Float: ~1.2e-7. Double: ~2.2e-16.

**Errore assoluto**: |valore vero - valore approssimato|
**Errore relativo**: |valore vero - valore approssimato| / |valore vero|

### Cancellazione numerica (catastrophic cancellation)

Quando sottrai due numeri molto vicini, le cifre significative si annullano.

```
Esempio:
a = 1.23456789  (9 cifre significative)
b = 1.23456780
a - b = 0.00000009  -> solo 1 cifra significativa!
```

**Mitigazione**: riorganizzare i calcoli per evitare sottrazioni tra numeri quasi uguali.

---

## 2. INTELLIGENZA ARTIFICIALE

### Evoluzione storica

| Periodo | Tappa | Cosa |
|---|---|---|
| 1940s-50s | Cibernetica (Wiener) | Sistemi di controllo. Primi modelli del neurone (McCulloch-Pitts). |
| 1958 | Perceptron (Rosenblatt) | Rete neurale a singolo strato. Classifica pattern linearmente separabili. |
| 1960s-70s | Chatbot primitivi | ELIZA (1966): pattern matching, simula psicoterapeuta. |
| 1980s | Sistemi esperti | Regole IF...THEN, codifica conoscenza esperta. Diagnosi medica. |
| 2010s | Deep Learning | Reti neurali profonde, GPU. AlexNet vince ImageNet (2012). |
| 2020s | IA Generativa | LLM (ChatGPT), modelli di diffusione (DALL-E, Midjourney). |

### Machine Learning

Il computer **impara dai dati** invece di essere programmato esplicitamente.

### Apprendimento supervisionato

Hai dati **etichettati** (input -> output corretto). L'algoritmo impara la mappatura.

**Classificazione**: output e una categoria discreta.
- "Questa email e spam o no?" (binaria)
- "Questo animale e gatto, cane o uccello?" (multiclasse)

**Regressione**: output e un valore continuo.
- "Quanto costera questa casa?"
- "Qual e la temperatura domani?"

### Algoritmi di ML

#### KNN (K-Nearest Neighbors)

Classifica un punto in base ai **K vicini piu prossimi** nello spazio delle feature. Non c'e un vero "addestramento": memorizza tutti i dati e decide al momento.

- Pro: semplice, nessuna assunzione sui dati
- Contro: lento con dataset grandi, sensibile alla scala

```
Nuovo punto "?" da classificare:
Classe A:  o   o       ?
Classe B:      x   x x
Se K=3: i 3 vicini sono 2 B e 1 A -> classificato come B
```

#### Naive Bayes

Basato sul **teorema di Bayes** con l'assunzione "ingenua" (naive) che tutte le feature siano **indipendenti** tra loro.

```
P(classe | dati) = P(dati | classe) x P(classe) / P(dati)
```

- Pro: molto veloce, funziona bene con dati testuali (spam filter)
- Contro: l'assunzione di indipendenza e quasi sempre falsa (ma funziona comunque)

**Esempio**: classificare un'email come spam: conta parole come "gratis", "vinci", "clicca qui", calcola la probabilita che un'email con quelle parole sia spam.

#### Decision Tree (Albero decisionale)

Struttura ad albero dove ogni **nodo** e una domanda su una feature e ogni **foglia** e una classe.

```
               [Eta > 30?]
              /            \
           si                no
          /                    \
   [Reddito > 50k?]        [Studente?]
    /           \           /         \
  si            no         si          no
  /              \         /            \
ACQUISTA      NON ACQ.  ACQUISTA     NON ACQ.
```

- Pro: interpretabile (puoi spiegare PERCHE ha deciso cosi)
- Contro: tende a overfittare se non potato

### Reti Neurali e Deep Learning

#### Percettrone

Il neurone artificiale piu semplice. Riceve input, li pesa, li somma, applica una **funzione di attivazione** e produce un output.

```
x1 ----w1----\
x2 ----w2-----+----> somma pesata --> f() --> output
x3 ----w3----/

y = f(w1*x1 + w2*x2 + w3*x3 + b)
```

#### Funzioni di attivazione

| Funzione | Formula | Uso |
|---|---|---|
| **Sigmoide** | 1/(1+e^-x) | Output tra 0 e 1, usata in passato |
| **ReLU** | max(0, x) | La piu usata oggi: semplice, veloce |
| **Softmax** | e^xi / sum(e^xj) | Trasforma output in probabilita (classificazione) |

#### Backpropagation

Algoritmo per **addestrare** reti neurali:
1. **Forward pass**: l'input attraversa la rete, si calcola l'output
2. Si calcola **l'errore** (differenza tra output predetto e output desiderato)
3. **Backward pass**: l'errore viene propagato all'indietro, aggiornando i pesi per ridurlo (discesa del gradiente)
4. Si ripete per molte epoche

#### Deep Learning

Reti neurali con **molti strati nascosti** (hidden layers). Ogni strato impara caratteristiche sempre piu astratte:
- Primo strato: bordi, colori
- Strati intermedi: forme, texture
- Strati finali: oggetti, volti

### Big Data e Data Science

**Big Data**: dataset troppo grandi per essere gestiti con strumenti tradizionali. Caratterizzati dalle **3V** (a volte 5V):
- **Volume**: enorme quantita di dati (terabyte, petabyte)
- **Velocita**: generati in tempo reale (social media, sensori IoT)
- **Varieta**: formati diversi (testo, immagini, video, log)

**Data Science**: disciplina che estrae conoscenza dai dati combinando statistica, informatica e dominio specifico.

### IA Generativa

Sistemi che **creano contenuti nuovi**: testo (ChatGPT), immagini (DALL-E), codice, audio.

Differenza con IA tradizionale: l'IA tradizionale classifica/riconosce. L'IA generativa **produce** contenuti originali.

---

## DOMANDE PER L'AUTOVERIFICA

1. Cosa misura la notazione O-grande? Elenca 4 complessita in ordine crescente.
2. Differenza tra problemi di classe P e NP. Cosa significa "NP-Completo"?
3. Come e rappresentato un numero floating-point (IEEE 754)? Quali sono i 3 campi?
4. Cos'e l'errore di overflow e di underflow?
5. Errore assoluto vs errore relativo: differenza?
6. Cos'e la cancellazione numerica? Fai un esempio.
7. Evoluzione IA: cibernetica, perceptron, sistemi esperti, deep learning, IA generativa.
8. Differenza tra apprendimento supervisionato e non supervisionato.
9. Differenza tra classificazione e regressione.
10. Come funziona KNN? Pro e contro.
11. Naive Bayes: su cosa si basa? Perche "naive"?
12. Cos'e un albero decisionale? Vantaggio principale rispetto ad altri algoritmi?
13. Cos'e un percettrone? Cosa fa la funzione di attivazione?
14. Cos'e la backpropagation? A cosa serve?
15. Cos'e il Deep Learning? Cosa cambia rispetto a una rete neurale semplice?
16. Big Data: cosa sono le 3V?
17. IA generativa: in cosa si differenzia dall'IA tradizionale?
