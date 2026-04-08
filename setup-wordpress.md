# SETUP WORDPRESS — Content Farmer

## 1. Hosting e Dominio

### Hosting consigliato
**Hostinger Premium** o **SiteGround StartUp** — entrambi sotto €5/mese, con WordPress preinstallato e SSL gratuito.

Perché non hosting gratuiti: Google penalizza siti lenti. Questi due hanno server LiteSpeed/cache integrata che fanno la differenza sul Core Web Vitals.

### Dominio
Scegli un nome brand neutro che non ti vincoli a una sola categoria. Deve funzionare sia per friggitrici che per deumidificatori.

Esempi validi:
- casascelta.it
- lacasapratica.it
- sceltafurba.it
- prodottoperfetto.it
- casaintelligente.it

Evita:
- Nomi con keyword dentro (es. "migliorfriggitrice.it") — limita la crescita e Google li vede come spam
- Nomi troppo lunghi o difficili da scrivere
- Domini .com se il target è solo italiano — meglio .it per segnale di geo-rilevanza

**Azione:** Vai su [Hostinger](https://www.hostinger.it) o [SiteGround](https://www.siteground.it), scegli il piano base, registra il dominio e installa WordPress. Tempo: 15 minuti.

---

## 2. Tema WordPress

### Tema consigliato: GeneratePress (gratuito) + GenerateBlocks

Perché:
- È il tema più leggero in assoluto (~30KB). I siti affiliazione devono caricare sotto i 2 secondi.
- SEO-friendly di default (markup pulito, schema compatibile)
- Nessun page builder pesante (Elementor, Divi = lenti = penalizzazione)
- La versione gratuita basta per iniziare. La Premium (€49/anno) si valuta dopo.

**Azione:** Dashboard WP → Aspetto → Temi → Cerca "GeneratePress" → Installa e Attiva.

---

## 3. Plugin essenziali (solo questi, niente bloat)

| Plugin | Scopo | Gratuito? |
|---|---|---|
| **Rank Math SEO** | Meta title, description, sitemap, schema markup | Sì |
| **WP Fastest Cache** | Cache pagine, minifica CSS/JS | Sì |
| **ShortPixel** | Compressione immagini automatica | Sì (100 img/mese) |
| **TablePress** | Tabelle confronto prodotti (ranking fattore) | Sì |
| **Pretty Links** | Gestione e cloaking link affiliazione | Sì |
| **WPCode** | Snippet per Google Analytics/tag senza toccare il codice | Sì |

### Plugin da NON installare
- Elementor / Divi / WPBakery — troppo pesanti
- Yoast SEO — Rank Math fa tutto meglio e gratis
- Jetpack — bloatware, rallenta tutto
- Plugin social sharing — inutili per siti affiliazione
- Plugin sicurezza multipli — uno basta (Hostinger/SiteGround lo includono)

**Azione:** Dashboard WP → Plugin → Aggiungi nuovo → installa tutti quelli nella tabella sopra.

---

## 4. Struttura Categorie

Crea queste categorie in WordPress (corrispondono ai silos):

```
Dashboard → Articoli → Categorie

- Cucina
  - Friggitrici ad Aria
  - Robot da Cucina
  - Macchine Caffè
  - Piccoli Elettrodomestici Cucina (bollitori, tostapane, sottovuoto, ecc.)

- Climatizzazione
  - Deumidificatori
  - Purificatori Aria
  - Ventilatori
  - Condizionatori Portatili
  - Riscaldamento Elettrico (stufe, termoventilatori, termoconvettori)

- Guide all'Acquisto (categoria trasversale per le guide)
```

Ogni articolo va in UNA sola sotto-categoria. Mai in due. Google deve capire la tassonomia senza ambiguità.

---

## 5. Configurazione Rank Math SEO

Dopo l'installazione, segui la procedura guidata. Impostazioni chiave:

### Generali
- Tipo sito: Blog personale / Piccola azienda
- Sitemap: attiva
- Breadcrumb: attiva (aiuta Google a capire la struttura)

### Titoli e Meta
Imposta i template di default:

**Articolo singolo:**
```
%title% — [Nome Sito]
```

**Pagina categoria:**
```
%term% — Guida e Confronti | [Nome Sito]
```

**Homepage:**
```
[Nome Sito] — Recensioni e Guide per la Casa Intelligente
```

### Schema Markup
- Tipo predefinito articolo: **Article**
- Per gli articoli "pillar" e "vs": aggiungi manualmente lo schema **Product Review** (Rank Math lo supporta nel pannello laterale di ogni post)

---

## 6. Pagine statiche da creare

### Homepage
Non usare la lista articoli di default. Crea una pagina statica con:
- Titolo forte (es. "Le migliori recensioni di elettrodomestici, testate davvero")
- 3-4 box che puntano alle categorie principali
- I 6 articoli più recenti

**Dashboard → Impostazioni → Lettura → "La homepage mostra: Una pagina statica"**

### Pagina Chi Siamo
Obbligatoria per EEAT. Scrivi 200-300 parole su:
- Chi c'è dietro il sito (non serve il nome reale, basta un profilo credibile)
- Come testi/selezioni i prodotti
- Che il sito si mantiene tramite affiliazione (trasparenza = fiducia)

### Pagina Privacy Policy e Cookie Policy
Obbligatorie per legge (GDPR) e per AdSense futuro. Usa il generatore gratuito di Iubenda.

### Pagina Contatti
Un semplice form. Plugin: WPForms Lite (gratuito).

---

## 7. Configurazione Pretty Links (Affiliazione)

Pretty Links trasforma i link Amazon brutti in link puliti e tracciabili:

```
https://www.amazon.it/dp/B08XXXXX?tag=tuotag-21
→ tuosito.it/vai/friggitrice-philips
```

### Setup
1. Installa Pretty Links
2. Dashboard → Pretty Links → Opzioni:
   - Tipo redirect: **301**
   - Prefisso: `vai` (così tutti i link saranno tuosito.it/vai/nome-prodotto)
   - Tracciamento: attivo
3. Per ogni prodotto crea un Pretty Link con:
   - Target URL: link Amazon con tag affiliazione
   - Pretty Link: /vai/nome-prodotto
   - Nofollow: **SÌ** (obbligatorio per link affiliazione)
   - Sponsored: **SÌ** (attributo rel="sponsored" — Google lo richiede)

### Tag Amazon Associates
Se non hai ancora l'account:
1. Vai su [programma-affiliazione.amazon.it](https://programma-affiliazione.amazon.it)
2. Registrati (serve partita IVA o codice fiscale)
3. Il tag avrà formato: `tuonome-21`
4. I link prodotto saranno: `https://www.amazon.it/dp/[ASIN]?tag=tuonome-21`

---

## 8. Impostazioni Performance

### WP Fastest Cache
- Cache: attivo
- Minifica HTML: attivo
- Minifica CSS: attivo
- Combine CSS: attivo
- Minifica JS: attivo (se non rompe nulla)
- Browser Cache: attivo
- Disable Emojis: attivo

### Immagini
- ShortPixel: compressione **Lossy** (miglior rapporto peso/qualità)
- Dimensione massima upload: 1200px di larghezza
- Formato: WebP con fallback JPG (ShortPixel lo fa in automatico)

---

## 9. Checklist finale prima di pubblicare

- [ ] SSL attivo (lucchetto verde nel browser)
- [ ] Sito indicizzabile (Impostazioni → Lettura → "Scoraggia i motori" NON spuntato)
- [ ] Sitemap funzionante: tuosito.it/sitemap_index.xml
- [ ] Google Search Console collegata (verifica proprietà con Rank Math)
- [ ] Google Analytics 4 installato (tramite WPCode)
- [ ] Velocità pagina: testare su PageSpeed Insights, obiettivo score > 85 mobile
- [ ] Pagine legali pubblicate (Privacy, Cookie, Chi Siamo, Contatti)
- [ ] Pretty Links configurato con tag Amazon
- [ ] Categorie create
- [ ] Un articolo di test pubblicato per verificare formattazione
