# Guida alla Configurazione Sicura del Repository GitHub per un Team di Sviluppo Web

## 1. Crea l’organizzazione o il repository

Hai due possibilità:

### A) Usare un’organizzazione GitHub (consigliato per team)

* Vai su **GitHub → + → New Organization**
* Crea un team dedicato
* Aggiungi i collaboratori
* Crei dentro l'organizzazione il repository del progetto (es. `webapp-2025`)
  - Nome chiaro e descrittivo.
  - Privato se il codice non deve essere pubblico.
  - Inizializza con un `README.md` e, se serve, un `.gitignore` per il tuo stack (Node, Python, PHP, ecc.).
  - Scegli una licenza se necessario (MIT, GPL, ecc.).

### B) Usare un singolo repository

* Clicca **New Repository**
* Impostalo come pubblico o privato
* Aggiungi i collaboratori nel tab **Settings → Collaborators**
   
## 3. Clona il repository localmente:

```bash
git clone https://github.com/<utente>/<repo>.git
cd <repo>
````

### File importanti che dovrebbero essere presenti:

* **README.md** → spiega obiettivi, struttura, come installare ed eseguire
* **CONTRIBUTING.md** → regole per contribuire (branch naming, standard di codice, ecc.)
* **ISSUE_TEMPLATE.md** → modello per segnalare problemi o richiedere feature
* **PULL_REQUEST_TEMPLATE.md** → modello per le PR

## 4. Struttura consigliata dei branch

| Branch | Scopo | Protezione |
|--------|--------|-------------|
| `main` | Versione stabile e pronta per la produzione | Molto protetto |
| `develop` | Integrazione delle funzionalità testate | Protetto, ma più flessibile |
| `feature/*` | Branch temporanei per nuove funzionalità | Pochi vincoli |
| `fix/*` | Per le correzioni| Pochi vincoli |

Esempi:

```
feature/login-page
feature/api-users
fix/header-layout
```

> Tutti i merge verso `main` e `develop` devono avvenire tramite **pull request** approvate e testate.

## 5. Impostazione dei Ruleset

Regole di base:

* **Branch `main`**:

  * Blocca i push diretti.
  * Richiedi pull request prima del merge.
  * Richiedi almeno 1 approvazione (reviewers).

* **Branch `develop`**:

  * Blocca force push.
  * Richiedi PR per le modifiche.
  * Minimo 1 approvatore.

* **Branch `feature/*` e `fix/*`**:

  * Consentito sviluppo rapido e rebase locali.
  * Pochi vincoli, può essere cancellato dopo merge.

### Dove creare un Ruleset
1. Vai su **Settings → Rules → Rulesets → New ruleset**
2. Assegna un nome (es. `Main branch protection`)
3. Configura le regole come indicato sotto.

### Ruleset per `main`

| Sezione | Impostazione consigliata | Descrizione |
|----------|---------------------------|-------------|
| **Target** | `Include → Branch name pattern → main` | Applica le regole al branch principale |
| **Push rules** | ✅ *Restrict force pushes*<br>✅ *Restrict deletions* | Evita cancellazioni o push forzati |
| **Pull Requests** | ✅ *Require pull request before merging* | Obbliga all’uso delle PR |
| **Approvals** | ✅ *Require approvals → 1 approver* | Almeno una revisione prima del merge |
| **Code owners** | ✅ *Require review from code owners* | Se esiste un file `CODEOWNERS` |
| **Enforcement** | Apply to → *Everyone* o *Everyone except admins* | Definisce chi deve rispettare le regole |

### Ruleset per `develop`

| Sezione | Impostazione | Descrizione |
|----------|---------------|-------------|
| **Target** | `develop` | Applica al branch di sviluppo |
| **Require PR** | ✅ Sì | Tutte le modifiche passano da PR |
| **Approvals** | ✅ 1 approvatore | Controllo base sul codice |
| **Force push / Deletion** | ❌ Vietato | Mantiene la storia pulita |

### Ruleset per `feature/*` e `fix/*`

| Sezione | Impostazione | Descrizione |
|----------|--------------|-------------|
| **Target** | `feature/*` | Tutti i branch di sviluppo |
| **Require PR** | ❌ No | Permette sviluppo rapido |
| **Force push** | ✅ Sì | Utile per rebase locali |
| **Auto-cleanup** | (facoltativo) | Rimozione automatica branch inattivi |

## 6. Gestione del lavoro: GitHub Issues

### Ogni incarico = una issue

Per ogni funzionalità o problema crea una Issue.

Esempio di Issue:

```
Titolo: Implementare pagina di login

Descrizione:
- Form email + password
- Validazione lato frontend
- Collegamento API
- Test responsività

Assegnato a: @mario
Etichetta: feature, frontend
Deadline: 25 novembre
```

### Etichette utili:

* feature
* fix
* bug
* frontend
* backend
* high priority
* in progress
* to review
* documentation

## 7. Monitora lo stato con GitHub Projects (Kanban)

GitHub Projects ti permette di creare una **lavagna tipo Trello**, direttamente collegata con Issues e PR.

Tipica board Kanban:

### To Do

* Tutte le Issues pronte da assegnare

### In Progress

* Incarichi su cui il team sta lavorando

### In Review

* PR che aspettano revisione

### Done

* Completati

Ogni membro del team può spostare la sua issue quando cambia fase.

## 8. Workflow consigliato per il team

### 1. Il project manager crea:

* Issues per ogni incarico (con eventuali label e milestone)
* Board Kanban
* Assegna i compiti al team

### 2. Ogni sviluppatore:

* Si prende una Issue
* Crea il branch `feature/*` o `fix/*`
* Sviluppa
* Fa push
* Apre una Pull Request verso `develop`

### 3. Tu o un reviewer:

* Controlli la PR
* Commenti eventuali modifiche
* Approvazione
* Merge su `develop`

### 4. Alla fine del ciclo:

* Merge da `develop` a `main`
* Deploy

## 9. Gestione dei Ruoli Utente

### Ruoli base nel repository

| Ruolo | Permessi principali | Uso consigliato |
|--------|----------------------|----------------|
| **Read** | Leggere, clonare | Clienti, revisori, stakeholder |
| **Triage** | Gestire issue e PR | QA tester o project manager |
| **Write** | Creare branch, pushare, aprire PR | Sviluppatori |
| **Maintain** | Gestire PR, issue e impostazioni minori | Team leader o dev senior |
| **Admin** | Tutti i permessi (anche eliminare repo) | Owner del progetto |

*Percorso:*  
`Repository → Settings → Collaborators → Add people → Seleziona ruolo`

### Ruoli a livello di Organizzazione (consigliato per team)

| Ruolo | Descrizione |
|--------|-------------|
| **Owner** | Controllo completo su tutti i repository |
| **Member** | Accesso limitato, configurabile per repo |
| **Billing Manager** | Gestisce solo la fatturazione (non necessario) |

## 10. Creazione di Team (solo per Organizzazioni)

1. Vai su `Organization → Teams → New team`
2. Crea i gruppi di lavoro:
```text
WebProject
├── frontend-devs (Write)
├── backend-devs (Write)
├── maintainers (Maintain)
└── admins (Admin)
```
3. Assegna i team ai repository → *Add repository → Seleziona ruolo*

**Vantaggi:**
- Gestione semplificata dei permessi
- Aggiunta/rimozione membri immediata
- Controllo centralizzato su più progetti

## 11. File `CODEOWNERS`

Crea il file `.github/CODEOWNERS` per definire chi deve approvare le modifiche a specifiche parti del codice:

```text
# Tutto il progetto deve essere revisionato da @owneruser e @teamlead
* @owneruser @teamlead

# Sezioni specifiche
/backend/ @backend-dev
/frontend/ @frontend-dev
```

Se hai attivato la regola *Require review from code owners*, nessun merge sarà possibile senza la loro approvazione.

## 12. Best Practice di Sicurezza

✅ Attiva **autenticazione a due fattori (2FA)** per tutti i membri.\
✅ Blocca **push diretti** su `main` e `develop`.\
✅ Usa **GitHub Secrets** per le variabili d’ambiente.\
✅ Attiva **secret scanning** e **dependabot alerts**.\
✅ Nomina **almeno due Admin/Owner** per evitare blocchi.

## 13. Ruoli consigliati per un piccolo team (esempio)

| Membro             | Ruolo    | Branch accesso | Responsabilità               |
| ------------------ | -------- | -------------- | ---------------------------- |
| Project Owner      | Admin    | Tutti          | Gestione generale e release  |
| Team Leader        | Maintain | develop        | Revisione e merge            |
| Dev Frontend       | Write    | feature/* e fix/*      | Sviluppo interfacce          |
| Dev Backend        | Write    | feature/* e fix/*     | Sviluppo logica e API        |
| QA / Tester        | Triage   | develop        | Verifica e issue tracking    |
| Cliente / Revisore | Read     | main           | Consultazione codice stabile |

## 14. Riepilogo

| Area         | Regole chiave                                                                  |
| ------------ | ------------------------------------------------------------------------------ |
| **main**     | Merge solo via PR approvata, test obbligatori, no push diretti                 |
| **develop**  | PR con una review, no force push                                               |
| **feature/** e **fix/** | Lavoro libero, poi merge su `develop`                                          |
| **Ruoli**    | Admin (1–2), Maintain (1–2), Write (sviluppatori), Triage (QA), Read (clienti) |



## 15. Best Practice

* Usa **pull request sempre**.
* Blocca **push diretti su branch protetti**.
* Abilita **2FA per tutti i membri**.
* Mantieni `develop` stabile, non solo un branch di test.
* Mantieni una documentazione chiara (`README`, `CONTRIBUTING.md`).

## 16. Risultato finale
* Ogni compito è tracciato
* Ogni stato (da fare → in corso → review → fatto) è visibile
* Ogni modifica è controllata
* Tutto il team lavora in modo ordinato e professionale