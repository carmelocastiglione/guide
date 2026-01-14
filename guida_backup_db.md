# Guida al Backup Sicuro di un Database MySQL

Questa guida illustra i metodi migliori per eseguire backup affidabili e sicuri di un database MySQL.

## 1. Preparazione e Prerequisiti

### Prerequisiti
- MySQL Server installato e in esecuzione
- Accesso con credenziali di amministratore (utente `root` o con privilegi `BACKUP ADMIN`)
- Spazio disco sufficiente per il backup
- Connettività di rete (se il database è remoto)

### Verificare la Connessione
```bash
mysql -u root -p -e "SELECT VERSION();"
```

## 2. Metodo 1: Backup con `mysqldump` (Consigliato)

### Vantaggi
- Backup in formato SQL leggibile
- Portabilità tra diversi sistemi
- Facilità di ripristino selettivo

### Backup di un Singolo Database
```bash
mysqldump -u root -p nome_database > backup_nome_database.sql
```

### Backup di Tutti i Database
```bash
mysqldump -u root -p --all-databases > backup_completo.sql
```

### Backup con Opzioni di Sicurezza Avanzate
```bash
mysqldump -u root -p \
  --all-databases \
  --single-transaction \
  --quick \
  --lock-tables=false \
  --verbose \
  --routines \
  --triggers \
  --events > backup_completo_$(date +%Y%m%d_%H%M%S).sql
```

### Spiegazione delle Opzioni
- `--single-transaction`: Usa una transazione coerente (migliore per InnoDB)
- `--quick`: Recupera dati riga per riga (riduce il carico di memoria)
- `--lock-tables=false`: Non blocca le tabelle (importante per ambienti in produzione)
- `--routines`: Include stored procedures e funzioni
- `--triggers`: Include i trigger del database
- `--events`: Include gli eventi schedulati
- `--verbose`: Mostra i progressi durante il backup

### Backup Compresso
```bash
mysqldump -u root -p nome_database | gzip > backup_nome_database.sql.gz
```

## 3. Metodo 2: Backup con Variabili d'Ambiente (Senza Password Visibile)

Crea un file `.my.cnf` nella home directory:

```bash
[mysqldump]
user=root
password=tuapassword
```

Imposta i permessi corretti:
```bash
chmod 600 ~/.my.cnf
```

Usa mysqldump senza password interattiva:
```bash
mysqldump --all-databases > backup_completo.sql
```

## 4. Metodo 3: Backup Automatico con Script Bash

Crea uno script `backup_mysql.sh`:

```bash
#!/bin/bash

# Configurazione
DB_USER="root"
DB_PASSWORD="tuapassword"
BACKUP_DIR="/path/to/backups"
DATE=$(date +%Y%m%d_%H%M%S)
LOG_FILE="$BACKUP_DIR/backup_$DATE.log"

# Crea directory se non esiste
mkdir -p $BACKUP_DIR

# Esegui backup
mysqldump -u $DB_USER -p$DB_PASSWORD \
  --all-databases \
  --single-transaction \
  --quick \
  --routines \
  --triggers \
  --events | gzip > $BACKUP_DIR/backup_completo_$DATE.sql.gz

# Verifica successo
if [ $? -eq 0 ]; then
  echo "Backup completato con successo: $BACKUP_DIR/backup_completo_$DATE.sql.gz" >> $LOG_FILE
  echo "Dimensione: $(du -h $BACKUP_DIR/backup_completo_$DATE.sql.gz | cut -f1)" >> $LOG_FILE
else
  echo "ERRORE: Backup fallito!" >> $LOG_FILE
  exit 1
fi

# Elimina backup più vecchi di 30 giorni
find $BACKUP_DIR -type f -name "backup_*.sql.gz" -mtime +30 -delete

echo "Operazione completata: $(date)" >> $LOG_FILE
```

Rendi lo script eseguibile:
```bash
chmod +x backup_mysql.sh
```

### Pianificazione con Cron (Backup Quotidiano)

Apri l'editor cron:
```bash
crontab -e
```

Aggiungi la seguente riga per eseguire il backup ogni giorno alle 2 AM:
```
0 2 * * * /path/to/backup_mysql.sh
```

Per ogni 6 ore:
```
0 */6 * * * /path/to/backup_mysql.sh
```

## 5. Metodo 4: Backup Incrementale (Binlog)

### Abilita Binary Logging in MySQL

Modifica `/etc/mysql/my.cnf`:

```ini
[mysqld]
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
expire_logs_days = 14
```

Riavvia MySQL:
```bash
sudo systemctl restart mysql
```

