# WAA IMPLEMENTATION GUIDE FOR ELEVENTY v1.0

Author: WAA Consortium - Technical Steering Committee
Date: March 2026
Status: Official Reference Implementation

---

## TABLE OF CONTENTS


    INTRODUCTION

        1.1 Document Purpose

        1.2 What is WAA in Eleventy

        1.3 Core Philosophy: The Mini-Graph System
        

    GENERAL ARCHITECTURE

        2.1 The Four Mini-Graphs

        2.2 The Connector Pattern

        2.3 Build-Time Dependency Resolution
        

    FILE STRUCTURE

        3.1 Directory Organization

        3.2 Core Files Explained

        3.3 Optional Components
        

    THE REGISTRY PATTERN (waa-registry.js)

        4.1 Purpose and Function

        4.2 Entity Definition

        4.3 Placeholder System

        4.4 Multi-Project Extension
        

    THE CONTEXT BUILDER (core.njk)

        5.1 How It Works

        5.2 Parameters

        5.3 Resolution Logic

        5.4 Output Generation
        

    MINI-GRAPH COMPONENTS

        6.1 authority.njk - Organization, People, Method

        6.2 commerce.njk - Products, Offers, Services

        6.3 content.njk - Articles, Quizzes, Collections
        

    PAGE-LEVEL INTEGRATION

        7.1 Frontmatter Configuration

        7.2 Required Entities Declaration

        7.3 Semantic Properties (about, mentions, isBasedOn)
        

    PRACTICAL EXAMPLES

        8.1 Blog Article Page

        8.2 Product Page

        8.3 Quiz Page

        8.4 Thematic Hub Page
        

    MULTI-PROJECT ARCHITECTURE

        9.1 Shared Registry Across Sites

        9.2 Domain-Specific Override

        9.3 Maintaining Unique @ids
        

    VALIDATION AND TESTING

        10.1 Testing with Google Rich Results

        10.2 Common Errors

        10.3 Graph Resolution Debugging
        

    APPENDIX

        11.1 Complete Code

        11.2 Schema.org References

        11.3 WAA-Specific Predicates
        
        ---
        
        ## 1. INTRODUCTION
        

1.1 Document Purpose


This document provides the official reference implementation of the WAA (Web Application Architecture) v3.0 Standard for the Eleventy static site generator. It translates theoretical concepts of spherical topology and semantic orchestration into concrete, working code.

The target audience consists of developers who:

    Build websites with Eleventy

    Want to implement WAA-compliant architecture

    Need to manage complex semantic relationships across multiple projects

    Aim to maximize SEO and AI-readiness through structured data
    

1.2 What is WAA in Eleventy


WAA is not a plugin or a library. It is a design pattern and protocol for organizing code, content, and semantics. In Eleventy, WAA manifests as:


    A modular system of Mini-Graphs (JSON-LD fragments)

    A central registry of canonical entities

    A build-time dependency resolver

    A frontmatter convention for declaring semantic relationships
    
    The goal is to make every page of an Eleventy site a dense, self-contained node of the Knowledge Graph that references and includes all relevant entities, without code duplication.
    

1.3 Core Philosophy: The Mini-Graph System


Traditional approaches to structured data in Eleventy often result in:


    One huge global schema file (heavy, unfocused)

    Page-specific fragments (isolated, disconnected)

    Manual @id management (error-prone)
    
    WAA solves this through Mini-Graphs: small, focused semantic modules that are assembled at build time based on the page context. Each Mini-Graph represents a logical cluster of entities:
    

    BASE: Always present (Website, WebPage, Breadcrumb)

    AUTHORITY: Legal and technological origins (Organization, Person, Method)

    TRANSACTIONAL: Commercial entities (Product, Offer, Service)

    CONTENT: Educational and editorial entities (Article, Quiz, Collection)
    
    These Mini-Graphs are linked through @id references. The innovation is that when a page references an entity from another Mini-Graph (e.g., a Quiz referencing the WAA method), the build system automatically includes the complete definition of that entity, creating a dense graph in a single HTTP response.
    
    ---
    
    ## 2. GENERAL ARCHITECTURE
    

2.1 The Four Mini-Graphs


