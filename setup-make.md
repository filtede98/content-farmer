# SETUP MAKE.COM — Content Farmer

## Panoramica del flusso (Scenario)

Il nostro scenario Make.com esegue questa sequenza:

```
Google Sheets (leggi keyword) → LLM (genera articolo) → WordPress (pubblica)
```

Più in dettaglio:

```
[1. Google Sheets]     Legge una riga con keyword, intento, silo, tipo articolo
        ↓
[2. Router]            Controlla che lo stato sia "da scrivere"
        ↓
[3. API LLM]           Invia il Master Prompt + dati keyword → riceve articolo HTML
        ↓
[4. Text Parser]       Estrae meta_title, meta_description, slug, corpo articolo
        ↓
[5. WordPress]         Crea il post con titolo, contenuto HTML, categoria, slug, meta SEO
        ↓
[6. Google Sheets]     Aggiorna lo stato della riga a "pubblicato" + data pubblicazione
```

---

## Step 1: Prepara Google Sheets

### Crea il foglio
1. Vai su Google Sheets e crea un nuovo foglio chiamato **"Content Farmer - Database"**
2. Crea queste colonne nella riga 1:

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| Keyword | Intento | Silo | Tipo Articolo | Volume | SD | Priorità | Stato | Data Pubblicazione |

3. Importa le keyword dal file keyword-database.csv che abbiamo creato
4. La colonna "Stato" deve contenere: **da scrivere** (per le righe da processare)

---

## Step 2: Configura le connessioni in Make.com

### Connessione Google Sheets
1. In Make.com vai su **Connections** (menu a sinistra)
2. Clicca **Add Connection** → cerca **Google Sheets**
3. Autorizza con il tuo account Google

### Connessione WordPress
1. Clicca **Add Connection** → cerca **WordPress**
2. Inserisci:
   - **URL:** https://sceltacasa.com
   - **Username:** il tuo username WordPress
   - **Password:** serve una **Application Password** (NON la password di login)

#### Come creare Application Password in WordPress:
1. Dashboard WP → Utenti → Il tuo profilo
2. Scorri fino a **"Password per le applicazioni"**
3. Nome: "Make.com"
4. Clicca **"Aggiungi nuova password per le applicazioni"**
5. Copia la password generata (la vedrai solo una volta!)
6. Usala in Make.com come password

### Connessione API LLM
Dipende dal modello scelto:

#### Opzione A: Claude (Anthropic) — Consigliato
1. Vai su https://console.anthropic.com
2. Crea un account e aggiungi credito (minimo $5)
3. Vai su API Keys → Create Key
4. In Make.com usa il modulo **HTTP > Make a request** con:
   - URL: `https://api.anthropic.com/v1/messages`
   - Headers: `x-api-key: [TUA_API_KEY]`, `anthropic-version: 2023-06-01`, `content-type: application/json`

#### Opzione B: OpenAI
1. Vai su https://platform.openai.com
2. In Make.com cerca il modulo **OpenAI > Create a Chat Completion**
3. Connetti con la tua API key

#### Opzione C: Google Gemini
1. Vai su https://aistudio.google.com
2. In Make.com usa il modulo **Google Gemini > Generate Content**

---

## Step 3: Costruisci lo Scenario

### Modulo 1: Google Sheets — Search Rows
- **Spreadsheet:** Content Farmer - Database
- **Sheet:** Foglio1
- **Filter:** Colonna "Stato" = "da scrivere"
- **Maximum number of returned rows:** 1 (processa una keyword alla volta)
- **Sort order:** A → Z per colonna "Priorità" (così fa prima le "alta")

### Modulo 2: API LLM — HTTP Request (o modulo dedicato)
- **System Prompt:** Il contenuto del Master Prompt (da master-prompt.md)
- **User Message:** 
```
Scrivi un articolo completo per la keyword "{{1.Keyword}}".
Tipo di articolo: {{1.Tipo Articolo}}.
Categoria: {{1.Silo}}.
Intento di ricerca: {{1.Intento}}.
```

### Modulo 3: Text Parser — Match Pattern
Estrai dal testo generato:
- **Corpo articolo:** tutto il contenuto HTML prima di "meta_title:"
- **meta_title:** il testo dopo "meta_title:" fino a fine riga
- **meta_description:** il testo dopo "meta_description:" fino a fine riga
- **slug:** il testo dopo "slug:" fino a fine riga
- **categoria:** il testo dopo "categoria:" fino a fine riga

Pattern regex per meta_title:
```
meta_title:\s*(.+)
```

Pattern regex per meta_description:
```
meta_description:\s*(.+)
```

Pattern regex per slug:
```
slug:\s*(.+)
```

Pattern regex per il corpo (tutto prima di meta_title):
```
([\s\S]+?)(?=meta_title:)
```

### Modulo 4: WordPress — Create a Post
- **Title:** {{3.meta_title}} (dal text parser)
- **Content:** {{3.corpo_articolo}} (dal text parser)
- **Slug:** {{3.slug}}
- **Status:** Draft (inizia con bozza, poi passiamo a Publish quando siamo sicuri)
- **Categories:** mappa {{1.Silo}} alla categoria WordPress corrispondente

### Modulo 5: Google Sheets — Update a Row
- **Row number:** {{1.row_number}}
- **Colonna Stato:** "pubblicato"
- **Colonna Data Pubblicazione:** {{now}} (data corrente)

---

## Step 4: Scheduling

### Fase test (prima settimana)
- **Trigger manuale** — lancia a mano, controlla ogni articolo
- **Stato post:** Draft (bozza)

### Fase produzione
- **Schedule:** ogni giorno alle 03:00 (di notte, meno carico server)
- **Stato post:** Publish
- **Articoli al giorno:** 1-2 (non di più, Google sospetta se pubblichi troppo)

---

## Step 5: Costi stimati

| Servizio | Costo mensile stimato |
|---|---|
| Make.com (piano gratuito) | €0 (1000 operazioni/mese = ~150 articoli) |
| API LLM (Claude Sonnet) | ~€0.10-0.15 per articolo = ~€3-5/mese per 30 articoli |
| Hostinger | €2.99/mese |
| **Totale** | **~€6-8/mese** |

---

## Troubleshooting comune

### L'articolo non si pubblica su WordPress
- Verifica che l'Application Password sia corretta
- Verifica che l'URL sia https (non http)
- Controlla i permessi dell'utente WP (deve essere Amministratore o Editore)

### Il testo generato è troppo corto o generico
- Aumenta max_tokens a 4096
- Verifica che il System Prompt sia copiato per intero
- Prova temperature 0.7-0.8

### Google Sheets non trova le righe
- Verifica che il filtro "Stato = da scrivere" sia esatto (spazi, maiuscole)
- Verifica che il foglio sia condiviso con l'account Google collegato a Make.com
