# KNOTLY — Complete Product Research & Architecture Document
## Chatbase Competitor: RAG Chatbot Platform

> Compiled from 16 parallel research streams covering RAG techniques, competitive intelligence,
> architecture patterns, benchmarks, and product design decisions.

---

## EXECUTIVE SUMMARY

**What we're building:** A RAG chatbot platform that directly competes with Chatbase. Users upload documents, train an AI chatbot, and embed it on their website. Our advantages: better accuracy (67% fewer retrieval failures), lower price (4x more messages), and features Chatbase lacks (hybrid search, confidence scoring, human escalation, cross-session memory).

**Key positioning:** "The trust-first AI chatbot builder. Every answer cited. Never hallucinate."

---

## 1. COMPETITIVE LANDSCAPE

### Chatbase (Our Primary Target)
- **Pricing:** $32-400/mo, 500-15,000 messages, $0.04/msg overage (40x markup over cost)
- **Revenue model:** Per-message credits with expensive overages
- **Weaknesses:** Only 4 file formats, 10-40MB storage, no hybrid search, basic accuracy, $99/mo to remove branding, API gated at $120/mo
- **Users:** 10,000+ businesses, 4.3/5 rating (GetApp)

### Top Competitors
| Product | Price | Differentiator |
|---|---|---|
| CustomGPT | $99/mo | Anti-hallucination, 1400 formats |
| Dante AI | $40/mo | Unlimited conversations |
| SiteGPT | $39/mo | Auto-sync, HIPAA |
| DocsBot | $49/mo | Deep Research, Code Interpreter |
| Intercom Fin | $0.99/resolution | Purpose-built CX model |
| AnythingLLM | Free (OSS) | Self-hosted, MIT license |

### Market Gaps We Fill
1. No product combines: visual control + hybrid search + confidence scoring + embeddable + affordable
2. "Visual RAG builder" and "low-code RAG" are unclaimed positions on GitHub
3. Agency/reseller market underserved (Chatbase charges $1,188/yr for white-label)
4. Self-hosting option missing from all paid competitors

---

## 2. PRICING STRATEGY

### Our Model: Hybrid (Generous Base + Transparent Overage)

| Tier | Price | Messages | vs Chatbase | Key Unlocks |
|---|---|---|---|---|
| Free | $0 | 100/mo | Chatbase: 50 | 1 bot, 5MB |
| Starter | $29/mo | 2,000/mo | **4x more** | Branding removed, API |
| Pro | $99/mo | 10,000/mo | **2.5x more** | Unlimited bots, analytics |
| Business | $299/mo | 50,000/mo | **3.3x more** | White-label, SSO, priority |
| Enterprise | Custom | Unlimited | — | SLA, dedicated, self-host |

### Unit Economics
- Cost per RAG query (GPT-4o-mini): **$0.0007**
- Our charge (effective): $0.005-0.015/msg
- **Gross margin: 90-95%**
- Chatbase charges $0.04/msg on $0.001 cost (97% margin) — we undercut 4x and still have 90%+ margin

---

## 3. TECHNICAL ARCHITECTURE

### RAG Pipeline (Proven Best Practices)

```
User Query
  → [Query Reformulation] (resolve anaphora, expand)
  → [Hybrid Search] (BM25 + semantic, α=0.75)
      → Pinecone namespace per customer
      → Top-20 candidates
  → [Reranking] (Cohere/mxbai, reduce to top-5)
  → [Retrieval Sufficiency Gate] (score threshold + LLM check)
      → If INSUFFICIENT: "I don't know" + offer escalation
  → [Generation] (Anthropic Citations API, grounding prompt)
  → [Faithfulness Check] (HHEM model, <600MB, local)
      → If score < 0.7: suppress, escalate
  → [Response + Citations] delivered to user
  → [Async] RAGAS evaluation + semantic cache + analytics
```

### Proven Benchmark Results
| Technique | Improvement | Source |
|---|---|---|
| Contextual chunk headers | 35% fewer retrieval failures | Anthropic |
| + Hybrid search (BM25) | 49% fewer failures | Anthropic |
| + Reranking | **67% fewer failures** | Anthropic |
| Optimal chunk size (200 tokens) | 91.3% recall (vs 85.4% at 800) | Chroma Research |
| Voyage-3 embeddings | +7.55% NDCG over OpenAI | Voyage AI |
| GraphRAG (global questions) | 70-80% win rate | Microsoft |
| Semantic caching | 15x latency, 50% cost reduction | Redis |

### Tech Stack Decisions

