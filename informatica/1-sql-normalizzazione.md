# GIORNO 1 — SQL COMPLETO + NORMALIZZAZIONE

> Orale: qui ti chiedono sicuramente qualcosa. È il cuore di Informatica. Ritmo: 6 ore.

---

## 1. SQL: TIPI DI ISTRUZIONI

| Categoria | Cosa fa | Istruzioni |
|---|---|---|
| **DDL** (Data Definition) | Definisce la struttura | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** (Data Manipulation) | Manipola i dati | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** (Data Control) | Controlla accessi e permessi | `GRANT`, `REVOKE` |
| **TCL** (Transaction Control) | Gestisce transazioni | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

---

## 2. DDL — CREARE TABELLE

### CREATE TABLE

```sql
CREATE TABLE Clienti (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    telefono VARCHAR(20),
    settore VARCHAR(50),
    data_registrazione DATETIME DEFAULT CURRENT_TIMESTAMP,
    attivo BOOLEAN DEFAULT TRUE
);
```

### Vincoli

| Vincolo | Significato |
|---|---|
| `PRIMARY KEY` | Identificatore univoco, NOT NULL + UNIQUE |
| `FOREIGN KEY ... REFERENCES` | Chiave esterna, integrità referenziale |
| `UNIQUE` | Valore unico nella tabella |
| `NOT NULL` | Il campo non può essere vuoto |
| `CHECK` | Vincolo su dominio (es. `CHECK (eta >= 18)`) |
| `DEFAULT` | Valore predefinito se non specificato |
| `AUTO_INCREMENT` | Valore auto-incrementale (MySQL) |

### Integrità referenziale

```sql
CREATE TABLE Ordini (
    id INT PRIMARY KEY,
    id_cliente INT,
    FOREIGN KEY (id_cliente) REFERENCES Clienti(id)
        ON DELETE CASCADE    -- se cancello il cliente, cancello i suoi ordini
        ON UPDATE CASCADE    -- se cambio id cliente, aggiorno la FK
);
```

| Opzione | Comportamento |
|---|---|
| `CASCADE` | Propaga l'operazione (cancella/aggiorna i record figli) |
| `SET NULL` | Imposta la FK a NULL |
| `RESTRICT` / `NO ACTION` | Blocca l'operazione se ci sono record figli |

### ALTER TABLE

```sql
ALTER TABLE Clienti ADD COLUMN partita_iva VARCHAR(16);
ALTER TABLE Clienti MODIFY COLUMN telefono VARCHAR(30);
ALTER TABLE Clienti DROP COLUMN attivo;
ALTER TABLE Ordini ADD CONSTRAINT fk_ordine_cliente
    FOREIGN KEY (id_cliente) REFERENCES Clienti(id);
```

### DROP TABLE
```sql
DROP TABLE IF EXISTS Ordini;  -- cancella la tabella
TRUNCATE TABLE Ordini;        -- svuota i dati ma mantiene la struttura
```

---

## 3. DML — MANIPOLARE I DATI

### INSERT

```sql
INSERT INTO Clienti (nome, email, telefono)
VALUES ('Mario Rossi', 'mario@email.it', '3331234567');

INSERT INTO Clienti (nome, email)
VALUES ('Anna Verdi', 'anna@email.it'),
       ('Luca Bianchi', 'luca@email.it');
```

### SELECT base

```sql
SELECT nome, email FROM Clienti;
SELECT * FROM Clienti WHERE settore = 'IT';
SELECT * FROM Clienti WHERE id IN (1, 3, 5);
SELECT * FROM Clienti WHERE nome LIKE 'M%';      -- inizia con M
SELECT * FROM Clienti WHERE email IS NOT NULL;
SELECT * FROM Clienti WHERE data_reg BETWEEN '2025-01-01' AND '2025-12-31';
```

### UPDATE e DELETE

```sql
UPDATE Clienti SET telefono = '3385551234' WHERE id = 1;
UPDATE Clienti SET attivo = FALSE WHERE id = 3;
DELETE FROM Clienti WHERE id = 5;
```

---

## 4. JOIN — COLLEGARE TABELLE

### Tipi di JOIN

