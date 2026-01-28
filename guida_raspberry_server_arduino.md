# Guida: Ricevere Dati da Arduino via MQTT su Raspberry Pi

## Introduzione

Questa guida spiega come configurare un Raspberry Pi 2 Model B per ricevere dati da un Arduino tramite il protocollo MQTT e visualizzarli usando **InfluxDB**, **Telegraf** e **Grafana**.

### Architettura del Sistema

```
Arduino
   ↓
   └─→ MQTT (Mosquitto)
         ↓
         ├─→ Telegraf (ingestion)
         │     ↓
         └─→ InfluxDB (database time-series)
               ↓
               └─→ Grafana (visualizzazione)
```

## Prerequisiti

- Raspberry Pi 2 Model B con Raspberry Pi OS Lite installato (vedi: `guida_installazione_so_raspberry.md`)
- Connessione SSH attiva al Raspberry Pi
- Arduino con sketch MQTT configurato
- Arduino IDE o altra soluzione per programmare Arduino
- Accesso a internet sul Raspberry Pi per il download dei pacchetti

## Parte 1: Installazione e Configurazione di Mosquitto (MQTT Broker)

### Cos'è MQTT?

MQTT è un protocollo leggero di comunicazione publish/subscribe basato su TCP/IP, perfetto per dispositivi IoT come Arduino.

### Installazione di Mosquitto

```bash
# Aggiorna la lista dei pacchetti
sudo apt update

# Installa Mosquitto (broker MQTT) e client
sudo apt install mosquitto mosquitto-clients -y

# Abilita il servizio all'avvio
sudo systemctl enable mosquitto

# Avvia il servizio
sudo systemctl start mosquitto

# Verifica che sia in esecuzione
sudo systemctl status mosquitto
```

### Configurazione Mosquitto per Pubblico Accesso

Per permettere a dispositivi esterni (come Arduino) di connettersi:

```bash
# Apri il file di configurazione
sudo nano /etc/mosquitto/mosquitto.conf
```

Aggiungi o modifica le seguenti righe alla fine del file:

```conf
# Consenti connessioni su tutte le interfacce
listener 1883
protocol mqtt

# (Opzionale) Disabilita autenticazione per test, ma ricordati di abilitarla in produzione
allow_anonymous true
```

Salva con **CTRL+X**, poi **Y**, poi **Enter**.

Riavvia Mosquitto:

```bash
sudo systemctl restart mosquitto
```

### Test della Connessione MQTT

Apri due finestre SSH (o due tab di PowerShell).

**Terminal 1** - Sottoscrizione a un topic:

```bash
mosquitto_sub -h localhost -t "test/topic"
```

**Terminal 2** - Pubblica un messaggio di test:

```bash
mosquitto_pub -h localhost -t "test/topic" -m "Hello MQTT!"
```

Se il messaggio appare nel Terminal 1, MQTT funziona correttamente.

## Parte 2: Installazione e Configurazione di InfluxDB 1.8.x

### Cos'è InfluxDB?

InfluxDB è un database specializzato per dati time-series (serie temporali), perfetto per memorizzare misurazioni di sensori nel tempo.

**Nota**: Utilizziamo InfluxDB **1.8.x** invece di 2.x perché il Raspberry Pi 2 è un dispositivo a 32-bit con risorse limitate. InfluxDB 1.8 ha un footprint di memoria molto inferiore ed è più stabile su ARM32.

### Installazione di InfluxDB 1.8.x

Su Raspberry Pi 2, il repository ufficiale InfluxDB ha problemi di verifica GPG. Usa questo metodo alternativo:

```bash
# Aggiungi il repository InfluxDB disabilitando la verifica della firma
# (Necessario per ARM32 dove la chiave non è completamente supportata)
echo "deb [trusted=yes] https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

# Aggiorna i pacchetti
sudo apt update

# Se ricevi ancora warning, prova con:
sudo apt update --allow-insecure-repositories

# Installa InfluxDB 1.8.x
sudo apt install influxdb -y

# Abilita il servizio all'avvio
sudo systemctl enable influxdb

# Avvia il servizio
sudo systemctl start influxdb

# Verifica che sia in esecuzione
sudo systemctl status influxdb
```

**Nota sulla sicurezza**: L'opzione `[trusted=yes]` disabilita la verifica GPG per questo repository. Su una Raspberry Pi in una rete privata questo è accettabile, ma su ambienti di produzione esposti a internet, valuta alternative come compilare da source o usare Prometheus.

### Configurazione Iniziale di InfluxDB 1.8.x