| Component | Choice | Why |
|---|---|---|
| Vector DB | Pinecone (serverless, namespaces) | Zero-ops, 1 namespace/customer, instant GDPR delete |
| Embedding | Voyage-3 ($0.06/M tokens) | Best price/performance (7.5% better than OpenAI) |
| Reranker | Cohere Rerank or mxbai-rerank-v2 | 67% improvement proven |
| LLM (default) | GPT-4o-mini | $0.0007/query, good quality |
| LLM (premium) | Claude Sonnet 4.6 + Citations API | Best grounding, native citations |
| Document Parser | Docling/MinerU (free) + Gemini Vision fallback | 20+ formats, $0 for 70% of docs |
| Widget Framework | Preact (5KB) in iframe | Smallest bundle, full isolation |
| Backend | FastAPI (Python) | Async, fast, good ecosystem |
| Frontend Dashboard | React + Tailwind | Prototyped at ~/knotly-prototype/ |
| Database | PostgreSQL (Supabase) | App data + pgvector backup |
| Queue | Redis / BullMQ | Doc processing, async tasks |
| Auth | Clerk or Supabase Auth | Multi-tenant, SSO on enterprise |

### Multi-Tenant Architecture

```
Each customer = 1 Pinecone namespace
  - Upload: embed → upsert to index/namespace={customer_id}
  - Query: embed → search index/namespace={customer_id}
  - Delete: delete namespace (instant, GDPR)
  - Scales to 100K+ customers without architecture changes
```

---

## 4. DOCUMENT PARSING (20+ formats vs Chatbase's 4)

### Tiered Pipeline

| Tier | Tool | Formats | Cost | When Used |
|---|---|---|---|---|
| Fast | PyMuPDF | Clean PDFs (text layer) | $0 | 60-70% of uploads |
| Standard | Docling/MinerU | PDF, DOCX, PPTX, XLSX, HTML, images, audio | $0 (local) | Complex layouts |
| Premium | Gemini Vision | Scanned, handwritten, charts | $0.01-0.03/page | Tier 2 failures |

### Format Coverage: Us vs Chatbase
- Chatbase: PDF, TXT, DOC, DOCX (4 formats)
- **Knotly: PDF, DOCX, XLSX, PPTX, CSV, JSON, HTML, images (OCR), audio, email, EPUB, LaTeX, web crawl (20+ formats)**

---

## 5. EMBED WIDGET ARCHITECTURE

### Design
```
<script src="https://cdn.knotly.ai/widget.js" data-bot-id="..." async></script>
```

| Component | Size | Role |
|---|---|---|
| Loader (vanilla JS) | 3-5 KB | Renders bubble, creates iframe on click |
| Chat app (Preact, in iframe) | <80 KB | Full chat UI with streaming |

### Performance Targets
- Time to bubble visible: <100ms
- Time to chat ready: <500ms after click
- Zero main thread blocking on host page
- Full CSS/JS isolation (iframe)
- Streaming via Fetch + ReadableStream (SSE format)
- Session persistence via localStorage

