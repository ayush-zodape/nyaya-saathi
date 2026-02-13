# Nyaya Saathi — Design Document

## 1. High-Level Architecture

Nyaya Saathi follows a **three-tier architecture** with clear separation between the presentation layer, application logic layer, and data/AI layer. The system is built entirely on AWS managed services to minimize operational overhead and maximize scalability.

```
┌─────────────────────────────────────────────────────────────────┐
│                      USER LAYER                                  │
│  ┌──────────────────────┐   ┌──────────────────────┐           │
│  │   Mobile Browser     │   │   Desktop Browser    │           │
│  │   (Hindi/English)    │   │   (Hindi/English)    │           │
│  └──────────┬───────────┘   └──────────┬───────────┘           │
│              └──────────┬──────────────┘                        │
│                         ▼                                       │
│              ┌────────────────────┐                              │
│              │   Streamlit App    │                              │
│              │   (Frontend UI)    │                              │
│              └────────┬───────────┘                              │
├───────────────────────┼─────────────────────────────────────────┤
│                       ▼           APPLICATION LAYER              │
│              ┌────────────────────┐                              │
│              │  Amazon API        │                              │
│              │  Gateway (REST)    │                              │
│              └────────┬───────────┘                              │
│                       ▼                                          │
│              ┌────────────────────┐                              │
│              │  AWS Lambda        │                              │
│              │  (Orchestrator)    │                              │
│              └────────┬───────────┘                              │
│                       │                                          │
│         ┌─────────────┼─────────────┐                           │
│         ▼             ▼             ▼                            │
├─────────────────────────────────────────────────────────────────┤
│                    AI / DATA LAYER                               │
│  ┌──────────────┐ ┌───────────────┐ ┌────────────────┐         │
│  │ Amazon       │ │ Amazon Bedrock│ │ Amazon S3      │         │
│  │ Bedrock      │ │ Knowledge     │ │ (Legal Corpus  │         │
│  │ (LLM - Nova/ │ │ Bases (RAG)   │ │  + DLSA Data)  │         │
│  │  Claude)     │ │               │ │                │         │
│  └──────────────┘ └───────────────┘ └────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Rationale
- **Serverless-first:** Using Lambda + API Gateway means zero infrastructure management, automatic scaling, and pay-per-use pricing — ideal for a hackathon prototype and future scaling.
- **Managed AI services:** Amazon Bedrock provides access to frontier LLMs without needing to host models, manage GPUs, or handle model versioning.
- **RAG over fine-tuning:** We use Retrieval-Augmented Generation instead of fine-tuning because: (a) legal acts change — RAG allows instant updates by replacing S3 documents, (b) RAG provides source attribution (cite the exact Act & Section), (c) it's dramatically cheaper and faster to implement.

---

## 2. Component Design

### 2.1 Frontend — Streamlit Chat Interface

**Technology:** Streamlit (Python)

**Responsibilities:**
- Render a mobile-responsive chat interface
- Display conversation history with user messages and AI responses
- Show legal rights as expandable cards with Act/Section citations
- Display generated documents with copy-to-clipboard and download options
- Show the persistent disclaimer banner
- Language toggle (Hindi / English)

**Key UI Elements:**

| Element | Description |
|---------|-------------|
| Chat Input | Text input box at the bottom (WhatsApp-style) for natural language input |
| Message Bubbles | User messages (right-aligned, blue) and AI responses (left-aligned, white) |
| Rights Cards | Expandable cards showing: Right name → Simple explanation → Act & Section → Action steps |
| Document Panel | Side panel or modal showing generated legal document with Copy/Download buttons |
| Disclaimer Banner | Fixed top banner: "यह मार्गदर्शन है, कानूनी सलाह नहीं" |
| Language Toggle | Hindi/English switch in top-right corner |
| Legal Aid Finder | Button to search nearest DLSA by state/district dropdown |

**Design Decisions:**
- **Streamlit over React:** Chosen for speed of development (Python-only, no separate frontend build) and beginner-friendliness. For production, would migrate to Next.js/React.
- **Chat-style interface:** Familiar pattern for users (WhatsApp-like), lowers learning curve to zero.
- **Mobile-first:** CSS media queries ensure usability on 5-inch screens with touch input.

### 2.2 API Layer — Amazon API Gateway + AWS Lambda

**Technology:** Amazon API Gateway (REST API) + AWS Lambda (Python 3.12)

**Lambda Function: `nyaya-saathi-orchestrator`**

This is the central brain that orchestrates the entire flow:

```
User Message
    │
    ▼
