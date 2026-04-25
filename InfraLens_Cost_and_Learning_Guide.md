# InfraLens — Cost Breakdown & Learning Outcomes

## Cost Summary

There are two realistic paths depending on how much you want to spend. The good news: almost every tool in the stack has a generous free tier, so you can build the entire project for nearly nothing if you're willing to run things locally and use free credits strategically.

---

## Detailed Cost Breakdown

### 1. LLM API — The Biggest Variable

This is your main cost lever. You need an LLM for the Supervisor, Diagnosis, and Comms agents.

**Budget path ($0–5/mo):**
- Anthropic gives $5 in free credits when you sign up for the API
- OpenAI gives $5–10 in free credits for new accounts
- Use the **Batch API** (50% off) for non-real-time workloads like eval runs and postmortem indexing
- Use **prompt caching** (90% discount on cached input tokens) — your system prompts for each agent are static and cacheable
- Route simple tasks to **Claude Haiku** ($1/$5 per 1M tokens) instead of Sonnet/Opus
- For development and testing, use smaller open-source models via Ollama (completely free, runs locally)

**Comfortable path ($15–30/mo):**
- Claude Sonnet 4.6: $3 input / $15 output per 1M tokens
- GPT-4o: $2.50 input / $10 output per 1M tokens
- At ~200 simulated incidents/month with 5 agent calls per incident, expect ~2M input + 500K output tokens/month = roughly $15–25

**Cost optimization tip:** This is actually a *resume talking point*. Implement intelligent model routing — use Haiku/GPT-4o-mini for triage classification, Sonnet/GPT-4o only for complex reasoning. Document the cost reduction.

### 2. HuggingFace — Models & Inference

**Budget path ($0/mo):**
- Free Inference API tier: $0.10/month in credits, rate-limited but workable for development
- Run models locally with `transformers` library — distilbert, bert-base-NER are small enough for a laptop with 8GB RAM
- Fine-tuning on free Google Colab (T4 GPU, 12GB VRAM) — more than enough for BERT-class models

**Comfortable path ($9/mo):**
- HF Pro: $9/month gets you 2M inference credits, 25 min/day of H200 compute via ZeroGPU, and higher rate limits
- Worth it if you want to fine-tune larger models or run VQA models like BLIP-2 without a local GPU

### 3. Vector Database (pgvector)

