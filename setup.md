# DOCUMENTO DI PROGETTO: Content Farmer — Scelta Casa

## Obiettivo
Sistema di automazione che genera e pubblica articoli SEO-ottimizzati su elettrodomestici per la casa, monetizzati tramite Amazon Associates. Il sistema lavora in background ogni notte, costruendo un asset digitale profittevole nel tempo.

## Sito Web
- **Dominio:** sceltacasa.com
- **Hosting:** Hostinger Premium (data center Germania)
- **CMS:** WordPress con tema GeneratePress
- **Tag Amazon Associates:** sceltacasa21-21

## Stack Tecnologico

| Componente | Strumento | Ruolo |
|---|---|---|
| CMS | WordPress + GeneratePress | Pubblicazione articoli |
| Orchestratore | Make.com | Automazione del flusso |
| Ricerca Prodotti | Rainforest API | Dati reali da Amazon.it (ASIN, prezzi, rating, immagini) |
| Ricerca Informativa | Perplexity API (sonar) | Criteri di scelta, fasce prezzo, errori comuni (per articoli guida) |
| Scrittore | Claude API (Sonnet) | Generazione articoli HTML con dati reali |
| Database Input | Google Sheets | Keyword, intenti, categorie, stato pubblicazione |
| SEO | Rank Math | Meta title, description, sitemap, schema |
| Immagini | ShortPixel | Compressione e conversione WebP |
| Link Affiliazione | Pretty Links | Gestione e tracking link Amazon |
| Cache | LiteSpeed Cache | Performance e velocità |
| Analytics | Google Analytics 4 + Search Console | Monitoraggio traffico e indicizzazione |

## Nicchia e Architettura Contenuti

### Nicchia: Elettrodomestici e piccoli apparecchi casa

### Silos (Categorie WordPress)

**Cucina:**
- Friggitrici ad Aria (ID: 13)
- Robot da Cucina (ID: 14)
- Macchine Caffè (ID: 15)
- Piccoli Elettrodomestici Cucina (ID: 16) — bollitori, tostapane, sottovuoto, essiccatori, estrattori

**Climatizzazione:**
- Deumidificatori (ID: 17)
- Purificatori Aria (ID: 18)
- Ventilatori (ID: 19)
- Condizionatori Portatili (ID: 20)
- Riscaldamento Elettrico (ID: 21) — stufe, termoventilatori, termoconvettori

**Guide all'Acquisto** — categoria trasversale per le guide

### Tipi di Articolo

| Tipo | Descrizione | Esempio keyword | Fonte dati | Lunghezza |
|---|---|---|---|---|
| **pillar** | Classifica dei migliori prodotti | "miglior friggitrice ad aria" | Rainforest API (5 prodotti reali) | 2000-2500 parole |
| **vs** | Confronto diretto tra 2 prodotti/categorie | "friggitrice ad aria o forno ventilato" | Rainforest API (2 prodotti) | 1200-1800 parole |
| **guida** | Come scegliere una categoria di prodotti | "come scegliere friggitrice ad aria" | Perplexity API (criteri, fasce prezzo) | 1500-2200 parole |

### Struttura articolo pillar
1. Intro (150-200 parole) con keyword nel primo paragrafo
2. Tabella comparativa con i 5 prodotti reali a colpo d'occhio
3. Product box per ogni prodotto (badge, rating, pro/contro, bottone Amazon)
4. Come scegliere (info-box con criteri)
5. Errori da evitare (warning-box)
6. Domande frequenti (FAQ box)
7. Verdict box finale con raccomandazione netta e link Amazon

### Struttura articolo vs
1. Intro (100-150 parole)
2. Tabella comparativa con specifiche reali
3. Differenze che contano
4. Quando scegliere A (product-box)
5. Quando scegliere B (product-box)
6. Verdict box finale

### Struttura articolo guida
1. Intro
2. Fattori da valutare (info-box)
3. Fasce di prezzo con tabella reale
4. Errori comuni (warning-box)
5. FAQ
6. Raccomandazioni per budget con link Amazon

## Flusso Automazione Make.com

```
[1] Google Sheets (legge keyword + tipo + silo + ID categoria)
        ↓
[2] Rainforest API (cerca prodotti reali su Amazon.it)
        ↓
[3] Claude API (genera articolo HTML con dati reali, link Amazon, CSS classes)
        ↓
[4-7] Text Parser x4 (estrae: corpo HTML, meta_title, meta_description, slug)
        ↓
[8] WordPress (crea post con titolo, contenuto, slug, categoria)
        ↓
[9] Google Sheets (aggiorna stato → "pubblicato" + data)
```

### Scheduling
- **Frequenza:** ogni notte alle 03:00
- **Articoli al giorno:** 1 (non di più per evitare penalizzazioni Google)
- **Stato iniziale:** Draft (bozza) — controllo manuale prima della pubblicazione
- **Stato produzione:** Publish (dopo rodaggio)

