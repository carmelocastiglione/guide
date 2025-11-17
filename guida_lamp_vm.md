# Guida Completa alla Configurazione di un Server Linux (Ubuntu Server)

Questa guida ti accompagna passo dopo passo nell’installazione e configurazione di un server Linux basato su **Ubuntu Server**, includendo setup iniziale, rete, Apache, MySQL e ora anche **PHP**.

## 1. Scaricare Ubuntu Server

Scarica l’immagine ISO:\
👉 [https://ubuntu.com/download/server](https://ubuntu.com/download/server)

## 2. Preparare la Macchina Virtuale

Impostazioni consigliate:

* **RAM:** 2 GB min
* **CPU:** 2 core
* **Disco:** ≥ 20 GB
* **Rete:** modalità **Bridge**
* **Boot ISO:** caricare l’immagine di Ubuntu Server

## 3. Installare il Sistema Operativo

Durante la procedura:

* seleziona lingua, tastiera e rete
* crea utente amministratore
* attiva "Install OpenSSH" se necessario
* mantieni il partizionamento automatico

## 4. Aggiornare il Sistema

```bash
sudo apt-get update
sudo apt-get upgrade
```

## 5. Verificare l’Indirizzo IP

```bash
ip a
```

Annota l’indirizzo per accedere al server da browser o SSH.

## 6. Installare e Configurare Apache

```bash
sudo apt-get install apache2
```

### Creazione directory del sito

```bash
sudo mkdir /var/www/test/
cd /var/www/test
sudo nano index.html
```

Inserisci un contenuto semplice:

```html
<h1>Server Test</h1>
<p>Apache funziona!</p>
```

### Virtual Host

```bash
cd /etc/apache2/sites-available/
sudo nano 000-default.conf
```

Modifica:

```apache
DocumentRoot /var/www/test
```

Poi abilita il sito:

```bash
sudo a2ensite 000-default.conf
sudo systemctl restart apache2
```

## 7. Installare MySQL

```bash
sudo apt-get install mysql-server
```

Mettere in sicurezza MySQL:

```bash
sudo mysql_secure_installation
```

## 8. Installare PHP e Configurarlo con Apache

Per eseguire script dinamici come applicazioni web in PHP, installiamo il pacchetto principale e i moduli utili.

### 8.1 Installazione di PHP

Installa PHP e il modulo di integrazione con Apache:

```bash
sudo apt-get install php libapache2-mod-php php-mysql
```

Verificare la versione installata

```bash
php -v
```

### 8.2 Creare un file PHP di test

Sostituiamo il file HTML con un file PHP:

```bash
sudo nano /var/www/test/index.php
```

Inserisci:

```php
<?php
phpinfo();
?>
```

Questo file mostrerà tutte le informazioni su PHP, moduli installati e configurazione Apache.

Riavvia Apache

```bash
sudo systemctl restart apache2
```

Ora visita nel browser:

```
http://<IP-del-server>/index.php
```

Dovresti vedere la pagina **phpinfo()**.

## 8.3 Moduli PHP utili da installare

Puoi installare moduli aggiuntivi in base alle necessità del tuo progetto:

```bash
sudo apt-get install php-cli php-curl php-zip php-xml php-mbstring php-gd php-intl
```

Poi riavvia Apache:

```bash
sudo systemctl restart apache2
```

## 8.4 Modificare il file php.ini

Il file di configurazione si trova tipicamente in:

```
/etc/php/<versione>/apache2/php.ini
```

Esempio:

```bash
sudo nano /etc/php/8.1/apache2/php.ini
```

Parametri utili da modificare:

* `upload_max_filesize`
* `post_max_size`
* `memory_limit`
* `max_execution_time`

Dopo ogni modifica:

```bash
sudo systemctl restart apache2
```

## 9. Test Conclusivi

Verifica servizi attivi:

```bash
systemctl status apache2
systemctl status mysql
```

Controlla la pagina PHP:

```
http://<IP>/index.php
```

## 10. Conclusione

Ora il tuo server Ubuntu è configurato con:

* Apache (web server)
* MySQL (database)
* PHP (interprete per applicazioni dinamiche)

Hai un ambiente **LAMP** completo, pronto per ospitare siti e web app.