┌─────────────────────┐
│ 1. Input Processing  │ ← Detect language (Hindi/English/Hinglish)
│    & Classification  │ ← Classify intent: new_situation | follow_up | document_request | legal_aid_search
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ 2. Situation         │ ← If new situation: Extract key facts
│    Analysis          │ ← If follow-up: Merge with conversation context
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ 3. RAG Retrieval     │ ← Query Bedrock Knowledge Base with extracted facts
│                      │ ← Retrieve top-k relevant legal provisions
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ 4. Response          │ ← Generate plain-language explanation
│    Generation        │ ← Generate action steps
│                      │ ← If requested: Generate legal document draft
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ 5. Safety & Format   │ ← Add disclaimers and source citations
│                      │ ← Format for display (Markdown)
└─────────────────────┘
          │
          ▼
     API Response
```

**Intent Classification (Step 1):**

| Intent | Trigger Examples | Action |
|--------|-----------------|--------|
| `new_situation` | "My husband beats me" / "Mera maalik paisa nahi de raha" | Full RAG pipeline |
| `follow_up` | "What if he threatens me?" / "Aur kya kar sakti hoon?" | Context-aware follow-up |
| `document_request` | "RTI likho" / "Draft a complaint" | Document generation |
| `legal_aid_search` | "Nearest legal aid" / "Free vakil kahan milega?" | DLSA directory lookup |
| `out_of_scope` | "What's the weather?" / "How to make a bomb?" | Polite refusal |

**Conversation Context Management:**
- Lambda maintains conversation state via session IDs passed from the frontend
- Context includes: identified situation type, extracted facts, previously identified legal provisions, and language preference
- Context is stored in-memory within the Lambda invocation (for prototype) — production would use DynamoDB

### 2.3 AI Layer — Amazon Bedrock

**Foundation Model:** Amazon Nova Lite (primary) or Anthropic Claude 3 Sonnet (fallback)

**Rationale for Amazon Nova Lite:**
- **Cost:** Significantly cheaper than Claude 3 Sonnet for high-volume usage
- **Speed:** Lower latency for conversational interactions
- **Hindi capability:** Strong multilingual support including Hindi
- **AWS-native:** Tighter integration with Bedrock Knowledge Bases

**Prompt Engineering Strategy:**

The system uses a **chain of specialized prompts** rather than a single mega-prompt:

**Prompt 1 — Situation Analyzer:**
```
System: You are a legal rights analyst for India. Given a person's description 
of their situation, extract structured facts including:
- Category (domestic violence, wage theft, discrimination, consumer complaint, etc.)
- Key actors (who is involved)
- Key events (what happened)
- Location/jurisdiction clues
- Urgency level (immediate danger vs. ongoing issue vs. past event)

Respond ONLY in structured JSON format.
```

**Prompt 2 — Rights Explainer (with RAG context):**
```
System: You are Nyaya Saathi, a legal rights guide for India. Based on the 
retrieved legal provisions and the user's situation:

1. List all applicable legal rights (max 5 most relevant)
2. For each right, provide:
   - The right in one simple sentence (Class 8 Hindi reading level)
   - The specific Act name and Section number
   - One sentence explaining what this means practically
3. Then provide step-by-step action guide (max 7 steps)

Rules:
- ALWAYS cite Act name and Section number
- Use simple Hindi or English as per user's language
- If unsure, say so — never fabricate legal provisions
- End with recommendation to consult DLSA for free legal aid

Retrieved Legal Context: {rag_results}
User Situation: {situation_analysis}
```

**Prompt 3 — Document Drafter:**
```
System: Draft a {document_type} in {language} for the following situation.
Follow the standard format for {document_type} in India.

Include:
- Proper addressing (To: appropriate authority)
- Date placeholder
- Subject line
- Body with relevant facts and legal provisions
- Specific Act & Section references
- Relief/information sought
- Placeholder markers: [YOUR NAME], [YOUR ADDRESS], [DATE], [AADHAAR NUMBER]

