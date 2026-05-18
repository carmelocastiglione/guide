# Guida: Installare e Usare MongoDB su Docker

Questa guida ti mostra come installare, configurare e utilizzare MongoDB utilizzando Docker e VSCode.

## Sommario
1. [Prerequisiti](#prerequisiti)
2. [Setup Rapido con Docker Compose](#setup-rapido-con-docker-compose)
3. [Comandi Docker Compose](#comandi-docker-compose)
4. [Connessione a MongoDB](#connessione-a-mongodb)
5. [Integrazione con VSCode](#integrazione-con-vscode)
6. [Comandi MongoDB Utili](#comandi-mongodb-utili) 
7. [Troubleshooting](#troubleshooting)

## Prerequisiti

- **Docker Desktop** installato e funzionante
  - [Scarica Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Verifica l'installazione: `docker --version`
- **VSCode** (opzionale, per gestione migliore)
  - [Scarica VSCode](https://code.visualstudio.com/)
  - Estensione consigliata: **MongoDB for VS Code**

## Setup Rapido con Docker Compose

Docker Compose è il metodo **consigliato e predefinito** per gestire MongoDB con persistenza automatica dei dati.

### 1. Crea il file `docker-compose.yml`

Nella cartella del tuo progetto, crea un file denominato `docker-compose.yml`:

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
    volumes:
      - mongodb_data:/data/db
    networks:
      - mongodb-network
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 5s
      timeout: 5s
      retries: 15
      start_period: 15s

  # Opzionale: Mongo Express (interfaccia web per gestire MongoDB)
  mongo-express:
    image: mongo-express:latest
    container_name: mongo-express
    restart: on-failure
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: root
      ME_CONFIG_MONGODB_URL: mongodb://root:root@mongodb:27017/?authSource=admin
      ME_CONFIG_MONGODB_ENABLE_ADMIN: 'true'
      ME_CONFIG_SITE_BASEURL: /
    networks:
      - mongodb-network
    depends_on:
      mongodb:
        condition: service_healthy

volumes:
  mongodb_data:
    driver: local

networks:
  mongodb-network:
    driver: bridge
```

### 2. Avviare MongoDB con Docker Compose

Apri il terminale nella cartella dove si trova `docker-compose.yml` ed esegui:

```powershell
docker-compose up -d
```

**Cosa accade:**
- ✅ MongoDB si avvia in background
- ✅ I dati sono automaticamente salvati nel volume `mongodb_data`
- ✅ Mongo Express è disponibile su `http://localhost:8081`
- ✅ MongoDB è raggiungibile su `localhost:27017`

### 3. Verificare che tutto sia funzionante

```powershell
docker-compose ps
```

Dovresti vedere sia `mongodb` che `mongo-express` in status `Up`.

### 4. Accedere a Mongo Express (Interfaccia Web)

Apri il browser e vai a:
```
http://localhost:8081
```

**Username e Password di Mongo Express (non MongoDB!)**
- Username: `admin`  
- Password: `pass`

Questi sono i dati di default per accedere all'interfaccia web di Mongo Express.
MongoDB usa credenziali separate (`root:root`).

**Se vuoi personalizzare le credenziali di Mongo Express**, aggiungi queste variabili nel `docker-compose.yml`:

```yaml
mongo-express:
  environment:
    ME_CONFIG_BASICAUTH: 'true'
    ME_CONFIG_BASICAUTH_USERNAME: tuousername
    ME_CONFIG_BASICAUTH_PASSWORD: tuapassword
```

Qui puoi visualizzare, creare e modificare i database in modo grafico.

## Comandi Docker Compose

### Avviare i servizi

```powershell
docker-compose up -d
```

### Arrestare i servizi

```powershell
docker-compose stop
```

### Avviare di nuovo i servizi

```powershell
docker-compose start
```

### Stoppare ed eliminare i container (ma i dati rimangono)

```powershell
docker-compose down
```

### Stoppare e eliminare TUTTO (container, volumi, reti) - CANCELLA I DATI!

```powershell
docker-compose down -v
```

### Visualizzare i log

```powershell
# Log di entrambi i servizi
docker-compose logs -f

# Log solo di MongoDB
docker-compose logs -f mongodb

# Log solo di Mongo Express
docker-compose logs -f mongo-express
```

### Accedere a MongoDB da terminale

```powershell
docker-compose exec mongodb mongosh --username root --password root --authenticationDatabase admin
```

### Ricostruire i container (dopo modifica docker-compose.yml)

```powershell
docker-compose up -d --force-recreate
```

## Connessione a MongoDB

### Connessione da terminale

Accedi alla shell MongoDB interattiva:

```powershell
docker exec -it mongodb mongosh --username root --password root --authenticationDatabase admin
```

### String di connessione

Usa questo formato per connetterti da applicazioni:

```
mongodb://root:root@localhost:27017/?authSource=admin
```

Se non usi autenticazione:
```
mongodb://localhost:27017
```

## Integrazione con VSCode

### 1. Installare l'estensione MongoDB

1. Apri VSCode
2. Vai su **Estensioni** (Ctrl+Shift+X)
3. Cerca: `MongoDB for VS Code`
4. Clicca su **Installa**

### 2. Connettere VSCode a MongoDB

1. Apri la sezione **MongoDB** nella sidebar
2. Clicca su **Add Connection**
3. Seleziona **Connect with connection string**
4. Incolla la stringa di connessione:
   ```
   mongodb://root:root@localhost:27017/?authSource=admin
   ```
5. Clicca su **Connect**

### 3. Usare MongoDB Explorer in VSCode

Una volta connesso:
- Visualizza **Databases** e le tue collezioni
- Fai clic destro su una collezione per:
  - Visualizzare i documenti
  - Aggiungere nuovi documenti
  - Editare documenti
  - Eliminare documenti
- Usa la **Playgrounds** per scrivere query MongoDB

### Creare un MongoDB Playgrounds (VSCode)

1. Clicca su **Create MongoDB Playgrounds**
2. Scrivi query MongoDB (sintassi: `db.nomeCollezione.metodo()`):

```javascript
// Connettiti al database
use('guida_mongodb');

// Inserisci un documento
db.utenti.insertOne({
  nome: "Mario",
  email: "mario@example.com",
  eta: 25
});

// Leggi tutti i documenti
db.utenti.find({});

// Leggi un documento specifico
db.utenti.findOne({ nome: "Mario" });

// Aggiorna un documento
db.utenti.updateOne(
  { nome: "Mario" },
  { $set: { eta: 26 } }
);

// Elimina un documento
db.utenti.deleteOne({ nome: "Mario" });
```

## Comandi MongoDB Utili

### Importante: Una Sola Sintassi

**In entrambi i contesti (MongoDB Shell e MongoDB Playgrounds) la sintassi è la stessa:**

```javascript
// Sintassi unica per SHELL e PLAYGROUNDS
db.nomeCollezione.metodo()

// Esempi:
db.utenti.insertOne({...})
db.utenti.find({})
db.utenti.updateOne({...})
```

**Non usare `db.collection()` - funziona solo con gli array aggregation nei playgrounds, non per le operazioni normali.**

### Accedere al database

```javascript
// SHELL MONGODB (mongosh)
// Usa un database (lo crea se non esiste)
use('mio_database')

// Vedi il database corrente
db

// Elenca tutti i database
show databases

// Elenca le collezioni del database corrente
show collections
```

### Operazioni CRUD - MongoDB Shell (mongosh)

**Questa è la sintassi corretta quando usi il terminale o `docker-compose exec`:**

**Puoi usare due sintassi equivalenti:**

```javascript
// SINTASSI 1: Accesso diretto (consigliato)
db.utenti.insertOne({...})

// SINTASSI 2: Con getCollection()
db.getCollection('utenti').insertOne({...})
```

#### Esempi con sintassi diretta:

```javascript
// CREATE - Inserisci un documento
db.utenti.insertOne({
  nome: "Luigi",
  email: "luigi@example.com",
  creato: new Date()
})

// CREATE - Inserisci più documenti
db.utenti.insertMany([
  { nome: "Anna", email: "anna@example.com" },
  { nome: "Paolo", email: "paolo@example.com" }
])

// READ - Leggi tutti
db.utenti.find({})

// READ - Leggi con filtro
db.utenti.find({ nome: "Luigi" })

// READ - Un documento specifico
db.utenti.findOne({ nome: "Luigi" })

// UPDATE - Modifica
db.utenti.updateOne(
  { nome: "Luigi" },
  { $set: { email: "luigi_nuovo@example.com" } }
)

// UPDATE - Modifica molti
db.utenti.updateMany(
  {},
  { $set: { attivo: true } }
)

// DELETE - Elimina uno
db.utenti.deleteOne({ nome: "Luigi" })

// DELETE - Elimina molti
db.utenti.deleteMany({ attivo: false })
```

#### Esempi con `getCollection()`:

Se il nome della collezione ha caratteri speciali o spazi, usa `getCollection()`:

```javascript
// Per nomi con spazi o caratteri speciali
db.getCollection('utenti-attivi').find({})
db.getCollection('my users').insertOne({nome: "Test"})

// Funziona anche per nomi normali
db.getCollection('utenti').insertOne({
  nome: "Luigi",
  email: "luigi@example.com"
})
```

### Operazioni CRUD - MongoDB Playgrounds (VSCode)

**La sintassi nei Playgrounds è la stessa della shell: `db.nomeCollezione.metodo()`**

```javascript
use('mio_database');

// CREATE
db.utenti.insertOne({
  nome: "Luigi",
  email: "luigi@example.com",
  creato: new Date()
})

// READ - Tutti i documenti
db.utenti.find({})

// READ - Con filtro
db.utenti.find({ nome: "Luigi" })

// READ - Un documento
db.utenti.findOne({ nome: "Luigi" })

// UPDATE
db.utenti.updateOne(
  { nome: "Luigi" },
  { $set: { email: "luigi_nuovo@example.com" } }
)

// DELETE
db.utenti.deleteOne({ nome: "Luigi" })
```

### Indici e Performance - MongoDB Shell (mongosh)

```javascript
// Crea un indice
db.utenti.createIndex({ email: 1 })

// Visualizza gli indici
db.utenti.getIndexes()

// Elimina un indice
db.utenti.dropIndex('email_1')
```

## Troubleshooting

### Errore "db.collection is not a function"

**Sintomo:** Ricevi questo errore quando provi a eseguire `db.collection('utenti').insertOne()`

**Causa:** Stai usando una sintassi non valida. Sia in MongoDB Shell che nei Playgrounds, la sintassi corretta è `db.nomeCollezione.metodo()`, non `db.collection()`.

**Soluzione:**
```javascript
// ❌ SBAGLIATO
db.collection('utenti').insertOne({...})

// ✅ CORRETTO (sia in Shell che in Playgrounds)
db.utenti.insertOne({...})
```

Usa sempre `db.nomeCollezione.metodo()` indipendentemente da dove stai scrivendo il codice.

### Errore di autenticazione su Mongo Express (browser)

**Sintomo:** La finestra di login di Mongo Express richiede continuamente username e password.

**Causa:** Confusione tra le credenziali di **Mongo Express** e le credenziali di **MongoDB**.

**Soluzione:**
- **Per accedere all'interfaccia web di Mongo Express** → usa `admin:pass` (credenziali di default)
- **Per connettersi a MongoDB da applicazioni** → usa `root:root` (credenziali MongoDB)

Se vuoi usare credenziali personalizzate per Mongo Express, modifica il `docker-compose.yml`:

```yaml
mongo-express:
  environment:
    ME_CONFIG_BASICAUTH: 'true'
    ME_CONFIG_BASICAUTH_USERNAME: mio_username
    ME_CONFIG_BASICAUTH_PASSWORD: mia_password
    # ... resto della configurazione
```

Poi riavvia i container:
```powershell
docker-compose down
docker-compose up -d
```

### Il container non si avvia

```powershell
# Visualizza i log per errori
docker logs mongodb

# Prova a rimuovere il container e ricrearlo
docker stop mongodb
docker rm mongodb
```

### Errore di connessione da VSCode

- Verifica che il container sia in esecuzione: `docker ps`
- Controlla la stringa di connessione (username, password, porta)
- Assicurati che la porta 27017 non sia occupata: `netstat -ano | findstr :27017`
- Usa le **credenziali MongoDB** (`root:root`), non quelle di Mongo Express

### Permessi di accesso

Se ricevi errori di autenticazione:
```javascript
// Accedi senza autenticazione (dal container)
docker exec -it mongodb mongosh

// Poi accedi al database admin
use admin

// Verifica gli utenti
db.getUsers()
```

### Pulire completamente MongoDB

```powershell
# Arresta i container
docker-compose down

# Rimuovi il volume (⚠️ CANCELLA I DATI!)
docker volume rm mongodb_data

# Ricrea da zero
docker-compose up -d
```

---

Risorse Utili

- [Documentazione ufficiale MongoDB](https://docs.mongodb.com/)
- [Immagine Docker MongoDB](https://hub.docker.com/_/mongo)
- [MongoDB Shell Reference](https://docs.mongodb.com/mongodb-shell/)
- [MongoDB for VS Code](https://www.mongodb.com/products/tools/vs-code)

---

## Riepilogo Quick Start (Docker Compose - Consigliato)

```powershell
# 1. Crea il file docker-compose.yml nella cartella del progetto
#    (Copia il contenuto yaml dalla sezione "Setup Rapido con Docker Compose")

# 2. Avvia MongoDB e Mongo Express
docker-compose up -d

# 3. Verifica che tutto sia attivo
docker-compose ps

# 4. Accedi a Mongo Express nel browser (credenziali di Mongo Express, non MongoDB!)
#    http://localhost:8081
#    Username: admin
#    Password: pass

# 5. Accedi a MongoDB da terminale (usa le credenziali MongoDB)
docker-compose exec mongodb mongosh --username root --password root --authenticationDatabase admin

# 6. Installa l'estensione MongoDB in VSCode e connettiti (credenziali MongoDB)
#    Stringa: mongodb://root:root@localhost:27017/?authSource=admin
```

**Ricorda la differenza:**
- 🌐 **Mongo Express (interfaccia web)** → `admin:pass`
- 📊 **MongoDB (database)** → `root:root`

**I dati sono automaticamente salvati e persistono anche se arresti i container con `docker-compose down`!**

---

Buona fortuna con MongoDB su Docker! 🚀
