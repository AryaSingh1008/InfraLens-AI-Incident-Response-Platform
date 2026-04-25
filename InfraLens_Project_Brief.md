# InfraLens — AI-Powered Incident Response & Infrastructure Intelligence Platform

## What It Does

InfraLens is a multi-agent AI system that automatically triages, diagnoses, and recommends fixes for infrastructure incidents in real time. When an alert fires — a spike in latency, a pod crash loop, a database connection exhaustion — InfraLens ingests the alert context, reads the relevant Grafana dashboards visually, extracts entities from logs, retrieves matching runbooks and past postmortems via RAG, performs root cause analysis, and drafts a remediation plan with Slack-ready communications.

**The real-world problem it solves:** On-call engineers spend 30–60 minutes per incident just gathering context — reading dashboards, searching Confluence, scrolling logs. InfraLens compresses that to under 2 minutes by doing the grunt work with AI agents while the human makes the final call.

---

## Why This Project Stands Out in 2026

1. **It bridges DevOps → AI Engineering directly.** Hiring managers see a candidate who understands both infrastructure operations *and* can build production AI systems. That's the exact gap most AI engineering JDs describe.

2. **It's not another chatbot or summarizer.** Multi-agent orchestration with specialized HuggingFace models, a RAG pipeline over real operational data, and a proper eval harness — this demonstrates systems thinking, not just API wrapping.

3. **It solves a problem every engineering org has.** Incident response is universal. Anyone who's been on-call immediately understands the value.

---

## HuggingFace Tasks Used

| HF Task | How It's Used | Example Model |
|---|---|---|
| **Text Classification** | Severity triage (P0–P4), alert category routing (network, compute, storage, app) | `distilbert-base-uncased` fine-tuned on PagerDuty/Datadog alert corpus |
| **Visual Question Answering** | Parse Grafana/CloudWatch dashboard screenshots — "Is there a latency spike?", "What's the error rate trend?" | `Salesforce/blip2-opt-2.7b` or `microsoft/Florence-2-large` |
| **Token Classification (NER)** | Extract entities from logs: service names, error codes, IP addresses, pod names, timestamps | `dslim/bert-base-NER` fine-tuned on infrastructure log data |
| **Document Question Answering** | Query runbooks and postmortems — "What's the remediation for Redis connection exhaustion?" | `impira/layoutlm-document-qa` |
| **Text Generation** | Draft incident summaries, Slack updates, and remediation steps | Llama 3.1 8B or Mistral 7B via HF Inference API |

---

## System Architecture

### Layer 1: Data Ingestion
- **Alert Streams:** Webhook receivers for PagerDuty, Datadog, OpsGenie (use their public APIs with synthetic/sample data)
- **Dashboard Screenshots:** Selenium or Playwright captures of Grafana dashboards triggered by alerts
- **Logs & Metrics:** Pull from ELK/Loki/Prometheus via API on alert trigger
- **Runbooks & Postmortems:** Indexed from Confluence/Notion exports or markdown files in Git

### Layer 2: HuggingFace Task Processing
Each data type routes through a specialized HF model pipeline:
- Alerts → Text Classification (severity + category)
- Dashboard images → Visual QA (anomaly detection + trend reading)
- Logs → Token Classification (entity extraction) + Text Classification (error categorization)
- Runbooks → Document QA (retrieval-augmented answers)

### Layer 3: Multi-Agent Orchestration (LangGraph)
Five agents coordinate via a LangGraph state machine:

1. **Triage Agent** — Classifies severity, determines blast radius, decides which downstream agents to activate
2. **Diagnosis Agent** — Correlates signals from logs, metrics, and dashboard readings to perform root cause analysis
3. **Runbook Agent** — RAG-powered retrieval over runbooks and postmortems to find matching remediation steps
4. **Comms Agent** — Drafts Slack incident updates, status page messages, and escalation notifications
5. **Supervisor Agent** — Orchestrates the workflow, handles retries, manages agent state transitions

