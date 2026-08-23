# Knotly Research — RAG Techniques & Competitive Intelligence

## Proven High-Impact Techniques (with benchmarks)

| Stage | Technique | Impact | Source |
|-------|-----------|--------|--------|
| Preprocessing | Contextual Retrieval (Anthropic) | **67% fewer retrieval failures** | Anthropic blog |
| Embedding | Voyage-3 | 7.5% better than OpenAI v3, 2x cheaper | Voyage AI |
| Chunking | Late Chunking (Jina) | Up to 6.5% nDCG improvement | Jina AI |
| Retrieval | Hybrid Search (BM25 + Dense) | Catches exact matches semantic search misses | Weaviate |
| Reranking | ColBERT late interaction | 180x fewer FLOPs than cross-encoder | Stanford/Jina |
| Query | HyDE + Multi-Query | Zero-shot parity with fine-tuned | arXiv |
| Robustness | CRAG (self-correcting) | Web fallback for low confidence | arXiv |
| Reasoning | GraphRAG | Multi-hop synthesis across docs | Microsoft |
| Cost | Semantic Caching | **50%+ cost reduction, 15x latency** | Redis |
| Evaluation | RAGAS | Automated faithfulness scoring | RAGAS docs |

## Architecture for Chatbase Competitor

### What Chatbase Does (Baseline)
- Upload docs → chunk → embed → retrieve → generate
- Embed widget on website via script tag
- Simple semantic search only (no hybrid)
- Limited customization of RAG pipeline

### What We Should Do (Differentiation)
1. **Hybrid search by default** (BM25 + semantic) — catches exact terms Chatbase misses
2. **Contextual Retrieval** — prepend context to chunks before embedding (67% improvement)
3. **Reranking** (Cohere/ColBERT) — second-pass scoring
4. **Unanswered question tracking** — show users what their bot CAN'T answer
5. **Source citations** — show exactly which document answered the question
6. **Semantic caching** — 15x faster for repeated questions (huge cost savings)
7. **Confidence scoring** — escalate to human when confidence is low
8. **Analytics dashboard** — top questions, resolution rate, gaps
9. **Multi-model support** — not locked to OpenAI
10. **Full embed customization** — script tag, React, iframe, headless API

## RAG Pipeline (Recommended Stack)

```
User Query
    ↓
[Query Processing]
├── Query rewriting (LLM rewrites for better retrieval)
├── Intent detection (does it need retrieval?)
└── Conversational resolution (resolve "it", "that", etc.)
    ↓
[Retrieval]
├── Hybrid search (BM25 + semantic, α=0.75)
├── Metadata filtering (auto-detected from query)
└── Top-K results (k=20)
    ↓
[Post-Retrieval]
├── Reranking (Cohere/ColBERT, reduce to top-5)
├── Contextual compression (extract only relevant parts)
└── Confidence scoring (threshold: 0.7)
    ↓
[Generation]
├── Context stuffing with citations
├── Grounding instructions in system prompt
├── Confidence check (low → escalate)
└── Streaming response
    ↓
[Post-Generation]
├── Hallucination check (claims vs sources)
├── Cache response (semantic caching)
└── Log for analytics
```

## Embedding Strategy

| Model | Dims | Context | Cost | NDCG@10 | Use Case |
|-------|------|---------|------|---------|----------|
| voyage-3 | 1024 | 32K | $0.06/M | 76.72 | Default production |
| text-embedding-3-small | 1536 | 8K | $0.02/M | 67.08 | Budget option |
| Cohere embed-v3 | 1024 | 512 | $0.10/M | ~73 | Multi-lingual |
| Local: nomic-embed | 768 | 8K | Free | ~68 | Self-hosted option |

## Chunking Strategy

- **Default**: Recursive character splitting, 512 tokens, 50 token overlap
- **Better**: Smart chunking (element-aware, by section headers)
- **Best**: Contextual Retrieval (prepend LLM context) + semantic chunking
- **For tables**: Preserve as single chunks
- **For code**: Split by functions/classes

## Prototype Status
- `~/knotly-prototype/index.html` — Full dashboard UI prototype
  - Dashboard (stats, recent conversations)
  - Sources (document management, file types)
  - Playground (chat test + config panel)
  - Embed (code generation, theming, live preview)
  - Analytics (top questions, unanswered gaps)
