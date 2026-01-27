# Guida Markdown - Sintassi Principale

Una guida completa sui comandi e la sintassi principale di Markdown.

## Indice

1. [Titoli](#titoli)
2. [Testo Formattato](#testo-formattato)
3. [Liste](#liste)
4. [Link e Immagini](#link-e-immagini)
5. [Codice](#codice)
6. [Blocchi di Citazione](#blocchi-di-citazione)
7. [Linee Orizzontali](#linee-orizzontali)
8. [Tabelle](#tabelle)
9. [Elenchi di Attività](#elenchi-di-attività)
10. [Elementi Speciali](#elementi-speciali)
11. [Consigli e Best Practices](#consigli-e-best-practices)
12. [Link utili](#link-utili)

## Titoli

I titoli in Markdown si creano usando il simbolo `#`. Più `#` significano livelli di titolo più bassi.

```markdown
# Titolo di Livello 1
## Titolo di Livello 2
### Titolo di Livello 3
#### Titolo di Livello 4
##### Titolo di Livello 5
###### Titolo di Livello 6
```

## Testo Formattato

### Grassetto
```markdown
**testo in grassetto**
__testo in grassetto__
```
**Risultato:** **testo in grassetto**

### Corsivo
```markdown
*testo in corsivo*
_testo in corsivo_
```
**Risultato:** *testo in corsivo*

### Grassetto e Corsivo
```markdown
***testo in grassetto e corsivo***
___testo in grassetto e corsivo___
```
**Risultato:** ***testo in grassetto e corsivo***

### Barrato
```markdown
~~testo barrato~~
```
**Risultato:** ~~testo barrato~~

### Sottolineato
```markdown
<u>testo sottolineato</u>
```
**Risultato:** <u>testo sottolineato</u>

## Liste

### Lista Non Ordinata
```markdown
- Elemento 1
- Elemento 2
- Elemento 3
  - Sotto-elemento 1
  - Sotto-elemento 2
```

Puoi anche usare `*` o `+`:
```markdown
* Elemento 1
* Elemento 2

+ Elemento 1
+ Elemento 2
```

### Lista Ordinata
```markdown
1. Primo elemento
2. Secondo elemento
3. Terzo elemento
   1. Sub-elemento
   2. Sub-elemento
```

## Link e Immagini

### Link Semplice
```markdown
[Testo del link](https://www.esempio.com)
```
**Risultato:** [Testo del link](https://www.esempio.com)

### Link con Titolo
```markdown
[Testo del link](https://www.esempio.com "Titolo del link")
```

### Link Automatico
```markdown
<https://www.esempio.com>
<email@esempio.com>
```

### Riferimento a Link
```markdown
[Testo del link][riferimento]

[riferimento]: https://www.esempio.com
```

### Immagine
```markdown
![Testo alternativo](path/to/image.jpg)
```

### Immagine con Link
```markdown
[![Testo alternativo](path/to/image.jpg)](https://www.esempio.com)
```

## Codice

### Codice Inline
Usa backtick singoli per il codice inline:
```markdown
Questo è una `variabile` nel testo.
```
Viene visualizzata una `variabile`

### Blocco di Codice
```markdown
    line 1
    line 2
    line 3
```

Oppure con tre backtick:
````markdown
```
line 1
line 2
line 3
```
````

### Blocco di Codice con Evidenziazione Sintattica
Specifica il linguaggio di programmazione:
````markdown
```javascript
function hello() {
  console.log("Hello, World!");
}
```
````

Linguaggi comuni: `python`, `javascript`, `java`, `html`, `css`, `bash`, `sql`, `php`, ecc.

---

## Blocchi di Citazione

### Citazione Semplice
```markdown
> Questa è una citazione.
```
**Risultato:**
> Questa è una citazione.

### Citazione Multipla
```markdown
> Questa è una citazione.
> Su più righe.
> Con più paragrafi.
```

### Citazione Nidificata
```markdown
> Citazione esterna.
>> Citazione nidificata.
```

---

## Linee Orizzontali

Puoi creare una linea orizzontale usando:
```markdown
---
***
___
```

## Tabelle

### Tabella Semplice
```markdown
| Intestazione 1 | Intestazione 2 | Intestazione 3 |
|---|---|---|
| Cella 1 | Cella 2 | Cella 3 |
| Cella 4 | Cella 5 | Cella 6 |
```

### Tabella con Allineamento
- `:---` = allineamento a sinistra
- `:---:` = allineamento al centro
- `---:` = allineamento a destra

```markdown
| Sinistra | Centro | Destra |
|:---|:---:|---:|
| A | B | C |
| D | E | F |
```

## Elenchi di Attività

Crea elenchi interattivi di attività:
```markdown
- [x] Attività completata
- [ ] Attività non completata
- [x] Altra attività completata
```

## Elementi Speciali

### Escape di Caratteri Speciali
Usa il backslash `\` per mostrare caratteri speciali:
```markdown
\* Non sarà un elenco
\# Non sarà un titolo
\[Non sarà un link\]
```

### HTML Inline
Markdown supporta il codice HTML diretto:
```markdown
<div style="color: red;">Questo è HTML</div>
```

### Note a Piè di Pagina (alcuni sapori di Markdown)
```markdown
Questo è un testo con una nota[^1].

[^1]: Questa è la nota a piè di pagina.
```

### Simboli e Caratteri Speciali
```markdown
© ® ™
€ £ ¥
° • → ← ↑ ↓
```

## Consigli e Best Practices

1. **Usa spazi coerenti**: Utilizza sempre lo stesso numero di spazi per l'indentazione.
2. **Lascia righe vuote**: Metti una riga vuota tra sezioni diverse per migliorare la leggibilità.
3. **Preferisci `#` per i titoli**: È più leggibile e universalmente supportato.
4. **Spiega il codice**: Usa sempre il linguaggio di programmazione nel blocco di codice.
5. **Usa liste quando possibile**: Rendono il contenuto più organizzato e leggibile.
6. **Includi il titolo**: Includi sempre un titolo principale (`#`) all'inizio del documento.

## Link utili

- [Markdown Guide](https://www.markdownguide.org/)
- [CommonMark Specification](https://spec.commonmark.org/)
- [GitHub Flavored Markdown](https://github.github.com/gfm/)
