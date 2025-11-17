# Guida all’Uso di Tilde (~) e Backtick (`) con rimappatura tramite PowerToys su Windows

## Introduzione

Sulle tastiere italiane i caratteri **tilde (~)** e **backtick (`)** non sono immediatamente intuitivi.
Con **Microsoft PowerToys** puoi rimapparli in modo semplice e veloce, impostando:

* **AltGr + '** per produrre **~**
* **AltGr + ì** per produrre **`**

## 1. Installare Microsoft PowerToys

### Cos’è PowerToys?

È un insieme di strumenti avanzati per Windows, tra cui **Keyboard Manager**, che permette di rimappare i tasti.

### Installazione (Microsoft Store – Consigliata)

1. Apri **Microsoft Store**.
2. Cerca **PowerToys**.
3. Clicca **Installa**.
4. Avvialo dal menu Start.

### Installazione alternativa (GitHub)

1. Vai alla pagina ufficiale delle release:
   [https://github.com/microsoft/PowerToys/releases](https://github.com/microsoft/PowerToys/releases)
2. Scarica **PowerToysSetup-x64.exe**.
3. Installa e avvia il programma.

## 2. Attivare Keyboard Manager

1. Apri **PowerToys**.
2. Nel menu laterale seleziona **Keyboard Manager**.
3. Attiva **Enable Keyboard Manager**.

Ora puoi configurare le rimappature.

## 3. Rimappare i Tasti

### Obiettivo

Configurare:

* **AltGr + '** → **Tilde (~)**
* **AltGr + ì** → **Backtick (`)**

> **Nota:** AltGr equivale a premere **Ctrl + Alt** contemporaneamente.

## 4. Rimappare AltGr + ' per ottenere la Tilde (~)

1. Vai in **Keyboard Manager** → **Remap a shortcut**.
2. Clicca **+** per aggiungere una nuova scorciatoia.
3. Nel campo **Shortcut** premi:
   **AltGr + '**
4. Nel campo **Mapped To (Text)**, scegli:
   **Tilde (~)**
5. Salva.

## 5. Rimappare AltGr + ì per ottenere il Backtick (`)

1. Vai in **Keyboard Manager** → **Remap a shortcut**.
2. Aggiungi una nuova remappatura con **+**.
3. Nel campo **Shortcut** premi:
   **AltGr + ì**
4. Nel campo **Mapped To (Text)**, seleziona:
   **Backtick (`)**
5. Salva.

## 6. Verifica del funzionamento

Apri un editor di testo (ad es. Notepad o Visual Studio Code) e prova:

* Premendo **AltGr + '** → deve comparire **~**
* Premendo **AltGr + ì** → deve comparire **`**

## 7. Ripristinare o modificare le rimappature

In qualsiasi momento puoi:

1. Tornare in **Keyboard Manager**.
2. Aprire **Remap a shortcut**.
3. Cancellare o modificare le assegnazioni con l’icona del cestino.
4. Oppure disattivare completamente Keyboard Manager.