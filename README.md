# RAG-Ready Markdown Converter & Chunker

<p align="center">
  <a href="https://apify.com/foxpink/apify-rag-markdown-chunker">
    <img src="https://img.shields.io/badge/Run_on_Apify_Cloud-0.01_per_1k_results-FF7754?style=for-the-badge&logo=apify" alt="Run on Apify Cloud">
  </a>
</p>

<p align="center">
  <a href="https://apify.com/foxpink/apify-rag-markdown-chunker">
    <img src="https://img.shields.io/badge/Apify-Store-FF7754?style=flat-square" alt="Apify Store">
  </a>
  <img src="https://img.shields.io/badge/license-MIT-green?style=flat-square" alt="MIT">
</p>

Convert raw HTML/text into **clean Markdown**, split into **token-aware chunks**, auto-embed, and upsert to Pinecone — all in one API call at **$0.01/1k results**.

---

## Standard vs Enterprise

| Feature | Standard | Enterprise |
|---------|----------|------------|
| HTML to Clean Markdown | Yes | Yes |
| Character-based chunking | Yes | -- |
| Token-aware chunking (cl100k_base) | -- | Yes |
| OpenAI Embeddings | -- | Yes |
| Pinecone auto-upsert | -- | Yes |
| Bulk processing | Yes | Yes |
| Zero DOM, no browser | Yes | Yes |
| **Price** | **$0.01/1k** | **$0.01/1k** |

Enterprise mode activates automatically when you provide `openaiApiKey` + `pineconeApiKey` + `pineconeIndex`.

## Quick Start

```bash
curl -X POST https://api.apify.com/v2/acts/foxpink~apify-rag-markdown-chunker/runs \
  -H "Content-Type: application/json" \
  -d '{
    "htmlContent": "<h1>Hello RAG</h1><p>Clean content here</p>",
    "chunkSize": 1000
  }' \
  "https://api.apify.com/v2/acts/foxpink~apify-rag-markdown-chunker/runs?token=YOUR_API_TOKEN"
```

## Output

| Field | Description |
|-------|-------------|
| recordId | ID of the processed record |
| status | ok or empty |
| rawMarkdown | Cleaned Markdown |
| chunks | Array of chunkIndex, content, characterCount, tokenCount |
| stats | Aggregated statistics |

## Pricing

**$0.01 per 1,000 results.** Enterprise mode costs the same — you only pay OpenAI and Pinecone directly.

---

<p align="center">
  Built by <a href="https://apify.com/foxpink">Nguyen Anh Duy</a> — FoxPink Studio
</p>
