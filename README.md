# Scriba

> **[English version available here](README.en.md)** 🇬🇧

Editor di scrittura minimale con autocompletamento LLM locale, supporto Markdown e export .docx.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Version](https://img.shields.io/badge/version-1.0.0-green.svg)

## 📖 Panoramica

Scriba è un'applicazione **single-file HTML** (~420 righe) che offre un editor di testo Markdown con suggerimenti di completamento automatico generati da un **LLM locale** tramite **LM Studio**. Non richiede server-side: si apre direttamente nel browser e comunica con LM Studio via API OpenAI-compatible su `localhost:1234`.

### Caratteristiche principali

- ✍️ **Editor Markdown** con auto-resize e font monospaced
- 🤖 **Autocompletamento AI** con ghost text in tempo reale
- 📊 **Equazioni matematiche** con supporto LaTeX/KaTeX
- 🔄 **Rifrasa e Correggi** testo selezionato con AI
- 👁️ **Anteprima Markdown** live con rendering professionale
- 📄 **Export .docx** diretto (senza conversioni esterne)
- 💾 **Carica/Salva .md** per gestire i tuoi documenti
- ⚙️ **Configurazione persistente** con localStorage

## 🚀 Quick Start

### 1. Installa LM Studio

Scarica e installa [LM Studio](https://lmstudio.ai/) per il tuo sistema operativo.

### 2. Carica un modello

Apri LM Studio e scarica uno dei modelli consigliati (vedi sezione "Modelli LLM consigliati").

### 3. Configura e avvia il server locale

In LM Studio:
1. Vai su **Local Server** (icona ↗️)
2. Carica il modello desiderato
3. **IMPORTANTE**: Apri **Server Settings** (⚙️ in alto a destra) e abilita **CORS**
4. Clicca su **Start Server**
5. Assicurati che sia in ascolto su `localhost:1234`

⚠️ **Senza CORS abilitato, Scriba non potrà comunicare con LM Studio dal browser!**

### 4. Apri Scriba

Apri `scriba.html` direttamente nel browser (Chrome, Firefox, Safari, Edge).

L'indicatore di connessione in alto a destra diventerà verde quando Scriba si connette a LM Studio.

## 💡 Utilizzo

### Autocompletamento AI

Scriba suggerisce automaticamente la continuazione del testo mentre scrivi:

- **Tab** - Accetta il suggerimento
- **Esc** - Rifiuta il suggerimento
- **Ctrl+Spazio** - Forza un suggerimento immediato

Il ghost text appare in grigio corsivo alla fine del testo. Il delay è configurabile nelle impostazioni (default: 1.5s).

### Rifrasa e Correggi

Seleziona del testo e usa i pulsanti che appaiono:

- **Rifrasa** - Riscrive il testo mantenendo il significato (temperatura 0.8)
- **Correggi** - Corregge errori grammaticali e di ortografia (temperatura 0.2)

Un pannello diff mostra il confronto tra originale e proposta. Puoi accettare o rifiutare.

### Equazioni matematiche

Scriba supporta formule LaTeX/KaTeX renderizzate in anteprima:

**Inline** (usa dollari singoli):
```markdown
La formula di Eulero è $e^{i\pi} + 1 = 0$.
```

**Block** (usa dollari doppi):
```markdown
$$
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
$$
```

**Sistemi ed equazioni multilinea** (usa `\\` per andare a capo):
```markdown
$$
\begin{cases}
x + y = 5 \\
x - y = 1
\end{cases}
$$
```

### Shortcuts da tastiera

- **Ctrl+P** - Toggle anteprima Markdown
- **Ctrl+Spazio** - Forza suggerimento AI
- **Tab** - Accetta ghost text
- **Esc** - Rifiuta ghost text

## 🎯 Modelli LLM consigliati

Per l'autocompletamento serve **bassa latenza** (<2-3s), quindi modelli **piccoli e veloci**:

### Migliori per qualità/velocità

| Modello | VRAM richiesta | Note |
|---------|---------------|------|
| **Qwen2.5-Coder-3B-Instruct** | 4-6 GB | Eccellente per codice e testo, velocissimo |
| **Qwen2.5-3B-Instruct** | 4-6 GB | Ottimo italiano, buon compromesso |
| **Phi-3.5-mini-instruct** | 4-6 GB | Microsoft, molto veloce |

### Per GPU più potenti

| Modello | VRAM richiesta | Note |
|---------|---------------|------|
| **Qwen2.5-7B-Instruct** | 8-10 GB | Qualità superiore, buon italiano |
| **Llama-3.2-8B-Instruct** | 10-12 GB | Meta, ottimo bilanciamento |
| **Gemma-2-9B-Instruct** | 12-16 GB | Google, eccellente italiano |

### Modelli reasoning (solo per Rifrasa/Correggi)

⚠️ **Non usare per autocompletamento** (troppo lenti):

- **DeepSeek-R1-Distill-Qwen-7B** - Ragionamento esteso
- **QwQ-32B-Preview** - Thinking mode avanzato

Attiva il **Thinking mode** nelle impostazioni per questi modelli.

## ⚙️ Configurazione

Clicca su **Impostazioni** per personalizzare:

### Parametri LLM

- **Indirizzo server** - URL del server LM Studio (default: `http://localhost:1234`)
- **Prompt di sistema** - Istruzioni per l'AI
- **Temperatura** - Creatività delle risposte (0.0-2.0, default: 0.7)
- **Max token** - Lunghezza massima suggerimenti (default: 80)
- **Ritardo (ms)** - Debounce prima della richiesta (default: 1500ms)
- **Contesto (caratteri)** - Quanti caratteri inviare come contesto (default: 1500)

### Thinking mode

Per modelli con ragionamento esteso (Qwen3, DeepSeek-R1, QwQ):
- Attiva **Thinking (ragionamento esteso)**
- Imposta **Thinking budget (token)** (default: 1024)

⚠️ Aumenta latenza - **non consigliato per autocompletamento**.

## 📦 Architettura tecnica

### Stack tecnologico

- **Vanilla JavaScript** - Nessun framework
- **Marked.js** - Parsing Markdown per anteprima
- **KaTeX** - Rendering equazioni matematiche
- **JSZip** - Generazione file .docx
- **FileSaver.js** - Download file
- **Google Fonts** - Literata (serif), JetBrains Mono (mono)

### Funzionamento autocompletamento

1. **Debounce** - Dopo 1.5s di pausa nella digitazione
2. **Estrazione contesto** - Ultimi N caratteri del testo
3. **Richiesta LLM** - POST a `/v1/chat/completions` (OpenAI-compatible)
4. **Post-processing** - Rimozione tag `<think>` e ripetizioni
5. **Ghost layer** - Overlay perfettamente allineato con la textarea

### Export .docx

Il file `.docx` viene assemblato **direttamente** senza servizi esterni:

1. **Parsing Markdown** - Conversione in OOXML tramite `markdownToDocxXml()`
2. **Supporto elementi** - Titoli, grassetto, corsivo, codice, liste, citazioni
3. **Assemblaggio ZIP** - Struttura `.docx` completa con JSZip
4. **Nome file** - Estratto dal primo `# heading` del documento

Elementi supportati:
- Headings `#`-`######` (con dimensioni decrescenti)
- **Grassetto**, *Corsivo*, ***Grassetto+Corsivo***
- `Codice inline` e blocchi di codice
- Liste puntate e numerate (anche nested)
- Citazioni `>`
- Linee orizzontali `---`

## 🔧 Risoluzione problemi

### L'indicatore rimane rosso (disconnesso)

- Verifica che LM Studio sia avviato
- Controlla che il server locale sia attivo su `localhost:1234`
- **Assicurati che CORS sia abilitato in LM Studio** (Server Settings → Enable CORS)
- Vai in Impostazioni e verifica l'URL del server
- Controlla la console JavaScript (F12) per errori CORS

### I suggerimenti sono troppo lenti

- Usa un modello più piccolo (es. Qwen2.5-3B)
- Disattiva il Thinking mode se attivo
- Riduci il "Contesto (caratteri)" nelle impostazioni
- Verifica che la GPU sia utilizzata in LM Studio

### I suggerimenti non appaiono

- Controlla che l'indicatore sia verde (connesso)
- Scrivi almeno una frase completa e aspetta 1.5s
- Premi Ctrl+Spazio per forzare un suggerimento
- Verifica i log in LM Studio per errori API

### Le equazioni non vengono renderizzate

- Le equazioni appaiono **solo in anteprima** (Ctrl+P)
- Verifica la sintassi LaTeX (usa `\\` per andare a capo)
- Controlla la console (F12) per errori KaTeX
- Nell'editor rimangono come testo `$...$` - è normale

### Errore HTTP 400 da LM Studio

- Il modello potrebbe non supportare alcuni parametri
- Disattiva il Thinking mode
- Verifica che il prompt di sistema non contenga caratteri speciali
- Controlla i log di LM Studio per dettagli

## 📝 Note di sviluppo

### Stile codice

- **Compatto** - Funzioni brevi su una riga dove possibile
- **camelCase** - Per variabili e funzioni JavaScript
- **Testi italiani** - Nell'interfaccia utente
- **Nessun modulo** - Tutto nello scope globale del tag `<script>`

### Limitazioni note

L'export .docx **non supporta**:
- Tabelle Markdown
- Immagini
- Link cliccabili
- Liste nested oltre 3 livelli
- Equazioni matematiche

Per estendere, modificare `markdownToDocxXml()`.

### Dipendenze esterne (CDN)

Tutte le librerie sono caricate da CDN per semplicità:
- JSZip 3.10.1
- FileSaver.js 2.0.5
- Marked 15.0.7
- KaTeX 0.16.9

## 📄 Licenza

MIT License - Copyright (c) 2026

## 🤝 Contributi

Contributi, issue e feature request sono benvenuti!

## 🔗 Link utili

- [LM Studio](https://lmstudio.ai/) - Runtime LLM locale
- [Marked.js](https://marked.js.org/) - Markdown parser
- [KaTeX](https://katex.org/) - Math rendering
- [OpenAI API](https://platform.openai.com/docs/api-reference) - Formato API

---

**Scriba** - Scrivi con l'intelligenza artificiale, mantieni il controllo totale 🖋️
