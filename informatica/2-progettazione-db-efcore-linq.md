# GIORNO 2 — PROGETTAZIONE DB + EF CORE + LINQ

> Orale: "Come struttureresti lo schema di un database per...?" — Qui trovi la risposta.

---

## 1. SISTEMI INFORMATIVI

Un **sistema informativo** è l'insieme di persone, processi, dati e tecnologie che raccolgono, elaborano, memorizzano e distribuiscono informazioni a supporto delle attività di un'organizzazione.

Componenti:
- **Dati** (la materia prima)
- **Persone** (chi li usa)
- **Procedure** (come si usano)
- **Tecnologie** (HW e SW con cui si gestiscono)

---

## 2. PROGETTAZIONE DI UN DATABASE

### Tre fasi

| Fase | Modello prodotto | Cosa si fa |
|---|---|---|
| **Concettuale** | Schema E/R | Analisi requisiti → entità, associazioni, attributi |
| **Logica** | Schema relazionale (tabelle) | Mapping E/R → tabelle con PK, FK |
| **Fisica** | DB implementato (SQL) | CREATE TABLE, indici, storage engine, ottimizzazioni |

---

## 3. MODELLO E/R (ENTITY-RELATIONSHIP)

### Entità

Un **oggetto** del mondo reale di cui vogliamo memorizzare informazioni. Si rappresenta con un rettangolo.

```
┌──────────┐
│ CLIENTE  │
└──────────┘
```

**Attributi**: proprietà dell'entità (ovali collegati al rettangolo). Es: nome, email, telefono.

**Chiave primaria**: attributo (o insieme) che identifica univocamente ogni istanza. Si sottolinea.

### Associazioni

Un **legame** tra due o più entità. Si rappresenta con un rombo.

```
┌──────────┐          ┌──────────┐
│ CLIENTE  │───(fa)───│  ORDINE  │
└──────────┘          └──────────┘
```

### Cardinalità delle associazioni

Quante istanze di un'entità partecipano all'associazione.

| Cardinalità | Significato | Esempio |
|---|---|---|
| **(1,1)** | Esattamente una | Ogni ordine ha esattamente 1 cliente |
| **(0,1)** | Zero o una | Ogni dipendente ha 0 o 1 ufficio |
| **(1,N)** | Almeno una | Ogni reparto ha da 1 a N dipendenti |
| **(0,N)** | Zero o molte | Ogni cliente fa da 0 a N ordini |

### Esempio completo

```
┌──────────┐ (1,1)    (0,N) ┌──────────┐
│ CLIENTE  │─────effettua────│  ORDINE  │
└──────────┘                 └─────┬────┘
                                   │
                                   │ (1,N)
                                   │
                              ┌────┴──────────┐
                              │ ORDINE_PRODOTTO│ (tabella ponte: N:N)
                              └────┬──────────┘
                                   │
                                   │ (0,N)
                                   │
                              ┌────┴────┐
                              │ PRODOTTO │
                              └─────────┘
```

**Spiegazione**: un cliente (1,1) può effettuare da 0 a N ordini. Un ordine (1,1) appartiene a esattamente 1 cliente. Un ordine contiene da 1 a N prodotti. Un prodotto può comparire in 0 a N ordini (N:N).

### Vincoli di integrità nel modello E/R

- **Vincolo di chiave**: la PK identifica univocamente
- **Vincolo di dominio**: ogni attributo ha un tipo (INT, VARCHAR, DATE...)
- **Vincolo di integrità referenziale**: una FK deve riferirsi a una PK esistente
- **Vincolo di cardinalità**: (1,1), (0,N), etc.

---

## 4. MAPPING E/R → MODELLO RELAZIONALE

### Regole di derivazione

| Caso | Regola |
|---|---|
| **Entità** | Diventa una tabella. Gli attributi diventano colonne. La PK dell'entità diventa PK della tabella. |
| **Associazione 1:N** | La FK va nella tabella dell'entità lato N (quella con cardinalità massima N). |
| **Associazione 1:1** | La FK può andare in una delle due tabelle (meglio dove la cardinalità minima è 1). |
| **Associazione N:N** | Crea una **tabella ponte** con: PK composta dalle FK delle due entità + eventuali attributi dell'associazione. |