The WAA implementation for Eleventy is built around four distinct Mini-Graphs, each with a specific semantic purpose:

BASE CONTAINER (Topology Layer)

    Always present on every page

    Declares the site container (@type WebSite)

    Declares the current page (@type WebPage, Article, etc.)

    Provides breadcrumb navigation

    File: schema/core.njk
    
    AUTHORITY (Origins Layer)

    Present on pages that need to establish credibility

    Declares the Organization (legal entity)

    Declares People (authors, founders)

    Declares the Method and the WAA framework

    File: schema/authority.njk
    
    TRANSACTIONAL (Well Layer)

    Present on product, service, or checkout pages

    Declares Products and variants

    Declares Offers (price, availability)

    Declares Sellers

    File: schema/commerce.njk
    
    EDUCATIONAL (Passage Layer)

    Present on content pages (blog, academy, resources)

    Declares Articles with educational intent

    Declares Quizzes for engagement

    Declares CollectionPages for comparisons

    File: schema/content.njk
    

2.2 The Connector Pattern


Mini-Graphs are not isolated silos. They connect through a system of semantic predicates defined by schema.org and extended by WAA:

PREDICATE CATEGORIES:

Hierarchical:

    isPartOf (child to parent)

    hasPart (parent to children)

    isBasedOn (derived from conceptual origin)
    
    Referential:

    about (primary topic)

    mentions (secondary reference)

    citation (academic reference)
    
    Attributional:

    author (creator)

    publisher (official publisher)

    creator (originator)

    discovererOf (WAA extension for methodological discovery)
    
    Transactional:

    offers (commercial availability)

    workExample (concrete instance of a framework)

    subjectOf (entity that discusses this)
    

2.3 Build-Time Dependency Resolution


The key technical innovation is build-time dependency resolution. When Eleventy generates a page:


    It reads the requiredEntities array from frontmatter

    It loads the central registry (waa-registry.js)

    It resolves each requested entity ID to its full definition

    It injects all resolved definitions into the page's graph

    It generates a single JSON-LD script tag with the complete graph
    
    This means:

    Pages declare what they need (semantic dependencies)

    The build system provides the definitions (dependency injection)

    The browser receives a complete graph at once (performance)

    Google sees dense, connected knowledge (SEO)
    
    ---
    
    ## 3. FILE STRUCTURE
    

3.1 Directory Organization


A WAA-compliant Eleventy project should include the following structure:



your-eleventy-site/

src/

_data/
   waa-registry.js              # Central entity registry

_includes/

components/

schema/
       core.njk                  # Base container (always included)
       authority.njk             # Organization, People, Method
       commerce.njk              # Products, Offers
       content.njk               # Articles, Quizzes, Collections

layouts/
   base.njk                      # Base layout that includes core.njk
   post.njk                      # Blog post layout
   product.njk                   # Product page layout

utils/
    schema-builder.js              # Utility functions (optional)
