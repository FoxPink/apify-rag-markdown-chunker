# RAG-Ready Markdown Converter & Chunker

[![Apify Marketplace](https://img.shields.io/badge/Apify-Marketplace-FF7754)](https://apify.com/foxpink/apify-rag-markdown-chunker)
[![GitHub](https://img.shields.io/badge/GitHub-Repo-181717)](https://github.com/FoxPink/apify-rag-markdown-chunker)

> **Standard mode**: HTML → clean Markdown → character-based chunks.  
> **Enterprise mode** (no extra charge): token-aware chunking + OpenAI Embedding + Pinecone / Qdrant auto-upsert.  
> **Same $0.01/1k price.**

---

## Quick Comparison

| Feature | Standard | Enterprise |
|---------|----------|------------|
| HTML → Clean Markdown | ✅ | ✅ |
| URL Fetching (auto-download HTML) | ✅ | ✅ |
| Character-based chunking | ✅ | — |
| Semantic chunking (heading-aware) | ✅ | ✅ |
| Token-aware chunking (cl100k_base) | — | ✅ |
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
- **Smart Chunking** — splits by natural boundaries (paragraph breaks, headers) with configurable overlap to preserve context; avoids cutting words mid-stream.
- **Token-Aware Chunking** (Enterprise) — uses `js-tiktoken` (cl100k_base) to split by actual LLM tokens instead of characters. Compatible with GPT-4, GPT-3.5, text-embedding-3-small.
- **Pinecone + Qdrant Auto-Upsert** (Enterprise) — generates embeddings via OpenAI `text-embedding-3-small` and upserts vectors directly to your Pinecone index **or** Qdrant collection. No glue code needed. Auto-detects which vector DB to use from your input.
- **Bulk Processing** — accepts an array of HTML documents and processes each independently with per-record chunk settings.
- **URL Fetching** — provide an array of URLs; the Actor automatically fetches and processes each one through the full pipeline.
- **Semantic Chunking** (new in v1.4) — heading-aware chunking that respects document structure. Splits on `#` headings, keeps related content together, preserves heading context in chunk metadata. Auto-selected for content >5000 characters.
- **JSONL Export** — download chunks as JSONL (one JSON object per line) for direct LLM fine-tuning, embedding batch jobs, or LangChain/LlamaIndex ingestion.
- **Zero DOM dependency** — pure string processing; runs on any Node.js platform without a browser or headless client.
- **MCP / AI Agent Ready** — callable via API; JSON output integrates directly with LangChain, LlamaIndex, Haystack, or custom RAG pipelines.
- **Backward Compatible** — Enterprise mode activates only when you provide API keys. Standard mode works exactly as before.
- **Quality Scoring** — scores chunk quality based on structural coherence, token efficiency, and metadata completeness.
- **Deduplication** — detects and removes duplicate or near-duplicate chunks across records, reducing embedding storage costs.

---

## How Enterprise Mode Works

```
HTML → Clean Markdown → Token-aware chunking (cl100k_base) → OpenAI Embedding → Pinecone or Qdrant upsert
```

Provide OpenAI API key + (Pinecone keys **or** Qdrant config) → the Actor auto-detects and runs the full pipeline. No configuration, no middleware, no extra services. You can even push to **both** Pinecone and Qdrant simultaneously by providing all keys.

---

## Use Cases

| Who | Why |
|-----|-----|
| RAG Pipeline Builders | Convert scraped pages → chunks → embeddings → Vector DB |
| LLM Fine-tuning | Clean training data by removing structural HTML garbage |
| AI Agents | Feed clean Markdown context to tool-calling LLMs |
| Content Analysts | Extract structured text from raw website dumps |

---

## Input

### Standard Mode

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `htmlContent` | string | — | Raw HTML or text content to process |
| `urls` | array | `[]` | URLs to auto-fetch and process (alternative to htmlContent) |
| `chunkingStrategy` | string | `auto` | `auto`, `character`, or `semantic`. Semantic respects heading boundaries |
| `chunkSize` | integer | `1000` | Target chunk length in characters or tokens |
| `chunkOverlap` | integer | `200` | Overlap between consecutive chunks (character mode only) |
| `mode` | string | `both` | Output mode: `both`, `markdown`, or `chunks` |
| `inputRecords` | array | `[]` | Bulk input `[{ id, html, chunkSize?, chunkOverlap? }]` |

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
| `qdrantUrl` | string | Your Qdrant instance URL (e.g. `https://xxx.us-east-1-0.aws.cloud.qdrant.io`) |
| `qdrantApiKey` | string (secret) | Your Qdrant API key |
| `qdrantCollection` | string | Your Qdrant collection name (1536-dimension vectors) |

When Enterprise fields are detected, the Actor automatically:

1. Switches from character-based to **token-aware chunking** (cl100k_base)
2. Generates **1536-dimension embeddings** via `text-embedding-3-small`
3. **Upserts vectors** to Pinecone and/or Qdrant with metadata (chunkIndex, tokenCount, source text)

The Actor auto-detects which vector DB to use:
- Pinecone → requires `openaiApiKey + pineconeApiKey + pineconeIndex`
- Qdrant → requires `openaiApiKey + qdrantUrl + qdrantApiKey + qdrantCollection`
- Both → provide all keys; runs both pipelines simultaneously

#### Where to get the keys

| Key | How to get |
|-----|------------|
| `openaiApiKey` | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) — create a new secret key |
| `pineconeApiKey` | [app.pinecone.io](https://app.pinecone.io) → API Keys → Copy |
| `pineconeIndex` | Create a **serverless index** with dimension **1536** (matching `text-embedding-3-small`) |
| `qdrantUrl` | [cloud.qdrant.io](https://cloud.qdrant.io) → Clusters → REST API Endpoint |
| `qdrantApiKey` | [cloud.qdrant.io](https://cloud.qdrant.io) → Clusters → API Key |
| `qdrantCollection` | Create a collection with dimension **1536** and `Cosine` distance |

---

## Output

Each processed record returns:

| Field | Type | Description |
|-------|------|-------------|
| `recordId` | string | ID of the processed record |
| `status` | string | `ok` or `empty` |
| `rawMarkdown` | string | Cleaned Markdown (if mode includes `markdown` or `both`) |
| `chunks` | array | Array of `{ chunkIndex, content, characterCount, tokenCount?, headingPath?, chunkType? }` |
| `stats` | object | `{ rawChars, cleanedChars, totalChunks, chunkSize, chunkOverlap, chunkingMode }` |

A `summary` entry is appended at the end with aggregate statistics across all records.

---

## Pricing

**Pay Per Event** — $0.01 per 1,000 results.

One result = one processed record (not per chunk). Processing 5 records with 200 total chunks = 5 billable results. Enterprise mode costs **the same** — you only pay OpenAI and Pinecone directly for their API usage.

---

## Examples

### Standard Mode

**Input:**
```json
{
  "htmlContent": "<html><body><h1>Hello World</h1><p>This is <strong>important</strong> content.</p></body></html>",
  "chunkSize": 500,
  "chunkOverlap": 50
}
```

**Output:**
```json
{
  "recordId": "default",
  "status": "ok",
  "rawMarkdown": "# Hello World\n\nThis is **important** content.",
  "chunks": [
    { "chunkIndex": 0, "content": "# Hello World\n\nThis is **important** content.", "characterCount": 49 }
  ],
  "stats": { "rawChars": 107, "cleanedChars": 49, "totalChunks": 1, "chunkSize": 500, "chunkOverlap": 50, "chunkingMode": "character" }
}
```

### Enterprise Mode (Token Chunking + Pinecone)

**Input:**
```json
{
  "htmlContent": "<h1>Enterprise RAG Pipeline</h1><p>Your HTML content here...</p>",
  "chunkSize": 500,
  "chunkOverlap": 100,
  "openaiApiKey": "sk-...",
  "pineconeApiKey": "pc-...",
  "pineconeIndex": "my-rag-index"
}
```

**Output includes:**
```json
{
  "stats": { "totalChunks": 12, "totalTokens": 482, "chunkingMode": "token" },
  "chunks": [
    { "chunkIndex": 0, "content": "...", "tokenCount": 42, "characterCount": 185 }
  ]
}
```

Vectors are upserted to Pinecone automatically.

### Enterprise Mode (Token Chunking + Qdrant)

**Input:**
```json
{
  "htmlContent": "<h1>Enterprise RAG Pipeline</h1><p>Your HTML content here...</p>",
  "chunkSize": 500,
  "chunkOverlap": 100,
  "openaiApiKey": "sk-...",
  "qdrantUrl": "https://xxx.us-east-1-0.aws.cloud.qdrant.io",
  "qdrantApiKey": "qdrant-...",
  "qdrantCollection": "my-rag-collection"
}
```

Vectors are upserted to Qdrant automatically via REST API (no SDK required).

---

## Usage with AI Agents / MCP

```json
// POST https://api.apify.com/v2/acts/foxpink~apify-rag-markdown-chunker/runs?token=YOUR_API_TOKEN
{
  "htmlContent": "<h1>Your HTML here</h1>",
  "chunkSize": 1000,
  "chunkOverlap": 200
}
```

Note: This tool operates on **already-crawled** content. Use with any Apify Web Scraper, Puppeteer, or browser-based Actor by piping its output into this Actor's `inputRecords`.

---

## Compatibility

- 100% Node.js (18+)
- No browser, no headless, no DOM
- ESM (ECMAScript Modules)
