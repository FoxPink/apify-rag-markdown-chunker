# RAG-Ready Markdown Converter & Chunker

[![Apify Marketplace](https://img.shields.io/badge/Apify-Marketplace-FF7754)](https://apify.com/foxpink/apify-rag-markdown-chunker)
[![GitHub](https://img.shields.io/badge/GitHub-Repo-181717)](https://github.com/FoxPink/apify-rag-markdown-chunker)

> **Standard mode**: HTML/PDF/DOCX → clean Markdown → chunks.  
> **Enterprise mode** (no extra charge): token-aware chunking + OpenAI Embedding + Pinecone / Qdrant auto-upsert.  
> **Same $0.01/1k price.**

---

## Quick Comparison

| Feature | Standard | Enterprise |
|---------|----------|------------|
| HTML → Clean Markdown | ✅ | ✅ |
| URL Fetching (auto-download HTML) | ✅ | ✅ |
| **PDF/DOCX File Parsing** (new) | ✅ | ✅ |
| Character-based chunking | ✅ | — |
| Semantic chunking (heading-aware) | ✅ | ✅ |
| Token-aware chunking (cl100k_base) | — | ✅ |
| Quality scoring + content hashing | ✅ | ✅ |
| Code block detection | ✅ | ✅ |
| Natural boundary detection | ✅ | ✅ |
| Configurable overlap | ✅ | ✅ |
| Embeddings via text-embedding-3-small | — | ✅ |
| Pinecone auto-upsert | — | ✅ |
| Qdrant auto-upsert | — | ✅ |
| Bulk processing | ✅ | ✅ |
| JSONL export (LLM-ready) | ✅ | ✅ |
| Zero DOM / no browser | ✅ | ✅ |
| **Price** | $0.01/1k | $0.01/1k |

---

## Why this exists

Hundreds of Apify crawlers output raw HTML full of nav bars, footers, scripts, and ads. Feeding that into a Vector DB or LLM wastes tokens and pollutes embeddings. This Actor takes any **already-crawled** content and delivers production-ready chunks — with or without a Vector DB pipeline.

---

## Features

- **HTML → Clean Markdown** — strips scripts, styles, nav, footer, iframes, SVG, canvas, and comment garbage; converts headers, lists, tables, blockquotes, links, images, bold, italic, code into proper Markdown syntax.
- **PDF/DOCX Parsing** (new) — provide direct download URLs to PDF (.pdf) or Word (.docx) files. Parsed to text using pure Node.js binaries (`pdf-parse`, `mammoth`). No browser, no DOM.
- **Smart Chunking** — splits by natural boundaries (paragraph breaks, headers) with configurable overlap to preserve context; avoids cutting words mid-stream.
- **Quality Scoring** — every chunk scored 0-100 based on structural coherence, heading presence, list/table density, and code block richness.
- **Content Hashing** — SHA256 hash on every chunk for reliable dedup across records.
- **Code Block Detection** — identifies fenced code blocks with language tags; includes in chunk metadata.
- **Token-Aware Chunking** (Enterprise) — uses `js-tiktoken` (cl100k_base) to split by actual LLM tokens instead of characters. Compatible with GPT-4, GPT-3.5, text-embedding-3-small.
- **Pinecone + Qdrant Auto-Upsert** (Enterprise) — generates embeddings via OpenAI `text-embedding-3-small` and upserts vectors directly to your Pinecone index **or** Qdrant collection.
- **Bulk Processing** — accepts an array of HTML documents and processes each independently.
- **URL Fetching** — provide an array of URLs; the Actor automatically fetches and processes each one.
- **Semantic Chunking** — heading-aware chunking that respects document structure. Splits on `#` headings, keeps related content together, preserves heading context in chunk metadata.
- **JSONL Export** — download chunks as JSONL (one JSON object per line) for direct LLM fine-tuning.
- **Zero DOM dependency** — runs on any Node.js platform without a browser or headless client.
- **Deduplication** — detects and removes duplicate or near-duplicate chunks across records.

---

## How Enterprise Mode Works

```
HTML → Clean Markdown → Token-aware chunking (cl100k_base) → OpenAI Embedding → Pinecone or Qdrant upsert
```

Provide OpenAI API key + (Pinecone keys **or** Qdrant config) → the Actor auto-detects and runs the full pipeline. No configuration, no middleware, no extra services.

---

## Use Cases

| Who | Why |
|-----|-----|
| RAG Pipeline Builders | Convert scraped pages → chunks → embeddings → Vector DB |
| LLM Fine-tuning | Clean training data by removing structural HTML garbage |
| AI Agents | Feed clean Markdown context to tool-calling LLMs |
| Content Analysts | Extract structured text from raw website or document dumps |

---

## Input

### Standard Mode

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `htmlContent` | string | — | Raw HTML or text content to process |
| `urls` | array | `[]` | URLs to auto-fetch and process (alternative to htmlContent) |
| `fileUrls` | array (new) | `[]` | Direct download URLs to PDF/DOCX files (alternative to urls/htmlContent) |
| `chunkingStrategy` | string | `auto` | `auto`, `character`, or `semantic` |
| `chunkSize` | integer | `1000` | Target chunk length in characters or tokens |
| `chunkOverlap` | integer | `200` | Overlap between consecutive chunks |
| `mode` | string | `both` | Output mode: `both`, `markdown`, or `chunks` |
| `inputRecords` | array | `[]` | Bulk input `[{ id, html, chunkSize?, chunkOverlap? }]` |
| `deduplicate` | boolean | `false` | Remove duplicate chunks across records |
| `minQualityScore` | integer | `0` | Filter out low-quality chunks (0-100) |

### Enterprise Mode

Provide **OpenAI key + either Pinecone or Qdrant config** to activate the full pipeline.

#### Pinecone Pipeline

| Field | Type | Description |
|-------|------|-------------|
| `openaiApiKey` | string (secret) | Your OpenAI API key for token chunking + embeddings |
| `pineconeApiKey` | string (secret) | Your Pinecone API key |
| `pineconeIndex` | string | Your Pinecone index name (must be 1536-dimension) |

#### Qdrant Pipeline

| Field | Type | Description |
|-------|------|-------------|
| `openaiApiKey` | string (secret) | Same key — shared across all Enterprise features |
| `qdrantUrl` | string | Your Qdrant instance URL |
| `qdrantApiKey` | string (secret) | Your Qdrant API key |
| `qdrantCollection` | string | Your Qdrant collection name (1536-dimension vectors) |

---

## Output

Each processed record returns:

| Field | Type | Description |
|-------|------|-------------|
| `recordId` | string | ID of the processed record |
| `status` | string | `ok` or `empty` |
| `rawMarkdown` | string | Cleaned Markdown (if mode includes `markdown` or `both`) |
| `chunks` | array | Array of `{ chunkIndex, content, characterCount, qualityScore, contentHash, codeBlocks, headingPath?, chunkType? }` |
| `stats` | object | `{ rawChars, cleanedChars, totalChunks, chunkSize, chunkOverlap, chunkingMode, avgQualityScore }` |

A `summary` entry is appended at the end with aggregate statistics across all records.

---

## Pricing

**Pay Per Event** — $0.01 per 1,000 results.

One result = one processed record (not per chunk). Processing 5 records with 200 total chunks = 5 billable results. Enterprise mode costs **the same** — you only pay OpenAI and Pinecone directly for their API usage.

---

## Examples

### PDF File Parsing

**Input:**
```json
{
  "fileUrls": ["https://example.com/report.pdf"],
  "chunkSize": 500,
  "mode": "both"
}
```

### Standard HTML

**Input:**
```json
{
  "htmlContent": "<html><body><h1>Hello World</h1><p>This is <strong>important</strong> content.</p></body></html>",
  "chunkSize": 500,
  "chunkOverlap": 50
}
```

---

## Usage with AI Agents / MCP

```json
{
  "mcpServers": {
    "apify": {
      "command": "npx",
      "args": ["-y", "@apify/mcp-server"],
      "env": { "APIFY_TOKEN": "YOUR_API_TOKEN" }
    }
  }
}
```

---

## Why FoxPink?

| vs. Competitor | Their Price | Our Price | Advantage |
|----------------|-------------|-----------|-----------|
| Unstructured.io API | $10/1k pages | **$0.01/1k** | **1,000x cheaper** |
| LangChain/LlamaIndex | Open source (self-host) | **$0.01/1k** | **No infra management** |
| labrat-0/rag-content-chunker | Free (GitHub) | **$0.01/1k** | **Apify-native, no self-host** |

**Unique features they don't have:** `qualityScore` per chunk, `contentHash` (SHA256), `codeBlocks` detection, PDF/DOCX binary parsing, semantic + character dual-mode, enterprise fallback (no API key required).

---

## FoxPink Studio Ecosystem

Combine with other FoxPink actors for a complete data pipeline:

| Actor | Purpose | Price |
|-------|---------|-------|
| [Email Enricher+](https://apify.com/foxpink/email-enricher-plus) | Email verification & spam trap detection | $0.01/1k |
| [Shopify Hidden API Spy](https://apify.com/foxpink/shopify-hidden-api-spy) | Zero-DOM Shopify product intelligence | $0.01/1k |
| [Odoo Market Intel](https://apify.com/foxpink/odoo-apps-market-intelligence) | Odoo Apps Store scraper & analysis | $0.05/1k |

**Workflow example:** RAG chunk product descriptions → store in Pinecone → semantic search for customer queries.

---

## Compatibility

- 100% Node.js (18+)
- No browser, no headless, no DOM
- ESM (ECMAScript Modules)
