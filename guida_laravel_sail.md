# Guida installazione Laravel Sail su Windows

## Requisiti
- Windows 10 o superiore
- Docker Desktop installato e configurato
- Composer installato
- Un progetto Laravel generato o esistente

## Preparazione
1. Su Docker Desktop, assicurati di avere abilitato WSL 2 nella sezione "Settings > General".
2. Nella sezione "Settings > Resources > WSL Integration", abilita l'integrazione per la distribuzione WSL che stai utilizzando (ad esempio, Ubuntu).
Se non hai ancora una distribuzione WSL, puoi installarla dal Microsoft Store (ad esempio, Ubuntu).

## Passaggi per l'installazione
1. Naviga nella directory del tuo progetto.
```bash
cd nome-del-tuo-progetto
```

2. Installa Laravel Sail come dipendenza di sviluppo.
```bash
composer require laravel/sail --dev
```

3. Esegui il comando per pubblicare i file di configurazione di Sail.
```bash
php artisan sail:install
```

4. Scegli i servizi che desideri utilizzare (ad esempio MySQL, Redis, ecc.) e conferma.

5. Avvia un terminale WSL
```bash
bash
cd /mnt/c/dir/del/tuo/progetto
```

6. Avvia i container Docker con Sail.
```bash
./vendor/bin/sail up
```

7. Creare un alias per Sail (opzionale ma consigliato):
```bash
nano ~/.bash_aliases
```

Aggiungi la seguente riga al file:
```bash
alias sail='bash vendor/bin/sail'
```
E ricarica il file .bashrc:
```bash
source ~/.bash_aliases
```

8. Migra il database (se necessario):
```bash
sail artisan migrate
```

9. Modifica il file `.env`, se necessario. Ad esempio:
```env
APP_URL=http://localhost
```

10. Ricarica le variabili d'ambiente:
```bash
sail artisan config:cache
```

9. Una volta avviati i container, puoi accedere al tuo progetto Laravel visitando `http://localhost` nel tuo browser.

## Comandi utili
- Per fermare i container:
```bash
./vendor/bin/sail down
```
- Per eseguire comandi Artisan all'interno del container:
```bash
./vendor/bin/sail artisan comando
```
- Per eseguire comandi Composer all'interno del container:
```bash
./vendor/bin/sail composer comando
```

## Risoluzione dei problemi
- Assicurati che Docker Desktop sia in esecuzione e configurato correttamente.
- Verifica che la porta 80 non sia occupata da altri servizi.
- Se incontri problemi di permessi, prova a eseguire il terminale come amministratore.
- Consulta la documentazione ufficiale di Laravel Sail per ulteriori dettagli e supporto: [Laravel Sail Documentation](https://laravel.com/docs/sail).

## Conclusione
Hai ora un ambiente di sviluppo Laravel completamente funzionante su Windows utilizzando Sail e Docker. Buon coding!