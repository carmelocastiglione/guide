# Guida completa: installare Laravel con Sail (zero PHP, Composer e Node locali)

Questa guida mostra come installare **Laravel usando esclusivamente Docker e Laravel Sail**, senza installare **PHP**, **Composer** o **Node.js** sul computer.

È una soluzione ideale per:

* laboratorio scolastico
* ambienti misti (Windows / Linux / macOS)
* evitare conflitti di versione

## Requisiti

Serve **solo**:

* **Docker Desktop** installato e **avviato**

Nient’altro.

## STEP 1 - Scegli la cartella di lavoro

Apri il terminale (PowerShell su Windows) e posizionati nella cartella dove vuoi creare il progetto:

```powershell
cd C:\Users\TuoNome\Documents
```

## STEP 2 - Crea un nuovo progetto Laravel

Usiamo un container Docker con PHP e Composer già pronti.

```powershell
docker run --rm `
  -v ${PWD}:/var/www/html `
  -w /var/www/html `
  laravelsail/php84-composer:latest `
  composer create-project laravel/laravel myapp
```

`myapp` è il nome del progetto (puoi cambiarlo).

Alla fine verrà creata la cartella:

```
myapp/
```

## STEP 3 - Entra nel progetto

```powershell
cd myapp
```

## STEP 4 - Installa Laravel Sail

Anche questo comando viene eseguito **senza PHP locale**, usando Docker:

```powershell
docker run --rm `
  -v ${PWD}:/var/www/html `
  -w /var/www/html `
  laravelsail/php84-composer:latest `
  php artisan sail:install
```

Durante la procedura seleziona:

* `mysql` (consigliato)
* `redis` / `mailpit` (opzionali)

Premi **Invio** per confermare.

## STEP 5 - Avvia Sail

```powershell
.\vendor\bin\sail up -d
```

Al primo avvio Docker scaricherà le immagini necessarie.

## STEP 6 - Apri Laravel nel browser

Apri il browser e vai su:

**[http://localhost](http://localhost)**

Se vedi la pagina di benvenuto di Laravel, l’installazione è completata

## Regola fondamentale

Da questo momento **NON usare mai**:

```text
php
composer
npm
```

Tutti i comandi vanno eseguiti tramite **Sail**.

## STEP 7 - Crea una shortcut per Sail (consigliato)

### Windows (PowerShell)

Crea una funzione per usare `sail` come comando breve.

Apri il profilo PowerShell:

```powershell
notepad $PROFILE
```

Incolla in fondo al file:

```powershell
function sail {
    if (Test-Path ".\\vendor\\bin\\sail") {
        .\\vendor\\bin\\sail @args
    } else {
        Write-Host "Sail non trovato nella cartella corrente"
    }
}
```

Salva e **riapri PowerShell**.

## Verifica shortcut

Dentro la cartella del progetto:

```powershell
sail ps
```

## Comandi Sail più usati

```powershell
sail up -d
sail down
sail artisan migrate
sail artisan make:controller TestController
sail npm install
sail npm run dev
```

## Controllo versioni (opzionale)

```powershell
sail php -v
sail composer --version
sail node -v
```

## Vantaggi di questa soluzione

* nessuna installazione locale
* ambiente identico per tutti
* ideale per didattica
* introduce Docker in modo pratico

## Riepilogo rapido

| Componente | Dove gira   |
| ---------- | ----------- |
| PHP        | Container   |
| Composer   | Container   |
| Node.js    | Container   |
| MySQL      | Container   |
| PC         | Solo Docker |

## Fine

Laravel è ora pronto all’uso con Sail.

Da qui puoi iniziare a sviluppare, migrare il database o lavorare sul frontend usando **solo Docker**.
