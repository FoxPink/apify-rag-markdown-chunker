# RAG-Ready Markdown Converter & Chunker

<p align="center">
  <a href="https://apify.com/foxpink/apify-rag-markdown-chunker">
    <img src="https://img.shields.io/badge/Run_on_Apify_Cloud-0.01_per_1k_results-FF7754?style=for-the-badge&logo=apify" alt="Run on Apify Cloud - $0.01/1k results">
  </a>
</p>

<p align="center">
  <a href="https://apify.com/foxpink/apify-rag-markdown-chunker">
    <img src="https://img.shields.io/badge/Apify-Store-FF7754?style=flat-square" alt="Apify Store">
  </a>
  <img src="https://img.shields.io/badge/license-MIT-green?style=flat-square" alt="MIT">
</p>

**Convert raw HTML/text into clean Markdown, split into token-aware chunks, auto-embed, and upsert to Pinecone — all in one API call.**

---

## Why this Actor?

| Problem | Solution |
|---------|----------|
| Crawlers output raw HTML with nav, scripts, ads | Strips them → clean Markdown |
| Character-level chunking cuts words mid-sentence | Token-aware chunking (cl100k_base) |
| Need vectors in Pinecone but don't want to glue code | Built-in OpenAI Embedding + Pinecone upsert |
| Enterprise features locked behind premium plans | Same $0.01/1k price — Enterprise included free |

---

## Features

| Feature | Standard | Enterprise |
|---------|----------|------------|
| HTML → Clean Markdown | ✅ | ✅ |
| Smart chunking (character) | ✅ | — |
| Token-aware chunking (cl100k_base) | — | ✅ |
| Natural boundary detection | ✅ | ✅ |
| Configurable overlap | ✅ | ✅ |
| OpenAI Embeddings (text-embedding-3-small) | — | ✅ |
| Pinecone auto-upsert | — | ✅ |
| Bulk processing | ✅ | ✅ |
| Zero DOM / no browser | ✅ | ✅ |
| **Price** | **$0.01/1k** | **$0.01/1k** |

---

## Quick Start

```bash
curl -X POST https://api.apify.com/v2/acts/foxpink~apify-rag-markdown-chunker/runs \
  -H "Content-Type: application/json" \
  -d '{
    "htmlContent": "<h1>Hello RAG</h1><p>Clean content here</p>",
    "chunkSize": 1000,
    "chunkOverlap": 200
  }' \
  "https://api.apify.com/v2/acts/foxpink~apify-rag-markdown-chunker/runs?token=YOUR_API_TOKEN"
```

**Enterprise** — add 3 keys to auto-activate token chunking + Pinecone:

```json
{
  "htmlContent": "...",
  "openaiApiKey": "sk-...",
  "pineconeApiKey": "pc-...",
  "pineconeIndex": "my-index"
}
```

---

## Input & Output

Full schema at [Apify Store → Input](https://apify.com/foxpink/apify-rag-markdown-chunker#input-tab).

| Input | Type | Default |
|-------|------|---------|
| htmlContent | string | — |
| chunkSize | integer | 1000 |
| chunkOverlap | integer | 200 |
| mode | enum | "both" |
| inputRecords | array | [] |
| openaiApiKey | secret | — (Enterprise) |
| pineconeApiKey | secret | — (Enterprise) |
| pineconeIndex | string | — (Enterprise) |

Output: `{ recordId, status, rawMarkdown, chunks[], stats }` + summary entry.

---

## Use Cases

- ✅ **RAG Pipelines** — Scrape → Chunk → Embed → Query
- ✅ **LLM Fine-tuning** — Clean training data fast
- ✅ **AI Agents** — Feed clean Markdown context
- ✅ **Content Analysis** — Structured text from raw HTML

---

## Pricing

**$0.01 per 1,000 results.** Enterprise mode (token chunking + Pinecone) costs the same — you only pay OpenAI & Pinecone directly.

---

<p align="center">
  <a href="https://apify.com/foxpink/apify-rag-markdown-chunker">
    <img src="https://img.shields.io/badge/▶_Run_on_Apify_Cloud-FF7754?style=for-the-badge" alt="Run on Apify Cloud">
  </a>
</p>

<p align="center">
  <sub>Built by <a href="https://apify.com/foxpink">Nguyễn Anh Duy</a> — FoxPink Studio</sub>
</p>