### Effettuare un Backup Incrementale

```bash
# Backup iniziale
mysqldump -u root -p --all-databases --single-transaction > backup_base.sql

# Visualizza posizione binlog
mysql -u root -p -e "SHOW MASTER STATUS;"

# Il backup incrementale è automaticamente registrato in binary log
```

## 6. Misure di Sicurezza Essenziali

### 6.1 Protezione dei File di Backup

```bash
# Cambia proprietario e permessi
sudo chown root:root backup_completo.sql.gz
sudo chmod 600 backup_completo.sql.gz
```

### 6.2 Crittografia del Backup

Con OpenSSL:
```bash
mysqldump -u root -p nome_database | gzip | openssl enc -aes-256-cbc -salt -out backup_criptato.sql.gz.enc
```

Per il ripristino:
```bash
openssl enc -d -aes-256-cbc -in backup_criptato.sql.gz.enc | gunzip | mysql -u root -p
```

### 6.3 Verifica dell'Integrità

```bash
# Calcola checksum
sha256sum backup_completo.sql.gz > backup_completo.sql.gz.sha256

# Verifica checksum
sha256sum -c backup_completo.sql.gz.sha256
```

### 6.4 Salvataggio Remoto (Best Practice)

```bash
# Con SCP
scp backup_completo.sql.gz user@remote_server:/remote/backup/path/

# Con rsync
rsync -avz --delete backup_completo.sql.gz user@remote_server:/remote/backup/path/
```

## 7. Ripristino da Backup

### Ripristino Completo

```bash
mysql -u root -p < backup_completo.sql
```

### Ripristino Compresso

```bash
gunzip < backup_completo.sql.gz | mysql -u root -p
```

### Ripristino Selettivo (Un Database)

```bash
mysql -u root -p nome_database < backup_nome_database.sql
```

### Ripristino da Backup Criptato

```bash
openssl enc -d -aes-256-cbc -in backup_criptato.sql.gz.enc | gunzip | mysql -u root -p
```

## 8. Checklist per un Backup Sicuro

- [ ] Backup eseguito regolarmente (almeno quotidianamente)
- [ ] File di backup protetti con permessi restrictivi (chmod 600)
- [ ] Backup crittografati con OpenSSL o simili
- [ ] Copia di backup conservata in location remota
- [ ] Backup periodicamente testati per verificare il ripristino
- [ ] Checksum calcolato e conservato separatamente
- [ ] Script di backup monitorato e loggato
- [ ] Password non hardcoded negli script (usare .my.cnf)
- [ ] Spazio disco monitorato per evitare riempimenti
- [ ] Rotazione dei backup per risparmiare spazio

## 9. Monitoraggio e Notifiche

### Script con Notifica Email

```bash
#!/bin/bash

BACKUP_DIR="/path/to/backups"
EMAIL="admin@example.com"
DATE=$(date +%Y%m%d_%H%M%S)
LOG_FILE="$BACKUP_DIR/backup_$DATE.log"

# Esegui backup
mysqldump -u root -p --all-databases | gzip > $BACKUP_DIR/backup_$DATE.sql.gz 2>> $LOG_FILE

# Controlla risultato
if [ $? -eq 0 ]; then
  SIZE=$(du -h $BACKUP_DIR/backup_$DATE.sql.gz | cut -f1)
  echo "✓ Backup completato - Dimensione: $SIZE" | mail -s "Backup MySQL OK" $EMAIL
else
  echo "✗ Backup fallito!" | mail -s "ERRORE Backup MySQL" $EMAIL
  exit 1
fi
```

## 10. Troubleshooting

### Errore: "Access denied for user"
- Verificare le credenziali di MySQL
- Assicurarsi che l'utente abbia i privilegi necessari

### Errore: "Got error 1045 from storage engine"
- Riavviare il servizio MySQL
- Controllare lo spazio disco disponibile

### Backup Lento
- Usare `--quick` per ridurre l'uso di memoria
- Usare `--single-transaction` per InnoDB
- Considerare backup off-peak (ore notturne)

### File di Backup Troppo Grande
- Comprimere il backup con gzip
- Effettuare backup incrementali
- Pulire vecchi backup automaticamente

## Conclusione

Un backup MySQL sicuro richiede:
1. **Frequenza**: Backup almeno giornalieri
2. **Affidabilità**: Test periodici di ripristino
3. **Sicurezza**: Crittografia e protezione dei file
4. **Ridondanza**: Copie in location diverse
5. **Monitoraggio**: Tracking dell'integrità dei backup

Ricorda: **Un backup non testato non è un backup!**