```

### 3.2 Core Files Explained

**waa-registry.js**
- **Purpose:** Central repository of all canonical entity definitions
- **Location:** src/_data/
- **Behavior:** Exported object with organization, method, waa, authors, website, products
- **Access:** Automatically available in all templates as waaRegistry

**core.njk**
- **Purpose:** Context Builder that assembles the final graph
- **Location:** src/_includes/components/schema/
- **Parameters:** pageContext, requiredEntities, registry
- **Output:** JSON-LD script tag with @graph array

**authority.njk, commerce.njk, content.njk**
- **Purpose:** Complete definitions of each Mini-Graph's entities
- **Usage:** Never included directly; entities are accessible via the registry

### 3.3 Optional Components

**schema-builder.js**
Utility functions for generating @id strings, validating graphs, merging entities. Can be used in .eleventy.js to add filters or shortcodes.

**examples/**
Directory containing example pages demonstrating different use cases. Not required for production but useful for learning.

---

## 4. THE REGISTRY PATTERN (waa-registry.js)

### 4.1 Purpose and Function

The registry pattern is the foundation of the WAA implementation for Eleventy. Instead of duplicating entity definitions across templates or multiple pages, all canonical definitions live in a central file that serves as the source of truth.

This registry:
- Is loaded once at build time
- Is available to all templates via the waaRegistry global
- Uses placeholder syntax for project-specific values
- Can be extended or overridden per project

### 4.2 Entity Definition

Each entity in the registry follows a strict structure:

```javascript
{
  "@id": "https://{{ site.domain }}/#entity-name",
  "@type": "SchemaOrgType",
  "property1": "value",
  "property2": { "@id": "reference-to-other-entity" },
  "property3": [ "array", "of", "values" ]
}
```

**CRITICAL RULES:**
- Each entity MUST have a unique @id
- @id values MUST be absolute URIs (including domain)
- References to other entities MUST use @id objects
- Arrays MUST be used for multiple values of the same property

### 4.3 Placeholder System

The registry uses Nunjucks-style placeholders ({{ variable }}) for values that change per project or environment:

```javascript
{
  "@id": "https://{{ site.domain }}/#org",
  "name": "{{ site.orgName }}",
  "legalName": "{{ site.legalName }}"
}
```

These placeholders are resolved when:
- The registry is loaded by Eleventy
- Site variables are defined in _data/site.js or _data/site.json
- Environment-specific overrides are applied

### 4.4 Multi-Project Extension

For agencies managing multiple client sites, the registry can be extended through composition:

```javascript
module.exports = (siteConfig) => {
  const baseRegistry = {
    organization: { ... },
    method: { ... },
    waa: { ... }
  };
  
  // Merge with client-specific overrides
  return {
    ...baseRegistry,
    ...siteConfig.overrides,
    products: siteConfig.products || {}
  };
};
```

This allows maintaining a core WAA identity while customizing per project.

---

## 5. THE CONTEXT BUILDER (core.njk)

### 5.1 How It Works

core.njk is the brain of the implementation. It:

1. Initializes an empty graph array
2. Always adds the WebSite entity (from registry)
3. Builds the current WebPage entity from pageContext
4. Iterates through the requiredEntities array
5. For each entity ID, looks up the definition in the registry
6. Adds each found definition to the graph
7. Outputs the complete array as JSON-LD

### 5.2 Parameters

The template accepts three parameters:

**pageContext (required)**
Object containing:
- url: Current page URL
- title: Page title
- description: Page description
- type: Schema.org type (WebPage, Article, Quiz, etc.)
- about: Primary topic (string or array)
- mentions: Secondary references (array)
- isBasedOn: Conceptual origins (array)

**requiredEntities (optional)**
Array of entity ID suffixes to resolve:
- "#org" -> organization
- "#method" -> method
- "#waa" -> waa
- "#product-olive" -> products.olive
- "#author-1" -> authors[0]

**registry (injected automatically)**
The waaRegistry object from _data/

### 5.3 Resolution Logic

The resolver uses a simple mapping system:

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

This mapping can be extended for additional entity types.

### 5.4 Output Generation

The final output is a JSON-LD script tag containing an @graph array:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { ... WebSite ... },
    { ... WebPage ... },
    { ... Organization ... },
    { ... Product ... },
    { ... all resolved entities ... }
  ]
}
```

This format is Google's preferred format for multiple entities on the same page.

---

## 6. MINI-GRAPH COMPONENTS

### 6.1 authority.njk - Organization, People, Method

This component contains the complete definitions of authority entities. It is never included directly; instead, its entities are accessible through the registry.

**Key Entities:**

**Organization**
- Legal name, VAT number, address
- knowsAbout relationship with Method and WAA
- publisher relationship with Website
- provider relationship with services

**Person (Authors/Team)**
- Name, job title, sameAs (social profiles)
- worksFor relationship with Organization
- creator/discovererOf relationship with Method/WAA

**Method**
- CreativeWork representing the methodology
- isBasedOn relationship with Manifesto (if applicable)
- teaches relationship with educational resources
- hasPart relationship with WAA

**WAA**
- SoftwareSourceCode + Product (double typing)
- isBasedOn relationship with Method
- softwareRequirements for implementation
- workExample relationship with concrete sites
- offers relationship with commercial availability

### 6.2 commerce.njk - Products, Offers, Services

Transactional entities for e-commerce and service pages:

