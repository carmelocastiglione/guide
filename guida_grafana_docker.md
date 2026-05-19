# Guida: Installare e Usare Grafana su Docker

Questa guida ti mostra come installare, configurare e utilizzare Grafana con Docker, integrando MySQL, MongoDB e InfluxDB come fonti dati.

## Sommario
1. [Prerequisiti](#prerequisiti)
2. [Setup Rapido con Docker Compose](#setup-rapido-con-docker-compose)
3. [Avvio e Accesso a Grafana](#avvio-e-accesso-a-grafana)
4. [Configurazione Data Sources](#configurazione-data-sources)
   - [MySQL](#mysql)
   - [InfluxDB](#influxdb)
5. [Dashboard e Visualizzazioni](#dashboard-e-visualizzazioni)
6. [Comandi Docker Compose](#comandi-docker-compose)
7. [Troubleshooting](#troubleshooting)

## Prerequisiti

- **Docker Desktop** installato e funzionante
  - [Scarica Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Verifica l'installazione: `docker --version`
- **Docker Compose** (incluso in Docker Desktop)
  - Verifica: `docker-compose --version`
- **VSCode** (opzionale, per miglior gestione)
  - [Scarica VSCode](https://code.visualstudio.com/)

## Setup Rapido con Docker Compose

Docker Compose è il metodo **consigliato** per gestire Grafana e i servizi di database in un ambiente integrato.

### 1. Crea il file `docker-compose.yml`

Nella cartella del tuo progetto, crea un file denominato `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Grafana
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_SECURITY_ADMIN_EMAIL: admin@example.com
      GF_USERS_ALLOW_SIGN_UP: 'false'
      GF_INSTALL_PLUGINS: grafana-piechart-panel
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    networks:
      - monitoring-network
    depends_on:
      - mysql
      - influxdb
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 10s
      timeout: 5s
      retries: 5

  # MySQL
  mysql:
    image: mysql:8.0
    container_name: mysql-metrics
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: metrics
      MYSQL_USER: grafana
      MYSQL_PASSWORD: grafana
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - monitoring-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # InfluxDB 2.x
  influxdb:
    image: influxdb:2.7-alpine
    container_name: influxdb-metrics
    restart: always
    ports:
      - "8086:8086"
    environment:
      INFLUXDB_DB: metrics
      INFLUXDB_ADMIN_USER: admin
      INFLUXDB_ADMIN_PASSWORD: admin
      INFLUXDB_HTTP_AUTH_ENABLED: 'true'
      INFLUXDB_ADMIN_ENABLED: 'true'
    volumes:
      - influxdb_data:/var/lib/influxdb2
    networks:
      - monitoring-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8086/health"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  grafana_data:
    driver: local
  mysql_data:
    driver: local
  influxdb_data:
    driver: local

networks:
  monitoring-network:
    driver: bridge
```

### 2. Crea la struttura di directory per provisioning (opzionale)

```bash
mkdir -p grafana/provisioning/datasources
mkdir -p grafana/provisioning/dashboards
```

## Avvio e Accesso a Grafana

### 1. Avvia i servizi

Dalla cartella dove hai creato `docker-compose.yml`:

```bash
docker-compose up -d
```

I servizi si avvieranno in background. Verifica lo stato:

```bash
docker-compose ps
```

Dovresti vedere:
```
NAME        STATUS           PORTS
grafana     Up (healthy)     0.0.0.0:3000->3000/tcp
mysql       Up (healthy)     0.0.0.0:3306->3306/tcp
influxdb    Up (healthy)     0.0.0.0:8086->8086/tcp
```

### 2. Accedi a Grafana

- **URL**: http://localhost:3000
- **Username**: `admin`
- **Password**: `admin`

**Nota**: Al primo accesso, Grafana ti chiederà di cambiare la password di admin. È consigliato farlo per motivi di sicurezza.

## Configurazione Data Sources

### MySQL

#### 1. Vai in Configuration → Data Sources

Nel menu di Grafana, clicca su **Configuration** (icona ingranaggio) → **Data Sources**.

#### 2. Aggiungi MySQL

Clicca su **Add data source** e seleziona **MySQL**.

Compila i seguenti campi:

```
Name:               MySQL Metrics
Host:               mysql-metrics:3306
Database:           metrics
Username:           grafana
Password:           grafana
SSL Mode:           disable
```

**Nota**: Se accedi da dentro il container, usa `mysql-metrics:3306`. Se accedi da host, usa `localhost:3306`.

#### 3. Testa la connessione

Clicca su **Save & test**. Dovresti vedere il messaggio "Database connection ok".

#### 4. Crea una query di test

Dopo aver configurato il data source:
1. Crea un nuovo Dashboard
2. Aggiungi un Panel
3. Nel field della query, seleziona MySQL come data source
4. Scrivi una query di test:

```sql
SELECT 
  UNIX_TIMESTAMP(NOW()) as time,
  RAND() * 100 as value
LIMIT 1
```

### InfluxDB

#### 1. Setup iniziale di InfluxDB (prima volta)

Accedi a InfluxDB tramite il browser:
- **URL**: http://localhost:8086

Completa la configurazione iniziale:
- Username: `admin`
- Password: `admin`
- Organization: `MyOrg`
- Bucket: `metrics`

#### 2. Genera un token API

Nel UI di InfluxDB:
1. Vai su **API Tokens** (menu in basso a sinistra)
2. Clicca su **Generate API Token**
3. Seleziona **All Access API Token** (per sviluppo) o configura permessi specifici
4. Copia il token generato

#### 3. Configura il data source in Grafana

Nel menu di Grafana:
1. **Configuration** → **Data Sources** → **Add data source**
2. Seleziona **InfluxDB**

Compila i campi:

```
Name:               InfluxDB Metrics
Query Language:     Flux (opzionale: InfluxQL per versioni 1.x)
URL:                http://influxdb-metrics:8086
Organization:       MyOrg
Token:              <PASTE_YOUR_TOKEN_HERE>
Default Bucket:     metrics
```

#### 4. Testa la connessione

Clicca su **Save & test**. Dovresti vedere "Success".

#### 5. Query di test

Nel tuo dashboard, aggiungi un panel con una query Flux:

```flux
from(bucket:"metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
```

---

## Dashboard e Visualizzazioni

### Importare un Dashboard predefinito

1. Nel menu di Grafana, clicca su **Dashboards**
2. Clicca su **New Dashboard**
3. Clicca su **Import**
4. Cerca tra i dashboard disponibili (es. "MySQL" o "MongoDB")
5. Seleziona il dashboard e importalo

### Creare un Dashboard personalizzato

1. **Crea un nuovo dashboard**: Home → **New Dashboard**
2. **Aggiungi un panel**: Clicca su **Add a new panel**
3. **Configura la query**:
   - Seleziona il data source (MySQL, MongoDB o InfluxDB)
   - Scrivi la query desiderata
4. **Personalizza la visualizzazione**:
   - Tipo di grafico (line, bar, gauge, etc.)
   - Titolo, descrizione, colori
5. **Salva il dashboard**: Clicca su **Save dashboard**

### Esempio: Dashboard multi-source

Crea un dashboard che combina dati da più fonti:

**Panel 1 - MySQL**: CPU Usage
```sql
SELECT 
  DATE_FORMAT(timestamp, '%Y-%m-%d %H:%i:%s') as time,
  cpu_usage as value
FROM metrics
WHERE timestamp >= DATE_SUB(NOW(), INTERVAL 1 HOUR)
ORDER BY timestamp DESC
LIMIT 100
```

**Panel 2 - InfluxDB**: Memory Usage
```flux
from(bucket:"metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "memory")
  |> filter(fn: (r) => r._field == "used")
```

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

# Log di un servizio specifico
docker-compose logs -f grafana
docker-compose logs -f mysql-metrics
docker-compose logs -f influxdb-metrics
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
docker-compose restart grafana
```

### Accedere alla shell di un container

```bash
# Grafana
docker-compose exec grafana /bin/sh

# MySQL
docker-compose exec mysql-metrics mysql -u root -p

# InfluxDB
docker-compose exec influxdb-metrics influx
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

### Grafana non si connette a MySQL

**Problema**: Connection refused o timeout

**Soluzione**:
1. Verifica che il container MySQL sia in esecuzione: `docker-compose ps`
2. Controlla i log: `docker-compose logs mysql-metrics`
3. Assicurati di usare il nome del servizio (`mysql-metrics:3306`) quando accedi da dentro Docker
4. Se accedi da host, usa `localhost:3306` ma verifica che la porta sia esposta

```bash
# Testa la connessione dal container di Grafana
docker-compose exec grafana curl -v mysql-metrics:3306
```

### InfluxDB non si connette

**Problema**: Authentication failed

**Soluzione**:
1. Assicurati di aver completato la configurazione iniziale di InfluxDB: http://localhost:8086
2. Verifica le credenziali nel token API
3. Controlla che l'organizzazione e il bucket esistano:
```bash
docker-compose exec influxdb influx org list
docker-compose exec influxdb influx bucket list
```

4. Rigenera il token API se necessario

### Porta già in uso

**Problema**: Error binding to port 3000 (o altra porta)

**Soluzione**:
```bash
# Uccidi il processo in ascolto sulla porta
# Windows:
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Linux/Mac:
lsof -i :3000
kill -9 <PID>
```

Oppure, cambia la porta nel `docker-compose.yml`:
```yaml
ports:
  - "3001:3000"  # Accedi da http://localhost:3001
```

### Permessi insufficienti sui volumi

**Problema**: Permission denied su volumi

**Soluzione**:
1. Elimina i volumi e ricrea:
```bash
docker-compose down -v
docker-compose up -d
```

2. Oppure, ripara i permessi (Linux):
```bash
sudo chown -R $USER:$USER grafana_data mysql_data mongodb_data influxdb_data
docker-compose up -d
```

### Servizi non comunicano tra loro

**Problema**: Errore di connessione tra servizi

**Soluzione**:
1. Verifica che tutti i servizi siano nella stessa rete (`monitoring-network`)
2. Usa i nomi dei servizi (non `localhost`) per le connessioni interne
3. Riavvia con una network pulita:
```bash
docker-compose down
docker network prune
docker-compose up -d
```

### grafana-cli: executable file not found

**Problema**: `OCI runtime exec failed: exec failed: unable to start container process: exec: "grafana-cli": executable file not found in $PATH: unknown`

**Soluzione**: `grafana-cli` non è disponibile nel container Grafana ufficiale. Per installare i plugin, usa **solo** la variabile d'ambiente `GF_INSTALL_PLUGINS` nel `docker-compose.yml`:

```yaml
services:
  grafana:
    image: grafana/grafana:latest
    environment:
      GF_INSTALL_PLUGINS: grafana-mongodb-datasource,grafana-piechart-panel
```

Poi riavvia:
```bash
docker-compose down
docker-compose up -d
```

Verifica che i plugin siano installati nei log:
```bash
docker-compose logs grafana | grep -i plugin
```

**Nota**: Non eseguire mai `grafana-cli` direttamente nel container, non funzionerà. Usa sempre `GF_INSTALL_PLUGINS`.

---

## Tips & Tricks

### Backup e Restore

```bash
# Backup di Grafana
docker-compose exec -T grafana sqlite3 /var/lib/grafana/grafana.db ".backup '/var/lib/grafana/backup.db'"
docker cp grafana:/var/lib/grafana/backup.db ./backup.db

# Backup di MySQL
docker-compose exec -T mysql-metrics mysqldump -u root -proot --all-databases > backup.sql

# Backup di InfluxDB
docker-compose exec influxdb-metrics influx backup /tmp/backup
```

### Creare utenti aggiuntivi in Grafana

I nuovi utenti devono essere creati dalla **UI web di Grafana**, non da CLI (grafana-cli non è disponibile nel container ufficiale).

**Steps**:
1. Accedi a http://localhost:3000 con l'utente `admin`
2. Vai su **Configuration** → **Users**
3. Clicca su **New user**
4. Compila i dati e clicca **Create**

Oppure, usa l'**API REST** di Grafana:

```bash
curl -X POST http://admin:admin@localhost:3000/api/admin/users \
  -H "Content-Type: application/json" \
  -d '{"login":"username","email":"user@example.com","password":"password","name":"User Name"}'
```

### Monitorare le risorse dei container

```bash
# CPU, memoria, I/O di tutti i container
docker stats

# In tempo reale per un container specifico
docker stats grafana
```

### Configurare alerting in Grafana

1. Nel dashboard, vai sul panel che vuoi monitorare
2. Clicca su **Edit** → **Alert**
3. Configura le condizioni di trigger
4. Aggiungi un notification channel (Email, Slack, etc.)
5. Salva il panel

---

## Risorse Utili

- **Documentazione Grafana**: https://grafana.com/docs/
- **Docker Images**:
  - Grafana: https://hub.docker.com/r/grafana/grafana
  - MySQL: https://hub.docker.com/_/mysql
  - MongoDB: https://hub.docker.com/_/mongo
  - InfluxDB: https://hub.docker.com/_/influxdb
- **Data Source Plugins**: https://grafana.com/grafana/plugins
- **Community Dashboards**: https://grafana.com/grafana/dashboards

---

**Ultimo aggiornamento**: 2026-05-19
