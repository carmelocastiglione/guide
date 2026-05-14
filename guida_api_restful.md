# Guida: API RESTful con Node.js, Express e MySQL in Docker

## Indice
1. [Struttura del Progetto](#struttura-del-progetto)
2. [Setup package.json](#setup-packagejson)
3. [Setup MySQL](#setup-mysql)
4. [Dockerfile e Docker Compose](#dockerfile-e-docker-compose)
5. [Endpoint dell'API](#endpoint-dellapi)
6. [Codice app.js](#codice-appjs)
7. [Avvio del Progetto](#avvio-del-progetto)
8. [Test dell'API](#test-dellapi)
9. [Esercizi](#esercizi)

## Struttura del Progetto

La struttura del progetto è organizzata in modo chiaro per separare i file di configurazione, il codice dell'applicazione e i dati del database:

```
api-docker/
├── app.js
├── package.json
├── nodemon.json
├── Dockerfile
├── docker-compose.yml
├── database/
│   └── init.sql
└── .dockerignore
```

## Setup package.json

### Crea package.json

```json
{
  "name": "api-docker",
  "version": "1.0.0",
  "description": "API RESTful con Node.js, Express e MySQL in Docker",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mysql2": "^3.3.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

## Setup MySQL

### Crea database/init.sql

Questo file inizializza il database con tabelle e dati di esempio:

```sql
CREATE DATABASE IF NOT EXISTS scuola_db;
USE scuola_db;

-- Tabella Studenti
CREATE TABLE studenti (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  cognome VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  data_nascita DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabella Corsi
CREATE TABLE corsi (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  docente VARCHAR(100),
  crediti INT,
  semestre INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabella Iscrizioni
CREATE TABLE iscrizioni (
  id INT AUTO_INCREMENT PRIMARY KEY,
  id_studente INT NOT NULL,
  id_corso INT NOT NULL,
  voto DECIMAL(3,1),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_studente) REFERENCES studenti(id),
  FOREIGN KEY (id_corso) REFERENCES corsi(id)
);

-- Dati di esempio: Studenti
INSERT INTO studenti (nome, cognome, email, data_nascita) VALUES
('Marco', 'Rossi', 'marco.rossi@example.com', '2005-03-15'),
('Anna', 'Bianchi', 'anna.bianchi@example.com', '2004-07-22'),
('Luigi', 'Verdi', 'luigi.verdi@example.com', '2005-11-08'),
('Giulia', 'Ferrari', 'giulia.ferrari@example.com', '2004-05-30'),
('Paolo', 'Moretti', 'paolo.moretti@example.com', '2005-09-12');

-- Dati di esempio: Corsi
INSERT INTO corsi (nome, docente, crediti, semestre) VALUES
('Matematica', 'Prof. Esposito', 6, 1),
('Informatica', 'Prof. Conti', 9, 1),
('Fisica', 'Prof. Russo', 6, 2),
('Inglese', 'Prof. Gallo', 3, 1),
('Storia', 'Prof. Giordano', 6, 2);

-- Dati di esempio: Iscrizioni
INSERT INTO iscrizioni (id_studente, id_corso, voto) VALUES
(1, 1, 28),
(1, 2, 30),
(2, 1, 25),
(2, 4, 27),
(3, 2, 29),
(3, 3, 26),
(4, 1, 30),
(4, 5, 28),
(5, 2, 27);
```

## Dockerfile e Docker Compose

### Crea Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

ENV NODE_ENV=development

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### Crea .dockerignore

```
node_modules
npm-debug.log
.git
.gitignore
.env
```

### Crea nodemon.json

Questo file configura nodemon per monitorare correttamente i file in Docker:

```json
{
  "watch": ["app.js"],
  "ext": "js",
  "delay": 1000,
  "env": {
    "NODE_ENV": "development"
  }
}
```

### Crea docker-compose.yml

```yaml
version: '3.8'

services:
  # Servizio MySQL
  db:
    image: mysql:8.0
    container_name: scuola_db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: scuola_db
    ports:
      - "3306:3306"
    volumes:
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
      - db_data:/var/lib/mysql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  # Servizio API Node.js
  api:
    build: .
    container_name: scuola_api
    environment:
      NODE_ENV: development
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: root
      DB_NAME: scuola_db
      PORT: 3000
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    command: npm run dev

  # Servizio PHPMyAdmin per gestione database
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: scuola_phpmyadmin
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: root
      PMA_DATABASE: scuola_db
    ports:
      - "8888:80"
    depends_on:
      - db
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```

**Accesso a PHPMyAdmin:**
- URL: `http://localhost:8888`
- Username: `root`
- Password: `root`

## Endpoint dell'API

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | `/api/studenti` | Lista tutti gli studenti |
| GET | `/api/studenti/:id` | Ottieni uno studente |
| POST | `/api/studenti` | Crea nuovo studente |
| PUT | `/api/studenti/:id` | Aggiorna studente |
| DELETE | `/api/studenti/:id` | Cancella studente |
| GET | `/api/corsi` | Lista tutti i corsi |
| GET | `/api/iscrizioni/:id_studente` | Corsi di uno studente |
| POST | `/api/iscrizioni` | Iscrivi studente a corso |
| GET | `/health` | Health check API |

## Codice app.js

```javascript
const express = require('express');
const mysql = require('mysql2/promise');
const app = express();

// Middleware
app.use(express.json());

// Pool di connessioni MySQL
const pool = mysql.createPool({
  host: process.env.DB_HOST || 'localhost',
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASSWORD || 'root',
  database: process.env.DB_NAME || 'scuola_db',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// ============== STUDENTI ==============

// GET - Lista tutti gli studenti
app.get('/api/studenti', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query('SELECT * FROM studenti');
    connection.release();
    res.json(rows);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});

// GET - Uno studente per ID
app.get('/api/studenti/:id', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query(
      'SELECT * FROM studenti WHERE id = ?',
      [req.params.id]
    );
    connection.release();
    
    if (rows.length === 0) {
      return res.status(404).json({ errore: 'Studente non trovato' });
    }
    
    res.json(rows[0]);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});

// POST - Crea nuovo studente
app.post('/api/studenti', async (req, res) => {
  const { nome, cognome, email, data_nascita } = req.body;
  
  if (!nome || !cognome || !email) {
    return res.status(400).json({ errore: 'Nome, cognome ed email sono obbligatori' });
  }
  
  try {
    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'INSERT INTO studenti (nome, cognome, email, data_nascita) VALUES (?, ?, ?, ?)',
      [nome, cognome, email, data_nascita || null]
    );
    connection.release();
    
    res.status(201).json({
      id: result.insertId,
      nome,
      cognome,
      email,
      data_nascita: data_nascita || null,
      created_at: new Date()
    });
  } catch (err) {
    if (err.code === 'ER_DUP_ENTRY') {
      return res.status(409).json({ errore: 'Email già esistente' });
    }
    res.status(500).json({ errore: err.message });
  }
});

// PUT - Aggiorna studente
app.put('/api/studenti/:id', async (req, res) => {
  const { nome, cognome, email, data_nascita } = req.body;
  
  try {
    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'UPDATE studenti SET nome = ?, cognome = ?, email = ?, data_nascita = ? WHERE id = ?',
      [nome, cognome, email, data_nascita || null, req.params.id]
    );
    connection.release();
    
    if (result.affectedRows === 0) {
      return res.status(404).json({ errore: 'Studente non trovato' });
    }
    
    res.json({ messaggio: 'Studente aggiornato', id: req.params.id });
  } catch (err) {
    if (err.code === 'ER_DUP_ENTRY') {
      return res.status(409).json({ errore: 'Email già esistente' });
    }
    res.status(500).json({ errore: err.message });
  }
});

// DELETE - Elimina studente
app.delete('/api/studenti/:id', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'DELETE FROM studenti WHERE id = ?',
      [req.params.id]
    );
    connection.release();
    
    if (result.affectedRows === 0) {
      return res.status(404).json({ errore: 'Studente non trovato' });
    }
    
    res.json({ messaggio: 'Studente eliminato', id: req.params.id });
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});

// ============== CORSI ==============

// GET - Lista tutti i corsi
app.get('/api/corsi', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query('SELECT * FROM corsi');
    connection.release();
    res.json(rows);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});

// ============== ISCRIZIONI ==============

// GET - Corsi di uno studente
app.get('/api/iscrizioni/:id_studente', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query(
      `SELECT c.*, i.voto FROM corsi c 
       INNER JOIN iscrizioni i ON c.id = i.id_corso 
       WHERE i.id_studente = ?`,
      [req.params.id_studente]
    );
    connection.release();
    res.json(rows);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});

// POST - Iscrivi studente a corso
app.post('/api/iscrizioni', async (req, res) => {
  const { id_studente, id_corso } = req.body;
  
  if (!id_studente || !id_corso) {
    return res.status(400).json({ errore: 'id_studente e id_corso sono obbligatori' });
  }
  
  try {
    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'INSERT INTO iscrizioni (id_studente, id_corso) VALUES (?, ?)',
      [id_studente, id_corso]
    );
    connection.release();
    
    res.status(201).json({
      id: result.insertId,
      id_studente,
      id_corso,
      created_at: new Date()
    });
  } catch (err) {
    if (err.code === 'ER_DUP_ENTRY') {
      return res.status(409).json({ errore: 'Studente già iscritto a questo corso' });
    }
    res.status(500).json({ errore: err.message });
  }
});

// Health Check
app.get('/health', (req, res) => {
  res.json({ status: 'OK' });
});

// 404 - Rotta non trovata
app.use((req, res) => {
  res.status(404).json({ errore: 'Rotta non trovata' });
});

// Avvia server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server in esecuzione sulla porta ${PORT}`);
});
```

## Avvio del Progetto

### 1. Crea la struttura delle cartelle:

```bash
mkdir api-docker
cd api-docker
mkdir database
```

### 2. Copia i file nella directory:

```
mia-api-docker/
├── app.js
├── package.json
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
└── database/
    └── init.sql
```

### 3. Avvia con Docker Compose:

```bash
docker-compose up --build
```

Dovresti vedere:
```
db_1         | ready for connections
phpmyadmin_1 | ✔ All checks passed
api_1        | Server in esecuzione sulla porta 3000
```

### 4. Visualizza i log:

```bash
docker-compose logs -f api
docker-compose logs -f db
docker-compose logs -f phpmyadmin
```

### 5. Ferma i servizi:

```bash
docker-compose down
```

### Accesso ai servizi:
- **API:** http://localhost:3000
- **PHPMyAdmin:** http://localhost:8888
- **MySQL:** localhost:3306

## Test dell'API

### Con cURL

```bash
# GET - Lista studenti
curl http://localhost:3000/api/studenti

# GET - Uno studente
curl http://localhost:3000/api/studenti/1

# POST - Nuovo studente
curl -X POST http://localhost:3000/api/studenti \
  -H "Content-Type: application/json" \
  -d '{"nome":"Francesco","cognome":"Neri","email":"francesco@example.com","data_nascita":"2005-06-10"}'

# PUT - Aggiorna studente
curl -X PUT http://localhost:3000/api/studenti/1 \
  -H "Content-Type: application/json" \
  -d '{"nome":"Marco","cognome":"Rossi","email":"marco.rossi@example.com","data_nascita":"2005-03-15"}'

# DELETE - Elimina studente
curl -X DELETE http://localhost:3000/api/studenti/5

# GET - Corsi
curl http://localhost:3000/api/corsi

# GET - Iscrizioni studente 1
curl http://localhost:3000/api/iscrizioni/1

# POST - Iscrivi studente a corso
curl -X POST http://localhost:3000/api/iscrizioni \
  -H "Content-Type: application/json" \
  -d '{"id_studente":5,"id_corso":1}'
```

### Con REST Client (Plugin VSCode)

**Installazione:**
1. Apri VSCode
2. Vai su Extensions (Ctrl+Shift+X)
3. Cerca "REST Client" di Huachao Mao
4. Clicca "Install"

**Utilizzo:**
1. Crea un file `.http` o `.rest` nella root del progetto, es: `test-api.http`
2. Scrivi le request nel seguente formato:

```http
### Base URL
@baseUrl = http://localhost:3000

### GET - Lista tutti gli studenti
GET {{baseUrl}}/api/studenti

### GET - Uno studente
GET {{baseUrl}}/api/studenti/1

### POST - Crea nuovo studente
POST {{baseUrl}}/api/studenti
Content-Type: application/json

{
  "nome": "Francesco",
  "cognome": "Neri",
  "email": "francesco@example.com",
  "data_nascita": "2005-06-10"
}

### PUT - Aggiorna studente
PUT {{baseUrl}}/api/studenti/1
Content-Type: application/json

{
  "nome": "Marco",
  "cognome": "Rossi",
  "email": "marco.rossi@example.com",
  "data_nascita": "2005-03-15"
}

### DELETE - Elimina studente
DELETE {{baseUrl}}/api/studenti/5

### GET - Lista corsi
GET {{baseUrl}}/api/corsi

### GET - Corsi di uno studente
GET {{baseUrl}}/api/iscrizioni/1

### POST - Iscrivi studente a corso
POST {{baseUrl}}/api/iscrizioni
Content-Type: application/json

{
  "id_studente": 5,
  "id_corso": 1
}

### GET - Health check
GET {{baseUrl}}/health
```

**Per usare il file:**
- Clicca su "Send Request" sopra ogni richiesta
- I risultati appariranno in una nuova scheda
- Puoi salvare i risultati e vederli in formato JSON

### Con PHPMyAdmin

Accedi all'indirizzo `http://localhost:8888` per visualizzare e gestire il database direttamente:

- **Username:** `root`
- **Password:** `root`
- **Database:** `scuola_db`

Da PHPMyAdmin puoi:
- Visualizzare il contenuto delle tabelle
- Eseguire query SQL personalizzate
- Gestire users e permissions
- Fare backup del database

## Esercizi

### Esercizio 1: Aggiungi una Nuova Rotta per Voti

**Difficoltà:** Facile

**Obiettivo:** Creare un endpoint per ottenere il voto di uno studente in un corso.

**Istruzioni:**
Aggiungi a `app.js`:

```javascript
// GET voto di uno studente in un corso
app.get('/api/iscrizioni/:id_studente/:id_corso', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query(
      'SELECT * FROM iscrizioni WHERE id_studente = ? AND id_corso = ?',
      [req.params.id_studente, req.params.id_corso]
    );
    connection.release();
    
    if (rows.length === 0) {
      return res.status(404).json({ errore: 'Iscrizione non trovata' });
    }
    
    res.json(rows[0]);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});
```

**Test:**
```bash
curl http://localhost:3000/api/iscrizioni/1/1
```

---

### Esercizio 2: Aggiungi Endpoint per Docenti

**Difficoltà:** Media

**Obiettivo:** Creare una nuova tabella DOCENTI e gli endpoint relativi.

**Istruzioni:**

1. Modifica `database/init.sql` e aggiungi prima di `CREATE TABLE corsi`:

```sql
CREATE TABLE docenti (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  cognome VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  specializzazione VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO docenti (nome, cognome, email, specializzazione) VALUES
('Giuseppe', 'Esposito', 'esposito@example.com', 'Matematica'),
('Anna', 'Conti', 'conti@example.com', 'Informatica'),
('Roberto', 'Russo', 'russo@example.com', 'Fisica'),
('Maria', 'Gallo', 'gallo@example.com', 'Lingue'),
('Paolo', 'Giordano', 'giordano@example.com', 'Storia');
```

2. Modifica `app.js` e aggiungi gli endpoint:

```javascript
// GET tutti i docenti
app.get('/api/docenti', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query('SELECT * FROM docenti');
    connection.release();
    res.json(rows);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});

// GET corsi di un docente
app.get('/api/docenti/:id/corsi', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query(
      'SELECT * FROM corsi WHERE docente LIKE (SELECT CONCAT(nome, " ", cognome) FROM docenti WHERE id = ?)',
      [req.params.id]
    );
    connection.release();
    res.json(rows);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});
```

3. Riavvia i container:
```bash
docker-compose down
docker-compose up
```

4. Testa:
```bash
curl http://localhost:3000/api/docenti
curl http://localhost:3000/api/docenti/1/corsi
```

---

### Esercizio 3: Filtro Studenti per Nome

**Difficoltà:** Media

**Obiettivo:** Aggiungere un parametro di query per filtrare studenti per nome.

**Istruzioni:**

Modifica l'endpoint GET `/api/studenti`:

```javascript
// GET studenti con filtro opzionale
app.get('/api/studenti', async (req, res) => {
  try {
    const nome = req.query.nome;
    const connection = await pool.getConnection();
    
    let query = 'SELECT * FROM studenti';
    let params = [];
    
    if (nome) {
      query += ' WHERE nome LIKE ? OR cognome LIKE ?';
      params = [`%${nome}%`, `%${nome}%`];
    }
    
    const [rows] = await connection.query(query, params);
    connection.release();
    res.json(rows);
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});
```

**Test:**
```bash
curl http://localhost:3000/api/studenti?nome=Marco
curl http://localhost:3000/api/studenti?nome=Rossi
```

---

### Esercizio 4: Aggiorna Voto

**Difficoltà:** Media

**Obiettivo:** Creare un endpoint per aggiornare il voto di uno studente.

**Istruzioni:**

Aggiungi a `app.js`:

```javascript
// PUT - Aggiorna voto iscrizione
app.put('/api/iscrizioni/:id', async (req, res) => {
  const { voto } = req.body;
  
  if (typeof voto !== 'number' || voto < 0 || voto > 30) {
    return res.status(400).json({
      errore: 'Voto deve essere un numero tra 0 e 30'
    });
  }
  
  try {
    const connection = await pool.getConnection();
    await connection.query(
      'UPDATE iscrizioni SET voto = ? WHERE id = ?',
      [voto, req.params.id]
    );
    connection.release();
    
    res.json({ messaggio: 'Voto aggiornato' });
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});
```

**Test:**
```bash
curl -X PUT http://localhost:3000/api/iscrizioni/1 \
  -H "Content-Type: application/json" \
  -d '{"voto":29}'
```

---

### Esercizio 5: Media Voti Studente

**Difficoltà:** Difficile

**Obiettivo:** Creare un endpoint che calcola la media voti di uno studente.

**Istruzioni:**

Aggiungi a `app.js`:

```javascript
// GET media voti di uno studente
app.get('/api/studenti/:id/media', async (req, res) => {
  try {
    const connection = await pool.getConnection();
    const [rows] = await connection.query(
      `SELECT AVG(voto) as media, COUNT(*) as corsi
       FROM iscrizioni
       WHERE id_studente = ? AND voto IS NOT NULL`,
      [req.params.id]
    );
    connection.release();
    
    const media = rows[0].media ? parseFloat(rows[0].media).toFixed(2) : 'N/A';
    
    res.json({
      id_studente: req.params.id,
      media_voti: media,
      corsi_superati: rows[0].corsi
    });
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});
```

**Test:**
```bash
curl http://localhost:3000/api/studenti/1/media
```

---

### Esercizio 6: Validazione Avanzata

**Difficoltà:** Difficile

**Obiettivo:** Aggiungere validazione dell'email e prevenzione di inserti duplicati.

**Istruzioni:**

Modifica POST `/api/studenti`:

```javascript
// Funzione helper per validare email
function validateEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

// POST - Crea nuovo studente con validazione
app.post('/api/studenti', async (req, res) => {
  const { nome, cognome, email, data_nascita } = req.body;
  
  // Validazioni
  if (!nome || !cognome || !email) {
    return res.status(400).json({
      errore: 'Nome, cognome ed email sono obbligatori'
    });
  }
  
  if (!validateEmail(email)) {
    return res.status(400).json({
      errore: 'Email non valida'
    });
  }
  
  if (nome.length < 2 || cognome.length < 2) {
    return res.status(400).json({
      errore: 'Nome e cognome devono avere almeno 2 caratteri'
    });
  }
  
  try {
    const connection = await pool.getConnection();
    
    // Controlla se email esiste già
    const [existing] = await connection.query(
      'SELECT id FROM studenti WHERE email = ?',
      [email]
    );
    
    if (existing.length > 0) {
      connection.release();
      return res.status(409).json({
        errore: 'Email già registrata'
      });
    }
    
    const [result] = await connection.query(
      'INSERT INTO studenti (nome, cognome, email, data_nascita) VALUES (?, ?, ?, ?)',
      [nome, cognome, email, data_nascita]
    );
    connection.release();
    
    res.status(201).json({
      id: result.insertId,
      nome,
      cognome,
      email,
      data_nascita,
      messaggio: 'Studente creato con successo'
    });
  } catch (err) {
    res.status(500).json({ errore: err.message });
  }
});
```

**Test:**
```bash
# Email non valida
curl -X POST http://localhost:3000/api/studenti \
  -H "Content-Type: application/json" \
  -d '{"nome":"Test","cognome":"User","email":"invalid-email","data_nascita":"2005-01-01"}'

# Email già esistente
curl -X POST http://localhost:3000/api/studenti \
  -H "Content-Type: application/json" \
  -d '{"nome":"Marco","cognome":"Rossi","email":"marco.rossi@example.com","data_nascita":"2005-01-01"}'
```

---

## Comandi Utili

```bash
# Avvia i container
docker-compose up

# Avvia in background
docker-compose up -d

# Visualizza log API
docker-compose logs -f api

# Visualizza log DB
docker-compose logs -f db

# Accedi a MySQL da terminale
docker exec -it scuola_db mysql -u root -ppassword scuola_db

# Rimuovi tutto
docker-compose down -v

# Ricostruisci le image
docker-compose build --no-cache
```