Situation: {facts}
Applicable Laws: {identified_provisions}
```

### 2.4 Knowledge Base — Amazon Bedrock Knowledge Bases + Amazon S3

**Data Source:** Indian legal acts sourced from [indiacode.nic.in](https://www.indiacode.nic.in/) (government public domain)

**S3 Bucket Structure:**
```
s3://nyaya-saathi-legal-corpus/
├── acts/
│   ├── ipc-bharatiya-nyaya-sanhita.pdf
│   ├── crpc-bharatiya-nagarik-suraksha-sanhita.pdf
│   ├── domestic-violence-act-2005.pdf
│   ├── dowry-prohibition-act-1961.pdf
│   ├── sc-st-atrocities-act-1989.pdf
│   ├── rti-act-2005.pdf
│   ├── mgnrega-act-2005.pdf
│   ├── consumer-protection-act-2019.pdf
│   ├── motor-vehicles-act-1988.pdf
│   ├── payment-of-wages-act-1936.pdf
│   ├── minimum-wages-act-1948.pdf
│   ├── senior-citizens-act-2007.pdf
│   ├── pocso-act-2012.pdf
│   ├── rte-act-2009.pdf
│   └── legal-services-authorities-act-1987.pdf
├── simplified-guides/
│   ├── domestic-violence-rights-hindi.md
│   ├── labor-rights-hindi.md
│   ├── rti-guide-hindi.md
│   └── free-legal-aid-guide-hindi.md
└── dlsa-directory/
    └── dlsa-all-india.json
```

**Knowledge Base Configuration:**
- **Chunking Strategy:** Fixed-size chunks of 512 tokens with 50-token overlap (optimized for legal act sections which are typically paragraph-length)
- **Embedding Model:** Amazon Titan Embeddings V2
- **Vector Store:** Amazon OpenSearch Serverless (managed by Bedrock Knowledge Bases)
- **Retrieval:** Top-5 chunks per query with metadata filtering (can filter by act name)

**Why This Structure:**
- **Acts as PDFs:** Legal acts are available as PDFs from indiacode.nic.in — direct upload with no preprocessing
- **Simplified guides:** Hand-curated plain-language summaries of key rights in Hindi — ensures RAG retrieval can find simple explanations, not just legal jargon
- **DLSA directory:** JSON file with all District Legal Services Authorities for the nearest-center finder

---

## 3. Data Flow Diagrams

### 3.1 Core Conversation Flow

```
User                    Frontend              Lambda              Bedrock           Knowledge Base
 │                        │                     │                   │                    │
 │  "Mere pati mujhe     │                     │                   │                    │
 │   marte hain"         │                     │                   │                    │
 │───────────────────────>│                     │                   │                    │
 │                        │  POST /chat         │                   │                    │
 │                        │  {msg, session_id,  │                   │                    │
 │                        │   lang: "hi"}       │                   │                    │
 │                        │────────────────────>│                   │                    │
 │                        │                     │  Prompt 1:        │                    │
 │                        │                     │  Analyze situation│                    │
 │                        │                     │──────────────────>│                    │
 │                        │                     │  {category:       │                    │
 │                        │                     │   "domestic_      │                    │
 │                        │                     │   violence",      │                    │
 │                        │                     │   actors:...}     │                    │
 │                        │                     │<──────────────────│                    │
 │                        │                     │                   │                    │
 │                        │                     │  Query: "domestic │                    │
 │                        │                     │  violence wife    │                    │
 │                        │                     │  beaten husband   │                    │
 │                        │                     │  dowry jewelry"   │                    │
 │                        │                     │───────────────────────────────────────>│
 │                        │                     │  Retrieved:       │                    │
 │                        │                     │  - DV Act Sec 3   │                    │
 │                        │                     │  - DV Act Sec 17  │                    │
 │                        │                     │  - IPC 498A       │                    │
 │                        │                     │  - Dowry Act Sec 3│                    │
 │                        │                     │<───────────────────────────────────────│
 │                        │                     │                   │                    │
 │                        │                     │  Prompt 2: Rights │                    │
 │                        │                     │  + RAG context    │                    │
 │                        │                     │──────────────────>│                    │
 │                        │                     │  Hindi response:  │                    │
 │                        │                     │  Rights + Steps   │                    │
 │                        │                     │<──────────────────│                    │
 │                        │                     │                   │                    │
 │                        │  {response + disclaimer}                │                    │
 │                        │<────────────────────│                   │                    │
 │  आपके ये अधिकार हैं:  │                     │                   │                    │
 │  1. घरेलू हिंसा...     │                     │                   │                    │
 │  2. दहेज वापसी...      │                     │                   │                    │
 │<───────────────────────│                     │                   │                    │
