# Guida alla Configurazione Sicura del Repository GitHub per un Team di Sviluppo Web

## Obiettivo
Impostare correttamente **utenti, ruoli e regole (ruleset)** per lavorare in modo collaborativo su GitHub **senza rischiare errori o danni** al codice in produzione.

## 1. Creazione del repository

1. Accedi a GitHub e crea un nuovo repository:
   - Nome chiaro e descrittivo (es. `progetto-web`).
   - Privato se il codice non deve essere pubblico.
   - Inizializza con un `README.md` e, se serve, un `.gitignore` per il tuo stack (Node, Python, PHP, ecc.).
   - Scegli una licenza se necessario (MIT, GPL, ecc.).

2. Clona il repository localmente:

```bash
git clone https://github.com/<utente>/<repo>.git
cd <repo>
````

## 2. Struttura consigliata dei branch

| Branch | Scopo | Protezione |
|--------|--------|-------------|
| `main` | Versione stabile e pronta per la produzione | Molto protetto |
| `develop` | Integrazione delle funzionalità testate | Protetto, ma più flessibile |
| `feature/*` | Branch temporanei per nuove funzionalità | Pochi vincoli |

> Tutti i merge verso `main` devono avvenire tramite **pull request** approvate e testate.

## 3. Impostazione dei Ruleset

Regole di base:

* **Branch `main`**:

  * Blocca i push diretti.
  * Richiedi pull request prima del merge.
  * Richiedi almeno 1 approvazione (reviewers).

* **Branch `develop`**:

  * Blocca force push.
  * Richiedi PR per le modifiche.
  * Minimo 1 approvatore.

* **Branch `feature/*`**:

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

---

### Ruleset per `feature/*`

| Sezione | Impostazione | Descrizione |
|----------|--------------|-------------|
| **Target** | `feature/*` | Tutti i branch di sviluppo |
| **Require PR** | ❌ No | Permette sviluppo rapido |
| **Force push** | ✅ Sì | Utile per rebase locali |
| **Auto-cleanup** | (facoltativo) | Rimozione automatica branch inattivi |

## 4. Workflow consigliato

1. **Creazione di feature branch**: ogni funzionalità viene sviluppata su un branch separato (`feature/login`, `feature/carrello`).
2. **Pull request su `develop`**: una volta pronta, la feature viene unita in `develop`.
3. **Test e revisione**: il team rivede il codice 
4. **Merge su `main`**: dopo approvazione, il branch `develop` viene unito in `main` per la produzione.

## 5. Gestione dei Ruoli Utente

### Ruoli base nel repository

| Ruolo | Permessi principali | Uso consigliato |
|--------|----------------------|----------------|
| **Read** | Leggere, clonare | Clienti, revisori, stakeholder |
| **Triage** | Gestire issue e PR | QA tester o project manager |
| **Write** | Creare branch, pushare, aprire PR | Sviluppatori |
| **Maintain** | Gestire PR, issue e impostazioni minori | Team leader o dev senior |
| **Admin** | Tutti i permessi (anche eliminare repo) | Owner del progetto |

👉 *Percorso:*  
`Repository → Settings → Collaborators → Add people → Seleziona ruolo`

---

### Ruoli a livello di Organizzazione (consigliato per team)

| Ruolo | Descrizione |
|--------|-------------|
| **Owner** | Controllo completo su tutti i repository |
| **Member** | Accesso limitato, configurabile per repo |
| **Billing Manager** | Gestisce solo la fatturazione (non necessario) |

---

## 6. Creazione di Team (solo per Organizzazioni)

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

## 7. File `CODEOWNERS`

Crea il file `.github/CODEOWNERS` per definire chi deve approvare le modifiche a specifiche parti del codice:

```text
# Tutto il progetto deve essere revisionato da @owneruser e @teamlead
* @owneruser @teamlead

# Sezioni specifiche
/backend/ @backend-dev
/frontend/ @frontend-dev
```

Se hai attivato la regola *Require review from code owners*, nessun merge sarà possibile senza la loro approvazione.

## 8. Best Practice di Sicurezza

✅ Attiva **autenticazione a due fattori (2FA)** per tutti i membri.
✅ Blocca **push diretti** su `main` e `develop`.
✅ Usa **GitHub Secrets** per le variabili d’ambiente.
✅ Attiva **secret scanning** e **dependabot alerts**.
✅ Nomina **almeno due Admin/Owner** per evitare blocchi.

## 9. Ruoli consigliati per un piccolo team (esempio)

| Membro             | Ruolo    | Branch accesso | Responsabilità               |
| ------------------ | -------- | -------------- | ---------------------------- |
| Project Owner      | Admin    | Tutti          | Gestione generale e release  |
| Team Leader        | Maintain | develop        | Revisione e merge            |
| Dev Frontend       | Write    | feature/*      | Sviluppo interfacce          |
| Dev Backend        | Write    | feature/*      | Sviluppo logica e API        |
| QA / Tester        | Triage   | develop        | Verifica e issue tracking    |
| Cliente / Revisore | Read     | main           | Consultazione codice stabile |

## 10. Riepilogo

| Area         | Regole chiave                                                                  |
| ------------ | ------------------------------------------------------------------------------ |
| **main**     | Merge solo via PR approvata, test obbligatori, no push diretti                 |
| **develop**  | PR con una review, no force push                                               |
| **feature/** | Lavoro libero, poi merge su `develop`                                          |
| **Ruoli**    | Admin (1–2), Maintain (1–2), Write (sviluppatori), Triage (QA), Read (clienti) |



## 11. Best Practice

* Usa **pull request sempre**.
* Blocca **push diretti su branch protetti**.
* Abilita **2FA per tutti i membri**.
* Mantieni `develop` stabile, non solo un branch di test.
* Mantieni una documentazione chiara (`README`, `CONTRIBUTING.md`).