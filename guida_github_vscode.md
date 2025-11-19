# Guida a Git e GitHub con Visual Studio Code

## Obiettivo
Imparare a **inviare i propri file su un branch di sviluppo** e fare il **merge** con il branch principale, usando **solo Visual Studio Code**.

## 1. Prerequisiti

Prima di iniziare, assicurati di avere:
- Un **account GitHub** → [https://github.com](https://github.com)
- **Git installato** → [https://git-scm.com/downloads](https://git-scm.com/downloads)
- **Visual Studio Code (VS Code)** → [https://code.visualstudio.com/](https://code.visualstudio.com/)
- L’estensione **GitHub Pull Requests and Issues** (facoltativa ma consigliata)
- L’estensione **Git Graph** (facoltativa ma consigliata)
- La configurazione iniziale di Git (da terminale):
```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

## 2. Clonare il repository in VS Code

1. Apri **VS Code**.  
2. Nella barra in alto o tramite **Ctrl + Shift + P**, scrivere **Git: clone** 
3. Clicca su **Clone from Github**.  
4. Seleziona il repository GitHub o incolla l’URL del repo (es. `https://github.com/nomeutente/nome-progetto.git`).  
5. Scegli una cartella locale dove salvare il progetto.  
6. Quando richiesto, clicca su **Apri il progetto**.

## 3. Creare un branch di sviluppo

1. In basso a sinistra, clicca sul nome del branch (di solito `main`).  
2. Seleziona **Create new branch...**  
3. Scrivi il nome del tuo branch, ad esempio `feature/login`.  
4. Premi **Invio**: ora stai lavorando su quel branch.

## 4. Aggiungere e salvare i file

1. Aggiungi o modifica i file del progetto nella cartella.  
2. Clicca sull’icona **Source Control** (rametto).  
3. Vedrai tutti i file modificati.  
4. Clicca su **+** accanto ai file per aggiungerli, oppure su **+ Changes** per aggiungerli tutti.  
5. Nella barra in alto, clicca sull'icona per generare un messaggio di *commit* oppure scrivi un messaggio (es. “Aggiunti file iniziali”) e premi **Commit** per confermare.

## 5. Inviare il branch su GitHub

1. Dopo aver fatto il commit, clicca su **Sincronizza le modifiche** (icona con le due frecce circolari).  
2. VS Code ti chiederà di pubblicare il branch: clicca **Pubblica branch**.  
3. Ora il tuo branch è visibile anche su GitHub.

## 6. Creare una Pull Request da VS Code

Hai due opzioni:

### Metodo 1: Da Visual Studio Code (consigliato)
1. Apri la **Command Palette** (Ctrl + Shift + P) e digita `GitHub: Create Pull Request`, oppure tramite l'icona Create Pull Request nel menù Changes
3. Seleziona il branch principale come **base** (`main`) e il tuo come **compare** (`sviluppo-mario`).  
4. Inserisci titolo e descrizione, poi conferma.  

### Metodo 2: Da GitHub
1. Vai su GitHub e apri il repository.  
2. Ti verrà mostrato un messaggio: **Compare & pull request**.  
3. Clicca e conferma la creazione della PR.

## 7. Fare il Merge (unire le modifiche)

Può farlo:
- Il **docente** o **responsabile del progetto**.
- Oppure tu, se hai i permessi.

### In GitHub:
1. Apri la Pull Request.  
2. Se non ci sono conflitti, clicca **Merge pull request** → **Confirm merge**.  
3. Puoi eliminare il branch remoto cliccando su **Delete branch**.

## 8. Aggiornare la copia locale dopo il merge

1. In VS Code, clicca di nuovo sul nome del branch (in basso a sinistra).  
2. Seleziona `main`.  
3. Clicca su **Sincronizza** per aggiornare i file del branch principale con le ultime modifiche dal server.

## Suggerimenti utili

- Usa **nomi chiari** per i branch (`sviluppo/mario`, `feature/login`, ecc.).  
- Fai **commit frequenti** con messaggi descrittivi.  
- Se appaiono conflitti, VS Code li evidenzia nel codice: scegli quali versioni mantenere e salva.  
- Dopo il merge, puoi eliminare i branch che non servono più.
