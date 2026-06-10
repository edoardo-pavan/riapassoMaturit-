# GIORNO 4 — DOCKER + FRONTEND + SICUREZZA WEB

> Ritmo leggero. Docker l'hai già usato, frontend lo conosci, sicurezza è teoria. L'AI la trovi nel file di Ed. Civica separato.

---

## 1. DOCKER — RIPASSO GUIDATO

### Perché Docker per un developer

- **Ambiente identico** ovunque: "sulla mia macchina funziona" non esiste più
- **Isolamento**: ogni container ha le sue dipendenze, senza conflitti
- **Riproducibilità**: il Dockerfile descrive ESATTAMENTE l'ambiente
- **Facilità di deployment**: stesso container in dev, staging, produzione
- **Microservizi**: un container per servizio, orchestrazione con Compose

### Comandi essenziali

```bash
docker pull mysql:8.0                    # scarica immagine
docker images                             # elenca immagini locali
docker run -d --name mio-mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -p 3306:3306 mysql:8.0                 # avvia container (detached)
docker ps                                 # container in esecuzione
docker ps -a                              # tutti i container
docker stop mio-mysql                     # ferma
docker start mio-mysql                    # riavvia
docker rm mio-mysql                       # rimuove container
docker rmi mysql:8.0                      # rimuove immagine
docker exec -it mio-mysql bash            # entra nel container
docker logs mio-mysql                     # guarda i log
```

### Docker Networking

| Tipo | Descrizione |
|---|---|
| **Default bridge** | Rete predefinita. I container comunicano solo via IP (non via hostname). |
| **User-defined bridge** | Rete creata da te. I container comunicano via **hostname** (= nome container). Quella che si usa sempre. |

```bash
docker network create mia-rete
docker run -d --name db --network mia-rete mysql:8.0
docker run -d --name app --network mia-rete -p 8080:80 mia-app
# 'app' può raggiungere 'db' semplicemente con l'hostname 'db'!
```

### Port mapping

Espone una porta del container sull'host.

```
docker run -p 8080:80 mia-app
            └── host : container
```

- L'app è accessibile su `http://localhost:8080`
- Dentro il container, l'app ascolta sulla porta `80`

### Volumi e Bind Mounts

I container sono **effimeri**: quando li cancelli, i dati spariscono. Per la persistenza:

| Tipo | Descrizione | Comando |
|---|---|---|
| **Volume** | Gestito da Docker, in `/var/lib/docker/volumes/` | `-v nome_volume:/path/container` |
| **Bind mount** | Monta una cartella dell'host nel container | `-v /host/path:/container/path` |

```bash
# Volume: Docker lo gestisce
docker volume create dati-mysql
docker run -v dati-mysql:/var/lib/mysql mysql:8.0

# Bind mount: cartella del tuo PC
docker run -v $(pwd)/mio-progetto:/app mia-app
```

**Caso pratico — MySQL con volume:**
```bash
docker run -d --name db \
  --network mia-rete \
  -v dati-mysql:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

### Dockerfile

File di istruzioni per costruire un'immagine.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MiaApp.dll"]
```

### Docker Compose

Orchestra più container insieme. File `docker-compose.yml`:

```yaml
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: mio_db
    volumes:
      - dati-mysql:/var/lib/mysql
    networks:
      - rete-app

  app:
    build: .
    ports:
      - "5000:80"
    depends_on:
      - db
    networks:
      - rete-app

volumes:
  dati-mysql:

networks:
  rete-app:
    driver: bridge
```

**Comandi Compose:**
```bash
docker-compose up -d        # avvia tutto
docker-compose down          # ferma e rimuove
docker-compose logs          # log di tutti i servizi
```

**Perché i servizi comunicano tra loro**: in Compose, il nome del servizio (`db`, `app`) è l'hostname. `app` si connette a MySQL con `server=db`.

### Docker Security (per l'orale)

