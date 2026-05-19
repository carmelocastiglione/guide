# Guida: Installare e Usare Metabase con MongoDB su Docker

Questa guida ti mostra come installare, configurare e utilizzare Metabase con MongoDB su Docker. Metabase è una piattaforma open source per la visualizzazione e l'analisi dei dati, con **supporto nativo a MongoDB**.

## Sommario

1. [Cos'è Metabase](#cosè-metabase)
2. [Prerequisiti](#prerequisiti)
3. [Setup Rapido con Docker Compose](#setup-rapido-con-docker-compose)
4. [Avvio e Configurazione Iniziale](#avvio-e-configurazione-iniziale)
5. [Connessione a MongoDB](#connessione-a-mongodb)
6. [Creazione di Dashboard e Query](#creazione-di-dashboard-e-query)
7. [Comandi Docker Compose](#comandi-docker-compose)
8. [Troubleshooting](#troubleshooting)
9. [Tips & Tricks](#tips--tricks)

---

## Cos'è Metabase

**Metabase** è una piattaforma open source che ti permette di:

- ✅ Visualizzare dati da MongoDB **senza plugin**, supporto **nativo**
- ✅ Creare dashboard interattive
- ✅ Scrivere query in linguaggio naturale o MongoDB Query Language
- ✅ Condividere insights con il team
- ✅ Pianificare esportazioni automatiche
- ✅ Gestire utenti e permessi

**Vantaggi rispetto a Grafana per MongoDB**:
- Supporto nativo (non serve licenza Enterprise)
- Interface molto user-friendly
- Perfetto per visualizzare dati applicativi, non solo metriche
- Query builder intuitivo

---

## Prerequisiti

- **Docker Desktop** installato e funzionante
  - [Scarica Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Verifica: `docker --version`
- **Docker Compose** (incluso in Docker Desktop)
  - Verifica: `docker-compose --version`
- **Browser web** moderno (Chrome, Firefox, Safari, Edge)

---

## Setup Rapido con Docker Compose

### 1. Crea il file `docker-compose.yml`

Crea una cartella per il progetto e al suo interno crea il file `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Metabase
  metabase:
    image: metabase/metabase:latest
    container_name: metabase
    restart: always
    ports:
      - "3000:3000"
    environment:
      MB_DB_TYPE: h2
      JAVA_TIMEZONE: UTC
    volumes:
      - metabase_data:/metabase-data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 5
    depends_on:
      - mongodb

  # MongoDB
  mongodb:
    image: mongo:7.0
    container_name: mongodb-data
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
    volumes:
      - mongodb_data:/data/db
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  metabase_data:
    driver: local
  mongodb_data:
    driver: local
```

**Spiegazione**:
- **Metabase** usa `MB_DB_TYPE: h2` (database H2 embedded)
- **MongoDB** su porta 27017
- Entrambi hanno volumi persistenti
- Health checks per verificare che i servizi siano pronti

### 2. Avvia i servizi

Dalla cartella dove hai creato `docker-compose.yml`:

```bash
docker-compose up -d
```

Verifica che i container siano in esecuzione:

```bash
docker-compose ps
```

Dovresti vedere:
```
NAME           STATUS           PORTS
metabase       Up (healthy)     0.0.0.0:3000->3000/tcp
mongodb-data   Up (healthy)     0.0.0.0:27017->27017/tcp
```

---

## Avvio e Configurazione Iniziale

### 1. Accedi a Metabase

Apri il browser e vai a: **http://localhost:3000**

### 2. Setup wizard

Metabase ti guiderà attraverso i seguenti step:

1. **Welcome**: Clicca su **Let's get started**
2. **Language**: Seleziona la lingua (English o altre)
3. **User Information**: Crea l'account admin
   - Email: `admin@example.com`
   - Password: Scegli una password robusta
4. **Company Information**: Nome della tua organizzazione (opzionale)
5. **Database Connection**: Configureremo MongoDB dopo

Per ora, salta la connessione al database (clicca su **I'll add my data later**) e completa il wizard.

---

## Connessione a MongoDB

### 1. Accedi a Metabase

Vai a http://localhost:3000 e accedi con l'account admin che hai creato.

### 2. Aggiungi MongoDB come data source

1. Clicca sull'icona **Settings** (ingranaggio) in alto a destra
2. Vai su **Admin settings** → **Databases** → **Add a database**

### 3. Seleziona MongoDB

Seleziona **MongoDB** dall'elenco dei database supportati.

### 4. Configura la connessione

Compila i seguenti campi:

```
Name:                       MongoDB
Host:                       mongodb-data
Port:                       27017
Username:                   root
Password:                   root
Authentication Database:    admin
```

**Note**:
- **Host**: Usa il nome del container `mongodb-data` (non `localhost` o `127.0.0.1`)
- **Port**: 27017 (porta standard MongoDB)
- **Authentication Database**: `admin` (il database dove è stato creato l'utente root)

### 5. Testa la connessione

Clicca su **Validate your connection**. Se tutto è configurato correttamente, vedrai "Successfully connected to the database".

### 6. Salva la connessione

Clicca su **Save**. Metabase inizierà a scansionare i database e le collection di MongoDB.

---

## Creazione di Dashboard e Query

### 1. Creare una prima query

1. Dal menu principale, clicca su **+ New** in alto a sinistra
2. Seleziona **Question** (o **Query**)
3. Seleziona **MongoDB** come data source

### 2. Query builder - Interfaccia semplice

Metabase offre due modi per scrivere query:

#### Metodo 1: Query Builder (interface GUI)

```
1. Seleziona la Collection
2. Usa i filtri e aggregazioni visivamente
3. Configura visualizzazione (grafico, tabella, etc.)
4. Salva la query
```

Esempio: Contare il numero di documenti in una collection:
```
Collection: users
Summarize: Count
```

#### Metodo 2: Native Query (MongoDB Query Language)

Se preferisci scrivere query MongoDB nativi:

1. Clicca su **Native query**
2. Scrivi la tua query MongoDB

Esempio di query nativa:
```javascript
db.users.aggregate([
  {
    $group: {
      _id: "$country",
      count: { $sum: 1 }
    }
  },
  {
    $sort: { count: -1 }
  },
  {
    $limit: 10
  }
])
```

### 3. Visualizzazione

Dopo aver scritto la query:

1. Clicca su **Visualize**
2. Scegli il tipo di visualizzazione:
   - **Table**: Visualizza i dati in tabella
   - **Number**: Mostra un numero singolo
   - **Bar chart**: Grafico a barre
   - **Line chart**: Grafico a linee
   - **Pie chart**: Grafico a torta
   - **Scatter**: Grafico a dispersione
   - Ecc.

3. Personalizza colori, etichette, etc.

### 4. Salvare una query

Clicca su **Save** in alto a destra. La query verrà salvata e potrai riutilizzarla in dashboard.

### 5. Creare un Dashboard

1. Clicca su **+ New** → **Dashboard**
2. Dai un titolo al dashboard
3. Clicca su **Create dashboard**
4. Nel dashboard, clicca su **+ Add a question**
5. Seleziona una delle query che hai creato
6. Aggiungi altre query
7. Clicca su **Save**

---

## Comandi Docker Compose

### Avviare i servizi

```bash
# Avvia in background
docker-compose up -d

# Avvia in foreground (vedi i log)
docker-compose up
```

### Visualizzare log

```bash
# Log di tutti i servizi
docker-compose logs -f

# Log di Metabase
docker-compose logs -f metabase

# Log di MongoDB
docker-compose logs -f mongodb-data
```

### Fermare i servizi

```bash
# Ferma i servizi (mantiene i dati)
docker-compose stop

# Ferma e rimuove i container
docker-compose down

# Ferma, rimuove i container E i volumi (cancella i dati!)
docker-compose down -v
```

### Riavviare i servizi

```bash
docker-compose restart

# Un servizio specifico
docker-compose restart metabase
docker-compose restart mongodb-data
```

### Accedere alla shell di un container

```bash
# Metabase
docker-compose exec metabase /bin/sh

# MongoDB
docker-compose exec mongodb-data mongosh
```

### Aggiornare le immagini

```bash
# Scarica le versioni più recenti
docker-compose pull

# Riavvia i servizi con le nuove immagini
docker-compose up -d
```

---

## Troubleshooting

### Metabase non si connette a MongoDB

**Problema**: Connection refused o timeout

**Soluzione**:
1. Verifica che MongoDB sia in esecuzione: `docker-compose ps`
2. Controlla i log: `docker-compose logs mongodb-data`
3. Assicurati di usare il **nome del container** (`mongodb-data`), non `localhost`
4. Verifica che le credenziali siano corrette (root/root)

```bash
# Testa la connessione da dentro il container Metabase
docker-compose exec metabase curl -v mongodb-data:27017
```

### Errore: "Authentication failed"

**Problema**: Username/password errati oppure authentication database sbagliato

**Soluzione**:
1. Verifica che le credenziali siano `root` / `root`
2. L'**Authentication Database** DEVE essere `admin`
3. Se hai cambiato le credenziali in `docker-compose.yml`, usale in Metabase

```bash
# Testa le credenziali da dentro il container MongoDB
docker-compose exec mongodb-data mongosh -u root -p root --authenticationDatabase admin
```

### Metabase non si avvia

**Problema**: Errore di avvio o container che si riavvia continuamente

**Soluzione**:
```bash
# Visualizza i log dettagliati
docker-compose logs metabase

# Se il database H2 è corrotto, elimina il volume e ricrea
docker-compose down -v
docker-compose up -d
```

### Le collection di MongoDB non compaiono

**Problema**: Metabase non scansiona le collection

**Soluzione**:
1. Vai in **Settings** → **Admin settings** → **Databases**
2. Seleziona MongoDB
3. Clicca su **Rescan database tables**
4. Aspetta che la scansione termini

Oppure, ricrea la connessione:
```bash
# Riavvia Metabase
docker-compose restart metabase
```

### Porta 3000 già in uso

**Problema**: Error binding to port 3000

**Soluzione**:
```bash
# Uccidi il processo in ascolto sulla porta (Windows)
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Oppure, cambia la porta nel docker-compose.yml
ports:
  - "3001:3000"  # Accedi da http://localhost:3001
```

### Dati MongoDB non persistono

**Problema**: I dati scompaiono quando fermi i container

**Soluzione**: Assicurati che il volume MongoDB sia configurato correttamente:

```yaml
volumes:
  - mongodb_data:/data/db
```

Se l'hai già cancellato, ricrea:
```bash
docker-compose down
docker-compose up -d
```

---

## Tips & Tricks

### Backup e Restore

#### Backup di Metabase

```bash
# Backup del database H2 di Metabase
docker cp metabase:/metabase-data ./metabase-backup
```

#### Backup di MongoDB

```bash
# Dump di MongoDB
docker-compose exec -T mongodb-data mongodump --archive=/tmp/backup.archive --username=root --password=root --authenticationDatabase=admin
docker cp mongodb-data:/tmp/backup.archive ./mongodb-backup.archive

# Restore di MongoDB
docker cp ./mongodb-backup.archive mongodb-data:/tmp/backup.archive
docker-compose exec -T mongodb-data mongorestore --archive=/tmp/backup.archive --username=root --password=root --authenticationDatabase=admin
```

### Creare dati di test in MongoDB

```bash
# Accedi a MongoDB
docker-compose exec mongodb-data mongosh -u root -p root --authenticationDatabase admin

# Nel prompt mongosh, crea una collection con dati
db.products.insertMany([
  { name: "Laptop", price: 1200, stock: 5 },
  { name: "Mouse", price: 25, stock: 100 },
  { name: "Keyboard", price: 75, stock: 50 }
])

# Verifica i dati
db.products.find()
```

### Gestione utenti in Metabase

1. Vai in **Settings** → **Admin settings** → **People**
2. Clicca su **Invite someone**
3. Inserisci email e seleziona il ruolo:
   - **Admin**: Accesso completo a tutte le impostazioni
   - **Normal**: Accesso alle query e dashboard
   - **Read-only**: Solo visualizzazione

### Esportare dati da Metabase

Dalla query o dashboard:
1. Clicca su **Download** (icona freccia in basso)
2. Scegli il formato: CSV, JSON, Excel, ecc.

### Pianificare esportazioni automatiche

1. Salva una query
2. Clicca su **Schedules and alerts**
3. Configura quando esportare (giornaliero, settimanale, ecc.)
4. Inserisci un'email per ricevere il file automaticamente

### Performance: Aumentare la memoria di Metabase

Se Metabase è lento, aumenta la memoria Java:

```yaml
services:
  metabase:
    image: metabase/metabase:latest
    environment:
      JAVA_OPTS: "-Xmx2g"  # 2GB di memoria
```

Poi riavvia:
```bash
docker-compose down
docker-compose up -d
```

---

## Risorse Utili

- **Documentazione Metabase**: https://metabase.com/docs/
- **MongoDB Query Language**: https://docs.mongodb.com/manual/reference/operator/
- **Docker Images**:
  - Metabase: https://hub.docker.com/r/metabase/metabase
  - MongoDB: https://hub.docker.com/_/mongo
- **Metabase Community**: https://discourse.metabase.com/
- **Esempio di Dashboard**: https://metabase.com/dashboards/samples
