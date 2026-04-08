# CLAUDE.md — Contesto operativo Content Farmer

## Identità progetto
- **Sito:** sceltacasa.com
- **Nicchia:** Elettrodomestici e piccoli apparecchi casa (Italia)
- **Monetizzazione:** Amazon Associates tag `sceltacasa21-21`
- **Email:** info@sceltacasa.com
- **Hosting:** Hostinger Premium (data center Germania)
- **GitHub:** github.com/filtede98/content-farmer
- **Proprietario:** Filippo Tedeschi (filippotedeschi98@gmail.com)

## Ruolo di Claude
Agisco come **Project Leader / CTO**. Filippo preferisce che io guidi il processo con decisioni concrete, non opzioni. Comunica in italiano. Ha familiarità con strumenti no-code (Make.com, WordPress) ma non è uno sviluppatore.

## Stack tecnico attuale

### WordPress
- **Tema:** GeneratePress (gratuito)
- **Plugin attivi:** Rank Math SEO, ShortPixel, TablePress, Pretty Links, WPCode, WPForms Lite, LiteSpeed Cache, Make Connector
- **CSS:** tutto in Aspetto → Personalizza → CSS aggiuntivo. Il file sorgente è `style-completo.css`
- **Sidebar:** rimossa via CSS (`display: none`)
- **Footer fix:** usa `#page { display: flex; flex-direction: column; }` (NON body, GeneratePress usa #page come wrapper)
- **Commenti:** disabilitati via CSS e impostazioni
- **Permalink:** struttura "Nome articolo"
- **Logo:** generato con Gemini, sfondo trasparente necessario per header scuro
- **Footer widget:** HTML personalizzato con link inline (Chi Siamo, Contatti, Privacy Policy)

### Categorie WordPress (con ID)
| Categoria | ID |
|---|---|
| Friggitrici ad Aria | 13 |
| Robot da Cucina | 14 |
| Macchine Caffè | 15 |
| Piccoli Elettrodomestici Cucina | 16 |
| Deumidificatori | 17 |
| Purificatori Aria | 18 |
| Ventilatori | 19 |
| Condizionatori Portatili | 20 |
| Riscaldamento Elettrico | 21 |

### Make.com — Scenario

**Flusso moduli (con numeri):**
```
[1] Google Sheets (Search Rows) → [11] HTTP/Rainforest API (GET) → [2] HTTP/Claude (POST, Data Structure) → [3] Text Parser (corpo) → [4] Text Parser (meta_title) → [7] Text Parser (meta_description) → [8] Text Parser (slug) → [9] WordPress (Create Post) → [10] Google Sheets (Update Row)
```

**Modulo [1] — Google Sheets: Search Rows**
- Spreadsheet: "Content Farmer - Database"
- Filter: colonna I (Stato) = "da scrivere"
- Max rows: 1

**Modulo [11] — HTTP: Rainforest API (GET)**
- URL: `https://api.rainforestapi.com/request`
- Method: GET
- Query String: api_key (da .env), type=search, amazon_domain=amazon.it, search_term={{1.Keyword (A)}}, language=it_IT, sort_by=featured
- Parse response: Yes
- Timeout: 30000
- Restituisce: `data.search_results[]` con title, asin, price.raw, prices[].list_price, rating, ratings_total, image, link

**Modulo [2] — HTTP: Claude API (POST, Data Structure)**
- URL: `https://api.anthropic.com/v1/messages`
- Method: POST
- Headers: x-api-key, anthropic-version: 2023-06-01, content-type: application/json
- Body input method: **Data Structure** (NON JSON string — risolve problema escape caratteri da Rainforest)
- Data Structure "Claude API Request": model (text), max_tokens (number), temperature (number), system (text), messages (array con role + content)
- model: claude-sonnet-4-20250514
- max_tokens: 8192 (NON 4096 — articoli HTML ricchi sono lunghi)
- temperature: 0.7
- system: contenuto di `system-prompt-v5.txt`
- messages[0].role: user
- messages[0].content: user message con variabili Rainforest (vedi sotto)
- Parse response: Yes
- Timeout: **120000** (Claude impiega 60-90 secondi per articoli lunghi)

**User message di Claude (campo content):**
```
Scrivi un articolo completo per la keyword "{{1.Keyword (A)}}". Tipo di articolo: {{1.Tipo Articolo (D)}}. Categoria: {{1.Silo (C)}}. Intento di ricerca: {{1.Intento (B)}}. Anno corrente: 2026. Lunghezza minima: 2000 parole.

PRODOTTI REALI DA AMAZON.IT (dati verificati):

Prodotto 1: {{11.data.search_results[1].title}}
ASIN: {{11.data.search_results[1].asin}}
Prezzo: {{11.data.search_results[1].price.raw}}
Prezzo originale: {{11.data.search_results[1].prices[].list_price}}
Rating: {{11.data.search_results[1].rating}}/5 ({{11.data.search_results[1].ratings_total}} recensioni)
Immagine: {{11.data.search_results[1].image}}
Link: https://www.amazon.it/dp/{{11.data.search_results[1].asin}}?tag=sceltacasa21-21

[Ripetere per prodotti 2-5 con search_results[2] fino a search_results[5]]

ISTRUZIONI: Usa SOLO questi 5 prodotti reali. NON inventarne altri. Usa prezzi e valutazioni ESATTAMENTE come forniti. Se prezzo originale diverso dal prezzo attuale, mostra prezzo barrato.
```

**Moduli [3][4][7][8] — Text Parser: Match Pattern**
- Tutti con: Multiline YES, Singleline **NO** (importante! Se Singleline=YES il `.+` cattura anche righe successive)
- Text source: modulo [2] → data.content[].message (o simile dalla risposta Claude)
- Pattern modulo [3] (corpo): `([\s\S]+?)(?=meta_title:)` — Singleline YES solo per questo
- Pattern modulo [4] (meta_title): `meta_title:\s*(.+)`
- Pattern modulo [7] (meta_description): `meta_description:\s*(.+)`
- Pattern modulo [8] (slug): `slug:\s*([\w-]+)`

**Modulo [9] — WordPress: Create a Post**
- Connection: tramite plugin Make Connector (API key nel plugin WP)
- Title: modulo [4] → $1
- Content: modulo [3] → $1
- Type: Articoli
- Slug: modulo [8] → $1
- Status: Draft (cambiare a Publish quando rodato)
- Categories: Map ON → {{1.ID Categoria (K)}} dal Google Sheet

**Modulo [10] — Google Sheets: Update a Row**
- Row number: {{1.Row number}}
- Colonna I (Stato): "pubblicato"
- Colonna J (Data Pubblicazione): {{now}}

**Scheduling:** ogni giorno alle 03:00

### Google Sheets — Struttura
Colonne: A-Keyword, B-Intento, C-Silo, D-Tipo Articolo, E-Volume, F-SD, G-CPC, H-Priorità, I-Stato, J-Data Pubblicazione, K-ID Categoria

### API Keys
- **Rainforest API:** in file `.env` (NON su GitHub)
- **Claude/Anthropic:** configurata direttamente in Make.com headers
- **Perplexity:** configurata ma non attiva nel flusso attuale (serve per articoli "guida" futuri)

## File del progetto

| File | Contenuto | Quando usarlo |
|---|---|---|
| `setup.md` | Documento di progetto completo | Panoramica generale |
| `system-prompt-v5.txt` | System prompt Claude attuale | Da copiare in Make.com campo System |
| `style-completo.css` | CSS completo del sito | Da copiare in WP → CSS aggiuntivo |
| `keyword-database.csv` | 48 keyword con ID categoria | Da importare in Google Sheets |
| `make-body-perplexity-v4.json` | Body Perplexity (per articoli guida futuri) | Modulo HTTP Perplexity |
| `pagine-statiche.html` | Testi Chi Siamo, Contatti, Privacy | Pagine WordPress |
| `.env` | API key Rainforest | NON committare su GitHub |

## Problemi risolti e lezioni apprese

### Make.com
- **JSON string + dati Perplexity/Rainforest = errore JSON** → Soluzione: usare **Data Structure** per il modulo Claude, che fa escape automatico
- **Timeout 40s troppo basso per Claude** → Impostare a **120000ms**
- **max_tokens 4096 insufficiente per HTML ricco** → Impostare a **8192**
- **Variabili Make.com nel JSON raw** appaiono come `{{1.0}}` nel codice ma come tag verdi nell'editor
- **Categorie WordPress vogliono ID numerico**, non il nome testuale

### Text Parser
- **Singleline YES** fa sì che `.+` catturi anche newline → meta_title include anche meta_description e slug
- **Soluzione:** Singleline NO per meta_title, meta_description, slug. Singleline YES solo per il corpo (che usa `[\s\S]+?`)
- **Slug pattern:** usare `([\w-]+)` per catturare solo caratteri URL-valid

### WordPress / GeneratePress
- **Footer gap bianco:** il fix è su `#page` (NON body): `#page { min-height: 100vh; display: flex; flex-direction: column; }`
- **Sidebar:** rimossa via CSS, non via impostazioni tema
- **Connessione Make.com:** richiede plugin "Make Connector" che genera API key
- **Nome autore:** modificare in Utenti → Profilo → "Nome da visualizzare pubblicamente"

### Contenuti
- **Claude inventa prodotti** se non gli dai dati reali → Rainforest API risolve al 100%
- **Perplexity inventa ASIN e modelli** → non affidabile per dati prodotto
- **Claude genera Markdown invece di HTML** se non specificato esplicitamente nel prompt
- **Anno nel titolo:** Claude scrive "2024" se non gli passi l'anno corrente nel user message

## Tipi di articolo

| Tipo | Stato | Fonte dati | Note |
|---|---|---|---|
| **pillar** | ✅ Attivo | Rainforest API | 35 keyword pronte |
| **vs** | ⏸ In attesa | Da implementare (serve Router Make.com) | 5 keyword |
| **guida** | ⏸ In attesa | Perplexity API (da aggiungere) | 8 keyword |

## Roadmap monetizzazione
1. **Ora:** solo affiliazione Amazon
2. **Dopo 20+ articoli:** applicare a Google AdSense
3. **Dopo 10K visite/mese:** passare a Ezoic
4. **Dopo 50K visite/mese:** passare a Mediavine

## To-Do prioritari
1. Collegare Google Search Console
2. Collegare Google Analytics 4
3. Pubblicare primi 5 articoli (review manuale)
4. Implementare Router per articoli vs/guida
5. Espandere keyword database a 100+
6. Raggiungere 3 vendite Amazon (attiva PA API per immagini prodotto native)