### Evoluzione futura del flusso
- Aggiungere Router per smistare pillar/vs → Rainforest e guida → Perplexity
- Aggiungere generazione immagine di copertina AI per ogni articolo
- Automatizzare meta SEO Rank Math via REST API WordPress
- Immagini prodotto Amazon nei product box (dopo attivazione PA API)

## Keyword Database

Il database è nel Google Sheet "Content Farmer - Database" con queste colonne:

| Colonna | Contenuto |
|---|---|
| A - Keyword | La query target |
| B - Intento | transazionale / informazionale / comparativo |
| C - Silo | Cucina / Climatizzazione |
| D - Tipo Articolo | pillar / vs / guida |
| E - Volume | Volume di ricerca mensile |
| F - SD | SEO Difficulty (da Ubersuggest) |
| G - CPC | Costo per click |
| H - Priorità | alta / media / bassa |
| I - Stato | da scrivere / pubblicato |
| J - Data Pubblicazione | Data di pubblicazione |
| K - ID Categoria | ID WordPress della sotto-categoria |

### Keyword validate (con dati Ubersuggest)

**Cucina — Friggitrici (SD 17-28, Volume 110-18100):**
- miglior friggitrice ad aria (18.1K, SD 22)
- friggitrice ad aria come funziona (3.6K, SD 17)
- miglior friggitrice ad aria qualità prezzo (880, SD 22)
- friggitrice ad aria con doppia resistenza (720, SD 20)
- come scegliere friggitrice ad aria (320, SD 19)

**Cucina — Robot (SD 15-36, Volume 50-1300):**
- miglior robot da cucina (1.3K, SD 23)
- miglior robot da cucina tipo bimby (390, SD 19)
- robot da cucina con frullatore (170, SD 15)

**Climatizzazione — Deumidificatori (SD 17-21, Volume 90-6600):**
- miglior deumidificatore (6.6K, SD 21, CPC €0.60)
- come scegliere deumidificatore (260, SD 17)
- miglior deumidificatore casa (90, SD 21, CPC €0.90)

Totale: **48 keyword** nel database, di cui ~25 validate con dati reali.

## Design e CSS

Il sito usa un CSS custom professionale con:
- **Header sticky scuro** (#1e293b) con logo e navigazione
- **Product box** con badge colorati (Scelta Top, Miglior Q/P, Budget), rating stelle, pro/contro in grid verde/rosso, bottone CTA arancione Amazon
- **Tabelle comparative** con header scuro, righe alternate, riga vincitore evidenziata
- **Info box** (azzurro) e **Warning box** (giallo) per suggerimenti ed errori
- **FAQ box** con domande/risposte stilizzate
- **Verdict box** con sfondo scuro gradient e CTA
- **Footer** con link Chi Siamo, Contatti, Privacy Policy
- **Full-width** senza sidebar
- **Responsive** per mobile
- File CSS completo: `style-completo.css`

## Pagine Statiche
- **Chi Siamo** — presentazione del sito e disclosure affiliazione
- **Contatti** — form WPForms + email info@sceltacasa.com
- **Privacy Policy** — GDPR compliant

## Plugin Installati
- Rank Math SEO
- ShortPixel Image Optimizer
- TablePress
- Pretty Links
- WPCode
- WPForms Lite
- LiteSpeed Cache
- Make Connector

## Costi Mensili Stimati

| Servizio | Costo |
|---|---|
| Hostinger Premium | €2,99/mese |
| Claude API (~30 articoli) | ~€3-5/mese |
| Rainforest API | ~$30/mese (500 ricerche) |
| Perplexity API (opzionale per guide) | ~$5/mese |
| **Totale** | **~€35-40/mese** |

## Ruoli

- **Automazione (95%):** Make.com + Claude + Rainforest API gestiscono tutto
- **Supervisione umana (5%):** Ricerca keyword mensile, quality check a campione, fact-checking prodotti, raffinamento prompt

## To-Do (prossimi step)

- [ ] Collegare Google Search Console
- [ ] Collegare Google Analytics 4
- [ ] Configurare Rainforest API nel flusso Make.com (sostituisce Perplexity per pillar/vs)
- [ ] Testare flusso completo con dati Rainforest
- [ ] Aggiungere generazione immagine copertina AI
- [ ] Pubblicare primi 5 articoli e monitorare indicizzazione
- [ ] Aggiungere Router per smistare pillar/vs/guida a fonti dati diverse
- [ ] Validare keyword rimanenti su Ubersuggest
- [ ] Espandere database a 100+ keyword
- [ ] Raggiungere 3 vendite Amazon per attivare PA API (immagini prodotto)