```
INNER JOIN:  solo i record che hanno corrispondenza in ENTRAMBE le tabelle
LEFT JOIN:   TUTTI i record della tabella sinistra + match da destra (NULL se non c'è)
RIGHT JOIN:  TUTTI i record della tabella destra + match da sinistra
SELF JOIN:   JOIN di una tabella con se stessa (usa alias)
```

### INNER JOIN

```sql
-- Clienti che hanno fatto ordini
SELECT c.nome, o.data_ordine, o.importo
FROM Clienti c
INNER JOIN Ordini o ON c.id = o.id_cliente;
```

### LEFT JOIN

```sql
-- TUTTI i clienti, anche quelli senza ordini (importo sarà NULL)
SELECT c.nome, o.importo
FROM Clienti c
LEFT JOIN Ordini o ON c.id = o.id_cliente;
```

### SELF JOIN

```sql
-- Dipendenti con il loro manager
SELECT d1.nome AS dipendente, d2.nome AS manager
FROM Dipendenti d1
LEFT JOIN Dipendenti d2 ON d1.id_manager = d2.id;
```

### Tabella riepilogativa JOIN

| JOIN | Tabella sx | Tabella dx |
|---|---|---|
| INNER | Solo match | Solo match |
| LEFT | TUTTI | Match o NULL |
| RIGHT | Match o NULL | TUTTI |
| CROSS (prodotto cartesiano) | Ogni riga con ogni riga | |

---

## 5. FUNZIONI DI AGGREGAZIONE, GROUP BY, HAVING

### Funzioni di aggregazione

```sql
SELECT COUNT(*) FROM Clienti;                    -- conteggio
SELECT AVG(importo) FROM Ordini;                  -- media
SELECT SUM(importo) FROM Ordini;                  -- somma
SELECT MAX(importo), MIN(importo) FROM Ordini;    -- max e min
```

### GROUP BY

Aggruppa i dati per un campo e applica funzioni di aggregazione a ciascun gruppo.

```sql
-- Totale speso da ogni cliente
SELECT c.nome, SUM(o.importo) AS totale_speso
FROM Clienti c
INNER JOIN Ordini o ON c.id = o.id_cliente
GROUP BY c.id, c.nome;
```

### HAVING

Filtra i gruppi (come WHERE filtra le righe). **WHERE si applica PRIMA del GROUP BY, HAVING DOPO.**

```sql
-- Solo i clienti che hanno speso più di 1000€
SELECT c.nome, SUM(o.importo) AS totale
FROM Clienti c
INNER JOIN Ordini o ON c.id = o.id_cliente
GROUP BY c.id, c.nome
HAVING SUM(o.importo) > 1000;
```

### Ordine delle clausole SQL

```sql
SELECT   ...     -- cosa voglio vedere
FROM     ...     -- da quali tabelle
JOIN     ...     -- come sono collegate
WHERE    ...     -- filtri sulle righe (prima del GROUP BY!)
GROUP BY ...     -- raggruppamento
HAVING   ...     -- filtri sui gruppi
ORDER BY ...     -- ordinamento
LIMIT    ...;    -- quante righe
```

---

## 6. SUBQUERY

Query dentro un'altra query. Possono stare in: SELECT, FROM, WHERE.

```sql
-- Clienti che hanno speso più della media
SELECT nome FROM Clienti
WHERE id IN (
    SELECT id_cliente FROM Ordini
    GROUP BY id_cliente
    HAVING SUM(importo) > (SELECT AVG(importo) FROM Ordini)
);
```

| Tipo subquery | Dove | Esempio |
|---|---|---|
| Scalare | SELECT | `(SELECT MAX(prezzo) FROM Prodotti)` |
| Di colonna | WHERE ... IN | `WHERE id IN (SELECT ...)` |
| Correlata | WHERE | `WHERE EXISTS (SELECT ... WHERE tabella_esterna.id = ...)` |

---

## 7. VISTE (VIEW)

Una vista è una **query salvata con un nome**. Si comporta come una tabella virtuale.