- **MAI eseguire container come root**: specifica `USER` nel Dockerfile
- **Least privilege**: il container deve avere solo i permessi che servono
- **Secrets**: non mettere password nel Dockerfile. Usa variabili d'ambiente, file `.env`, Docker Secrets
- **Immagini ufficiali**: usa immagini da Docker Hub verificate (`mysql:8.0`, non `mysql-hacker:latest`)
- **Scanning vulnerabilità**: `docker scan` per controllare vulnerabilità note
- **Read-only filesystem**: `docker run --read-only` impedisce modifiche al filesystem del container
- **Risorse limitate**: `--memory`, `--cpus` per evitare che un container monopolizzi l'host

---

## 2. FRONTEND — RIPASSO RAPIDO

### HTML — tag essenziali

```html
<h1>Titolo</h1>                    <!-- heading -->
<p>Paragrafo</p>                   <!-- paragrafo -->
<a href="/pagina">Link</a>         <!-- collegamento -->
<img src="foto.jpg" alt="desc">   <!-- immagine -->
<ul><li>Elemento</li></ul>        <!-- lista -->
<div>Blocco</div>                  <!-- contenitore generico -->
<span>In linea</span>              <!-- contenitore inline -->

<form method="POST" action="/api/endpoint">
    <input type="text" name="nome" required>
    <input type="email" name="email">
    <input type="password" name="password">
    <input type="number" name="eta" min="0" max="120">
    <textarea name="messaggio"></textarea>
    <select name="categoria">
        <option value="1">Cat 1</option>
    </select>
    <button type="submit">Invia</button>
</form>
```

### CSS — concetti essenziali

```css
/* Selettori */
p { color: blue; }                /* elemento */
.classe { font-size: 16px; }      /* classe */
#id { margin: 10px; }             /* id */

/* Box model */
.box {
    width: 200px;
    padding: 10px;      /* spazio interno */
    border: 1px solid #ccc;
    margin: 20px;       /* spazio esterno */
}

/* Flexbox */
.container {
    display: flex;
    justify-content: center;  /* allineamento orizzontale */
    align-items: center;      /* allineamento verticale */
    gap: 20px;
}

/* Grid */
.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);  /* 3 colonne uguali */
    gap: 15px;
}

/* Responsive: Media Query */
@media (max-width: 768px) {
    .grid { grid-template-columns: 1fr; }  /* 1 colonna su mobile */
}
```

### JavaScript — essenziale per fetch API

```javascript
// Selezionare elementi
const btn = document.querySelector('#mio-bottone');
const nome = document.querySelector('#nome');

// Event listener
btn.addEventListener('click', () => {
    console.log('cliccato!');
});

// Fetch API — chiamata al backend
async function caricaClienti() {
    const response = await fetch('/api/clienti');
    const clienti = await response.json();
    console.log(clienti);
}

// POST con fetch
async function creaCliente() {
    const response = await fetch('/api/clienti', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ nome: 'Mario', email: 'mario@test.it' })
    });
    if (response.ok) {
        const nuovo = await response.json();
        console.log('Creato:', nuovo);
    }
}
```

### Bootstrap (cenni)

Framework CSS. Basta aggiungere classi predefinite:

```html
<div class="container">
    <div class="row">
        <div class="col-md-6">Colonna 1</div>
        <div class="col-md-6">Colonna 2</div>
    </div>
</div>
<button class="btn btn-primary">Clicca</button>
<table class="table table-striped">...</table>
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">...</nav>
```

### Responsive Design

Tecnica per adattare il layout a qualsiasi schermo. Strumenti:
- **Media Query** CSS: `@media (max-width: 768px)`
- **Flexbox/Grid**: layout flessibili
- **Viewport meta tag**: `<meta name="viewport" content="width=device-width">`
- **Unità relative**: `%`, `vw`, `vh`, `rem` invece di `px` fissi

---

## 3. SICUREZZA WEB

### CSRF (Cross-Site Request Forgery)

**Attacco**: un sito malevolo fa fare al browser dell'utente una richiesta a un sito dove l'utente è autenticato, sfruttando i cookie di sessione.

