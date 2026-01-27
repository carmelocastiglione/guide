# Guida Git - Comandi Principali

Una guida completa ai comandi Git per il controllo di versione e la collaborazione nello sviluppo del codice.

## 1. Configurazione iniziale di Git

Prima di iniziare ad usare Git, è necessario configurare il tuo nome e la tua email. Questi dati saranno associati ai tuoi commit.

### Configurare nome e email
```bash
git config --global user.name "Il Tuo Nome"
git config --global user.email "tua.email@example.com"
```

### Verificare la configurazione
```bash
git config --list
```

## 2. Iniziare un nuovo repository

E' possibile inizializzare una directory locale (ad esempio per utilizzare Git su un nuovo progetto) oppure clonare un repository (progetto) già esistente, ad esempio su Github

### Creare un nuovo repository locale
```bash
git init
```
Questo crea una nuova cartella `.git` nel tuo progetto.

### Clonare un repository remoto
```bash
git clone <URL_REPOSITORY>
git clone <URL_REPOSITORY> <NOME_CARTELLA>  # con nome personalizzato
```

## 3. Verificare lo Stato

### Visualizzare lo stato del repository
```bash
git status
```
Mostra i file modificati, non tracciati e in staging.

### Visualizzare i cambiamenti (diff)
```bash
git diff                    # differenze dei file non staged
git diff --staged           # differenze dei file in staging
git diff <BRANCH1> <BRANCH2> # differenze tra branch
```

## 4. Aggiungere Modifiche (Staging)

L'area di stage rappresenta una sorta di entità intermedia tra la directory di lavoro e la directory che conterrà i cambiamenti ai file; una volta eseguite le operazioni relative allo staging, sarà possibile effettuare il commit delle modifiche apportate al proprio progetto, tenendo a mente però che qualsiasi elemento unstaged, cioè qualunque file che sia stato creato o modificato senza essere stato passato come argomento a git add, non potrà essere committato.

### Aggiungere file specifici
```bash
git add <NOME_FILE>
git add <CARTELLA>/
```

### Aggiungere tutti i file modificati
```bash
git add .
git add --all
```

### Rimuovere file dalla staging area
```bash
git reset <NOME_FILE>
git reset                   # rimuove tutti i file
```

## 5. Commit

Un commit in Git è un'istantanea del progetto in un determinato momento. Quando si crea un commit, Git tiene traccia di tutti i cambiamenti presenti nell'area di staging. 

### Creare un commit
```bash
git commit -m "Messaggio del commit"
```

### Modificare l'ultimo commit
```bash
git commit --amend
git commit --amend --no-edit  # senza modificare il messaggio
```

### Visualizzare la cronologia dei commit
```bash
git log                        # cronologia completa
git log --oneline              # formato compatto
git log --graph --all          # con rami visualizzati
git log -p                      # mostra le modifiche
git log -n 5                    # ultimi 5 commit
git log --author="Nome"         # commit di un autore specifico
```

## 6. Rami (Branches)

### Visualizzare i rami
```bash
git branch                  # rami locali
git branch -a               # tutti i rami (locali e remoti)
git branch -r               # solo rami remoti
```

### Creare un nuovo ramo
```bash
git branch <NOME_RAMO>
```

### Spostarsi su un ramo
```bash
git checkout <NOME_RAMO>
git switch <NOME_RAMO>      # sintassi più recente
```

### Creare e spostarsi su un nuovo ramo
```bash
git checkout -b <NOME_RAMO>
git switch -c <NOME_RAMO>   # sintassi più recente
```

### Eliminare un ramo
```bash
git branch -d <NOME_RAMO>        # elimina se già mergiato
git branch -D <NOME_RAMO>        # forza l'eliminazione
git push origin --delete <NOME_RAMO>  # elimina da remoto
```

### Rinominare un ramo
```bash
git branch -m <NOME_VECCHIO> <NOME_NUOVO>
git branch -m <NOME_NUOVO>   # rinomina il ramo corrente
```

## 7. Merge

### Unire un ramo nel ramo corrente
```bash
git merge <NOME_RAMO>
```

### Unire senza creare un commit di merge
```bash
git merge --squash <NOME_RAMO>
git commit -m "Merge squash"
```

### Annullare un merge in corso
```bash
git merge --abort
```

## 8. Rebase

### Riapplicare i commit su un altro ramo
```bash
git rebase <NOME_RAMO>
```

### Rebase interattivo (modificare i commit)
```bash
git rebase -i HEAD~3   # ultimi 3 commit
```

### Annullare un rebase in corso
```bash
git rebase --abort
```

## 9. Repository Remoto

### Visualizzare i repository remoti
```bash
git remote                  # nomi
git remote -v               # URL
git remote show <NOME>      # dettagli completi
```

### Aggiungere un repository remoto
```bash
git remote add <NOME> <URL>
git remote add origin <URL>
```

### Modificare l'URL di un repository remoto
```bash
git remote set-url <NOME> <NUOVO_URL>
git remote set-url origin <NUOVO_URL>
```

