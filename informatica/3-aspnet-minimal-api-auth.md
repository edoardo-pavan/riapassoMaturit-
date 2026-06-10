# GIORNO 3 — ASP.NET CORE MINIMAL API + AUTENTICAZIONE

> Orale: ti chiedono come funziona un'API REST, il routing, l'autenticazione. Niente panico.

---

## 1. ASP.NET CORE — ARCHITETTURA

### Cos'è

ASP.NET Core è un **framework open-source** di Microsoft per creare applicazioni web e API. È cross-platform (Windows, Linux, macOS), performante e modulare.

### Architettura a strati

```
┌─────────────────────────────┐
│       Client (Browser)      │  ← HTML/CSS/JS (Frontend)
├─────────────────────────────┤
│       ASP.NET Core API      │  ← C# Minimal API (Backend)
├─────────────────────────────┤
│       EF Core (ORM)         │  ← Traduzione C# ↔ SQL
├─────────────────────────────┤
│       Database (MySQL)      │  ← Dati persistenti
└─────────────────────────────┘
```

### Web Server Kestrel

Kestrel è il **web server integrato** di ASP.NET Core. Riceve le richieste HTTP e le inoltra all'applicazione. È cross-platform e performante.

### Web Host

Il **Web Host** è l'ambiente che configura e avvia l'applicazione. In `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
// configurazione servizi...
var app = builder.Build();
// configurazione middleware...
app.Run();
```

### Middleware

I **middleware** sono componenti software che formano una **pipeline**: ogni richiesta HTTP li attraversa in ordine. Ogni middleware può elaborare la richiesta e/o passarla al successivo.

```
Request → [Middleware 1] → [Middleware 2] → [Middleware 3] → Endpoint
                                                               ↓
Response ← [Middleware 1] ← [Middleware 2] ← [Middleware 3] ←
```

Esempi di middleware: autenticazione, CORS, routing, logging, gestione errori, file statici.

```csharp
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
app.MapGet("/", () => "Hello World!");
```

**Ordine importante**: i middleware vengono eseguiti nell'ordine in cui sono dichiarati in `Program.cs`.

---

## 2. MINIMAL API — STRUTTURA

### Cos'è

Le **Minimal API** sono un modo "leggero" di creare API REST in ASP.NET Core, con meno codice boilerplate rispetto ai Controller tradizionali. Introdotte in .NET 6.

### Routing

Il **routing** associa una richiesta HTTP (metodo + URL) a una funzione (endpoint handler).

```
GET    /api/clienti      →  Lista tutti i clienti
GET    /api/clienti/{id} →  Dettaglio di un cliente
POST   /api/clienti      →  Crea un nuovo cliente
PUT    /api/clienti/{id} →  Modifica un cliente
DELETE /api/clienti/{id} →  Cancella un cliente
```

### Esempio completo CRUD

```csharp
// GET tutti
app.MapGet("/api/clienti", async (AppDbContext db) =>
    await db.Clienti.ToListAsync());

// GET per ID
app.MapGet("/api/clienti/{id}", async (int id, AppDbContext db) =>
    await db.Clienti.FindAsync(id) is Cliente c ? Results.Ok(c) : Results.NotFound());

// POST (crea)
app.MapPost("/api/clienti", async (ClienteDto dto, AppDbContext db) =>
{
    var cliente = new Cliente { Nome = dto.Nome, Email = dto.Email };
    db.Clienti.Add(cliente);
    await db.SaveChangesAsync();
    return Results.Created($"/api/clienti/{cliente.Id}", cliente);
});

// PUT (modifica)
app.MapPut("/api/clienti/{id}", async (int id, ClienteDto dto, AppDbContext db) =>
{
    var cliente = await db.Clienti.FindAsync(id);
    if (cliente is null) return Results.NotFound();
    cliente.Nome = dto.Nome;
    cliente.Email = dto.Email;
    await db.SaveChangesAsync();
    return Results.NoContent();
});

// DELETE
app.MapDelete("/api/clienti/{id}", async (int id, AppDbContext db) =>
{
    var cliente = await db.Clienti.FindAsync(id);
    if (cliente is null) return Results.NotFound();
    db.Clienti.Remove(cliente);
    await db.SaveChangesAsync();
    return Results.NoContent();
});
```

### Dependency Injection

Invece di creare oggetti a mano (`new AppDbContext()`), li **inietti** come parametri. ASP.NET Core li fornisce automaticamente.

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));

// Poi negli endpoint:
app.MapGet("/api/clienti", async (AppDbContext db) => ...); // db viene iniettato!
```

### Service Lifetimes

| Lifetime | Quando viene creato | Esempio |
|---|---|---|
| **Transient** | Ogni volta che viene richiesto | Servizi leggeri e stateless |
| **Scoped** | Una volta per richiesta HTTP | DbContext (il più comune!) |
| **Singleton** | Una volta per tutta la vita dell'app | Cache, configurazioni globali |

---

## 3. VALIDAZIONE

### Server-side (nel backend)

```csharp
app.MapPost("/api/clienti", async (ClienteDto dto, AppDbContext db) =>
{
    if (string.IsNullOrWhiteSpace(dto.Nome))
        return Results.BadRequest("Il nome è obbligatorio");
    if (!dto.Email.Contains('@'))
        return Results.BadRequest("Email non valida");
    // ... inserimento
});
```

### Client-side (nel frontend con JS)

```javascript
form.addEventListener('submit', (e) => {
    if (nome.value.trim() === '') {
        e.preventDefault();
        errore.textContent = 'Il nome è obbligatorio';
    }
});