**Product**
- name, description, image
- sku, mpn, brand
- isBasedOn relationship with Method/WAA
- workExample relationship with WAA implementations

**Offer**
- price, priceCurrency, availability
- seller relationship with Organization
- validFrom, validThrough

**Service**
- serviceType, provider
- areaServed, availableChannel

### 6.3 content.njk - Articles, Quizzes, Collections

Educational and editorial entities:

**Article**
- headline, description, articleBody
- about topic, mentions references
- citation relationship with sources/WAA
- isRelatedTo relationship with Quiz

**Quiz**
- name, description
- educationalUse relationship with Method
- isBasedOn relationship with Resources/WAA
- teaches competencies
- leads to conversion pages

**CollectionPage**
- name, description
- isBasedOn relationship with Data/Method/WAA
- about compared entities
- mentions technologies/products

---

## 7. PAGE-LEVEL INTEGRATION

### 7.1 Frontmatter Configuration

Every page that wants to participate in the WAA semantic graph must configure its frontmatter:

```yaml
---
title: "Page Title"
description: "Page description"
type: "Article"  # Schema.org type
layout: "post.njk"

# WAA-specific frontmatter
about: "Primary topic, SEO focus"
mentions: 
  - "Secondary concept 1"
  - "Secondary concept 2"
isBasedOn:
  - { "@id": "https://domain.com/#method" }
  - { "@id": "https://domain.com/#waa" }
requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#product-olive"
---
```

### 7.2 Required Entities Declaration

The requiredEntities array tells the Context Builder which additional entities to include. Think of it as import statements for semantic dependencies.

**Rules:**
- Use only the suffix (the domain is prepended automatically)
- Order doesn't matter
- Duplicates are handled gracefully
- Missing entities are silently ignored (no build error)

**Standard patterns:**
- Blog posts: ["#org", "#method", "#waa"]
- Product pages: ["#org", "#waa", "#product-{id}"]
- About Us pages: ["#org", "#method", "#waa", "#author-1", "#author-2"]
- Quiz pages: ["#org", "#method", "#waa", "#product-{id}"]

### 7.3 Semantic Properties (about, mentions, isBasedOn)

These properties directly populate the WebPage entity:

**about**
- Primary topic of the page
- Used for SEO and content classification
- Can be a string or array of strings

**mentions**
- Secondary concepts or entities
- Creates weaker semantic connections
- Always an array

**isBasedOn**
- Conceptual origins (Method, WAA)
- Creates derivation relationships
- Array of @id objects

---

## 8. PRACTICAL EXAMPLES

### 8.1 Blog Article Page

File: src/blog/waa-eleventy-guide.md

```yaml
---
title: "Complete Guide to WAA in Eleventy"
description: "Learn how to implement spherical web architecture"
date: 2026-03-08
type: Article
layout: post.njk

about: "WAA implementation, Eleventy, spherical web"
mentions:
  - "semantic SEO"
  - "knowledge graph"
  - "mini-graphs"

isBasedOn:
  - { "@id": "https://example.com/#method" }
  - { "@id": "https://example.com/#waa" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#author-1"
---

This guide explains how to implement WAA in Eleventy...
```

### 8.2 Product Page

File: src/products/extra-virgin-olive-oil.md

```yaml
---
title: "Extra Virgin Olive Oil - 5L Can"
description: "Cold-pressed organic olive oil"
type: Product
layout: product.njk

sku: "OIL-5L-ORG"
price: 49.90
priceCurrency: "EUR"
availability: "https://schema.org/InStock"

about: "extra virgin olive oil, organic olive oil"
mentions:
  - "sustainable production"
  - "controlled supply chain"

isBasedOn:
  - { "@id": "https://example.com/#method" }
  - { "@id": "https://example.com/#waa" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#product-oil-5l"
---

Detailed product description...
```

### 8.3 Quiz Page

File: src/quizzes/oil-scams.md

```yaml
---
title: "Test Your Knowledge on Olive Oil Scams"
description: "Interactive quiz on food fraud"
type: Quiz
layout: quiz.njk

about: "olive oil scams, food fraud"
mentions:
  - "oil quality"
  - "certifications"

educationalUse: "interactive learning"
teaches: "identifying food fraud"

isBasedOn:
  - { "@id": "https://example.com/#method" }
  - { "@id": "https://example.com/#waa" }
  - { "@id": "https://example.com/#resource-scams" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
  - "#product-oil-5l"
---

Quiz content...
```