```sql
CREATE VIEW ClientiTop AS
SELECT c.nome, SUM(o.importo) AS totale
FROM Clienti c
JOIN Ordini o ON c.id = o.id_cliente
GROUP BY c.id, c.nome
HAVING totale > 5000;

-- Ora posso interrogarla come una tabella normale
SELECT * FROM ClientiTop;
```

**Vantaggi**: semplifica query complesse, sicurezza (nascondi colonne sensibili), riutilizzo.

---

## 8. STORED PROCEDURE E FUNCTIONS

### Stored Procedure

Blocco di codice SQL salvato nel DB, eseguibile con `CALL`.

```sql
DELIMITER //
CREATE PROCEDURE InserisciCliente(
    IN p_nome VARCHAR(100),
    IN p_email VARCHAR(150)
)
BEGIN
    INSERT INTO Clienti (nome, email) VALUES (p_nome, p_email);
    SELECT LAST_INSERT_ID() AS nuovo_id;
END //
DELIMITER ;

CALL InserisciCliente('Paolo Neri', 'paolo@email.it');
```

**Parametri**: `IN` (input), `OUT` (output), `INOUT` (input/output).

### Stored Function

Simile alla procedure ma **restituisce SEMPRE un valore** con `RETURN`.

```sql
CREATE FUNCTION TotaleOrdiniCliente(cliente_id INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE totale DECIMAL(10,2);
    SELECT SUM(importo) INTO totale FROM Ordini WHERE id_cliente = cliente_id;
    RETURN IFNULL(totale, 0);
END;

SELECT nome, TotaleOrdiniCliente(id) FROM Clienti;
```

### Differenza Procedure vs Function

| | Procedure | Function |
|---|---|---|
| **Chiamata** | `CALL nome()` | `SELECT nome()` o in espressioni |
| **Valore ritorno** | Opzionale (OUT) | Obbligatorio (RETURN) |
| **Dentro SQL** | NO (solo CALL) | SÌ (SELECT, WHERE) |

---

## 9. TRIGGER

Codice SQL che si esegue **automaticamente** quando avviene un evento su una tabella.

```sql
CREATE TRIGGER prima_inserimento_ordine
BEFORE INSERT ON Ordini
FOR EACH ROW
BEGIN
    IF NEW.importo < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Importo non può essere negativo';
    END IF;
END;
```

**Eventi trigger**: `BEFORE` / `AFTER` + `INSERT` / `UPDATE` / `DELETE`.

**NEW vs OLD**: `NEW` = valore dopo l'operazione, `OLD` = valore prima (solo UPDATE/DELETE).

---

## 10. SCHEDULED EVENTS

Task pianificati che si eseguono a intervalli regolari.

```sql
CREATE EVENT pulizia_log
ON SCHEDULE EVERY 1 DAY
STARTS '2025-01-01 03:00:00'
DO
    DELETE FROM Log WHERE data < DATE_SUB(NOW(), INTERVAL 30 DAY);
```

---

## 11. TRANSAZIONI ACID

Una **transazione** è una sequenza di operazioni che devono essere eseguite **tutte o nessuna**.

### Proprietà ACID

| Lettera | Significato | Spiegazione |
|---|---|---|
| **A** | Atomicity | O tutto o niente. Se un'operazione fallisce, si annulla tutto (ROLLBACK). |
| **C** | Consistency | Il DB passa da uno stato valido a un altro stato valido (vincoli rispettati). |
| **I** | Isolation | Transazioni concorrenti non si influenzano (come se fossero eseguite in serie). |
| **D** | Durability | Una volta fatto COMMIT, i dati sono permanenti (anche in caso di crash). |

### Sintassi

```sql
START TRANSACTION;
    UPDATE Conti SET saldo = saldo - 100 WHERE id = 1;
    UPDATE Conti SET saldo = saldo + 100 WHERE id = 2;
    -- Se qualcosa va storto:
    -- ROLLBACK;
    -- Se tutto ok:
COMMIT;
```

---

## 12. DBA, BACKUP, SICUREZZA

### Ruolo del DBA (Database Administrator)

- Progettazione schema DB
- Gestione utenti e permessi (GRANT/REVOKE)
- Backup e restore
- Monitoraggio performance
- Tuning indici
- Manutenzione e aggiornamenti

### GRANT e REVOKE

