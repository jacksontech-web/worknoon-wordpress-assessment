# short-answers.md

## 1. Difference between Google Knowledge Graph and Google Knowledge Panel

- **Google Knowledge Graph** is the *underlying database/system* that stores structured information about entities (people, places, things, brands, etc.) and their relationships.
- **Google Knowledge Panel** is the *visual interface/output* you see on Google Search results, which displays information pulled from the Knowledge Graph.
- In simple terms:  
  - Knowledge Graph = backend data system  
  - Knowledge Panel = frontend display shown to users

---

## 2. How Google determines entity identity

Google identifies entities using a combination of signals, including:

- **Structured data (Schema.org markup)** on websites
- **Consistency of information** across the web (name, address, phone, etc.)
- **Wikipedia/Wikidata references** for verification and authority
- **Backlinks and citations** from trusted sources
- **Contextual language processing (NLP)** to understand mentions in content
- **Search behavior signals**, such as user engagement and queries
- **Entity disambiguation**, where Google distinguishes between similar names using context (location, category, relationships)

---

## 3. When to create Custom Post Types (CPTs) instead of pages

Use **Custom Post Types** when:

- Content has a **repeating structure** (e.g., listings, products, properties, events, team members)
- You need **custom fields and taxonomies** (e.g., price, location, category filters)
- You want **dynamic archives and filtering systems**
- Content is **not static or informational like normal pages**
- You need better **scalability and content organization**

Use **Pages instead** when:

- Content is static (About, Contact, Privacy Policy)
- No need for structured repetition or filtering
- Simple hierarchy is enough

---

## 4. Recommended plugins for speed optimization and why

- **WP Rocket**
  - Easy-to-use caching plugin
  - Improves page load speed via caching, minification, lazy loading, and database optimization

- **LiteSpeed Cache**
  - Best performance when used with LiteSpeed servers
  - Offers server-level caching (faster than PHP-based caching)
  - Includes image optimization and CSS/JS optimization tools

- **Autoptimize**
  - Focuses on optimizing CSS, JavaScript, and HTML
  - Reduces render-blocking resources

- **Perfmatters**
  - Disables unnecessary WordPress features (e.g., emojis, scripts)
  - Reduces bloat for faster performance

- **W3 Total Cache**
  - Advanced caching system for developers
  - Supports page, object, and database caching

**Note:** Only use one main caching plugin at a time (e.g., WP Rocket OR LiteSpeed Cache) to avoid conflicts.