### Esempio di mapping

**Schema E/R**: CLIENTE (1,1) ──effettua── (0,N) ORDINE (1,N) ──contiene── (0,N) PRODOTTO

**Schema logico**:

```sql
CLIENTE (id_cliente PK, nome, email, telefono)
ORDINE (id_ordine PK, data, importo, id_cliente FK → CLIENTE)
PRODOTTO (id_prodotto PK, nome, prezzo, descrizione)
ORDINE_PRODOTTO (id_ordine FK, id_prodotto FK, quantita, PK(id_ordine, id_prodotto))
```

---

## 5. DIAGRAMMI UML DEI CASI D'USO (RASD)

Il **RASD** (Requirements Analysis and Specification Document) è il documento di analisi dei requisiti. Include:

1. **Casi d'uso**: cosa può fare l'utente col sistema
2. **Diagrammi UML**: rappresentazione grafica attori e casi d'uso

### Elementi UML Use Case

```
  👤                  ┌─────────────────┐
 Attore ──────────────│ Caso d'uso       │
(Utente)              │ "Registrarsi"    │
                      └─────────────────┘
```

**Attore**: chi interagisce col sistema (utente, admin, sistema esterno).

**Relazioni tra casi d'uso**:
- **Include** (`<<include>>`): il caso A include SEMPRE il caso B
- **Extend** (`<<extend>>`): il caso B estende A in certe condizioni (opzionale)
- **Generalizzazione**: un attore/caso d'uso è specializzazione di un altro

### Esempio per un e-commerce

```
                          ┌──────────────┐
                    ┌─────│  Registrarsi  │
                    │     └──────────────┘
                    │     ┌──────────────┐
         👤 ────────┼─────│  Effettuare   │
       Cliente       │     │  login       │
                    │     └──────────────┘
                    │     ┌──────────────┐
                    ├─────│  Cercare      │
                    │     │  prodotti     │
                    │     └──────────────┘
                    │     ┌──────────────┐   <<extend>>
                    └─────│  Fare ordine  │────────────── Applicare sconto
                          └──────────────┘
```

> All'orale basta dire: "Il RASD documenta i requisiti. Si usano i diagrammi UML dei casi d'uso per mostrare attori e funzionalità, con relazioni include/extend."

---

## 6. ENTITY FRAMEWORK CORE (EF CORE)

### Cos'è un ORM

**ORM** = Object Relational Mapper. Traduce le tabelle del DB in classi C# (e viceversa). Invece di scrivere SQL a mano, scrivi LINQ e EF Core genera la query.

```
┌──────────┐     EF Core      ┌──────────┐
│Classi C# │ ←────────────→  │ Tabelle   │
│(Model)   │    (ORM)        │ MySQL     │
└──────────┘                  └──────────┘
```

### DbContext

È il "ponte" tra il codice C# e il database. Contiene i `DbSet` (tabelle) e gestisce la connessione.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Cliente> Clienti { get; set; }
    public DbSet<Ordine> Ordini { get; set; }
    public DbSet<Prodotto> Prodotti { get; set; }
}
```

### Model

Una classe C# che rappresenta una tabella.

```csharp
public class Cliente
{
    public int Id { get; set; }           // PK per convenzione
    public string Nome { get; set; }
    public string Email { get; set; }
    public List<Ordine> Ordini { get; set; }  // navigation property (1:N)
}
```

### DTO (Data Transfer Object)

Una classe "leggera" per trasferire solo i dati necessari, senza esporre l'intero Model.

```csharp
public class ClienteDto
{
    public string Nome { get; set; }
    public string Email { get; set; }
}
```

### Code-First vs Database-First

| Approccio | Da cosa parti | Come |
|---|---|---|
| **Code-First** | Scrivi le classi C# (Model) | EF Core crea il DB e le tabelle via migration |
| **Database-First** | Hai già il DB esistente | EF Core genera le classi C# dallo schema DB |

A scuola usate Code-First: scrivete il Model, lanciate `dotnet ef migrations add NomeMigration`, poi `dotnet ef database update`.

---

## 7. LINQ (Language Integrated Query)

Linguaggio di query integrato in C#. Invece di scrivere SQL, scrivi query direttamente nel codice.

### Sintassi base

```csharp
// LINQ method syntax (la più usata)
var clientiTop = _db.Clienti
    .Where(c => c.Ordini.Sum(o => o.Importo) > 1000)
    .OrderBy(c => c.Nome)
    .Select(c => new { c.Nome, c.Email })
    .ToList();