### Layer 4: RAG Pipeline & Knowledge Store
- **Vector Store:** pgvector (PostgreSQL) for runbook/postmortem embeddings
- **Embedding Pipeline:** `BAAI/bge-large-en-v1.5` via HuggingFace, with semantic chunking (not naive fixed-size)
- **Incident Knowledge Graph:** Neo4j to store causal relationships — "Redis connection exhaustion → caused by → connection pool misconfiguration → fixed by → increasing pool size to 50"
- **Hybrid Search:** BM25 + vector similarity for retrieval, with reranking via `BAAI/bge-reranker-large`

### Layer 5: Eval Harness & LLMOps
- **DeepEval:** Faithfulness, answer relevancy, contextual precision/recall on RAG outputs
- **LangSmith:** Full trace logging for every agent interaction — latency, token usage, cost per incident
- **Prometheus + Grafana:** Operational metrics — agent SLOs, model inference latency, retrieval hit rates
- **CI/CD Eval Gates:** GitHub Actions pipeline that runs eval suite on every PR; blocks merge if faithfulness drops below threshold

---

## Recommended Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Agent Framework** | LangGraph (LangChain) | Best-in-class for stateful multi-agent workflows with human-in-the-loop |
| **LLM Backbone** | Claude API or GPT-4o (orchestration) + HF models (specialized tasks) | Hybrid approach — use frontier models for reasoning, open-source for classification/NER |
| **RAG Framework** | LlamaIndex | Superior document parsing, hierarchical indexing, and retrieval abstractions |
| **Vector DB** | pgvector (PostgreSQL) | Production-proven, no vendor lock-in, familiar to DevOps engineers |
| **Knowledge Graph** | Neo4j | Incident causality chains are naturally graph-structured |
| **Embeddings** | BAAI/bge-large-en-v1.5 (HuggingFace) | Top-tier open-source embeddings, self-hosted |
| **Eval** | DeepEval + LangSmith | DeepEval for RAG metrics, LangSmith for traces and cost analysis |
| **Observability** | Prometheus + Grafana | Dogfooding — use the same tools the system monitors |
| **Infra** | Docker Compose → Kubernetes (stretch) | Start simple, show you can scale |
| **CI/CD** | GitHub Actions | Eval gates, model artifact versioning, automated testing |
| **Frontend** | Streamlit or Next.js | Streamlit for MVP, Next.js if you want to flex full-stack |

---

## Implementation Roadmap (12 Weeks)

### Phase 1: Foundation (Weeks 1–3)
- Set up project structure, Docker Compose environment
- Build alert ingestion webhook (mock PagerDuty/Datadog payloads)
- Implement Text Classification pipeline for severity triage (fine-tune on labeled alert data)
- Set up pgvector and initial runbook indexing with LlamaIndex
- Deliverable: Alerts come in, get classified, matching runbooks retrieved

### Phase 2: Multi-Modal Intelligence (Weeks 4–6)
- Integrate Visual QA for Grafana dashboard screenshot analysis
- Build Token Classification pipeline for log entity extraction
- Implement Document QA over runbooks
- Build the embedding pipeline with semantic chunking and hybrid search
- Deliverable: System can read dashboards, parse logs, and answer questions about runbooks

### Phase 3: Agent Orchestration (Weeks 7–9)
- Build LangGraph state machine with all 5 agents
- Implement Supervisor agent with routing logic and retry handling
- Add human-in-the-loop approval for P0/P1 remediation suggestions
- Build the Comms agent with Slack webhook integration
- Deliverable: End-to-end incident response flow — alert in, diagnosis + remediation out

### Phase 4: Eval, LLMOps & Polish (Weeks 10–12)
- Build DeepEval test suite (faithfulness, relevancy, precision/recall)
- Set up LangSmith tracing and cost dashboards
- Implement CI/CD eval gates in GitHub Actions
- Add Prometheus metrics for agent SLOs
- Build incident knowledge graph with Neo4j for pattern learning
- Write technical blog post and record demo video
- Deliverable: Production-ready system with eval harness, monitoring, and documentation