```bash
# Accedi alla shell InfluxDB CLI
influx
```

Dentro la shell InfluxDB, esegui i seguenti comandi:

```sql
-- Crea un database per i dati Arduino
CREATE DATABASE arduino_db

-- Visualizza i database creati
SHOW DATABASES

-- Esci dalla shell
exit
```

### Creare un Utente (Opzionale ma Consigliato)

Per aggiungere autenticazione:

```bash
influx
```

Dentro la shell:

```sql
-- Crea un utente per Telegraf
CREATE USER telegraf WITH PASSWORD 'telegraf_password'

-- Assegna i permessi
GRANT ALL ON arduino_db TO telegraf

-- Esci
exit
```

## Parte 3: Installazione e Configurazione di Telegraf

### Cos'è Telegraf?

Telegraf è un agent per raccogliere, elaborare e trasmettere metriche. In questo caso, leggerà i dati MQTT da Mosquitto e li inoltrerà a InfluxDB.

### Installazione di Telegraf

```bash
# Aggiungi il repository di InfluxData (disabilitando verifica GPG per ARM32)
echo "deb [trusted=yes] https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

# Aggiorna i pacchetti
sudo apt update

# Se ricevi warning, prova con:
sudo apt update --allow-insecure-repositories

# Installa Telegraf
sudo apt install telegraf -y

# Abilita il servizio all'avvio
sudo systemctl enable telegraf

# Verifica la configurazione di default
sudo cat /etc/telegraf/telegraf.conf | less
```

### Configurazione Telegraf per MQTT e InfluxDB

Crea un nuovo file di configurazione specifico per il tuo progetto:

```bash
sudo nano /etc/telegraf/telegraf.d/mqtt-influxdb.conf
```

Inserisci la seguente configurazione:

```toml
# Definisci l'intervallo di raccolta dati
[agent]
interval = "10s"
round_interval = true
metric_batch_size = 1000
metric_buffer_limit = 10000
collection_jitter = "0s"
flush_interval = "10s"
flush_jitter = "0s"

# Input: Sottoscrizione al topic MQTT da Mosquitto
[[inputs.mqtt_consumer]]
servers = ["tcp://localhost:1883"]
topics = [
  "arduino/temperature",
  "arduino/humidity",
  "arduino/pressure"
]
# Se Mosquitto richiede autenticazione:
# username = "telegraf"
# password = "your_password"
qos = 0
persistent_session = false
client_id = "telegraf"
data_format = "value"
data_type = "float"

# Output: Invia i dati a InfluxDB 1.8.x
[[outputs.influxdb]]
urls = ["http://localhost:8086"]
database = "arduino_db"
# Se hai creato un utente (opzionale):
# username = "telegraf"
# password = "telegraf_password"
timeout = "5s"
```

**Configura:**
- `database = "arduino_db"` - Il database creato in InfluxDB
- Se hai abilitato l'autenticazione, inserisci username e password

Salva con **CTRL+X**, poi **Y**, poi **Enter**.

### Avvio di Telegraf

```bash
# Riavvia Telegraf per applicare la configurazione
sudo systemctl restart telegraf

# Verifica che sia in esecuzione
sudo systemctl status telegraf

# Visualizza i log per eventuali errori
sudo journalctl -u telegraf -n 20 --no-pager
```

## Parte 4: Installazione e Configurazione di Grafana

### Cos'è Grafana?

Grafana è una piattaforma di visualizzazione che ti permette di creare dashboard per monitorare i dati in tempo reale.

### Installazione di Grafana

Grafana su Raspberry Pi ARM32 potrebbe avere nomi di pacchetto diversi. Prova prima con il metodo ufficiale:

```bash
# Aggiungi il repository Grafana con chiave GPG
curl -s https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/grafana-archive-keyring.gpg

# Aggiungi il repository
echo "deb [signed-by=/usr/share/keyrings/grafana-archive-keyring.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Aggiorna i pacchetti
sudo apt update

# Installa Grafana (prova prima con "grafana", se non funziona usa "grafana-server")
sudo apt install grafana -y

# Se il comando precedente fallisce, prova:
# sudo apt install grafana-server -y

# Abilita il servizio all'avvio
sudo systemctl enable grafana-server

# Avvia il servizio
sudo systemctl start grafana-server

# Verifica che sia in esecuzione
sudo systemctl status grafana-server
```

### Accesso all'Interfaccia Grafana