### vs Chatbase
- Our loader: 3-5 KB vs their 15-20 KB
- Deferred loading (facade pattern) vs their eager load
- npm React component option (they don't have this)
- WCAG 2.1 AA accessibility (they lack this)

---

## 6. ACCURACY & TRUST SYSTEM (Key Differentiator)

### Trust Architecture

```
Pre-Retrieval:
  - Hybrid search (catches what semantic misses)
  - Contextual chunk headers (35% improvement alone)
  - 200-token chunks (proven optimal)

Post-Retrieval:
  - Reranker quality gate (score threshold)
  - Retrieval sufficiency check (LLM: "SUFFICIENT/PARTIAL/INSUFFICIENT")

Generation:
  - Anthropic Citations API (every claim linked to source)
  - Grounding system prompt (emphatic, few-shot)

Post-Generation:
  - HHEM faithfulness check (local, <600MB, ~100ms GPU)
  - Suppress if faithfulness < 0.7

Always Available:
  - "I don't know" with human escalation
  - Source citations shown to user
  - Confidence indicator (green/yellow/red)
```

---

## 7. CONVERSATION MEMORY (Solves "Loses Context" Complaint)

### 5-Layer Stack

| Layer | Purpose | Solves |
|---|---|---|
| Recent buffer (10 msgs) | Immediate context | Basic continuity |
| **Query reformulation** | "the second one" → "Tesla Model Y" | **Anaphora** |
| Session summary | Compress long conversations | 100+ msg sessions |
| Cross-session facts | Remember returning users | "Loses context between visits" |
| Knowledge base RAG | The actual document search | Core Q&A |

### Key Technique: Query Reformulation
```
Before every search, rewrite the query:
"What about the second one?" → "What is the price of the Tesla Model Y?"
Cost: ~150 tokens/turn. Non-negotiable.
```

---

## 8. HUMAN ESCALATION

### Trigger Signals
- Explicit user request ("talk to a human")
- Retrieval confidence < 0.6
- Sentiment < -0.5 (frustration detected)
- Topic = billing/legal/complaint
- 2+ failed attempts on same topic
- 8+ turns without resolution

### Handoff Flow
```
Trigger → Check agent availability
  → Online: WebSocket handoff + context transfer (transcript + AI analysis + confidence)
  → Offline: Create ticket (Zendesk/Intercom webhook) + ETA + callback option
```

---

## 9. ANALYTICS & OBSERVABILITY

### Dashboard Features (Differentiator)
- Top questions (what users ask most)
- Unanswered questions (knowledge gaps → "Add Source" CTA)
- Resolution rate (% answered without escalation)
- Average confidence score
- Response latency (p50, p95)
- User satisfaction (thumbs up/down)
- Escalation patterns (which topics, when)
- Source utilization (which docs get cited most/least)

---

## 10. GO-TO-MARKET POSITIONING

### One-Line Pitch
"Build AI chatbots your customers actually trust. Every answer cited. 4x more messages than Chatbase at half the price."

### Key Messages
1. **For frustrated Chatbase users:** "Tired of hallucinating chatbots and $0.04/message overages? Switch to Knotly."
2. **For agencies:** "White-label included at $299/mo. Chatbase charges $1,188/year just to remove their branding."
3. **For developers:** "Free API access on all plans. Open-source widget. Bring your own LLM keys."
4. **For compliance teams:** "Every answer cited with source. Confidence scoring. Automatic escalation when uncertain."

### Distribution Channels
- SEO: "Chatbase alternative", "best RAG chatbot builder", "AI customer support widget"
- ProductHunt launch
- Developer community (open-source widget, free API tier)
- Agency partnerships (white-label reseller program)
- Integration marketplace (Zendesk, Intercom, Shopify apps)

---

## 11. PROTOTYPE

- **Dashboard:** `~/knotly-prototype/index.html` (5 views: Dashboard, Sources, Playground, Embed, Analytics)
- **Design system:** Based on Vercel Geist patterns (rounded-[14px] buttons, rounded-[18px] cards, Inter font)
- **Components:** Button, Card, Input, Modal, Navbar, Tabs, Toast, Table, Badge (from skills folder)

---

## 12. DEVELOPMENT ROADMAP

### Phase 1: MVP (4 weeks)
- [ ] Document upload + parsing (PDF, DOCX, TXT, web crawl)
- [ ] Chunking + embedding (Voyage-3, 200-token chunks with contextual headers)
- [ ] Hybrid search (BM25 + semantic via Pinecone)
- [ ] Reranking (Cohere)
- [ ] Basic chat generation (GPT-4o-mini)
- [ ] Embeddable widget (script tag, iframe)
- [ ] Dashboard (create bot, manage sources, test chat)
- [ ] Multi-tenant (Pinecone namespaces)
- [ ] Stripe billing

### Phase 2: Trust & Quality (2 weeks)
- [ ] Anthropic Citations API integration
- [ ] HHEM faithfulness checking
- [ ] Confidence scoring + "I don't know"
- [ ] Query reformulation for multi-turn
- [ ] Analytics dashboard (top questions, gaps)

### Phase 3: Growth (2 weeks)
- [ ] Human escalation (Zendesk/Intercom webhook)
- [ ] Cross-session memory (Mem0 or custom)
- [ ] White-label mode
- [ ] React/npm widget component
- [ ] API documentation

### Phase 4: Scale (ongoing)
- [ ] Self-hosting option (Docker)
- [ ] GraphRAG for complex knowledge bases
- [ ] Voice/phone channel
- [ ] Multi-language optimization
- [ ] Agency reseller program

---

## APPENDIX: Research Sources

16 parallel research agents covering:
1. RAG Techniques (150+ cataloged) — NirDiamant repo + web research
2. Chatbase Product Analysis — pricing, features, API, 12 gaps
3. Chatbase Competitors (22 products) — features, pricing, positioning
4. Community Feedback — Reddit, HN real user experiences
5. Tech Blogs — Anthropic, Pinecone, Weaviate, Cohere, LlamaIndex
6. RAG Benchmarks — actual numbers from published research
7. Open-Source Implementations — AnythingLLM, DocsGPT, RAGFlow, Onyx, Quivr
8. Vector DB Comparison — Pinecone, Weaviate, Qdrant, pgvector, Milvus, Chroma, LanceDB
9. Document Parsing — MinerU, Docling, Marker, Unstructured, LlamaParse, Azure, AWS
10. Embed Widget UX — Intercom, Crisp, Chatbase, Chatwoot architectures inspected
11. Pricing Models — per-message, per-resolution, flat-rate, hybrid analysis
12. Accuracy/Hallucination — grounding, citations, HHEM, CRAG, confidence scoring
13. Multi-Tenant Architecture — namespaces, partitions, RLS, vendor recommendations
14. Conversation Memory — Mem0, Zep, LangGraph, query reformulation
15. Human Escalation — trigger patterns, handoff mechanisms, context transfer
16. Kaggle/GitHub Landscape — competitive intelligence from open repos