// MAI fidarsi solo della validazione client-side!
// Il backend deve SEMPRE riverificare.
```

> **Regola d'oro**: la validazione server-side è OBBLIGATORIA. Quella client-side è un optional per l'UX.

---

## 4. AUTENTICAZIONE E AUTORIZZAZIONE

### Concetti

| Termine | Significato |
|---|---|
| **Autenticazione** | Verifica l'identità: "Chi sei?" (username/password) |
| **Autorizzazione** | Verifica i permessi: "Cosa puoi fare?" (ruoli, policy) |

### Cookie-based Authentication

1. L'utente fa login con username e password
2. Il server verifica e crea una **sessione**
3. Il server invia un **cookie** con un ID di sessione
4. Il browser rimanda il cookie a ogni richiesta successiva
5. Il server riconosce l'utente dal cookie

**Pro**: semplice, nativo nei browser.
**Contro**: il server deve tenere lo stato della sessione (non scala benissimo in orizzontale). Vulnerabile a CSRF se non protetto.

### JWT (JSON Web Token)

Un token **autocontenuto** e **firmato digitalmente** che contiene le informazioni dell'utente.

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJtYXJpb0B3ZWJkZXYuaXQiLCJyb2xlIjoiYWRtaW4ifQ.sig
└────── Header ──────┘└────────── Payload ──────────────┘└─ Firma ─┘
```

**Flusso JWT**:
1. L'utente fa login → il server genera un JWT (Access Token) e lo invia al client
2. Il client lo salva (localStorage / cookie) e lo allega a ogni richiesta nell'header: `Authorization: Bearer <token>`
3. Il server verifica la firma del JWT (senza bisogno di sessioni lato server → stateless!)

**Access Token vs Refresh Token**:

| | Access Token | Refresh Token |
|---|---|---|
| **Durata** | Breve (15 min – 1 ora) | Lunga (giorni/settimane) |
| **Scopo** | Autorizzare richieste API | Ottenere un nuovo Access Token |
| **Dove si salva** | Memoria / cookie | HttpOnly cookie (più sicuro) |

**Perché due token?** Se un Access Token viene rubato, scade in fretta. Per ottenerne uno nuovo serve il Refresh Token.

### Cookie vs JWT: confronto rapido

| | Cookie + Sessione | JWT |
|---|---|---|
| **Stato** | Stateful (server tiene la sessione) | Stateless (il token contiene tutto) |
| **Scalabilità** | Meno scalabile (sessione su un server) | Molto scalabile (qualsiasi server può verificare) |
| **CSRF** | Vulnerabile (serve protezione anti-CSRF) | Immune (se inviato via header, non cookie) |
| **Revoca** | Immediata (cancelli la sessione) | Complessa (serve blacklist) |

### OAuth 2.0 e OpenID Connect (cenni)

- **OAuth 2.0**: framework per **autorizzazione** delegata. "Permetto a questa app di accedere ai miei dati su Google senza darle la mia password."
- **OpenID Connect** (OIDC): strato di **autenticazione** sopra OAuth 2.0. "Fai login col tuo account Google/Microsoft."

**Flusso tipico**: l'utente clicca "Login con Google" → reindirizzato a Google → autorizza → Google rimanda un token al sito → il sito autentica l'utente.

---

## 5. CONFIGURAZIONE E SEGRETI

### appsettings.json

Contiene configurazioni (stringhe di connessione, chiavi API, impostazioni).

```json
{
  "ConnectionStrings": {
    "Default": "server=localhost;database=miodb;user=root;password=***"
  },
  "JwtSettings": {
    "SecretKey": "chiave-super-segreta-lunga-almeno-32-caratteri"
  }
}
```

### Secrets (sviluppo)

Mai mettere password nei file committati! In sviluppo usa **User Secrets**:

```bash
dotnet user-secrets init
dotnet user-secrets set "JwtSettings:SecretKey" "valore-segreto"
```

In produzione: variabili d'ambiente, Azure Key Vault, file `.env`.

---

## DOMANDE PER L'AUTOVERIFICA

1. Cos'è ASP.NET Core? Su quali sistemi operativi gira?
2. Cos'è Kestrel? Che ruolo ha?
3. Cosa sono i middleware in ASP.NET Core? Fai 3 esempi.
4. Perché l'ordine dei middleware è importante?
5. Cosa sono le Minimal API? Differenza rispetto ai Controller tradizionali?
6. Cos'è il routing? Come si associa un URL a un handler?
7. Implementa a parole un endpoint CRUD per la tabella Prodotti (GET, POST, PUT, DELETE).
8. Cos'è la Dependency Injection? A cosa serve?
9. Spiega i 3 service lifetimes (Transient, Scoped, Singleton).
10. Differenza tra validazione server-side e client-side. Quale è obbligatoria?
11. Differenza tra autenticazione e autorizzazione.
12. Come funziona l'autenticazione basata su cookie? Pro e contro?
13. Cos'è un JWT? Come è strutturato (header, payload, firma)?
14. Perché si usano Access Token + Refresh Token?
15. Cookie vs JWT: 2 differenze chiave.
16. Cos'è OAuth 2.0? A cosa serve?
17. Come gestisci i segreti (password, chiavi) in un progetto ASP.NET Core?
