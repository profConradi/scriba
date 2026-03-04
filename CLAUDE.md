# Scriba

Editor di scrittura minimale con autocompletamento LLM locale, supporto Markdown e export .docx.

## Panoramica

Scriba è un'applicazione single-file HTML (~420 righe) che offre un editor di testo Markdown con suggerimenti di completamento automatico generati da un LLM locale tramite LM Studio. Non ha dipendenze server-side: si apre direttamente nel browser e comunica con LM Studio via API OpenAI-compatible su `localhost:1234`.

## Architettura

**File unico**: `scriba.html` contiene tutto — HTML, CSS e JavaScript. Nessun build system, nessun bundler.

**Dipendenze CDN** (caricate a runtime):
- JSZip 3.10.1 — generazione del file .docx (assemblaggio diretto di OOXML dentro un archivio ZIP)
- FileSaver.js 2.0.5 — download del file generato
- marked 15.0.7 — parsing Markdown per l'anteprima
- Google Fonts: Literata (serif, per anteprima e UI), JetBrains Mono (monospace, per l'editor)

**Nessun framework**: vanilla JS, nessun React/Vue/etc. Lo stato è gestito con variabili globali nel modulo script.

## Componenti funzionali

### Editor Markdown
- `<textarea id="editor">` con font monospaced (JetBrains Mono)
- Auto-resize verticale: la textarea cresce col contenuto
- Placeholder che suggerisce di iniziare con `# Titolo`
- Nessun campo titolo separato — il primo `# heading` nel testo funge da titolo del documento

### Autocompletamento LLM (ghost text)
- **Meccanismo**: dopo una pausa nella digitazione (debounce configurabile, default 1500ms), viene inviata una richiesta al server LLM con gli ultimi N caratteri come contesto
- **Visualizzazione**: un `<div id="ghost-layer">` sovrapposto alla textarea mostra il suggerimento in grigio corsivo, perfettamente allineato col testo reale tramite mirror di font/padding/dimensioni
- **Interazione**: `Tab` accetta, `Esc` rifiuta, `Ctrl+Spazio` forza una richiesta
- **API**: endpoint `POST /v1/chat/completions` (OpenAI-compatible), modello `local-model`
- **Post-processing**: `stripThinkTags()` rimuove tag `<think>` dai modelli reasoning, `cleanSuggestion()` elimina ripetizioni della coda del testo esistente
- **Stop sequences**: `['\n\n\n']` per limitare la lunghezza delle risposte

### Anteprima Markdown
- Toggle con bottone "Anteprima" o `Ctrl+P`
- Usa `marked.parse()` per il rendering
- Nasconde l'editor e mostra `<div id="preview">` con tipografia serif (Literata)
- L'autocompletamento è implicitamente disattivato in modalità anteprima (la textarea non è visibile)

### Rifrasa / Correggi (context popup)
- Selezionando del testo nell'editor appare un popup flottante (`#ctxPopup`) posizionato vicino alla fine della selezione
- Due azioni: **Rifrasa** (riscrittura con significato preservato, temperatura 0.8) e **Correggi** (correzione errori, temperatura 0.2)
- Il testo selezionato viene inviato al LLM con un prompt di sistema dedicato
- Il risultato appare in un pannello diff (`#diffOverlay`) con testo originale barrato in rosso e proposta in verde
- L'utente può accettare (sostituzione nel testo) o rifiutare (ripristino selezione)
- Il posizionamento del popup usa un div mirror invisibile per calcolare le coordinate dalla posizione del cursore nella textarea

### Export .docx
- Converte il Markdown in OOXML strutturato tramite `markdownToDocxXml()`
- Elementi supportati: titoli (`#` → Heading1..6 con dimensioni decrescenti), **grassetto**, *corsivo*, ***grassetto+corsivo***, `codice inline`, blocchi di codice (sfondo grigio, Courier New), elenchi puntati (`-`, `*`, `+`) e numerati, citazioni (`>`), linee orizzontali
- Il parsing inline usa una regex sequenziale in `inlineToRuns()` che produce run OOXML (`<w:r>`)
- Il file .docx viene assemblato con JSZip: `[Content_Types].xml`, `_rels/.rels`, `word/document.xml`, `word/_rels/document.xml.rels`
- Formato pagina: A4 (11906×16838 twip), margini 1" (1440 twip), interlinea 1.5 (360 twip)
- Il nome file è derivato dal primo `# heading` nel testo

### Impostazioni
- Pannello modale overlay (`#settingsOverlay`)
- Configurazioni: URL server, prompt di sistema, temperatura, max token, ritardo debounce, dimensione contesto
- Toggle per thinking mode con budget token configurabile
- Persistenza in `localStorage` (chiave `scriba-config`)

### Thinking mode
- Per modelli con ragionamento esteso (Qwen3, DeepSeek-R1, QwQ)
- Quando attivo: aggiunge `reasoning_effort: 'medium'` alla richiesta e somma il thinking budget ai max token
- Quando disattivo: appende al prompt di sistema un'istruzione per sopprimere i tag `<think>`
- Badge visivo "THINK" nella barra di stato
- `stripThinkTags()` rimuove comunque eventuali tag di ragionamento dalla risposta

### Connessione
- Check periodico ogni 15s su `GET /v1/models`
- Indicatore visivo: dot rosso/verde + nome del modello caricato
- Spinner durante le richieste LLM

## Design system

Palette calda e minimale definita in CSS custom properties:
- `--bg: #faf8f5` (crema), `--fg: #2c2825` (marrone scuro), `--accent: #6b5c4d` (marrone medio), `--ghost: #9a918a` (grigio caldo), `--border: #e0dbd6`
- Larghezza massima editor: 720px, centrato
- Nessuna dipendenza da framework CSS

## Convenzioni di codice

- Lo stile è compatto: funzioni brevi su una riga dove possibile, senza spazi inutili
- Le stringhe OOXML sono concatenate direttamente (non template literals) perché sono XML verbose e la concatenazione le tiene leggibili come blocchi
- I nomi delle funzioni e variabili sono in camelCase inglese; i testi dell'interfaccia sono in italiano
- Nessun modulo, nessun import/export — tutto è nello scope globale del tag `<script>`

## Note per lo sviluppo

- **Testare sempre con un server LM Studio attivo** su `localhost:1234` con un modello caricato
- La ghost text overlay è fragile: qualsiasi cambiamento al font, padding o line-height dell'editor deve essere replicato identico nel `#ghost-layer`
- L'export .docx non supporta: tabelle, immagini, link cliccabili, liste nested oltre 3 livelli. Se servono, va esteso `markdownToDocxXml()`
- Il popup contestuale calcola la posizione con un mirror div: se cambia il layout dell'editor, `showCtxPopup()` potrebbe posizionare il popup in modo errato
- `marked` è usato solo per l'anteprima; l'export .docx ha il suo parser Markdown indipendente in `markdownToDocxXml()`. Cambiamenti alla sintassi supportata vanno fatti in entrambi i posti
- max_tokens per rifrasa/correggi è `Math.max(256, selText.length)` — potrebbe non bastare per testi molto lunghi con modelli che producono output verbosi

## Modelli LLM consigliati

Per l'autocompletamento serve bassa latenza (<2-3s), quindi modelli piccoli e veloci:
- **Qwen3-30B-A3B** (MoE, 3B attivi): miglior rapporto qualità/velocità, 4-6 GB VRAM
- **Qwen3-4B-Instruct-2507**: per GPU con 8-10 GB VRAM
- **Qwen3-8B**: per 16+ GB VRAM, buon italiano
- **Gemma 3 12B**: per 24+ GB, eccellente italiano

Evitare modelli reasoning-heavy (DeepSeek-R1 full, QwQ-72B) per l'autocompletamento — troppo lenti. Possono essere usati con il thinking mode per rifrasa/correggi dove la latenza è accettabile.
