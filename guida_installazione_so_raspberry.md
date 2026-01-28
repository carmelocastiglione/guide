# Guida Installazione Raspberry Pi OS su Raspberry Pi 2 Model B

## Introduzione

Questa guida ti accompagna nel processo di installazione di Raspberry Pi OS su un Raspberry Pi 2 Model B configurando un **server headless** (senza schermo) utilizzando Raspberry Pi Imager.

## Hardware Necessario

Per configurare un server headless su Raspberry Pi 2 Model B, avrai bisogno di:

### Componenti Essenziali
- **Raspberry Pi 2 Model B** - Il microcomputer principale
- **Scheda MicroSD** - Minimo 16GB (consigliato 32GB per maggiore affidabilità)
- **Alimentatore USB** - 2.5A o superiore (importante per la stabilità)
- **Cavo Ethernet** - Per la connessione di rete (categoria 5e o superiore)
- **Router/Switch di rete** - Per collegare il Pi alla rete tramite DHCP

### Componenti Opzionali ma Consigliati
- **Case protettivo** - Per proteggere l'hardware
- **Dissipatori di calore** - Opzionali ma utili per la dissipazione termica
- **Lettore di schede SD** - Se il tuo computer non ha uno slot integrato

### Non Necessari per Server Headless
- **Schermo HDMI**
- **Mouse e tastiera**
- **Cavi di alimentazione GPIO**

## Preparazione

### 1. Scarica Raspberry Pi Imager

1. Accedi al sito ufficiale: [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)
2. Scarica la versione per il tuo sistema operativo (Windows, macOS, Linux)
3. Installa il programma seguendo le istruzioni

### 2. Preparazione della scheda MicroSD

1. Inserisci la scheda MicroSD nel lettore collegato al tuo computer
2. Avvia Raspberry Pi Imager

## Configurazione con Raspberry Pi Imager

### Passo 1: Seleziona il Sistema Operativo

1. Apri **Raspberry Pi Imager**
2. Scegli **"Raspberry Pi 2 Model B"** come dispositivo
3. Seleziona **"Raspberry Pi OS (other)"**
4. Scegli **"Raspberry Pi OS Lite (32-bit)"** (versione minimale, senza interfaccia grafica - perfetta per server headless)

### Passo 2: Seleziona la Scheda di Memoria

1. Seleziona la tua scheda MicroSD (attento a non selezionare il drive sbagliato o cancellarai tutto)

### Passo 3: Scegli il Nome Host
- **hostname**: `raspberrypi` (o il nome che preferisci, ad es. `raspberry-server`)
- Questo sarà il nome con cui accederai al Pi sulla rete

### Passo 4: Configurazione Locale
- **Città capitale**: Rome (Italy)
- **Fuso orario**: `Europe/Rome` (o il tuo fuso orario)
- **Locale della tastiera**: `it` (Italiano)

### Passo 5: Username e Password
- **Nome utente**: `pi` (predefinito)
- **Password**: Scegli una password sicura e annotala
  - Esempio: una combinazione di lettere maiuscole, minuscole, numeri e simboli
  - **Importante**: Annota questa password, la userai per SSH

### Passo 6: WiFi (OPZIONALE - Non necessario per Ethernet)
- Se intendi usare solo Ethernet per il server, puoi saltare questa sezione

### Passo 7: Configurazione SSH
- ☑️ **Abilita SSH** (Enable SSH)
- Seleziona **"Usa autenticazione con password**

### Passo 8: Raspberry Pi Connect
- Non serve abilitarlo

### Passo 9: Scrivi immagine
1. Rivedere i parametri inseriti e procedere con la scrittura sulla scheda microSD
2. Conferma quando richiesto (attenzione: i dati sulla scheda verranno cancellati)
3. Attendi il completamento dell'operazione (può durare 5-10 minuti)
4. Un messaggio di successo apparirà al termine
5. Rimuovi la scheda MicroSD dal computer

## Installazione Fisica del Raspberry Pi

### Preparazione Hardware

1. **Spegni il Raspberry Pi** se è acceso
2. **Inserisci la scheda MicroSD** nello slot MicroSD (sul lato inferiore della scheda)
3. **Collega il cavo Ethernet** alla porta Ethernet integrata del Pi e al tuo router/switch di rete

### Avvio del Sistema

1. Collega l'alimentatore USB al Raspberry Pi
2. Il Pi si avvierà automaticamente
3. L'LED rosso (Power) deve rimanere acceso
4. L'LED verde (Activity) lampeggerà durante l'inizializzazione (può durare 1-2 minuti)