1. Apri il browser: `http://192.168.1.100:3000` (sostituisci con l'IP del Raspberry)
2. Credenziali di default:
   - **Username**: `admin`
   - **Password**: `admin`
3. Al primo accesso, ti verrà chiesto di cambiare la password

### Configurazione della Sorgente Dati (Data Source)

1. Clicca su **Connections** nel menu a sinistra
2. Clicca **Add new connection**
3. Scegli **InfluxDB**
4. Clicca su **Add new data source**
5. Configura:
   - **Name**: `InfluxDB Arduino`
   - **Query Language**: `InfluxQL`
   - **URL**: `http://localhost:8086`
   - **Auth**: Seleziona **Basic Auth** solo se hai abilitato autenticazione
   - **Database**: `arduino_db`
   - **User**: `telegraf` (se hai creato un utente, altrimenti lascia vuoto)
   - **Password**: `telegraf_password` (se hai creato un utente, altrimenti lascia vuoto)
6. Clicca **Save & test**

Dovresti vedere un messaggio di successo.

### Creazione di un Dashboard

1. Nel menu a sinistra, clicca su **Dashboards** (quadrato con più quadrati)
2. Clicca **New** → **New Dashboard**
3. Clicca **Add panel**
4. Nel pannello di query:
   - Seleziona `InfluxDB Arduino` come Data Source
   - Costruisci una query per visualizzare i dati (es. temperature da Arduino)

**Esempio di Query InfluxQL (linguaggio InfluxDB 1.8):**

Nell'interfaccia Grafana, usa il **Query Inspector** e scrivi:

```sql
SELECT mean("value") FROM "mqtt_consumer" WHERE topic='arduino/temperature' AND time > now() - 1h GROUP BY time(1m)
```

Oppure crea una query visualmente:
1. Nel pannello Query, scegli **InfluxDB Arduino** come source
2. Seleziona **arduino_db** come database
3. Scegli la measurement **mqtt_consumer**
4. Filtra per il topic desiderato (es. `arduino/temperature`)
5. Scegli l'aggregazione (es. **mean**, **last**, **max**)

5. Personalizza il grafico (tipo, colori, assi, ecc.)
6. Salva il dashboard con un nome significativo

## Parte 5: Configurazione Arduino per MQTT

### Sketch Arduino Base

Ecco un esempio di sketch Arduino che legge sensori e pubblica i dati su MQTT:

```cpp
#include <PubSubClient.h>
#include <WiFi.h>

// Configurazione WiFi (se usi Arduino con WiFi, es. Arduino MKR WiFi 1010)
const char* ssid = "NOME_WIFI";
const char* password = "PASSWORD_WIFI";

// Configurazione MQTT
const char* mqtt_server = "192.168.1.100"; // IP del Raspberry
const int mqtt_port = 1883;
const char* mqtt_client_id = "Arduino_Client";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(9600);
  
  // Connessione WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi Connesso!");
  
  // Configurazione MQTT
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {
  // Mantieni la connessione MQTT
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  
  // Leggi un sensore (esempio: temperatura da A0)
  int sensorValue = analogRead(A0);
  float temperature = sensorValue * 0.48828125; // Converte in gradi Celsius
  
  // Pubblica il valore
  char msg[50];
  snprintf(msg, 50, "%.2f", temperature);
  client.publish("arduino/temperature", msg);
  
  delay(5000); // Pubblica ogni 5 secondi
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentativo di connessione MQTT...");
    if (client.connect(mqtt_client_id)) {
      Serial.println("Connesso!");
    } else {
      Serial.print("Fallito, rc=");
      Serial.print(client.state());
      Serial.println(" Riprovo in 5 secondi");
      delay(5000);
    }
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Messaggio ricevuto [");
  Serial.print(topic);
  Serial.print("]: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}
```

**Nota**: Se usi Arduino Uno o Mega senza WiFi integrato, dovrai aggiungere uno shield Ethernet o WiFi (es. Arduino WiFi Shield).

### Per Arduino con Ethernet

Se usi uno shield Ethernet:

```cpp
#include <PubSubClient.h>
#include <Ethernet.h>

byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED};
IPAddress server(192, 168, 1, 100); // IP del Raspberry

EthernetClient ethClient;
PubSubClient client(ethClient);

// Resto dello sketch è simile
```

## Parte 6: Test del Sistema Completo

### 1. Verifica Mosquitto

```bash
mosquitto_sub -h localhost -t "arduino/#"
```

Dovresti vedere i messaggi MQTT pubblicati da Arduino.

### 2. Verifica InfluxDB

Accedi tramite CLI:

```bash
influx
```

Esegui:

```sql
USE arduino_db
SHOW MEASUREMENTS
SELECT * FROM mqtt_consumer LIMIT 10
```

Dovresti vedere i dati inviati da Telegraf.

### 3. Verifica Telegraf

```bash
sudo journalctl -u telegraf -f
```

Dovresti vedere i log di Telegraf che riceve e invia dati.

### 4. Verifica Grafana

Accedi a: `http://192.168.1.100:3000` e controlla il dashboard.

## Troubleshooting

### Telegraf non riceve dati da MQTT
- Verifica che Mosquitto sia in esecuzione: `sudo systemctl status mosquitto`
- Testa la connessione MQTT: `mosquitto_sub -h localhost -t "arduino/#"`
- Controlla la configurazione di Telegraf: `sudo cat /etc/telegraf/telegraf.d/mqtt-influxdb.conf`
- Verifica i log: `sudo journalctl -u telegraf -n 50 --no-pager`

### I dati non appaiono in InfluxDB
- Verifica che il database esista: `influx` poi `SHOW DATABASES`
- Controlla i log di Telegraf: `sudo journalctl -u telegraf -n 50`
- Verifica la connessione: `curl http://localhost:8086/ping`
- Se usi autenticazione, verifica username/password: `influx -username telegraf -password telegraf_password`

### Grafana non visualizza i dati
- Verifica la connessione alla sorgente dati in Grafana
- Controlla che la query Flux sia corretta
- Assicurati che il bucket e l'organizzazione siano corretti

### Arduino non si connette a MQTT
- Verifica che l'IP del Raspberry sia corretto
- Controlla che Mosquitto sia in ascolto: `ss -tlnp | grep mosquitto`
- Testa la connessione dal Raspberry: `telnet 192.168.1.100 1883`

## Parte 7: Sicurezza in Produzione

### Abilita Autenticazione su Mosquitto

```bash
# Crea un file password
sudo mosquitto_passwd -c /etc/mosquitto/passwd mqtt_user

# Configura Mosquitto per usare le password
sudo nano /etc/mosquitto/mosquitto.conf
```

Aggiungi:

```conf
password_file /etc/mosquitto/passwd
allow_anonymous false
```

Riavvia Mosquitto:

```bash
sudo systemctl restart mosquitto
```

Aggiorna la configurazione di Telegraf:

```conf
[[inputs.mqtt_consumer]]
servers = ["tcp://localhost:1883"]
username = "mqtt_user"
password = "your_password"
```

### Cambia le Password di Default in InfluxDB 1.8

```bash
# Accedi a InfluxDB
influx
```

Dentro la shell:

```sql
-- Cambia la password dell'utente telegraf
ALTER USER telegraf WITH PASSWORD 'new_secure_password'

-- Esci
exit
```

**Grafana**: Accedi e cambia la password di default via interfaccia web nelle impostazioni utente.

### Abilita SSL/TLS (Opzionale)

Per comunicazioni crittografate, configura certificati SSL su Mosquitto e Grafana.

## Comandi Utili per Manutenzione

```bash
# Visualizza lo stato di tutti i servizi
sudo systemctl status mosquitto influxdb telegraf grafana-server

# Riavvia tutti i servizi
sudo systemctl restart mosquitto influxdb telegraf grafana-server

# Visualizza l'uso della CPU e memoria
top

# Visualizza l'uso del disco
df -h

# Pulisci il buffer del kernel
sudo sync

# Riavvia il Raspberry
sudo reboot

# Spegni il Raspberry
sudo shutdown -h now
```

## Conclusione

Hai completato la configurazione di un sistema completo di monitoraggio con Arduino, MQTT, InfluxDB, Telegraf e Grafana!

Questo setup ti permette di:
- ✅ Raccogliere dati da Arduino via MQTT
- ✅ Memorizzare i dati in un database time-series
- ✅ Visualizzare i dati in tempo reale su Grafana
- ✅ Scalare il sistema aggiungendo più sensori e dispositivi

## Riferimenti Utili

- [Documentazione MQTT Mosquitto](https://mosquitto.org/documentation/)
- [Documentazione InfluxDB](https://docs.influxdata.com/influxdb/v2.0/)
- [Documentazione Telegraf](https://docs.influxdata.com/telegraf/)
- [Documentazione Grafana](https://grafana.com/docs/grafana/latest/)
- [Arduino Official Documentation](https://docs.arduino.cc/)
- [PubSubClient Library](https://github.com/knolleary/pubsubclient)
