**You:**

What would hyperlexia look like as a workflow in N8N

* * *

**ChatGPT:**

Love this question. If “hyperlexia” is your superpower for hoovering text and turning it into structure + questions, then in n8n it looks like a set of small, opinionated pipelines that:

1. capture text from everywhere
    
2. normalize and chunk it
    
3. extract meaning + links + dates
    
4. generate questions & spaced-review prompts
    
5. file everything into a durable home (Notion/DB)
    
6. surface daily “study packs” and on-demand retrieval
    

Here’s a clean, practical blueprint you can build today.

# 1) Capture & Normalize (Workflow A)

**Goal:** vacuum text from multiple sources, dedupe, and convert to tidy records.

**Trigger nodes (pick what you use):**

* Gmail (watch emails) / IMAP → body + subject + from
    
* Webhook (drop text, URLs, or PDFs from Shortcuts/Share Sheet)
    
* Telegram/Discord/Slack trigger
    
* RSS Feed Read (articles, blogs)
    
* Google Drive/Dropbox watch (new .txt/.pdf/.docx)
    
* Speech-to-text (e.g., Whisper API via HTTP Request) for voice memos
    

**Processing nodes:**

* **If** (MIME/type) → route PDFs to OCR path
    
* **PDF/OCR path:** HTTP Request → your OCR/LLM to pull clean text
    
* **Function** (“NormalizeText”)
    
    * Strip HTML, fix whitespace, collapse quotes, keep original link
        
    * Build a canonical object:  
        `{source, author, url, title, datetime, raw_text, tokens_est}`
        
* **Function** (“Chunker”)
    
    * Split into ~800–1,200 token chunks on paragraph boundaries
        
    * Attach `chunk_id`, `parent_id`, `sequence`
        

**Deduping:**

* **Hash** each chunk (Function: SHA256 of title+first 500 chars)
    
* **DB Lookup** (Postgres/MySQL/SQLite/Notion query): skip if hash exists
    

**Enrichment (LLM):**

* **OpenAI (Chat Completions)**
    
    * System: “Extract key points, entities, dates, defs, quotes; generate 5 cloze deletions; 5 Q&A; 1-paragraph summary; capture unknown terms.”
        
    * Output JSON schema, e.g.:
        
        ```json
        {
          "summary": "...",
          "keywords": ["..."],
          "entities": [{"name":"...", "type":"person|org|concept"}],
          "dates": [{"label":"...", "iso":"..."}],
          "definitions": [{"term":"...", "definition":"..."}],
          "questions": [{"q":"...", "a":"...", "type":"recall|concept|application"}],
          "cloze": ["...", "..."],
          "followups": ["terms to learn…"]
        }
        ```
        

**Storage:**

* **Vector Embedding** (OpenAI Embeddings) → upsert to your vector DB (Weaviate/Pinecone/Supabase pgvector)
    
    * Metadata: `source, url, title, parent_id, chunk_id, hash`
        
* **Primary record**: Notion “Hyperlexic Library” (or a SQL table)
    
    * Properties:
        
        * Title, Source, URL, Author, CapturedAt
            
        * Summary (rich text)
            
        * Keywords (multi-select)
            
        * Entities (multi-select)
            
        * ReviewSchedule (date)
            
        * ParentID/ChunkID/Hash
            
* **Questions table** (Notion/DB)
    
    * Fields: ParentID, ChunkID, Q, A, Type, Ease, Interval, DueDate, LastReviewed
        
* **Error path:** if LLM fails, store raw chunk; mark `needs_enrichment = true`
    

# 2) Spaced Review & Daily Pack (Workflow B)

**Goal:** every morning, auto-assemble a review set sized to your energy.

**Trigger:**

* **Cron** daily at, say, 9:00 AM (America/Los_Angeles)
    

**Steps:**

* **DB Query**: select questions where `DueDate <= today` order by priority (new → due → leeches)
    
* **If** (Energy mode)
    
    * Pull your “energy” from a tiny Notion property or an n8n env var: `low=10`, `med=25`, `high=50`
        