### 8.4 Thematic Hub Page

File: src/hub/food-fraud.md

```yaml
---
title: "Food Fraud Hub"
description: "Complete resource on fraud and counterfeiting"
type: CollectionPage
layout: hub.njk

about: "food fraud, counterfeiting"
mentions:
  - "olive oil"
  - "fish"
  - "DOP cheeses"

hasPart:
  - { "@id": "https://example.com/articles/oil-fraud" }
  - { "@id": "https://example.com/quizzes/oil-scams" }
  - { "@id": "https://example.com/comparisons/oils" }

requiredEntities:
  - "#org"
  - "#method"
  - "#waa"
---

Hub content...
```

---

## 9. MULTI-PROJECT ARCHITECTURE

### 9.1 Shared Registry Across Sites

For agencies or consortia managing multiple WAA-compliant sites, the registry can be shared via:

- NPM package (@waa-consortium/registry)
- Git submodule
- Private npm registry
- JSON endpoint API

Each site imports the base registry and extends it:

```javascript
const baseRegistry = require('@waa-consortium/registry');

module.exports = {
  ...baseRegistry,
  organization: {
    ...baseRegistry.organization,
    name: "Specific Client Name",
    legalName: "Client Srl"
  },
  products: require('./client-products.json')
};
```

### 9.2 Domain-Specific Override

@id fields must be unique per domain. Use the site.domain variable:

```javascript
organization: {
  "@id": "https://{{ site.domain }}/#org",
  "name": "{{ site.orgName }}"
}
```

This ensures that entities from different sites do not collide in Google's index.

### 9.3 Maintaining Unique @ids

**Rules for @id management:**
- Always include the full domain
- Never use relative paths
- Use fragments (#) for on-page entities
- Keep fragments consistent across sites
- Document your fragment naming convention

**Good:** https://client1.com/#org
**Bad:** /#org
**Bad:** client1.com/#organization

---

## 10. VALIDATION AND TESTING

### 10.1 Testing with Google Rich Results

After implementation, validate your pages with:

1. **Google Rich Results Test**
   - Enter your URL
   - Check errors and warnings
   - Verify that all expected entities appear

2. **Schema.org Validator**
   - Paste your JSON-LD output
   - Validate against schema.org vocabulary

3. **Google Search Console**
   - Monitor enhancement reports
   - Track rich result impressions

### 10.2 Common Errors

**Missing @id references**
Entities referenced but not defined in requiredEntities
**Solution:** Ensure all @ids in page properties appear in requiredEntities

**Duplicate @id values**
Same @id used for different entities
**Solution:** Use unique fragments per entity type

**Invalid nesting**
Incorrectly placing Offer inside Product
**Solution:** Follow schema.org type hierarchy

**Missing domain**
Using relative @id paths
**Solution:** Always use absolute URLs with domain

### 10.3 Graph Resolution Debugging

If an entity doesn't appear in the output:

1. Check that requiredEntities includes the correct suffix
2. Verify the registry has that key defined
3. Confirm the mapping logic in core.njk matches the key
4. Check for typos in entityId comparison
5. Ensure the registry is correctly loaded in _data/

Use Eleventy debug mode:
```bash
npx @11ty/eleventy --dry-run --verbose
```

---

## 11. APPENDIX

### 11.1 Complete Code

Complete code for all files is available in the implementations/eleventy/ directory of the WAA standards repository.

### 11.2 Schema.org References

Core types used in WAA implementation:

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

### 11.3 WAA-Specific Predicates

Extensions and conventions:

**discovererOf**
Indicates that a Person discovered or originated a Method/WAA

**educationalUse (with WAA values)**
- "learning"
- "assessment"
- "simulation"

**leads**
Indicates that a Quiz or Content leads to a conversion page (Well)

**workExample**
Concrete website built with WAA framework

**softwareRequirements**
Technical requirements for WAA implementation

---

## END OF DOCUMENT