```
L'utente è loggato su banca.it (cookie di sessione presente).
L'utente visita sito-hacker.com.
sito-hacker.com ha un form nascosto che invia una POST a banca.it/trasferisci.
Il browser allega automaticamente il cookie di sessione.
La banca pensa che la richiesta sia legittima.
```

**Mitigazione**:
- Token anti-CSRF: un token unico generato dal server, incluso nel form e verificato a ogni POST
- SameSite cookie: `SameSite=Strict` o `Lax` impedisce l'invio del cookie da siti terzi
- Header `Origin` / `Referer`: verificare che la richiesta provenga dal sito giusto

### XSS (Cross-Site Scripting)

**Attacco**: un hacker inietta JavaScript malevolo in una pagina web, che viene eseguito nel browser della vittima.

```
Un utente inserisce in un commento: <script>rubareCookie()</script>
Se il server non sanitizza l'input, il commento viene visualizzato
e lo script si esegue nel browser di chiunque veda la pagina.
```

**Mitigazione**:
- **Output encoding**: trasformare `<` in `&lt;`, `>` in `&gt;` → il tag `<script>` diventa testo innocuo
- **Content Security Policy (CSP)**: header HTTP che limita quali script possono essere eseguiti
- **Input validation**: validare e sanitizzare TUTTO l'input utente
- Framework moderni (React, ASP.NET Core Razor) lo fanno automaticamente

### SQL Injection

**Attacco**: l'hacker inserisce codice SQL nei campi di input, manipolando la query.

```
Input utente nel campo nome: ' OR '1'='1

Query costruita con concatenazione:
"SELECT * FROM Utenti WHERE nome = '" + input + "' AND password = '" + pass + "'"

Diventa:
SELECT * FROM Utenti WHERE nome = '' OR '1'='1' AND password = 'qualsiasi'
                                   └──────── sempre vero! ─────────┘
Risultato: l'hacker entra senza password.
```

**Mitigazione**:
- **Prepared Statement / Query parametrizzate**: mai concatenare stringhe per costruire query SQL
- **EF Core / LINQ**: generano automaticamente query parametrizzate → protezione integrata
- Se usi SQL diretto in EF Core, usa `$"..."` (FormattableString), MAI `+`
- **Input validation**: validare i tipi di dato (un'email non può contenere `' OR '1`)

```csharp
// ✅ SICURO: FormattableString → EF Core parametrizza automaticamente
_db.Clienti.FromSql($"SELECT * FROM Clienti WHERE Nome = {nome}");

// ❌ INSICURO: concatenazione → SQL Injection!
_db.Clienti.FromSqlRaw("SELECT * FROM Clienti WHERE Nome = '" + nome + "'");
```

---

## DOMANDE PER L'AUTOVERIFICA

1. Perché Docker è utile per uno sviluppatore? (almeno 3 motivi)
2. Comandi Docker: spiega `docker run`, `docker ps`, `docker exec`, `docker-compose up`.
3. Differenza tra default bridge e user-defined bridge in Docker networking. Quale si usa in pratica?
4. Differenza tra volume e bind mount. Quando usi l'uno o l'altro?
5. A cosa serve un Dockerfile? Quali sono le istruzioni principali?
6. Cosa fa Docker Compose? Come comunicano tra loro i container in Compose?
7. Elenca almeno 4 best practice di sicurezza Docker.
8. Scrivi un form HTML con campi nome, email, messaggio e bottone submit.
9. Cos'è Flexbox? A cosa serve `justify-content` e `align-items`?
10. Cos'è una Media Query? Come la usi per rendere una pagina responsive?
11. A cosa serve `fetch()` in JavaScript? Scrivi un esempio di GET e uno di POST.
12. Cos'è un attacco CSRF? Come funziona? Come ti proteggi?
13. Cos'è un attacco XSS? Come funziona? Come ti proteggi?
14. Cos'è la SQL Injection? Come funziona? Perché EF Core e LINQ proteggono?
15. Perché `FromSql($"...{var}")` protegge dalla SQL Injection mentre la concatenazione no?
16. Cos'è il GDPR? Quali implicazioni ha per un sito web che raccoglie dati utenti?