### Rinominare un repository remoto
```bash
git remote rename <NOME_VECCHIO> <NOME_NUOVO>
```

### Rimuovere un repository remoto
```bash
git remote remove <NOME>
```

## 10. Push e Pull

### Inviare i commit al repository remoto
```bash
git push                          # al ramo tracciato
git push <REMOTO> <RAMO>          # a un ramo specifico
git push origin main              # esempio
git push --all                    # tutti i rami
git push --tags                   # tutti i tag
```

### Recuperare i cambiamenti dal repository remoto
```bash
git fetch                         # scarica senza unire
git fetch <REMOTO>                # da un remoto specifico
git fetch --all                   # da tutti i remoti
```

### Recuperare e unire i cambiamenti
```bash
git pull                          # fetch + merge
git pull <REMOTO> <RAMO>
git pull --rebase                 # fetch + rebase
```

### Impostare il ramo upstream
```bash
git push -u origin <RAMO>         # imposta il tracciamento
git branch --set-upstream-to=origin/<RAMO>
```

## 11. Stash (Salvataggio Temporaneo)

### Salvare le modifiche temporaneamente
```bash
git stash
git stash save "Descrizione"
```

### Visualizzare gli stash salvati
```bash
git stash list
```

### Ripristinare uno stash
```bash
git stash pop                     # ripristina e elimina
git stash pop stash@{0}           # uno specifico
git stash apply                   # ripristina senza eliminare
```

### Eliminare uno stash
```bash
git stash drop stash@{0}
git stash clear                   # elimina tutti
```

## 12. Tag

### Creare un tag
```bash
git tag <NOME_TAG>                           # tag leggero
git tag -a <NOME_TAG> -m "Descrizione"      # tag annotato
```

### Visualizzare i tag
```bash
git tag                           # lista di tutti i tag
git tag -l "v*"                   # con pattern
git show <NOME_TAG>               # dettagli di un tag
```

### Inviare i tag al remoto
```bash
git push origin <NOME_TAG>
git push origin --tags            # tutti i tag
```

### Eliminare un tag
```bash
git tag -d <NOME_TAG>             # locale
git push origin --delete <NOME_TAG>  # remoto
```

## 13. Annullare Modifiche

### Scartare le modifiche di un file
```bash
git checkout -- <NOME_FILE>
git restore <NOME_FILE>           # sintassi più recente
```

### Ripristinare il repository a un commit precedente
```bash
git reset <COMMIT>                # soft: mantiene le modifiche in staging
git reset --hard <COMMIT>         # elimina tutte le modifiche
git reset --mixed <COMMIT>        # default: modifiche in working dir
```

### Revert (creare un nuovo commit che annulla)
```bash
git revert <COMMIT>
```

### Visualizzare la cronologia di reset/revert
```bash
git reflog
```

## 14. Cherry-pick

### Applicare un commit specifico al ramo corrente
```bash
git cherry-pick <COMMIT>
git cherry-pick <COMMIT1> <COMMIT2>
```

### Annullare un cherry-pick in corso
```bash
git cherry-pick --abort
```

## 15. Ricerca e Debug

### Trovare il commit che ha introdotto un bug
```bash
git bisect start
git bisect bad                    # il commit corrente è cattivo
git bisect good <COMMIT>          # un commit noto buono
# Git ti guiderà nel restringere il commit problematico
```

### Visualizzare chi ha modificato una linea
```bash
git blame <NOME_FILE>
```

### Cercare in tutta la cronologia
```bash
git log -S "<TESTO>"              # commit che aggiungono/rimuovono il testo
git log --grep="<PATTERN>"        # nel messaggio del commit
```

## 16. Configurazioni Utili

### Alias per comandi lunghi
```bash
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.ci "commit"
git config --global alias.st "status"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
git config --global alias.visual "log --graph --oneline --all"
```

### Usare gli alias
```bash
git co <RAMO>
git br -a
git ci -m "Messaggio"
```

## 17. Consigli Utili

### Workflow consigliato
1. `git pull` - Aggiorna il codice locale
2. Modifica i file
3. `git status` - Verifica lo stato
4. `git add .` - Prepara i file
5. `git commit -m "Descrizione"` - Crea il commit
6. `git push` - Invia al remoto

### Evitare errori comuni
- Non usare mai `git push -f` su rami pubblici (usa `git push --force-with-lease`)
- Fai regolarmente `git pull` per sincronizzare il tuo lavoro
- Crea branch separati per ogni feature o bug fix
- Scrivi messaggi di commit significativi

## 18. Comandi di Emergenza

### Recuperare un commit eliminato
```bash
git reflog                        # trova l'ID del commit
git checkout <COMMIT_ID>
git merge <COMMIT_ID>             # o unisci di nuovo
```

### Annullare l'ultimo push
```bash
git push --force-with-lease origin HEAD~1:main
```

### Estrarre un file da un commit precedente
```bash
git checkout <COMMIT> -- <NOME_FILE>
```