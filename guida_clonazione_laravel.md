# Guida: Clonazione e Setup di un Progetto Laravel

Questa guida spiega come clonare un progetto Laravel da GitHub e renderlo completamente funzionante nel tuo ambiente locale.

## Prerequisiti

Prima di iniziare, assicurati di avere installato:
- **Git** - per clonare il repository
- **PHP** - versione 8.0 o superiore
- **Composer** - gestione delle dipendenze PHP

## Passaggi di Setup

### 1. Clonare il Repository

Clona il progetto dal repository GitHub usando:

```bash
git clone https://github.com/username/nome-progetto.git
```

Sostituisci `https://github.com/username/nome-progetto.git` con il link del tuo repository GitHub.

Quindi accedi alla cartella del progetto:

```bash
cd nome-progetto
```

### 2. Installare le Dipendenze

Installa tutte le dipendenze PHP necessarie usando Composer:

```bash
composer install
```

Questo comando legge il file `composer.json` e scarica tutti i pacchetti richiesti.

### 3. Configurare il File .env

Copia il file di configurazione di esempio:

**Su Linux/macOS:**
```bash
cp .env.example .env
```

**Su Windows (PowerShell):**
```powershell
copy .env.example .env
```

**Su Windows (CMD):**
```cmd
copy .env.example .env
```

### 4. Generare la Chiave dell'Applicazione

Genera una chiave di crittografia univoca per l'applicazione:

```bash
php artisan key:generate
```

Questo comando crea una chiave casuale nel file `.env`.

### 5. Migrare il Database

Esegui le migrazioni per creare le tabelle nel database:

```bash
php artisan migrate
```

Assicurati che il database sia configurato correttamente nel file `.env` prima di eseguire questo comando.

### 6. Popolare il Database (Opzionale)

Se il progetto include dei seeders, puoi popolare il database con dati di prova:

```bash
php artisan db:seed
```

### 7. Avviare il Server di Sviluppo

Avvia il server di sviluppo integrato di Laravel:

```bash
php artisan serve
```

### 8. Accedere all'Applicazione

Apri il tuo browser e accedi all'applicazione:

- **Localhost:** `http://localhost:8000`
- **Indirizzo IP locale:** `http://127.0.0.1:8000`

## Riepilogo dei Comandi

Ecco una sequenza rapida di tutti i comandi:

```bash
git clone https://github.com/username/nome-progetto.git
cd nome-progetto
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan db:seed
php artisan serve
```

Quindi accedi a `http://localhost:8000` nel browser.

## Risoluzione dei Problemi Comuni

### Errore: "Class '.env' not found"
Assicurati di aver copiato il file `.env.example` in `.env` e di aver eseguito `php artisan key:generate`.

### Errore di Connessione al Database
Verifica le credenziali del database nel file `.env` e assicurati che il database esista e sia accessibile.

### Errore durante `composer install`
Aggiorna Composer eseguendo `composer update` e assicurati di avere una versione recente di PHP installata.

## Note Importanti

- Non committare mai il file `.env` su Git (contiene credenziali sensibili)
- Se il progetto ha un file `.env.example` aggiornato, rileggi questo file se aggiungi nuove variabili di configurazione
- Usa sempre `php artisan migrate` per aggiornare il database in caso di nuove migrazioni