// Equivalente SQL:
// SELECT Nome, Email FROM Clienti
// WHERE (SELECT SUM(Importo) FROM Ordini WHERE id_cliente = Clienti.Id) > 1000
// ORDER BY Nome
```

### Operatori LINQ principali

| LINQ | SQL equivalente |
|---|---|
| `.Where()` | `WHERE` |
| `.Select()` | `SELECT` (proiezione) |
| `.OrderBy()` / `.OrderByDescending()` | `ORDER BY` |
| `.FirstOrDefault()` | `LIMIT 1` |
| `.ToList()` | esegue la query |
| `.Count()` | `COUNT` |
| `.Sum()` | `SUM` |
| `.GroupBy()` | `GROUP BY` |
| `.Include()` | JOIN su navigation property |
| `.Any()` | `EXISTS` |

### Join con Include (EF Core)

```csharp
// Cliente con tutti i suoi ordini (JOIN automatico via navigation property)
var clienteConOrdini = _db.Clienti
    .Include(c => c.Ordini)        // JOIN su tabella Ordini
    .FirstOrDefault(c => c.Id == 5);
```

---

## 8. SQL DIRETTO IN EF CORE

Quando LINQ non basta, puoi eseguire SQL direttamente.

### Query SQL (SELECT)

```csharp
// FromSql: restituisce entità del Model
int id = 5;
var cliente = _db.Clienti
    .FromSql($"SELECT * FROM Clienti WHERE Id = {id}")
    .ToList()
    .FirstOrDefault();

// SqlQuery: restituisce tipi arbitrari
int count = _db.Database
    .SqlQuery<int>($"SELECT COUNT(*) FROM Clienti")
    .ToList()[0];
```

### Comandi SQL (INSERT/UPDATE/DELETE)

```csharp
// ExecuteSql: per INSERT, UPDATE, DELETE (non SELECT!)
var righe = _db.Database
    .ExecuteSql($"DELETE FROM Log WHERE Data < {cutoffDate}");
```

**⚠️ Sicurezza**: usa SEMPRE `FormattableString` (con `$"..."`) o parametri, MAI concatenazione di stringhe (→ SQL Injection!).

---

## DOMANDE PER L'AUTOVERIFICA

1. Quali sono le 3 fasi della progettazione di un database? Cosa produce ciascuna?
2. Cos'è un'entità nel modello E/R? E un'associazione?
3. Cosa significa cardinalità (1,1), (0,N), (1,N)? Fai un esempio per ciascuna.
4. Come si mappa un'associazione 1:N nel modello relazionale? E una N:N?
5. Cos'è una tabella ponte? Quando serve?
6. Trasforma in schema logico: STUDENTE(1,N)─frequenta─(0,N)CORSO(1,1)─tenuto da─(0,N)DOCENTE
7. Cos'è un caso d'uso in UML? Differenza tra `<<include>>` e `<<extend>>`?
8. Cos'è un ORM? Cosa fa EF Core?
9. Cos'è il DbContext? E un DbSet?
10. Differenza tra Model e DTO.
11. Code-First vs Database-First: differenza?
12. Scrivi una query LINQ che restituisce tutti i prodotti con prezzo > 100, ordinati per nome.
13. Come si fa una JOIN con EF Core (Include)?
14. Quando usi FromSql / SqlQuery / ExecuteSql in EF Core?
15. Perché usare sempre FormattableString ($"...") invece della concatenazione?
16. Cos'è una migration in EF Core? A cosa serve?
