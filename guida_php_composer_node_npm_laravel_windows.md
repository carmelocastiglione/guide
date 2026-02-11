# Guida: Installazione di PHP Composer, Node.js e npm per Laravel su Windows

## Indice
1. [Prerequisiti](#prerequisiti)
2. [Installazione di PHP](#installazione-di-php)
3. [Installazione di PHP Composer](#installazione-di-php-composer)
4. [Installazione di Node.js e npm](#installazione-di-nodejs-e-npm)
5. [Creazione di un Progetto Laravel](#creazione-di-un-progetto-laravel)
6. [Configurazione del Progetto](#configurazione-del-progetto)
7. [Avvio del Server di Sviluppo](#avvio-del-server-di-sviluppo)

## Prerequisiti

Prima di iniziare, dovrai installare:

### **Git** (opzionale ma consigliato)
   - Scarica da: [git-scm.com](https://git-scm.com/download/win)

## Installazione di PHP

### Installazione Manuale

1. **Scarica PHP da [php.net](https://www.php.net/downloads/):**
   - Visita il sito ufficiale
   - Scarica la versione **Non-Thread Safe (NTS)** più recente (attualmente PHP 8.2 o superiore)
   - Seleziona il file ZIP (ad es.: `php-8.2.x-Win32-vs16-x64-nts.zip`)
   - NTS è consigliato per Laravel con `php artisan serve` e per development moderno

2. **Estrai il file ZIP:**
   - Crea una cartella `C:\php` sul disco C
   - Estrai il contenuto del file ZIP in questa cartella

3. **Configura il file `php.ini`:**
   - Nella cartella `C:\php`, copia `php.ini-development` in `php.ini`
   - Apri `php.ini` con un editor di testo (Notepad o VS Code)
   - Decommenta le seguenti righe (rimuovi il `;` all'inizio):
     ```
     extension_dir = "ext"
     extension=curl
     extension=fileinfo
     extension=mbstring
     extension=openssl
     extension=pdo_mysql
     extension=pdo_sqlite
     extension=zip
     ```

4. **Aggiungi PHP al PATH di Windows:**
   - Premi il tasto Windows e scrivi "variabili di ambiente" o "environment variables"
   - Clicca su "Modifica le variabili d'ambiente del sistema"
   - Clicca su "Variabili d'Ambiente"
   - Seleziona la variabile `PATH` e clicca "Modifica"
   - Clicca "Nuovo" e aggiungi: `C:\php`
   - Clicca "OK" su tutte le finestre per salvare

5. **Verifica l'installazione:**
   - Apri un nuovo **Prompt dei Comandi** (importante: apri una nuova finestra)
   - Digita:
   ```bash
   php -v
   ```
   - Dovresti vedere la versione di PHP installata

## Installazione di PHP Composer

### Metodo 1: Installatore Ufficiale (Consigliato)

1. **Scarica l'installatore di Composer:**
   - Visita [getcomposer.org](https://getcomposer.org/download/)
   - Scarica il file `Composer-Setup.exe`

2. **Esegui l'installatore:**
   - Fai doppio clic su `Composer-Setup.exe`
   - Segui le istruzioni della procedura guidata
   - Quando richiesto, specifica il percorso a `php.exe` (di solito in `C:\xampp\php\php.exe` se usi XAMPP)

3. **Verifica l'installazione:**
   - Apri un nuovo Prompt dei Comandi
   - Digita:
   ```bash
   composer --version
   ```

### Metodo 2: Installazione Manuale Sicura

1. **Naviga alla cartella di installazione:**
   ```bash
   cd C:\php
   ```
   (o dove vuoi installare Composer)

2. **Scarica l'installatore di Composer con verifica:**
   ```bash
   php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
   ```

3. **Verifica l'integrità del file:**
   ```bash
   php -r "if (hash_file('sha384', 'composer-setup.php') === 'c8b085408188070d5f52bcfe4ecfbee5f727afa458b2573b8eaaf77b3419b0bf2768dc67c86944da1544f06fa544fd47') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); exit(1); }"
   ```

4. **Esegui l'installazione:**
   ```bash
   php composer-setup.php
   ```

5. **Rimuovi il file dell'installatore:**
   ```bash
   php -r "unlink('composer-setup.php');"
   ```

6. **Crea un file .bat per eseguire Composer globalmente:**
   Con il Prompt dei Comandi normale (cmd):
   ```bash
   echo @php "%~dp0composer.phar" %*>composer.bat
   ```
   Con PowerShell:
   ```powershell
   Set-Content composer.bat '@php "%~dp0composer.phar" %*'
   ```

7. **Verifica l'installazione:**
   ```bash
   composer --version
   ```

## Installazione di Node.js e npm

### Installazione Manuale (Standalone Binary)

1. **Scarica Node.js:**
   - Visita [nodejs.org](https://nodejs.org/)
   - Scarica la versione **LTS** (consigliata per stabilità)
   - Seleziona il file ZIP per Windows a 64-bit (ad es.: `node-v20.x.x-win-x64.zip`)

2. **Estrai il file ZIP:**
   - Crea una cartella `C:\nodejs` sul disco C
   - Estrai il contenuto del file ZIP in questa cartella
   - Dovresti avere file come `node.exe` e `npm.cmd` nella cartella `C:\nodejs`

3. **Aggiungi Node.js al PATH di Windows:**
   - Premi il tasto Windows e scrivi "variabili di ambiente" o "environment variables"
   - Clicca su "Modifica le variabili d'ambiente del sistema"
   - Clicca su "Variabili d'Ambiente"
   - Seleziona la variabile `PATH` e clicca "Modifica"
   - Clicca "Nuovo" e aggiungi: `C:\nodejs`
   - Clicca "OK" su tutte le finestre per salvare

4. **Verifica l'installazione:**
   - Apri un nuovo Prompt dei Comandi (importante: apri una nuova finestra)
   - Digita:
   ```bash
   node --version
   npm --version
   ```
   - Dovresti vedere le versioni di Node.js e npm installate
   - Se vedi un errore in `npm`, prova a usare `npm.cmd --version`, apri il Prompt dei Comandi normale (cmd) invece di PowerShell, o modifica la PowerShell Execution Policy come descritto nella sezione "Risoluzione dei Problemi"

## Creazione di un Progetto Laravel

### Opzione 1: Usando Composer (Non consigliato)

1. **Navigare alla cartella di progetto:**
   ```bash
   cd C:\Codice
   ```
   (o la cartella dove vuoi creare il progetto)

2. **Creare un nuovo progetto Laravel:**
   ```bash
   composer create-project laravel/laravel mio-progetto
   ```
   
   Dove `mio-progetto` è il nome della tua applicazione.

3. **Entrare nella cartella del progetto:**
   ```bash
   cd mio-progetto
   ```

### Opzione 2: Usando Laravel Installer (Consigliato)

Se preferisci usare il Laravel Installer:

1. **Installa il Laravel Installer globalmente:**
   ```bash
   composer global require laravel/installer
   ```

2. **Aggiungi il percorso alla variabile PATH** (se necessario):
   - Apri Variabili di Ambiente
   - Aggiungi: `C:\Users\[TuoUsername]\AppData\Roaming\Composer\vendor\bin`

3. **Crea un nuovo progetto:**
   ```bash
   laravel new mio-progetto
   ```

4. **Avvio del Server di Sviluppo:**
   ```bash
   cd mio-progetto
   composer run dev
   ```

## Configurazione del Progetto (se si è scelta l'opzione 1)

### 1. **Installa le Dipendenze PHP:**
   ```bash
   composer install
   ```

### 2. **Installa le Dipendenze JavaScript:**
   ```bash
   npm install
   ```

### 3. **Copia il file di Configurazione:**
   ```bash
   copy .env.example .env
   ```
   oppure
   ```bash
   cp .env.example .env
   ```

### 4. **Genera la Chiave dell'Applicazione:**
   ```bash
   php artisan key:generate
   ```

### 5. **Configura il Database (opzionale):**
   - Apri il file `.env` con un editor di testo
   - Configura i dati di connessione al database:
   ```
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=nome_database
   DB_USERNAME=root
   DB_PASSWORD=
   ```

### 6. **Esegui le Migrazioni del Database (opzionale):**
   ```bash
   php artisan migrate
   ```

## Avvio del Server di Sviluppo

### Terminale 1: Avvia il Server Laravel

```bash
php artisan serve
```

Per default, il server sarà disponibile a: `http://localhost:8000`

### Terminale 2: Compila gli Asset (CSS, JavaScript)

```bash
npm run dev
```

Oppure, per la build di produzione:
```bash
npm run build
```

## Comandi Utili

### Composer
```bash
composer install       # Installa le dipendenze
composer update        # Aggiorna le dipendenze
composer require nome/pacchetto  # Installa un nuovo pacchetto
```

### npm
```bash
npm install           # Installa le dipendenze
npm update            # Aggiorna le dipendenze
npm install nome-pacchetto  # Installa un nuovo pacchetto
npm run dev          # Compila gli asset in modalità sviluppo
npm run build        # Compila gli asset per la produzione
```

### Artisan (Laravel)
```bash
php artisan serve                 # Avvia il server di sviluppo
php artisan migrate               # Esegue le migrazioni del database
php artisan tinker                # Apre la console interattiva di Laravel
php artisan make:model NomeModello  # Crea un nuovo modello
php artisan make:controller NomeController  # Crea un nuovo controller
php artisan make:migration nome_migrazione  # Crea una migrazione
```

## Risoluzione dei Problemi

### Errore: "Composer non riconosciuto"
- Verifica che Composer sia nel PATH
- Riavvia il Prompt dei Comandi dopo l'installazione

### Errore: "Node non riconosciuto"
- Verifica che Node.js sia nel PATH
- Riavvia il Prompt dei Comandi dopo l'installazione

### Errore PowerShell: "npm.ps1 - L'esecuzione di script è disabilitata"
**Soluzione consigliata: Usa il Prompt dei Comandi (cmd) invece di PowerShell**
- Apri il **Prompt dei Comandi** (cmd) invece di PowerShell
- npm funziona naturalmente senza problemi di execution policy

Se preferisci usare PowerShell, puoi risolvere il problema in due modi:

**Opzione 1: Usa npm.cmd direttamente in PowerShell**
```powershell
npm.cmd install
npm.cmd start
```

**Opzione 2: Modifica la PowerShell Execution Policy**
- Apri PowerShell come **Amministratore**
- Esegui:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
- Conferma premendo `Y` e invio
- Da questo momento, npm funzionerà normalmente in PowerShell

### Errore di permessi su Windows
- Esegui il Prompt dei Comandi come **Amministratore**

### Errore di Composer durante `create-project`
- Verifica la connessione internet
- Prova a svuotare la cache di Composer:
  ```bash
  composer clear-cache
  ```

### Porte in Conflitto
Se la porta 8000 è già in uso:
```bash
php artisan serve --port=8001
```

## Conclusione

Con questa guida hai tutti gli strumenti necessari per iniziare a sviluppare con Laravel su Windows. Ricorda di mantenere aggiornati Composer, Node.js e i pacchetti installati per avere le ultime features e correzioni di sicurezza.