## Trovare il Raspberry Pi sulla Rete

### Metodo 1: Tramite Router/Interfaccia DHCP

1. Accedi alla pagina amministrativa del tuo router (solitamente http://192.168.1.1 o http://192.168.0.1)
2. Accedi con le credenziali del router
3. Cerca la sezione **"DHCP Clients"** o **"Connected Devices"**
4. Cerca il dispositivo con il nome host che hai impostato (es. `raspberry-server`)
5. Annota l'indirizzo IP assegnato (es. `192.168.1.100`)

### Metodo 2: Tramite Comando (Windows PowerShell)

```powershell
# Scansiona i dispositivi della rete con ping
for ($i=1; $i -le 254; $i++) {
    $ip = "192.168.1.$i"
    ping -n 1 -w 100 $ip 2>$null | Select-String "reply" | ForEach-Object {
        Write-Host "Found: $ip"
    }
}
```

### Metodo 3: Tramite Nmap (se installato)

```bash
nmap -sn 192.168.1.0/24
```

### Metodo 4: Ricerca del Hostname (Windows PowerShell)

```powershell
# Se conosci il nome host, puoi cercare il dispositivo
ping raspberry-server -4
# Nota l'indirizzo IP dalla risposta
```

## Connessione via SSH

### Prerequisiti

- Indirizzo IP del Raspberry Pi (es. `192.168.1.100`)
- Nome utente: `pi`
- Password impostata durante la configurazione

### Metodo 1: Tramite PowerShell (Windows 10/11)

```powershell
# Apri PowerShell e digita:
ssh pi@192.168.1.100

# Oppure con il hostname (se risolto):
ssh pi@raspberry-server

# Quando richiesto, digita la password e premi Enter
```

### Metodo 2: Tramite PuTTY (Alternativa)

1. Scarica PuTTY da: https://www.putty.org/
2. Apri PuTTY
3. Nella sezione **Host Name**, inserisci: `pi@192.168.1.100`
4. Assicurati che **Port** sia impostato a `22`
5. Clicca **Open**
6. Quando richiesto, digita la password

### Metodo 3: Tramite Terminal Linux/macOS

```bash
ssh pi@192.168.1.100
# Digita la password quando richiesto
```

## Verificazione della Connessione

Una volta connesso tramite SSH, dovresti vedere un prompt simile a:

```
pi@raspberry-server:~ $
```

Digita i seguenti comandi per verificare la corretta installazione:

### Verifica Sistema Operativo
```bash
cat /etc/os-release
```

### Verifica Indirizzo IP
```bash
hostname -I
```

### Verifica Connessione di Rete
```bash
ping 8.8.8.8
```

### Visualizza Informazioni Hardware
```bash
cat /proc/cpuinfo
```

## Configurazione Successiva Consigliata

### Aggiorna il Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### Cambia la Password (Opzionale)

```bash
passwd
```

### Configura il Firewall (Opzionale)

```bash
sudo apt install ufw -y
sudo ufw allow 22/tcp
sudo ufw enable
```

## Troubleshooting

### Il Raspberry Pi non si connette alla rete
- Verifica che il cavo Ethernet sia collegato correttamente
- Controlla che il router sia acceso e funzionante
- Riavvia il Raspberry Pi staccando e ricollegando l'alimentatore

### Non riesco a connettermi via SSH
- Verifica di usare l'indirizzo IP corretto
- Controlla che SSH sia abilitato nelle impostazioni di Imager
- Assicurati di usare la password corretta
- Verifica che il Raspberry Pi sia raggiungibile con `ping 192.168.1.xxx`

### SSH rifiuta la connessione
- Assicurati che il Raspberry Pi abbia finito l'avvio (aspetta 1-2 minuti dopo l'alimentazione)
- Verifica le credenziali inserite
- Prova a riavviare il Raspberry Pi

## Conclusione

Hai completato l'installazione di Raspberry Pi OS su Raspberry Pi 2 Model B come server headless! Ora puoi:

- Installare servizi e applicazioni via SSH
- Monitorare il sistema remotamente
- Configurare automazioni e script
- Procedere con l'installazione di InfluxDB, Telegraf e Grafana (vedi: `guida_raspberry_server_arduino.md`)

## Riferimenti Utili

- [Sito ufficiale Raspberry Pi](https://www.raspberrypi.com/)
- [Documentazione ufficiale](https://www.raspberrypi.com/documentation/)
- [Raspberry Pi Imager GitHub](https://github.com/raspberrypi/rpi-imager)