```

### 3.2 Document Generation Flow

```
User                    Frontend              Lambda              Bedrock
 │                        │                     │                   │
 │  "RTI likho"          │                     │                   │
 │───────────────────────>│                     │                   │
 │                        │  POST /chat         │                   │
 │                        │  {msg, session_id}  │                   │
 │                        │────────────────────>│                   │
 │                        │                     │                   │
 │                        │                     │  Intent: doc_req  │
 │                        │                     │  Type: RTI        │
 │                        │                     │  Context: MGNREGA │
 │                        │                     │  wage dispute     │
 │                        │                     │                   │
 │                        │                     │  Prompt 3: Draft  │
 │                        │                     │  RTI application  │
 │                        │                     │──────────────────>│
 │                        │                     │                   │
 │                        │                     │  Formatted RTI    │
 │                        │                     │  application text │
 │                        │                     │<──────────────────│
 │                        │                     │                   │
 │                        │  {document +        │                   │
 │                        │   copy/download}    │                   │
 │                        │<────────────────────│                   │
 │  [RTI Document with   │                     │                   │
 │   Copy & Download]    │                     │                   │
 │<───────────────────────│                     │                   │
```

---

## 4. Technology Stack Summary

| Layer | Technology | Justification |
|-------|-----------|---------------|
| **Frontend** | Streamlit (Python) | Rapid prototyping, built-in chat components, mobile-responsive, Python-only (no JS needed) |
| **API** | Amazon API Gateway | Managed REST API, throttling, CORS support, integrates natively with Lambda |
| **Compute** | AWS Lambda (Python 3.12) | Serverless, auto-scaling, pay-per-invocation, no server management |
| **LLM** | Amazon Bedrock (Nova Lite / Claude 3 Sonnet) | Managed LLM access, no GPU management, strong Hindi support, prompt-based interaction |
| **RAG** | Amazon Bedrock Knowledge Bases | Managed RAG pipeline — handles chunking, embedding, vector storage, and retrieval automatically |
| **Embeddings** | Amazon Titan Embeddings V2 | Optimized for Bedrock Knowledge Bases, strong multilingual embedding quality |
| **Vector DB** | Amazon OpenSearch Serverless | Auto-provisioned by Bedrock Knowledge Bases, no configuration needed |
| **Storage** | Amazon S3 | Legal act PDFs, DLSA directory JSON, cost: < $0.01/month for our data size |
| **Hosting** | AWS EC2 (t3.micro) or AWS App Runner | Hosts Streamlit app; EC2 free tier eligible, App Runner for production |
| **IAM** | AWS IAM | Least-privilege access policies for Lambda → Bedrock, Lambda → S3 |

---

## 5. Security & Privacy Design

### Data Flow Security
- **In transit:** All API calls over HTTPS/TLS 1.2+
- **At rest:** S3 default encryption (AES-256) for legal corpus
- **No PII storage:** Conversations are session-scoped and not persisted to any database
- **No authentication required:** Public access for maximum accessibility (no barriers for target users)

### AI Safety Controls
- **System prompt hardening:** All prompts include instructions to refuse harmful/illegal guidance
- **Source attribution mandate:** LLM is instructed to always cite Act & Section; responses without citations are flagged
- **Confidence thresholds:** If RAG retrieval score is below threshold, system responds with "I'm not confident — please consult a legal aid center" instead of generating potentially incorrect guidance
- **Input sanitization:** Lambda validates input length, strips HTML/scripts

---

## 6. Error Handling & Resilience

| Scenario | Handling |
|----------|----------|
| Bedrock API timeout | Retry once with exponential backoff; if still fails, show "Service temporarily unavailable, please try again" |
| RAG returns no relevant results | Respond: "I couldn't find specific legal provisions for your situation. Please describe it differently or contact your nearest DLSA for help." |
| User input is gibberish/empty | Respond: "Could you please describe your situation in Hindi or English? For example: 'My employer hasn't paid my wages for 3 months.'" |
| User asks about illegal activity | Respond: "I cannot provide guidance on this topic. If you are facing a legal issue, I'm here to help you understand your rights." |
| Lambda cold start | Acceptable for prototype (under 3 seconds); mitigate with provisioned concurrency in production |

---

## 7. Cost Estimation (Prototype Phase)

| Service | Usage Estimate | Monthly Cost |
|---------|---------------|--------------|
| Amazon Bedrock (Nova Lite) | ~1,000 requests, avg 2,000 tokens each | ~$2–5 |
| Bedrock Knowledge Bases | Storage + queries | ~$1–2 |
| Amazon S3 | < 100 MB legal corpus | < $0.01 |
| AWS Lambda | ~3,000 invocations | Free tier |
| API Gateway | ~3,000 requests | Free tier |
| EC2 (t3.micro) | 1 instance for Streamlit | Free tier |
| **Total** | | **~$3–7/month** |

---

## 8. Development & Deployment Plan

### Phase 1: Data Preparation (Day 1)
- Download 15 key Indian legal acts as PDFs from indiacode.nic.in
- Create simplified rights guides in Hindi (Markdown files)
- Compile DLSA directory JSON from nalsa.gov.in
- Upload all to S3 bucket
- Create and sync Bedrock Knowledge Base

### Phase 2: Core AI Pipeline (Day 1–2)
- Set up Lambda function with Bedrock API integration
- Implement prompt chain (Situation Analyzer → Rights Explainer → Document Drafter)
- Test with 10 sample scenarios covering domestic violence, wage theft, RTI, consumer complaints, and caste discrimination
- Tune prompts for accuracy and Hindi quality

### Phase 3: Frontend & Integration (Day 2–3)
- Build Streamlit chat interface
- Connect to Lambda via API Gateway
- Add rights cards, document panel, language toggle, disclaimer banner
- Mobile responsiveness testing
- Add DLSA finder feature

### Phase 4: Polish & Submit (Day 3)
- End-to-end testing with complete user journeys
- Record 3-minute video pitch demo
- Finalize presentation deck
- Deploy and verify live demo URL
- Submit to Hack2Skill dashboard

---

## 9. Key Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Legal inaccuracy / hallucination | High — could misguide vulnerable users | RAG with source attribution; confidence thresholds; prominent disclaimers; instruct LLM to say "I'm not sure" |
| Hindi response quality | Medium — poor Hindi reduces trust | Test with native Hindi speakers; use simplified Hindi guides in RAG corpus; prompt engineering for natural Hindi |
| Bedrock API availability | Medium — demo could fail | Prepare cached sample responses for demo scenarios as fallback |
| Legal act PDFs poorly parsed | Medium — chunking issues | Pre-test with Bedrock Knowledge Base; add simplified Markdown versions as backup |
| Scope creep | Medium — trying to cover all 1,800 laws | Strictly limit to 15 most impactful acts; explicitly state this boundary |

---

## 10. Differentiation from Existing Solutions

| Existing Solution | Limitation | Nyaya Saathi Advantage |
|-------------------|-----------|----------------------|
| **MyScheme / Tele-Law** | Government schemes only; English-heavy; requires digital literacy | Covers legal RIGHTS (not just schemes); Hindi-first; plain language |
| **LegalKart / Vakilsearch** | Paid services ($5–50); targeted at urban, educated users | Free; targets marginalized, low-literacy users |
| **Generic ChatGPT/Gemini** | No Indian legal specialization; no RAG over acts; hallucination risk; no source citation | RAG over actual Indian acts; always cites Act & Section; India-specific |
| **NALSA website** | Static information; English; no situation-based guidance | Conversational; Hindi; situation → rights → action flow |
| **NGO helplines (181, 1098)** | Phone-only; limited hours; depends on human availability | 24/7; text-based; scalable; generates documents |
