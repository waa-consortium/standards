# WAA IMPLEMENTATION GUIDE FOR ELEVENTY v1.0

**Author:** WAA Consortium - Technical Steering Committee
**Date:** March 2026
**Status:** Official Reference Implementation

---

## INDICE

1. [INTRODUZIONE](#1-introduzione)
    - 1.1 Scopo del Documento
    - 1.2 Cos'è WAA in Eleventy
    - 1.3 Filosofia Core: Il Sistema dei Mini-Grafi

2. [ARCHITETTURA GENERALE](#2-architettura-generale)
    - 2.1 I Quattro Mini-Grafi
    - 2.2 Il Pattern dei Connettori
    - 2.3 Risoluzione Dipendenze a Build Time

3. [STRUTTURA FILE](#3-struttura-file)
    - 3.1 Organizzazione Directory
    - 3.2 File Core Spiegati
    - 3.3 Componenti Opzionali

4. [IL PATTERN REGISTRY (waa-registry.js)](#4-il-pattern-registry-waa-registryjs)
    - 4.1 Scopo e Funzione
    - 4.2 Definizione Entità
    - 4.3 Sistema Placeholder
    - 4.4 Estensione Multi-Progetto

5. [IL CONTEXT BUILDER (core.njk)](#5-il-context-builder-corenjk)
    - 5.1 Come Funziona
    - 5.2 Parametri
    - 5.3 Logica di Risoluzione
    - 5.4 Generazione Output

6. [COMPONENTI MINI-GRAFO](#6-componenti-mini-grafo)
    - 6.1 authority.njk - Organizzazione, Persone, Metodo
    - 6.2 commerce.njk - Prodotti, Offerte, Servizi
    - 6.3 content.njk - Articoli, Quiz, Collezioni

7. [INTEGRAZIONE A LIVELLO PAGINA](#7-integrazione-a-livello-pagina)
    - 7.1 Configurazione Frontmatter
    - 7.2 Dichiarazione Entità Richieste
    - 7.3 Proprietà Semantiche (about, mentions, isBasedOn)

8. [ESEMPI PRATICI](#8-esempi-pratici)
    - 8.1 Pagina Articolo Blog
    - 8.2 Pagina Prodotto
    - 8.3 Pagina Quiz
    - 8.4 Pagina Hub Tematico

9. [ARCHITETTURA MULTI-PROGETTO](#9-architettura-multi-progetto)
    - 9.1 Registry Condiviso tra Siti
    - 9.2 Override per Dominio
    - 9.3 Mantenere @id Unici

10. [VALIDAZIONE E TEST](#10-validazione-e-test)
    - 10.1 Test con Google Rich Results
    - 10.2 Errori Comuni
    - 10.3 Debug Risoluzione Grafo

11. [APPENDICE](#11-appendice)
    - 11.1 Codice Completo
    - 11.2 Riferimenti Schema.org
    - 11.3 Predicati WAA-Specifici

---

## 1. INTRODUZIONE

### 1.1 Scopo del Documento

Questo documento fornisce l'implementazione di riferimento ufficiale dello Standard WAA (Web Application Architecture) v3.0 per il generatore di siti statici Eleventy. Traduce i concetti teorici di topologia sferica e orchestrazione semantica in codice concreto e funzionante.

Il pubblico di riferimento è costituito da sviluppatori che:
- Costruiscono siti web con Eleventy
- Vogliono implementare architettura conforme a WAA
- Devono gestire relazioni semantiche complesse su più progetti
- Cercano di massimizzare SEO e AI-readiness attraverso dati strutturati

### 1.2 Cos'è WAA in Eleventy

WAA non è un plugin o una libreria. È un pattern di progettazione e un protocollo per organizzare codice, contenuto e semantica. In Eleventy, WAA si manifesta come:

- Un sistema modulare di Mini-Grafi (frammenti JSON-LD)
- Un registro centrale di entità canoniche
- Un risolutore di dipendenze a build time
- Una convenzione frontmatter per dichiarare relazioni semantiche

L'obiettivo è rendere ogni pagina di un sito Eleventy un nodo denso e auto-contenuto del Knowledge Graph che referenzia e include tutte le entità rilevanti, senza duplicazione di codice.

### 1.3 Filosofia Core: Il Sistema dei Mini-Grafi

Gli approcci tradizionali ai dati strutturati in Eleventy spesso risultano in:

- Un file schema globale enorme (pesante, poco focalizzato)
- Frammenti specifici per pagina (isolati, disconnessi)
- Gestione manuale degli @id (soggetta a errori)

WAA risolve questo attraverso i Mini-Grafi: moduli semantici piccoli e focalizzati che vengono assemblati a build time in base al contesto della pagina. Ogni Mini-Grafo rappresenta un cluster logico di entità:

- **BASE:** Sempre presente (Website, WebPage, Breadcrumb)
- **AUTORITÀ:** Origini legali e tecnologiche (Organization, Person, Method)
- **TRANSAZIONALE:** Entità commerciali (Product, Offer, Service)
- **CONTENUTO:** Entità educative ed editoriali (Article, Quiz, Collection)

Questi Mini-Grafi sono collegati attraverso riferimenti @id. L'innovazione è che quando una pagina referenzia un'entità da un altro Mini-Grafo (es. un Quiz che referenzia il metodo WAA), il sistema di build include automaticamente la definizione completa di quell'entità, creando un grafo denso in una singola risposta HTTP.

---

## 2. ARCHITETTURA GENERALE

### 2.1 I Quattro Mini-Grafi

L'implementazione WAA per Eleventy è costruita attorno a quattro Mini-Grafi distinti, ciascuno con uno specifico scopo semantico:

**CONTAINER BASE (Livello Topologia)**
- Sempre presente su ogni pagina
- Dichiara il contenitore sito (@type WebSite)
- Dichiara la pagina corrente (@type WebPage, Article, ecc.)
- Fornisce breadcrumb navigation
- **File:** schema/core.njk

**AUTORITÀ (Livello Origini)**
- Presente su pagine che necessitano di stabilire credibilità
- Dichiara l'Organizzazione (entità legale)
- Dichiara le Persone (autori, fondatori)
- Dichiara il Metodo e il framework WAA
- **File:** schema/authority.njk

**TRANSAZIONALE (Livello Pozzo)**
- Presente su pagine prodotto, servizio o checkout
- Dichiara Prodotti e varianti
- Dichiara Offerte (prezzo, disponibilità)
- Dichiara Venditori
- **File:** schema/commerce.njk

**EDUCATIVO (Livello Passaggio)**
- Presente su pagine di contenuto (blog, academy, risorse)
- Dichiara Articoli con intento educativo
- Dichiara Quiz per engagement
- Dichiara CollectionPages per confronti
- **File:** schema/content.njk

### 2.2 Il Pattern dei Connettori

I Mini-Grafi non sono silos isolati. Si connettono attraverso un sistema di predicati semantici definiti da schema.org ed estesi da WAA:

**CATEGORIE DI PREDICATI:**

**Gerarchici:**
- isPartOf (figlio verso padre)
- hasPart (padre verso figli)
- isBasedOn (derivato da origine concettuale)

**Referenziali:**
- about (topic primario)
- mentions (riferimento secondario)
- citation (riferimento accademico)

**Attribuzionali:**
- author (creatore)
- publisher (editore ufficiale)
- creator (originatore)
- discovererOf (estensione WAA per scoperta metodologica)

**Transazionali:**
- offers (disponibilità commerciale)
- workExample (istanza concreta di un framework)
- subjectOf (entità che discute di questa)

### 2.3 Risoluzione Dipendenze a Build Time

L'innovazione tecnica chiave è la risoluzione delle dipendenze a build time. Quando Eleventy genera una pagina:

1. Legge dal frontmatter l'array requiredEntities
2. Carica il registro centrale (waa-registry.js)
3. Risolve ogni ID entità richiesto nella definizione completa
4. Inietta tutte le definizioni risolte nel grafo della pagina
5. Genera un singolo tag script JSON-LD con il grafo completo

Questo significa:
- Le pagine dichiarano ciò di cui hanno bisogno (dipendenze semantiche)
- Il sistema di build fornisce le definizioni (dependency injection)
- Il browser riceve un grafo completo in una volta (performance)
- Google vede conoscenza densa e connessa (SEO)

---

## 3. STRUTTURA FILE

### 3.1 Organizzazione Directory

Un progetto Eleventy conforme a WAA dovrebbe includere la seguente struttura:

```
your-eleventy-site/

src/

_data/
   waa-registry.js              # Registro centrale entità

_includes/

components/

schema/
       core.njk                  # Container base (sempre incluso)
       authority.njk             # Organizzazione, Persone, Metodo
       commerce.njk              # Prodotti, Offerte
       content.njk               # Articoli, Quiz, Collezioni

layouts/
   base.njk                      # Layout base che include core.njk
   post.njk                      # Layout post blog
   product.njk                   # Layout pagina prodotto

utils/
    schema-builder.js              # Funzioni di utilità (opzionale)
```

### 3.2 File Core Spiegati

**waa-registry.js**
- **Scopo:** Repository centrale di tutte le definizioni di entità canoniche
- **Posizione:** src/_data/
- **Comportamento:** Oggetto esportato con organization, method, waa, authors, website, products
- **Accesso:** Automaticamente disponibile in tutti i template come waaRegistry

**core.njk**
- **Scopo:** Context Builder che assembla il grafo finale
- **Posizione:** src/_includes/components/schema/
- **Parametri:** pageContext, requiredEntities, registry
- **Output:** Tag script JSON-LD con array @graph

**authority.njk, commerce.njk, content.njk**
- **Scopo:** Definizioni complete delle entità di ciascun Mini-Grafo
- **Uso:** Mai inclusi direttamente; le entità sono accessibili tramite registry

### 3.3 Componenti Opzionali

**schema-builder.js**
Funzioni di utilità per generare stringhe @id, validare grafi, unire entità. Può essere usato in .eleventy.js per aggiungere filtri o shortcode.

**examples/**
Directory contenente pagine di esempio che dimostrano diversi casi d'uso. Non richiesto per produzione ma utile per apprendimento.

---

## 4. IL PATTERN REGISTRY (waa-registry.js)

### 4.1 Scopo e Funzione

Il pattern registry è il fondamento dell'implementazione WAA per Eleventy. Invece di duplicare le definizioni delle entità in template o pagine multiple, tutte le definizioni canoniche vivono in un file centrale che funge da fonte di verità.

Questo registro:
- Viene caricato una volta a build time
- È disponibile a tutti i template tramite il globale waaRegistry
- Usa sintassi placeholder per valori specifici del progetto
- Può essere esteso o sovrascritto per progetto

### 4.2 Definizione Entità

Ogni entità nel registro segue una struttura rigorosa:

```javascript
{
  "@id": "https://{{ site.domain }}/#entity-name",
  "@type": "SchemaOrgType",
  "property1": "value",
  "property2": { "@id": "reference-to-other-entity" },
  "property3": [ "array", "of", "values" ]
}
```

**REGOLE CRITICHE:**
- Ogni entità DEVE avere un @id unico
- I valori @id DEVONO essere URI assoluti (incluso dominio)
- I riferimenti ad altre entità DEVONO usare oggetti @id
- Gli array DEVONO essere usati per valori multipli della stessa proprietà

### 4.3 Sistema Placeholder

Il registro usa placeholder in stile Nunjucks ({{ variabile }}) per valori che cambiano per progetto o ambiente:

```javascript
{
  "@id": "https://{{ site.domain }}/#org",
  "name": "{{ site.orgName }}",
  "legalName": "{{ site.legalName }}"
}
```

Questi placeholder vengono risolti quando:
- Il registro viene caricato da Eleventy
- Le variabili del sito sono definite in _data/site.js o _data/site.json
- Vengono applicati override specifici per ambiente

### 4.4 Estensione Multi-Progetto

Per agenzie che gestiscono più siti clienti, il registro può essere esteso attraverso composizione:

```javascript
module.exports = (siteConfig) => {
  const baseRegistry = {
    organization: { ... },
    method: { ... },
    waa: { ... }
  };
  
  // Unisci con override specifici del cliente
  return {
    ...baseRegistry,
    ...siteConfig.overrides,
    products: siteConfig.products || {}
  };
};
```

Questo permette di mantenere un'identità WAA core personalizzando per progetto.

---

## 5. IL CONTEXT BUILDER (core.njk)

### 5.1 Come Funziona

core.njk è il cervello dell'implementazione. Esso:

1. Inizializza un array graph vuoto
2. Aggiunge sempre l'entità WebSite (dal registro)
3. Costruisce l'entità WebPage corrente da pageContext
4. Itera attraverso l'array requiredEntities
5. Per ogni ID entità, cerca la definizione nel registro
6. Aggiunge ogni definizione trovata al grafo
7. Output dell'array completo come JSON-LD

### 5.2 Parametri

Il template accetta tre parametri:

**pageContext (richiesto)**
Oggetto contenente:
- url: URL pagina corrente
- title: titolo pagina
- description: descrizione pagina
- type: tipo schema.org (WebPage, Article, Quiz, ecc.)
- about: topic primario (stringa o array)
- mentions: riferimenti secondari (array)
- isBasedOn: origini concettuali (array)

**requiredEntities (opzionale)**
Array di suffissi ID entità da risolvere:
- "#org" -> organization
- "#method" -> method
- "#waa" -> waa
- "#product-olive" -> products.olive
- "#author-1" -> authors[0]

**registry (iniettato automaticamente)**
L'oggetto waaRegistry da _data/

### 5.3 Logica di Risoluzione

Il risolutore usa un semplice sistema di mapping:

```twig
if entityId == "#org" -> return registry.organization
if entityId == "#method" -> return registry.method
if entityId == "#waa" -> return registry.waa
if entityId starts with "#product-" -> 
  key = entityId.replace("#product-", "")
  return registry.products[key]
if entityId starts with "#author-" -> 
  index = parseInt(entityId.replace("#author-", "")) - 1
  return registry.authors[index]
```

Questo mapping può essere esteso per tipi di entità aggiuntivi.

### 5.4 Generazione Output

L'output finale è un tag script JSON-LD contenente un array @graph:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { ... WebSite ... },
    { ... WebPage ... },
    { ... Organization ... },
    { ... Product ... },
    { ... tutte le entità risolte ... }
  ]
}
```

Questo formato è quello preferito da Google per entità multiple sulla stessa pagina.

---

## 6. COMPONENTI MINI-GRAFO

### 6.1 authority.njk - Organizzazione, Persone, Metodo

Questo componente contiene le definizioni complete delle entità di autorità. Non viene mai incluso direttamente; invece, le sue entità sono accessibili tramite il registro.

**Entità chiave:**

**Organization**
- Nome legale, partita IVA, indirizzo
- Relazione knowsAbout con Method e WAA
- Relazione publisher con Website
- Relazione provider con servizi

**Person (Autori/Team)**
- Nome, titolo, sameAs (profili social)
- Relazione worksFor con Organization
- Relazione creator/discovererOf con Method/WAA

**Method**
- CreativeWork che rappresenta la metodologia
- Relazione isBasedOn con Manifesto (se applicabile)
- Relazione teaches con risorse educative
- Relazione hasPart con WAA

**WAA**
- SoftwareSourceCode + Product (doppio typing)
- Relazione isBasedOn con Method
- softwareRequirements per implementazione
- Relazione workExample con siti concreti
- Relazione offers con disponibilità commerciale

### 6.2 commerce.njk - Prodotti, Offerte, Servizi

Entità transazionali per pagine e-commerce e servizi:

**Product**
- nome, descrizione, immagine
- sku, mpn, brand
- Relazione isBasedOn con Method/WAA
- Relazione workExample con implementazioni WAA

**Offer**
- prezzo, priceCurrency, disponibilità
- Relazione seller con Organization
- validFrom, validThrough

**Service**
- serviceType, provider
- areaServed, availableChannel

### 6.3 content.njk - Articoli, Quiz, Collezioni

Entità educative ed editoriali:

**Article**
- headline, description, articleBody
- about topic, mentions references
- Relazione citation con fonti/WAA
- Relazione isRelatedTo con Quiz

**Quiz**
- name, description
- Relazione educationalUse con Method
- Relazione isBasedOn con Resources/WAA
- teaches competenze
- leads a pagine di conversione

**CollectionPage**
- name, description
- Relazione isBasedOn con Data/Method/WAA
- about entità confrontate
- mentions tecnologie/prodotti

---

## 7. INTEGRAZIONE A LIVELLO PAGINA

### 7.1 Configurazione Frontmatter

Ogni pagina che vuole partecipare al grafo semantico WAA deve configurare il proprio frontmatter:

```yaml
---
title: "Titolo Pagina"
description: "Descrizione pagina"
type: "Article"  # Tipo schema.org
layout: "post.njk"

# Frontmatter specifico WAA
about: "Topic primario, focus SEO"
mentions: 
  - "Concetto secondario 1"
  - "Concetto secondario 2"
isBasedOn:
  - { "@id": "https://dominio.it/#method" }
  - { "@id": "https://dominio.it/#waa" }
requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#product-olive"
---
```

### 7.2 Dichiarazione Entità Richieste

L'array requiredEntities dice al Context Builder quali entità aggiuntive includere. Pensalo come istruzioni di import per dipendenze semantiche.

**Regole:**
- Usa solo il suffisso (il dominio viene preposto automaticamente)
- L'ordine non conta
- I duplicati vengono gestiti con grazia
- Entità mancanti vengono ignorate silenziosamente (nessun errore di build)

**Pattern standard:**
- Post blog: ["#org", "#method", "#waa"]
- Pagine prodotto: ["#org", "#waa", "#product-{id}"]
- Pagina Chi Siamo: ["#org", "#method", "#waa", "#author-1", "#author-2"]
- Pagine Quiz: ["#org", "#method", "#waa", "#product-{id}"]

### 7.3 Proprietà Semantiche (about, mentions, isBasedOn)

Queste proprietà popolano direttamente l'entità WebPage:

**about**
- Topic primario della pagina
- Usato per SEO e classificazione contenuti
- Può essere stringa o array di stringhe

**mentions**
- Concetti o entità secondari
- Crea connessioni semantiche più deboli
- Sempre un array

**isBasedOn**
- Origini concettuali (Method, WAA)
- Crea relazioni di derivazione
- Array di oggetti @id

---

## 8. ESEMPI PRATICI

### 8.1 Pagina Articolo Blog

File: src/blog/waa-eleventy-guida.md

```yaml
---
title: "Guida Completa a WAA in Eleventy"
description: "Impara come implementare architettura web sferica"
date: 2026-03-08
type: Article
layout: post.njk

about: "Implementazione WAA, Eleventy, web sferico"
mentions:
  - "SEO semantico"
  - "knowledge graph"
  - "mini-grafi"

isBasedOn:
  - { "@id": "https://esempio.it/#method" }
  - { "@id": "https://esempio.it/#waa" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#author-1"
---

Questa guida spiega come implementare WAA in Eleventy...
```

### 8.2 Pagina Prodotto

File: src/prodotti/olio-extravergine.md

```yaml
---
title: "Olio Extravergine di Oliva - Latta 5L"
description: "Olio biologico spremuto a freddo"
type: Product
layout: product.njk

sku: "OLIO-5L-BIO"
price: 49.90
priceCurrency: "EUR"
availability: "https://schema.org/InStock"

about: "olio extravergine, olio biologico"
mentions:
  - "produzione sostenibile"
  - "filiera controllata"

isBasedOn:
  - { "@id": "https://esempio.it/#method" }
  - { "@id": "https://esempio.it/#waa" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#product-olio-5l"
---

Descrizione dettagliata del prodotto...
```

### 8.3 Pagina Quiz

File: src/quiz/truffe-olio.md

```yaml
---
title: "Scopri se conosci le truffe dell'olio"
description: "Quiz interattivo sulle frodi alimentari"
type: Quiz
layout: quiz.njk

about: "truffe olio d'oliva, frodi alimentari"
mentions:
  - "qualità olio"
  - "certificazioni"

educationalUse: "apprendimento interattivo"
teaches: "riconoscimento truffe alimentari"

isBasedOn:
  - { "@id": "https://esempio.it/#method" }
  - { "@id": "https://esempio.it/#waa" }
  - { "@id": "https://esempio.it/#resource-truffe" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#product-olio-5l"
---

Contenuto del quiz...
```

### 8.4 Pagina Hub Tematico

File: src/hub/truffe-alimentari.md

```yaml
---
title: "Hub Truffe Alimentari"
description: "Risorsa completa su frodi e contraffazioni"
type: CollectionPage
layout: hub.njk

about: "truffe alimentari, frodi, contraffazioni"
mentions:
  - "olio d'oliva"
  - "pesce"
  - "formaggi DOP"

hasPart:
  - { "@id": "https://esempio.it/articoli/truffe-olio" }
  - { "@id": "https://esempio.it/quiz/truffe-olio" }
  - { "@id": "https://esempio.it/confronti/oli" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
---

Contenuto dell'hub...
```

---

## 9. ARCHITETTURA MULTI-PROGETTO

### 9.1 Registry Condiviso tra Siti

Per agenzie o consorzi che gestiscono più siti conformi a WAA, il registro può essere condiviso tramite:

- Pacchetto NPM (@waa-consortium/registry)
- Git submodule
- Registry npm privato
- API JSON endpoint

Ogni sito importa il registro base e lo estende:

```javascript
const baseRegistry = require('@waa-consortium/registry');

module.exports = {
  ...baseRegistry,
  organization: {
    ...baseRegistry.organization,
    name: "Nome Specifico Cliente",
    legalName: "Cliente Srl"
  },
  products: require('./client-products.json')
};
```

### 9.2 Override per Dominio

I campi @id devono essere unici per dominio. Usa la variabile site.domain:

```javascript
organization: {
  "@id": "https://{{ site.domain }}/#org",
  "name": "{{ site.orgName }}"
}
```

Questo assicura che entità di siti diversi non collidano nell'indice di Google.

### 9.3 Mantenere @id Unici

**Regole per la gestione @id:**
- Includi sempre il dominio completo
- Non usare mai percorsi relativi
- Usa frammenti (#) per entità nella pagina
- Mantieni frammenti coerenti tra siti
- Documenta la tua convenzione di denominazione frammenti

**Buono:** https://cliente1.it/#org
**Cattivo:** /#org
**Cattivo:** cliente1.it/#organizzazione

---

## 10. VALIDAZIONE E TEST

### 10.1 Test con Google Rich Results

Dopo l'implementazione, valida le tue pagine con:

1. **Google Rich Results Test**
   - Inserisci la tua URL
   - Controlla errori e avvisi
   - Verifica che tutte le entità attese appaiano

2. **Schema.org Validator**
   - Incolla il tuo output JSON-LD
   - Valida rispetto al vocabolario schema.org

3. **Google Search Console**
   - Monitora i report sui miglioramenti
   - Tieni traccia delle impressioni dei rich result

### 10.2 Errori Comuni

**Riferimenti @id mancanti**
Entità referenziate ma non definite in requiredEntities
**Soluzione:** Assicurati che tutti gli @id nelle proprietà della pagina appaiano in requiredEntities

**Valori @id duplicati**
Stesso @id usato per entità diverse
**Soluzione:** Usa frammenti unici per tipo di entità

**Annidamento non valido**
Mettere Offer dentro Product in modo errato
**Soluzione:** Segui la gerarchia dei tipi schema.org

**Dominio mancante**
Usare percorsi @id relativi
**Soluzione:** Usa sempre URL assoluti con dominio

### 10.3 Debug Risoluzione Grafo

Se un'entità non appare nell'output:

1. Controlla che requiredEntities includa il suffisso corretto
2. Verifica che il registro abbia quella chiave definita
3. Conferma che la logica di mapping in core.njk corrisponda alla chiave
4. Controlla errori di battitura nel confronto entityId
5. Assicurati che il registro sia caricato correttamente in _data/

Usa la modalità debug di Eleventy:
```bash
npx @11ty/eleventy --dry-run --verbose
```

---

## 11. APPENDICE

### 11.1 Codice Completo

Il codice completo per tutti i file è disponibile nella directory implementations/eleventy/ del repository degli standard WAA.

### 11.2 Riferimenti Schema.org

Tipi core usati nell'implementazione WAA:

- WebSite
- WebPage
- Article
- Quiz
- CollectionPage
- Product
- Offer
- Service
- Organization
- Person
- CreativeWork
- SoftwareSourceCode

### 11.3 Predicati WAA-Specifici

Estensioni e convenzioni:

**discovererOf**
Indica che una Person ha scoperto o originato un Metodo/WAA

**educationalUse (con valori WAA)**
- "apprendimento"
- "valutazione"
- "simulazione"

**leads**
Indica che un Quiz o Contenuto porta a una pagina di conversione (Pozzo)

**workExample**
Sito web concreto costruito con framework WAA

**softwareRequirements**
Requisiti tecnici per implementazione WAA

---

## FINE DOCUMENTO
