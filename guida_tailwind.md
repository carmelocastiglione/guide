# Guida Introduttiva a Tailwind CSS (senza npm)

Tailwind CSS è un framework CSS *utility-first*: si costruiscono interfacce moderne utilizando direttamente classi HTML, senza scrivere quasi CSS personalizzato.

## 1. Perché usare Tailwind?

- Sviluppo molto veloce  
- Stile coerente e standardizzato  
- Responsive semplice tramite prefissi (`sm:`, `md:`, `lg:`…)  
- Nessuna configurazione necessaria con la versione CDN  

Esempio:

```html
<button class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
  Clicca qui
</button>
```

📄 Documentazione:
[https://tailwindcss.com/docs/utility-first](https://tailwindcss.com/docs/utility-first)

## 2. Installazione Rapida (CDN)

Aggiungi nel `<head>` del tuo HTML:

```html
<script src="https://cdn.tailwindcss.com"></script>
```

📄 Documentazione:
[https://tailwindcss.com/docs/installation/play-cdn](https://tailwindcss.com/docs/installation/play-cdn)

## 3. Utility fondamentali

### Spaziatura (padding e margin)

```html
<div class="p-4 mt-6">
  Contenuto con padding e margin
</div>
```

📄 [https://tailwindcss.com/docs/padding](https://tailwindcss.com/docs/padding)\
📄 [https://tailwindcss.com/docs/margin](https://tailwindcss.com/docs/margin)

### Colori

```html
<p class="text-blue-500 bg-red-300">
  Testo colorato
</p>
```

📄 [https://tailwindcss.com/docs/text-color](https://tailwindcss.com/docs/text-color)\
📄 [https://tailwindcss.com/docs/background-color](https://tailwindcss.com/docs/background-color)

### Tipografia

```html
<h1 class="text-xl font-bold leading-tight">
  Titolo importante
</h1>
```

📄 [https://tailwindcss.com/docs/font-size](https://tailwindcss.com/docs/font-size)\
📄 [https://tailwindcss.com/docs/font-weight](https://tailwindcss.com/docs/font-weight)\
📄 [https://tailwindcss.com/docs/line-height](https://tailwindcss.com/docs/line-height)

### Bordi e Ombre

```html
<div class="border border-2 rounded-lg p-4 shadow">
  Box con bordi e ombra
</div>
```

📄 [https://tailwindcss.com/docs/border-width](https://tailwindcss.com/docs/border-width)\
📄 [https://tailwindcss.com/docs/border-radius](https://tailwindcss.com/docs/border-radius)\
📄 [https://tailwindcss.com/docs/box-shadow](https://tailwindcss.com/docs/box-shadow)

### Flexbox

```html
<div class="flex items-center justify-between gap-4">
  <span>Elemento A</span>
  <span>Elemento B</span>
</div>
```

📄 [https://tailwindcss.com/docs/flex](https://tailwindcss.com/docs/flex)\
📄 [https://tailwindcss.com/docs/align-items](https://tailwindcss.com/docs/align-items)\
📄 [https://tailwindcss.com/docs/justify-content](https://tailwindcss.com/docs/justify-content)\
📄 [https://tailwindcss.com/docs/gap](https://tailwindcss.com/docs/gap)

### Grid

```html
<div class="grid grid-cols-3 gap-6">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

📄 [https://tailwindcss.com/docs/grid-template-columns](https://tailwindcss.com/docs/grid-template-columns)\
📄 [https://tailwindcss.com/docs/gap](https://tailwindcss.com/docs/gap)

## 4. Responsive Design

```html
<div class="text-sm md:text-lg lg:text-xl">
  Testo responsive
</div>
```

📄 [https://tailwindcss.com/docs/responsive-design](https://tailwindcss.com/docs/responsive-design)

## 5. Stati: Hover, Focus, Active

```html
<button class="bg-green-500 hover:bg-green-600 active:bg-green-700 focus:ring-2 focus:ring-green-300">
  Bottone interattivo
</button>
```

📄 [https://tailwindcss.com/docs/hover-focus-and-other-states](https://tailwindcss.com/docs/hover-focus-and-other-states)\
📄 [https://tailwindcss.com/docs/ring-width](https://tailwindcss.com/docs/ring-width)

## 6. Componenti Base

### **Card**

```html
<div class="max-w-sm p-4 bg-white rounded-xl shadow">
  <h2 class="text-xl font-semibold mb-2">Titolo</h2>
  <p class="text-gray-700">Testo della card</p>
</div>
```

📄 [https://tailwindcss.com/docs/shadow](https://tailwindcss.com/docs/shadow)\
📄 [https://tailwindcss.com/docs/border-radius](https://tailwindcss.com/docs/border-radius)

---

### **Navbar**

```html
<nav class="flex justify-between items-center p-4 bg-gray-800 text-white">
  <div class="font-bold">Logo</div>
  <ul class="flex gap-4">
    <li><a href="#" class="hover:text-gray-300">Home</a></li>
    <li><a href="#" class="hover:text-gray-300">Blog</a></li>
    <li><a href="#" class="hover:text-gray-300">Contatti</a></li>
  </ul>
</nav>
```

📄 [https://tailwindcss.com/docs/flex](https://tailwindcss.com/docs/flex)\
📄 [https://tailwindcss.com/docs/background-color](https://tailwindcss.com/docs/background-color)\
📄 [https://tailwindcss.com/docs/padding](https://tailwindcss.com/docs/padding)

## 7. Esempio: Layout Completo

Header + Due Colonne + Footer

```html
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.tailwindcss.com"></script>
  <title>Layout Tailwind</title>
</head>
<body class="bg-gray-100">

  <!-- HEADER -->
  <header class="bg-blue-700 text-white p-4 flex justify-between items-center">
    <h1 class="text-xl font-bold">My Website</h1>
    <nav class="flex gap-4">
      <a href="#" class="hover:text-gray-300">Home</a>
      <a href="#" class="hover:text-gray-300">Blog</a>
      <a href="#" class="hover:text-gray-300">Contatti</a>
    </nav>
  </header>

  <!-- LAYOUT A DUE COLONNE -->
  <div class="max-w-6xl mx-auto grid grid-cols-1 md:grid-cols-4 gap-6 p-6">

    <!-- MAIN CONTENT -->
    <main class="md:col-span-3 bg-white p-6 rounded-xl shadow">
      <h2 class="text-2xl font-semibold mb-4">Articolo principale</h2>
      <p class="text-gray-700 leading-relaxed mb-4">
        Questo è il contenuto principale della pagina.
      </p>
      <p class="text-gray-700 leading-relaxed">
        Questo layout è completamente responsive.
      </p>
    </main>

    <!-- SIDEBAR -->
    <aside class="bg-white p-6 rounded-xl shadow">
      <h3 class="text-xl font-semibold mb-3">Sidebar</h3>
      <ul class="space-y-2">
        <li><a href="#" class="text-blue-600 hover:underline">Link 1</a></li>
        <li><a href="#" class="text-blue-600 hover:underline">Link 2</a></li>
        <li><a href="#" class="text-blue-600 hover:underline">Link 3</a></li>
      </ul>
    </aside>

  </div>

  <!-- FOOTER -->
  <footer class="bg-gray-800 text-white text-center p-4 mt-6">
    © 2025 My Website — Tutti i diritti riservati.
  </footer>

</body>
</html>
```

📄 Utility principali usate:\
[https://tailwindcss.com/docs/grid-template-columns](https://tailwindcss.com/docs/grid-template-columns)\
[https://tailwindcss.com/docs/padding](https://tailwindcss.com/docs/padding)\
[https://tailwindcss.com/docs/shadow](https://tailwindcss.com/docs/shadow)\
[https://tailwindcss.com/docs/text-color](https://tailwindcss.com/docs/text-color)\
[https://tailwindcss.com/docs/background-color](https://tailwindcss.com/docs/background-color)

## 8. Consigli Utili

* Usa la classe `max-w-*` per evitare layout troppo larghi
* Suddividi le classi su più righe se l’HTML diventa troppo lungo
* Usa il Playground ufficiale: [https://play.tailwindcss.com/](https://play.tailwindcss.com/)
* Installa l’estensione VS Code “Tailwind CSS IntelliSense”