**Budget path ($0/mo):**
- Run PostgreSQL + pgvector locally via Docker Compose (you're already using Docker)
- Or use **Supabase Free Tier**: 500MB storage, pgvector included, no credit card
- Or **Neon Free Tier**: 0.5GB storage, 100 CU-hours/month
- 500+ runbooks with embeddings will use ~50–100MB — well within free limits

**Comfortable path ($0–25/mo):**
- Supabase Pro ($25/mo) if you want always-on database without auto-pause
- Realistically, for a portfolio project, free tier is fine

### 4. Neo4j Knowledge Graph

**Both paths ($0/mo):**
- Neo4j AuraDB Free: no credit card, sufficient for prototyping
- Or run Neo4j Community Edition locally via Docker
- Your incident knowledge graph won't exceed free tier limits

### 5. LangSmith (Observability & Tracing)

**Both paths ($0/mo):**
- Developer plan: 5,000 traces/month, 14-day retention, completely free
- For a portfolio project with 200 simulated incidents, you'll use ~1,000 traces/month
- This is more than enough

### 6. DeepEval (LLM Evaluation)

**Both paths ($0/mo):**
- Fully open-source, Apache 2.0 license
- The only cost is the LLM API calls for "LLM-as-judge" evaluations (already counted in LLM API costs above)
- `pip install deepeval` and you're done

### 7. Infrastructure & CI/CD

**Both paths ($0/mo):**
- Docker Desktop: free for personal use
- Docker Compose: free
- GitHub Actions: 2,000 minutes/month on free tier (plenty for eval pipeline runs)
- Prometheus + Grafana: self-hosted via Docker Compose, or Grafana Cloud free tier (10K metrics, 50GB logs)

### 8. Optional Extras

| Item | Cost | Notes |
|---|---|---|
| Custom domain | $10–15/year | Optional — use HF Spaces for free demo hosting |
| Vercel (frontend) | $0 | Hobby tier is free |
| Google Colab Pro | $10/mo | Only if you need more GPU for fine-tuning |

---

## Total Cost Summary

| Path | Monthly | 12-Week Total | What You Get |
|---|---|---|---|
| **Budget (free tiers + local)** | $0 – $5 | **$0 – $15** | Fully functional project, local development, free hosted services |
| **Comfortable (paid tiers)** | $34 – $79 | **$100 – $240** | Faster iteration, better GPU access, always-on services |
| **Middle ground (recommended)** | $10 – $20 | **$30 – $60** | HF Pro + modest LLM API usage + everything else free |

**Bottom line:** You can build this entire project for under $60 total if you're strategic about free tiers and local development. The LLM API is the only meaningful cost, and even that can be minimized with caching, batching, and model routing.

---

## What You'll Learn (and How It Complements Your Trading Project)

Since you already have a project in the trading space, InfraLens fills very different gaps in your portfolio. Here's what's new versus what overlaps.

### Skills That Are Completely New from InfraLens

**1. Multi-Agent Orchestration (LangGraph)**
Your trading project likely uses a single pipeline or chain. InfraLens teaches you to build a *stateful multi-agent system* where 5 agents coordinate, pass state, handle failures, and make routing decisions. This is the #1 skill hiring managers are scanning for in 2026 AI engineering roles.

What you'll learn: LangGraph state machines, supervisor patterns, conditional routing, human-in-the-loop checkpoints, agent retry logic, and how to debug multi-agent failures via tracing.

**2. Multi-Modal AI (Vision + Text + Structured Data)**
Trading projects typically work with numerical/text data. InfraLens adds *visual understanding* — parsing Grafana dashboard screenshots with Visual QA models. This demonstrates you can work across modalities, not just text.

What you'll learn: HuggingFace vision-language models (BLIP-2, Florence-2), image preprocessing for model input, prompt engineering for VQA tasks, and when visual AI is practical vs. overkill.

**3. Fine-Tuning Open-Source Models**
Unless your trading project involved training models, this is likely new. You'll fine-tune BERT for alert classification and NER for log entity extraction — real model training, not just API calls.

What you'll learn: HuggingFace Trainer API, dataset preparation and labeling, training/validation splits, hyperparameter tuning, model evaluation metrics (precision, recall, F1), and pushing models to HuggingFace Hub with model cards.

**4. Production RAG Engineering (Beyond Basic)**
Even if your trading project uses some form of retrieval, InfraLens goes deeper: semantic chunking (not naive fixed-size), hybrid search (BM25 + vector), reranking with cross-encoders, and evaluation of retrieval quality.

What you'll learn: LlamaIndex document parsing, chunking strategies and why they matter, pgvector indexing, hybrid retrieval architectures, reranking with BGE-reranker, and how to measure retrieval precision/recall.

**5. LLM Evaluation & Testing (DeepEval)**
This is almost certainly new. Most portfolio projects skip evaluation entirely — and that's exactly why hiring managers notice when you include it. You'll build a proper eval harness that runs in CI/CD.

What you'll learn: Faithfulness metrics (does the output match the retrieved context?), answer relevancy, contextual precision/recall, building eval datasets, LLM-as-judge patterns, and CI/CD integration that blocks bad merges.

**6. Knowledge Graphs (Neo4j)**
Storing incident causality chains as a graph — "this error was caused by X, which was fixed by Y" — is a different data paradigm than anything in typical trading projects.

What you'll learn: Neo4j Cypher query language, graph data modeling for causal relationships, when to use graphs vs. relational vs. vector stores, and building a "learning" system that improves from past incidents.

**7. LLMOps & Observability**
Monitoring AI systems in production is different from monitoring traditional software. You'll instrument every agent call, track token costs, set SLOs, and build dashboards.

What you'll learn: LangSmith tracing integration, cost-per-inference tracking, latency percentile monitoring, agent SLO definition, Prometheus custom metrics for AI workloads, and Grafana dashboard design.

### Skills That Overlap with Trading (Deepened Here)

**8. API Integration & Webhook Design**
You probably already consume financial APIs. InfraLens adds webhook receivers (PagerDuty, Datadog) and outbound integrations (Slack), which is a different pattern — event-driven rather than polling.

**9. Docker & Infrastructure-as-Code**
If your trading project uses Docker, great — InfraLens reinforces this with a more complex multi-container setup (app + Postgres + Neo4j + Prometheus + Grafana).

**10. System Design & Architecture**
Both projects demonstrate systems thinking, but InfraLens operates in a fundamentally different domain. Having two well-architected projects across different domains signals versatility.

### The Portfolio Story This Creates

With a trading AI project *and* InfraLens, your portfolio tells a compelling narrative:

- **Trading project** → "I can build AI systems that handle financial data, real-time signals, and decision-making"
- **InfraLens** → "I can build multi-agent AI systems that handle unstructured operational data, visual inputs, and production-grade evaluation"
- **Together** → "I'm not a one-trick pony. I understand AI engineering *patterns* — RAG, agents, eval, LLMOps — and can apply them across domains."

This is exactly what separates a mid-level AI engineer from someone who followed one tutorial. You're showing *transferable AI engineering skills* across two very different problem domains.

---

## Quick Reference: Skills Inventory

| Skill | Trading Project | InfraLens | New? |
|---|---|---|---|
| Multi-agent orchestration (LangGraph) | Unlikely | Core | Yes |
| Vision/multi-modal AI (VQA) | No | Core | Yes |
| Fine-tuning HF models | Maybe | Core | Likely yes |
| Advanced RAG (hybrid + reranking) | Basic retrieval | Production-grade | Deepened |
| LLM evaluation (DeepEval) | No | Core | Yes |
| Knowledge graphs (Neo4j) | No | Core | Yes |
| LLMOps & observability | No | Core | Yes |
| CI/CD eval gates | No | Core | Yes |
| Prompt/context engineering | Some | Extensive | Deepened |
| API integration | Financial APIs | DevOps APIs + webhooks | Extended |
| Docker multi-container | Maybe basic | Complex compose stack | Deepened |
| Cost optimization (model routing) | Maybe | Documented & measured | Yes |