```sql
GRANT SELECT, INSERT ON Clienti TO 'utente'@'localhost';
GRANT ALL PRIVILEGES ON mio_db.* TO 'admin'@'%';
REVOKE DELETE ON Clienti FROM 'utente'@'localhost';
FLUSH PRIVILEGES;
```

### Backup con mysqldump

```bash
mysqldump -u root -p nome_db > backup.sql
mysql -u root -p nome_db < backup.sql   # restore
```

---

## 13. ALGEBRA RELAZIONALE (giusto per saperla spiegare)

Operatori fondamentali:

| Operatore | Simbolo | Equivalente SQL |
|---|---|---|
| **Selezione** | σ | `WHERE` |
| **Proiezione** | π | `SELECT` (colonne specifiche) |
| **Prodotto cartesiano** | × | `CROSS JOIN` |
| **Join naturale** | ⨝ | `NATURAL JOIN` / `INNER JOIN ... USING` |
| **Ridenominazione** | ρ | `AS` (alias) |
| **Unione** | ∪ | `UNION` |
| **Differenza** | − | `EXCEPT` / `NOT IN` |

A livello di orale: "L'algebra relazionale è il fondamento teorico dell'SQL. Gli operatori di base sono selezione, proiezione, join, unione e differenza."

---

## 14. NORMALIZZAZIONE

### Perché normalizzare

Evitare **anomalie**: ridondanza, inconsistenza, difficoltà di update/delete.

### Forme normali

| FN | Regola | Esempio di violazione |
|---|---|---|
| **1NF** | Ogni colonna contiene valori **atomici** (non ripetuti). Niente array o colonne multi-valore. | Tabella con colonne `telefono1, telefono2, telefono3` |
| **2NF** | 1NF + ogni colonna non-chiave dipende dall'**intera** chiave primaria (no dipendenze parziali). | In una tabella `OrdiniProdotti(id_ordine, id_prodotto, nome_prodotto)`, `nome_prodotto` dipende solo da `id_prodotto`, non dalla chiave composta |
| **3NF** | 2NF + nessuna dipendenza **transitiva** (colonna non-chiave dipende da un'altra non-chiave). | `Clienti(id, nome, citta, provincia)` → `provincia` dipende da `citta`, non direttamente da `id` |

### Esempio di normalizzazione

**Tabella non normalizzata:**
```
Ordini(id_ord, data, id_cliente, nome_cliente, citta, provincia,
        id_prodotto, nome_prodotto, prezzo, quantita)
```

**Dopo normalizzazione (3NF):**
```
Clienti(id, nome, id_citta)       Citta(id, nome, id_provincia)    Province(id, nome)
Ordini(id, data, id_cliente)      Prodotti(id, nome, prezzo)
Ordini_Prodotti(id_ordine, id_prodotto, quantita)   ← tabella ponte N:N
```

---

## DOMANDE PER L'AUTOVERIFICA

1. Differenza tra DDL, DML, DCL. Esempi per ciascuno.
2. Cos'è una FOREIGN KEY? Cosa significano ON DELETE CASCADE e ON DELETE SET NULL?
3. Differenza tra INNER JOIN e LEFT JOIN. Quando usi l'uno e quando l'altro?
4. Qual è l'ordine corretto delle clausole SQL (SELECT/FROM/WHERE/GROUP BY/HAVING/ORDER BY)?
5. Differenza tra WHERE e HAVING.
6. Cos'è una subquery? Dove può apparire?
7. Differenza tra Stored Procedure e Stored Function.
8. Cos'è un TRIGGER? Eventi su cui può scattare?
9. Cosa significa ACID? Spiega ogni lettera con un esempio.
10. Ruolo del DBA: elenca 4 compiti principali.
11. GRANT e REVOKE: a cosa servono? Fai un esempio.
12. Cos'è la normalizzazione? Spiega 1NF, 2NF, 3NF con esempi.
13. Normalizza questa tabella: `Ordini(id_ordine, data, cliente, citta, prodotto, prezzo, qta)`.
14. Algebra relazionale: cosa sono selezione e proiezione? Equivalente SQL?
15. Come fai il backup di un database MySQL?