* **Function** (“AssemblePack”):
    
    * Mix of types: 50% recall, 30% concept, 20% application
        
    * Interleave sources (avoid monotony)
        
* **Formatter/Markdown**: turn into a clean checklist + links back to source chunks
    
* **Send**:
    
    * Telegram/Email with the pack, or
        
    * Create a Notion “Today’s Review” page with toggles (Q on top, Answer hidden)
        
* **Review endpoint**: Include two buttons/links per Q (Correct / Wrong) pointing to a small **Webhook** that updates SRS fields (`Ease`, `Interval`, `DueDate` using SM-2-ish math).
    
    * n8n path: Webhook → Function (“SM2Update”) → Questions table row update
        

# 3) Retrieve on Demand (Workflow C)

**Goal:** ask questions in plain English and get cited answers.

**Trigger:**

* Webhook (from a shortcut, Telegram command, or a tiny web UI)
    

**Steps:**

* **Vector DB search** with the query → top-k chunks + metadata
    
* **OpenAI** with RAG prompt:
    
    * “Answer using only the provided chunks. Cite titles/URLs and chunk_id.”
        
* **Return**: concise answer + inline citations + “open in Notion” links
    
* **Log** the query + answer into a “Queries” table so the system learns what you ask most.
    

# 4) “Unknowns to Learning Plan” (Workflow D)

**Goal:** turn all the “I should learn this” into a weekly micro-curriculum.

**Trigger:**

* Weekly Cron (e.g., Sundays)
    

**Steps:**

* **Query**: collect `followups`/unknown terms from last week’s enrichments
    
* **OpenAI**: cluster into ~3 modules; define outcomes + 3–5 resources/module
    
* **Create** a Notion “Learning Plan (Week of YYYY-MM-DD)” with sections:
    
    * Modules, outcomes, links, and pre-generated questions
        
* **Optional**: add calendar blocks (Google Calendar node) for deep work sprints
    

# 5) Optional “Reading Copilot” (Workflow E)

**Goal:** paste a URL and instantly get a structured “hyperlexic packet.”

**Trigger:**

* Webhook with `{url}`
    

**Steps:**

* **HTTP Request** → fetch page (or Mercury Parser)
    
* Feed text into **Workflow A**’s Normalize → Chunk → Enrich → Store
    
* Return a Notion link + TL;DR + 5 questions in your chat app
    

* * *

## Minimal node list you can create right now

* Triggers: **Cron**, **Webhook**, **Gmail/IMAP**, **RSS Read**, **Google Drive**
    
* Processing: **Function** (NormalizeText), **Function** (Chunker), **If**, **Merge**
    
* LLM: **OpenAI (Chat Completions)**, **OpenAI (Embeddings)** (or HTTP Request to your LLM)
    
* Storage: **Notion** (Library, Questions), **Postgres/MySQL/SQLite** (optional), **Pinecone/Weaviate/Supabase** (vector)
    
* Delivery: **Telegram**, **Email**, **Notion page create/update**
    
* Utilities: **Set**, **Wait**, **Error Trigger** (for retry loops)
    

* * *

## Suggested field schemas

**Notion – Library**

* Title (Title)
    
* URL (URL)
    
* Source (Select: Email, Web, PDF, Note, Voice)
    
* Author (Text)
    
* CapturedAt (Date)
    
* Summary (Rich text)
    
* Keywords (Multi-select)
    
* Entities (Multi-select)
    
* ParentID (Text)
    
* ChunkID (Text)
    
* Hash (Text, unique)
    
* ReviewSchedule (Date)
    
* VectorId (Text)
    

**Notion – Questions**

* Q (Title)
    
* A (Rich text)
    
* Type (Select: recall | concept | application | cloze)
    
* ParentID (Text)
    
* ChunkID (Text)
    
* SourceTitle (Text)
    
* SourceURL (URL)
    
* Ease (Number, default 2.5)
    
* Interval (Number, days)
    