---

## How This Maps to Real AI Engineering JDs

| JD Requirement | How InfraLens Demonstrates It |
|---|---|
| "Build and deploy RAG pipelines" | pgvector + LlamaIndex + hybrid search + reranking — not a toy demo |
| "Design multi-agent systems" | 5-agent LangGraph architecture with supervisor pattern |
| "Implement LLM evaluation" | DeepEval suite with CI/CD gates blocking bad merges |
| "Production ML/AI systems" | Docker, Prometheus monitoring, LangSmith tracing, structured logging |
| "Fine-tune and deploy models" | Fine-tuned text classifier + NER model on HuggingFace Hub |
| "Work with unstructured data" | Logs, dashboards (images), runbooks (documents), alerts (text) |
| "MLOps / LLMOps experience" | Model versioning, eval pipelines, automated testing, cost tracking |
| "Cloud infrastructure knowledge" | The entire domain — plus you're using Kubernetes, Docker, Prometheus |
| "Cross-functional collaboration" | Comms agent demonstrates understanding of incident communication patterns |
| "Context engineering" | Careful prompt design, retrieval strategies, agent state management |

---

## Resume-Ready Impact Metrics

Use these quantified bullets on your resume (measure them against a baseline of manual incident response):

- **"Built a multi-agent AI incident response platform that reduced mean-time-to-diagnosis from 45 min to <3 min across 200+ simulated infrastructure incidents"**
- **"Designed a RAG pipeline over 500+ runbooks achieving 91% faithfulness and 87% answer relevancy scores (DeepEval), with hybrid BM25 + vector retrieval and reranking"**
- **"Implemented LLM eval gates in CI/CD that caught 23% of regressions before merge, reducing hallucinated remediation steps by 78%"**
- **"Orchestrated 5 specialized AI agents via LangGraph with <4s median end-to-end latency and 99.2% task completion rate"**
- **"Fine-tuned a BERT-based alert classifier on 10K labeled incidents achieving 94% accuracy on severity triage (P0–P4)"**
- **"Reduced per-incident LLM inference cost by 62% through intelligent model routing — frontier models for reasoning, open-source for classification"**

---

## Portfolio Presentation Strategy

### GitHub Repository
- Clean README with architecture diagram, quick-start instructions, and demo GIF
- Well-structured codebase: `/agents`, `/pipelines`, `/eval`, `/infra`, `/api`
- `eval/` directory with reproducible benchmark results
- GitHub Actions workflow visible in the repo

### Live Demo
- Record a 3–5 minute Loom showing an alert triggering the full pipeline
- Show the LangSmith trace of agents coordinating
- Show the Grafana dashboard of system metrics
- Show an eval run catching a regression

### Technical Blog Post
- Publish on Medium/Dev.to: "How I Built a Multi-Agent Incident Response System with HuggingFace and LangGraph"
- Focus on design decisions, trade-offs, and what you learned
- Include benchmarks comparing RAG retrieval strategies

### HuggingFace Presence
- Push fine-tuned models to HuggingFace Hub with model cards
- Create a HuggingFace Space with a Streamlit demo
- Contribute evaluation datasets for infrastructure NLP tasks

---

## What Makes This Beat Other Portfolio Projects

Most AI portfolio projects in 2026 fall into three buckets: chatbots, document summarizers, or code assistants. InfraLens is none of those. It demonstrates:

1. **Domain expertise** — You understand infrastructure operations, not just AI APIs
2. **Multi-modal reasoning** — Text, images, structured data, and documents
3. **Systems design** — Five agents coordinating through a state machine, not a single prompt chain
4. **Production discipline** — Eval harness, CI/CD gates, monitoring, cost optimization
5. **Real HuggingFace depth** — Fine-tuned models, not just API calls to GPT-4

The combination of DevOps domain knowledge + AI engineering skills is rare and highly valued. This project is the proof that you can bridge both worlds.