* Repetitions (Number)
    
* DueDate (Date)
    
* LastReviewed (Date)
    

> If you’d rather use Anki, replace the Questions table updates with **HTTP Request → AnkiConnect** to add/update cards and pull due counts.

* * *

## Prompts that work well

**Enrichment system prompt (terse & structured)**

> You are an expert text distiller for a hyperlexic user. Extract: 1) 7–12 bullet key points, 2) named entities with types, 3) normalized dates, 4) definitions for all bold terms, 5) 5 cloze deletions, 6) 5 Q&A questions mixed by difficulty and type, 7) a 120-word TL;DR, 8) up to 10 follow-up terms to learn. Output valid JSON matching the provided schema. Do not include commentary.

**RAG answer prompt**

> Using only the supplied chunks, answer succinctly. Quote short phrases (<20 words) where helpful and add inline citations as [Title §chunk_id]. If you don’t know, say so.

* * *

## Why this maps to “hyperlexia”

* **Aggressive capture**: everything textual becomes fuel.
    
* **Fast structure**: chunking + JSON enrichment turns raw text into usable units.
    
* **Questionification**: every chunk yields recall hooks and cloze cards.
    
* **Spaced repetition**: daily packs sized to your current bandwidth.
    
* **Retrieval**: ask anything, get cited answers instantly.
    
* **Metacognitive loop**: unknowns are auto-batched into next week’s plan.
    

* * *

If you want, I can generate a starter set of n8n workflows (export files) that implement:

* A) Webhook → Normalize/Chunk → LLM Enrich → Notion + Vector
    
* B) Cron → Due Questions → Pack → Telegram/Notion (+ SM-2 updates)
    
* C) Webhook → Vector Search → RAG Answer
    

Say the word and I’ll drop importable JSON you can load directly into n8n.

* * *

**You:**

Yes

* * *

**ChatGPT:**

Awesome — I built you a plug-and-play starter pack with three importable n8n workflows plus a README.

Download the n8n Hyperlexic Starter (ZIP)

### What’s inside

* **A – Capture → Normalize/Chunk → Enrich → Store**
    
    * Webhook: `/webhook/hyperlexic_ingest`
        
    * Normalizes pasted text/URLs, chunks ~1200 chars, enriches via OpenAI (summary, entities, dates, questions, cloze, follow-ups), and saves to two Notion DBs (Library + Questions).
        
    * Replace: `YOUR_NOTION_LIBRARY_DATABASE_ID`, `YOUR_NOTION_QUESTIONS_DATABASE_ID`. Add OpenAI & Notion credentials in n8n.
        
* **B – Daily Spaced Review Pack**
    
    * **Cron** 09:00 (America/Los_Angeles). Pulls due questions from your Notion “Questions” DB and emails a markdown review pack.
        
    * Set env var `ENERGY` to `low|med|high` (10/25/50 items). Replace SMTP creds.
        
* **C – Ask → Retrieve → Answer (RAG)**
    
    * Webhook: `/webhook/hyperlexic_query` expecting `{ "query": "..." }`.
        
    * Calls your vector DB search endpoint (`https://YOUR_VECTOR_DB/search`) and feeds results to OpenAI for a concise, cited answer. Swap that URL for your Pinecone/Supabase/Weaviate endpoint.
        

### Quick setup checklist

1. Import the three JSON workflows in n8n (top-right menu → Import from File).
    
2. Add credentials:
    
    * **OpenAI API** (for the two OpenAI nodes).
        
    * **Notion API** (and replace both Notion Database IDs).
        
    * **SMTP** (for the “Email: Send Pack” node in Workflow B).
        
3. Optional: implement the **/search** endpoint for your vector DB (or point to an existing one). Return a `results` array of top chunks + metadata.
    
4. Turn on the webhooks (A & C) and the cron (B).
    
5. Hit the A-webhook with a URL or pasted text to watch it flow end-to-end.
    

Want me to tailor these to your exact Notion schemas or switch B to Telegram/Discord delivery instead of email?