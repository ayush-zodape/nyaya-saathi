# Nyaya Saathi — Requirements Document

## Introduction

Nyaya Saathi (Justice Companion) is an AI-powered legal rights awareness and grievance drafting assistant designed to bridge the justice gap for marginalized communities in India. The system enables rural women, SC/ST communities, daily-wage workers, migrant laborers, and trafficking survivors to understand their legal rights and take action — in their own language, without needing a lawyer or expensive legal consultations.

The platform addresses a critical problem: while 80 million+ Indians are eligible for free legal aid, only 15 million access it. Target users often don't know their legal rights, cannot navigate 1,800+ central and state laws, and find legal language impenetrable. Nyaya Saathi provides conversational legal guidance in Hindi and English (with future regional language support), using Amazon Bedrock's AI capabilities with Retrieval-Augmented Generation (RAG) to match real-world situations to relevant legal acts and generate actionable documents.

### Target Track
- **Student Track — Problem Statement 3:** AI for Communities, Access & Public Impact

---

## Problem Statement

India has over 1,800 central and state laws, yet access to legal awareness remains critically low among marginalized communities. According to NALSA (National Legal Services Authority) data, approximately 80 million Indians are eligible for free legal aid, but only about 15 million actually access it. The gap exists because:

- **Legal language is impenetrable:** Acts and sections are written in complex English legal jargon that even educated urban Indians struggle with.
- **Lawyers are expensive:** A basic legal consultation costs ₹500–5,000, putting it out of reach for daily-wage workers earning ₹300–500/day.
- **Awareness is near-zero:** A rural woman facing domestic violence may not know that the Protection of Women from Domestic Violence Act (2005) gives her the right to reside in her marital home. A Dalit facing caste-based discrimination may not know the SC/ST Prevention of Atrocities Act protects them. A worker with unpaid wages may not know they can file an RTI to check their MGNREGA payment status.
- **Free legal aid is underutilized:** District Legal Services Authorities exist in every district but are severely underutilized due to lack of awareness and accessibility.
- **Filing complaints is daunting:** Even when people know their rights, drafting an FIR, RTI application, or formal complaint requires literacy and legal formatting knowledge they don't have.

---

## Why AI Is Needed (Not Just Digitization)

A static FAQ or searchable database cannot solve this problem because:

- **Contextual reasoning required:** Matching a person's messy, emotional, real-world description ("My husband beats me and my in-laws took my gold jewelry") to specific legal provisions across 1,800+ laws requires natural language understanding and multi-hop reasoning — not keyword search.
- **Combinatorial complexity:** A single situation may invoke multiple acts simultaneously (DV Act + IPC Section 498A + Dowry Prohibition Act). An AI must reason across these to provide comprehensive guidance.
- **Document generation:** Auto-drafting a personalized complaint letter or RTI application requires text generation capabilities that go beyond template filling — every situation is unique.
- **Language barriers:** Translating legal concepts (not just words) between English legal text and Hindi/regional languages requires semantic understanding, not dictionary lookup.

---

## Glossary

| Term | Definition |
|------|------------|
| **System** | The Nyaya Saathi platform |
| **User** | An individual seeking legal rights information or grievance drafting assistance |
| **Situation** | A user's description of their legal problem or concern in natural language |
| **Legal_Act** | Indian legislation (e.g., IPC, Domestic Violence Act, MGNREGA) |
| **Rights_Explanation** | Plain-language description of applicable legal rights at Class 8 reading level |
| **Grievance_Document** | Auto-generated complaint, FIR description, RTI application, or legal aid application |
| **DLSA** | District Legal Services Authority — provides free legal aid to eligible citizens |
| **RAG** | Retrieval-Augmented Generation — vector search over legal corpus to ground LLM responses |
| **Intent** | Classified user request type: `new_situation`, `follow_up`, `document_request`, `legal_aid_search`, `out_of_scope` |
| **Knowledge_Base** | Amazon Bedrock Knowledge Base containing 15+ Indian legal act PDFs indexed via Amazon Titan Embeddings V2 |
| **LLM** | Large Language Model — Amazon Nova Lite or Claude 3 Sonnet via Amazon Bedrock |
| **Session** | A temporary conversation context that does not persist PII beyond the interaction |
| **Source_Attribution** | Specific Act name and Section number cited in every response for independent verification |
| **Confidence_Score** | AI's certainty level in rights matching — low confidence triggers explicit uncertainty disclosure and DLSA referral |

---

## Requirements

### Requirement 1: Multilingual Conversational Intake

**User Story:** As a user with limited English proficiency, I want to describe my legal situation in Hindi, English, or Hinglish using text, so that I can access legal guidance without language barriers.

#### Acceptance Criteria

1. WHEN a user submits text input in Hindi (e.g., "Mere pati mujhe marte hain aur sasural waalon ne mera zewar le liya"), THEN THE System SHALL process and understand the input accurately
2. WHEN a user submits text input in English (e.g., "My employer hasn't paid my wages for 3 months"), THEN THE System SHALL process and understand the input accurately
3. WHEN a user submits text input in Hinglish (mixed Hindi-English), THEN THE System SHALL process and understand the input without requiring the user to switch languages
4. WHEN the System processes user input, THEN THE System SHALL ask clarifying follow-up questions if needed (e.g., "Are you currently living in your marital home?" or "Is your employer a government contractor or private?")
5. WHEN the input field is displayed, THEN THE System SHALL support text input of up to 2,000 characters per message
6. WHEN the input field is displayed, THEN THE System SHALL provide clear instructions in both Hindi and English for how to describe their situation

> **Future Enhancement:** Voice input via Amazon Transcribe for speech-to-text, enabling fully voice-based interaction for illiterate users.

---

### Requirement 2: Intent Classification

**User Story:** As the system, I want to classify user requests into specific intent categories, so that I can route them to appropriate processing workflows.

#### Acceptance Criteria

1. WHEN a user describes a new legal situation, THEN THE System SHALL classify the intent as `new_situation`
2. WHEN a user asks follow-up questions about a previous response, THEN THE System SHALL classify the intent as `follow_up`
3. WHEN a user requests document generation (RTI, FIR, complaint letter, legal aid application), THEN THE System SHALL classify the intent as `document_request`
4. WHEN a user asks for legal aid center information, THEN THE System SHALL classify the intent as `legal_aid_search`
5. WHEN a user request is outside legal assistance scope, THEN THE System SHALL classify the intent as `out_of_scope` and respond helpfully
6. WHEN intent classification confidence is below threshold, THEN THE System SHALL ask clarifying questions rather than making assumptions

---

### Requirement 3: Situation-to-Rights Mapping Using RAG

**User Story:** As a user describing my legal problem, I want the system to identify which laws apply to my situation, so that I can understand my legal rights.

#### Acceptance Criteria

1. WHEN a user describes a situation, THEN THE System SHALL query the Knowledge_Base using RAG to retrieve relevant legal provisions
2. WHEN querying the Knowledge_Base, THEN THE System SHALL use Amazon Titan Embeddings V2 to generate query embeddings for semantic matching
3. WHEN retrieving legal provisions, THEN THE System SHALL search across all 15+ Indian legal acts in the Knowledge_Base (see Requirement 6 for full list)
4. WHEN multiple legal acts are relevant to a single situation (e.g., DV Act + IPC Section 498A + Dowry Prohibition Act for a domestic violence case), THEN THE System SHALL return provisions from all applicable acts with cross-references
5. WHEN RAG retrieval returns results, THEN THE System SHALL include Source_Attribution with specific Act name and Section number for each identified right
6. WHEN RAG retrieval Confidence_Score is low, THEN THE System SHALL explicitly indicate uncertainty and suggest DLSA consultation instead of guessing
7. WHEN rights identification is complete, THEN THE System SHALL return the response within 15 seconds

---

### Requirement 4: Plain-Language Rights Explanation

**User Story:** As a user with Class 8 education level, I want legal rights explained in simple Hindi or English, so that I can understand my rights without legal jargon.

#### Acceptance Criteria

1. WHEN generating a Rights_Explanation, THEN THE System SHALL use vocabulary appropriate for Class 8 reading level (simple sentence structure, common words)
2. WHEN generating a Rights_Explanation in Hindi, THEN THE System SHALL use simple Hindi words, avoiding English legal terms wherever possible
3. WHEN explaining legal provisions, THEN THE System SHALL provide concrete examples relevant to the user's specific situation
4. WHEN multiple rights apply, THEN THE System SHALL organize the explanation with clear section headings and numbered points
5. WHEN citing legal provisions, THEN THE System SHALL include Source_Attribution in parentheses after each right (e.g., "Protection of Women from Domestic Violence Act 2005, Section 12")
6. WHEN the explanation is complete, THEN THE System SHALL include the disclaimer: "यह जानकारी केवल मार्गदर्शन के लिए है, यह कानूनी सलाह नहीं है। कृपया किसी वकील या जिला विधिक सेवा प्राधिकरण से संपर्क करें।"
7. WHEN explaining each right, THEN THE System SHALL cover three aspects: what the right means in practical terms, what the user can do about it, and what protection it offers

> **Example:** Instead of "Section 17 of PWDVA confers right of residence," the system says: "कानून कहता है कि आपको अपने ससुराल के घर में रहने का पूरा अधिकार है। कोई भी आपको घर से निकाल नहीं सकता।" (The law says you have full right to live in your marital home. No one can throw you out.)

---

### Requirement 5: Auto-Generation of Grievance Documents

**User Story:** As a user who needs to file a complaint, I want the system to generate the required document for me, so that I can submit it to authorities without hiring a lawyer.

#### Acceptance Criteria

1. WHEN a user requests an RTI application, THEN THE System SHALL generate a properly formatted RTI application per RTI Act requirements with proper addressing, subject line, and specific information sought
2. WHEN a user requests an FIR description, THEN THE System SHALL generate a structured narrative suitable for police complaint registration, including relevant IPC/BNS sections
3. WHEN a user requests a complaint letter, THEN THE System SHALL generate a formal complaint letter addressed to the appropriate authority (District Collector, Labor Commissioner, Women's Commission, etc.)
4. WHEN a user requests a legal aid application, THEN THE System SHALL generate a DLSA legal aid application form with eligibility justification
5. WHEN generating any Grievance_Document, THEN THE System SHALL include:
   - Proper formatting (address, date, subject line, body, signature block)
   - All relevant facts from the user's described situation
   - Applicable Legal_Acts and section numbers
   - Placeholder markers for personal details clearly marked as `[YOUR NAME]`, `[YOUR ADDRESS]`, `[YOUR AADHAAR NUMBER]`, etc.
6. WHEN a Grievance_Document is generated, THEN THE System SHALL provide it in the user's chosen language (Hindi or English)
7. WHEN a Grievance_Document is generated, THEN THE System SHALL provide step-by-step submission instructions (where to go, what to carry, what to expect)
8. WHEN a Grievance_Document is generated, THEN THE System SHALL include the disclaimer: "This is guidance, not legal advice"
9. WHEN a Grievance_Document is generated, THEN THE System SHALL allow the user to copy the text to clipboard or download it

---

### Requirement 6: Legal Acts Coverage

**User Story:** As a user with a legal issue, I want the system to cover major Indian laws relevant to marginalized communities, so that I can get guidance on common legal problems.

#### Acceptance Criteria

1. WHEN the Knowledge_Base is initialized, THEN THE System SHALL include the following 15+ legal acts:

| # | Legal Act | Key Coverage Area |
|---|-----------|-------------------|
| 1 | Indian Penal Code (IPC) / Bharatiya Nyaya Sanhita (BNS) | Criminal offenses |
| 2 | Code of Criminal Procedure (CrPC) / Bharatiya Nagarik Suraksha Sanhita (BNSS) | Criminal procedure |
| 3 | Protection of Women from Domestic Violence Act, 2005 | Domestic violence |
| 4 | Dowry Prohibition Act, 1961 | Dowry harassment |
| 5 | SC/ST (Prevention of Atrocities) Act, 1989 | Caste-based discrimination |
| 6 | Right to Information Act, 2005 | Government transparency |
| 7 | Mahatma Gandhi NREGA (MGNREGA), 2005 | Rural employment guarantee |
| 8 | Consumer Protection Act, 2019 | Consumer rights |
| 9 | Motor Vehicles Act, 1988 | Traffic challan grievances |
| 10 | Payment of Wages Act, 1936 | Wage disputes |
| 11 | Minimum Wages Act, 1948 | Minimum wage enforcement |
| 12 | Maintenance and Welfare of Parents and Senior Citizens Act, 2007 | Elder care |
| 13 | Protection of Children from Sexual Offences (POCSO) Act, 2012 | Child sexual abuse |
| 14 | Right to Education Act, 2009 | Education access |
| 15 | Legal Services Authorities Act, 1987 | Free legal aid eligibility |

2. WHEN all legal act PDFs are collected, THEN THE System SHALL store them in Amazon S3 with organized folder structure
3. WHEN the Knowledge_Base indexes new acts, THEN THE System SHALL use Amazon Titan Embeddings V2 for chunking and vector embedding
4. WHEN a user's situation spans multiple acts, THEN THE System SHALL cross-reference provisions across all applicable acts

---

### Requirement 7: Nearest Free Legal Aid Center Finder

**User Story:** As a user who needs in-person legal assistance, I want to find the nearest free legal aid center, so that I can access professional help without cost.

#### Acceptance Criteria

1. WHEN a user requests legal aid center information, THEN THE System SHALL query the DLSA directory
2. WHEN querying the DLSA directory, THEN THE System SHALL use the user's specified state and district
3. WHEN no location is provided, THEN THE System SHALL ask the user for their district or state
4. WHEN DLSA centers are found, THEN THE System SHALL return up to 3 nearest centers with: name, address, phone number, and operating hours
5. WHEN displaying DLSA information, THEN THE System SHALL explain who is eligible for free legal aid:
   - Women (all categories)
   - Members of SC/ST communities
   - Persons with disabilities
   - Victims of trafficking
   - Industrial workers
   - Persons in custody
   - Persons below the income threshold set by the state government
6. WHEN the DLSA directory is initialized, THEN THE System SHALL store the directory data in Amazon S3

---

### Requirement 8: Prompt Chaining Workflow

**User Story:** As the system, I want to process user requests through a multi-stage pipeline, so that I can provide comprehensive and accurate legal guidance through structured reasoning.

#### Acceptance Criteria

1. WHEN a `new_situation` intent is detected, THEN THE System SHALL execute Stage 1: Situation Analyzer (extract facts, identify legal domains)
2. WHEN Stage 1 completes, THEN THE System SHALL execute Stage 2: Rights Explainer (RAG retrieval + plain-language explanation)
3. WHEN Stage 2 completes, THEN THE System SHALL execute Stage 3: Action Guide (step-by-step guidance + DLSA information)
4. WHEN a `document_request` intent is detected, THEN THE System SHALL execute the Document Drafter stage using context from previous stages
5. WHEN a `follow_up` intent is detected, THEN THE System SHALL use the current Session context to provide continuity
6. WHEN any stage fails, THEN THE System SHALL provide a graceful error message and suggest DLSA consultation as fallback

---

### Requirement 9: Responsible AI and Safety Guardrails

**User Story:** As a system operator, I want the AI to refuse inappropriate requests and avoid harmful outputs, so that the system is used ethically and responsibly.

#### Acceptance Criteria

1. WHEN a user requests guidance on illegal activities, THEN THE System SHALL refuse and explain that it cannot assist with illegal actions
2. WHEN a user asks for case outcome predictions or sentencing predictions, THEN THE System SHALL refuse and explain that outcomes cannot be predicted
3. WHEN generating responses, THEN THE System SHALL avoid bias in coverage across different communities (SC/ST, women, workers, minorities)
4. WHEN a user describes a situation requiring immediate safety intervention (e.g., life-threatening domestic violence, child abuse), THEN THE System SHALL provide emergency contact information:
   - Police: 100
   - Women Helpline: 181
   - Child Helpline: 1098
   - National Commission for Women: 7827-170-170
5. WHEN Confidence_Score is low for rights matching, THEN THE System SHALL explicitly state uncertainty and recommend DLSA consultation
6. WHEN any response is generated, THEN THE System SHALL include the prominent disclaimer in both Hindi and English: "यह जानकारी केवल मार्गदर्शन के लिए है, यह कानूनी सलाह नहीं है। कृपया किसी वकील या जिला विधिक सेवा प्राधिकरण से संपर्क करें।" / "This information is for guidance only, it is not legal advice. Please contact a lawyer or District Legal Services Authority."
7. WHEN a situation is too complex or ambiguous for confident guidance, THEN THE System SHALL acknowledge limitations and recommend consulting a lawyer in person

---

### Requirement 10: Session Management and Privacy

**User Story:** As a user sharing sensitive legal information, I want my personal data to be protected and not stored permanently, so that my privacy is maintained.

#### Acceptance Criteria

1. WHEN a user starts a conversation, THEN THE System SHALL create a temporary Session with a unique identifier
2. WHEN a Session is active, THEN THE System SHALL maintain conversation context for follow-up questions within the same session
3. WHEN a Session ends (browser close or timeout), THEN THE System SHALL delete all conversation data including user situation details
4. THE System SHALL NOT store personally identifiable information (PII) in persistent storage at any time
5. THE System SHALL NOT store user names, addresses, phone numbers, Aadhaar numbers, or case details beyond the active Session
6. WHEN communicating with backend services, THEN THE System SHALL use HTTPS encryption for all API calls
7. THE System SHALL NOT require login or registration for basic use — access is anonymous
8. WHEN using AWS services, THEN THE System SHALL enforce IAM-based access control with least-privilege policies

---

### Requirement 11: Mobile-Responsive Web Interface

**User Story:** As a user accessing the system from a budget smartphone on a 2G/3G connection, I want the interface to work well on small screens with limited bandwidth, so that I can use it on my device without frustration.

#### Acceptance Criteria

1. WHEN the web interface loads on a mobile device, THEN THE System SHALL display a responsive layout optimized for screens as small as 320px width
2. WHEN the web interface loads, THEN THE System SHALL complete initial page load in under 500 KB total (HTML + CSS + JS)
3. WHEN the web interface displays text, THEN THE System SHALL use readable font sizes (minimum 16px body text) for users with limited vision
4. WHEN the web interface displays input fields/buttons, THEN THE System SHALL make them easily tappable on touchscreens (minimum 44px height)
5. WHEN the web interface loads on slow connections, THEN THE System SHALL function on 2G/3G networks with a text-first design (minimal images and JavaScript)
6. WHEN the web interface displays content, THEN THE System SHALL use high-contrast text for outdoor/bright-light readability

---

### Requirement 12: Performance and Response Time

**User Story:** As a user with limited patience and intermittent connectivity, I want the system to respond quickly, so that I can get answers without long waits.

#### Acceptance Criteria

1. WHEN a user submits a situation for rights identification, THEN THE System SHALL return a response within 15 seconds
2. WHEN a user requests document generation, THEN THE System SHALL return the generated document within 30 seconds
3. WHEN a user queries for legal aid centers, THEN THE System SHALL return results within 5 seconds
4. WHEN the Knowledge_Base is queried via RAG, THEN THE System SHALL return retrieval results within 10 seconds
5. WHEN the System experiences high load, THEN THE System SHALL maintain response times within acceptable limits using AWS Lambda auto-scaling
6. WHEN the system is under normal load, THEN THE System SHALL support at least 10 simultaneous user sessions

---

### Requirement 13: Error Handling and Graceful Degradation

**User Story:** As a user encountering system errors, I want clear error messages in my language and alternative options, so that I can still get help even when something goes wrong.

#### Acceptance Criteria

1. WHEN the LLM service (Amazon Bedrock) is unavailable, THEN THE System SHALL display an error message in Hindi explaining the issue and suggesting to try again later
2. WHEN the Knowledge_Base query fails, THEN THE System SHALL suggest contacting the nearest DLSA for assistance (with contact info)
3. WHEN document generation fails, THEN THE System SHALL provide a template outline that the user can fill manually
4. WHEN any error occurs, THEN THE System SHALL log the error for debugging without exposing technical details to the user
5. WHEN the System cannot understand user input after 2 clarification attempts, THEN THE System SHALL provide DLSA contact information as a fallback
6. WHEN RAG retrieval returns low-confidence results, THEN THE System SHALL state "I'm not confident about this — please consult a legal aid center" rather than guessing

---

### Requirement 14: Source Attribution and Transparency

**User Story:** As a user receiving legal guidance, I want to know which specific laws and sections apply to my situation, so that I can verify the information independently and cite it when needed.

#### Acceptance Criteria

1. WHEN the System provides a Rights_Explanation, THEN THE System SHALL cite the specific Legal_Act name for each identified right
2. WHEN the System provides a Rights_Explanation, THEN THE System SHALL cite the specific section number for each identified right
3. WHEN the System generates a Grievance_Document, THEN THE System SHALL include relevant Legal_Act names and section numbers in the document body
4. WHEN multiple sources support a single right, THEN THE System SHALL list all applicable sources
5. WHEN the System cites a source, THEN THE System SHALL format it clearly and consistently (e.g., "Protection of Women from Domestic Violence Act 2005, Section 12")
6. WHEN the System retrieves information via RAG, THEN THE System SHALL only present information that has been grounded in the retrieved legal text (no hallucinated sections)

---

### Requirement 15: Backend Architecture and AWS Integration

**User Story:** As a system architect, I want the backend to use AWS serverless services, so that the system is scalable, cost-effective, and maintainable for deployment.

#### Acceptance Criteria

1. WHEN a user request is received from the Streamlit frontend, THEN THE System SHALL route it through Amazon API Gateway (REST API)
2. WHEN API Gateway receives a request, THEN THE System SHALL invoke the appropriate AWS Lambda function (Python 3.12 runtime)
3. WHEN Lambda functions need AI capabilities, THEN THE System SHALL invoke Amazon Bedrock (Nova Lite for intake/classification, Claude 3 Sonnet for rights explanation and document generation)
4. WHEN Lambda functions need RAG capabilities, THEN THE System SHALL query Amazon Bedrock Knowledge Bases backed by Amazon OpenSearch Serverless as the vector store
5. WHEN Knowledge Bases perform vector search, THEN THE System SHALL use Amazon Titan Embeddings V2 for embedding generation
6. WHEN the System needs to store legal corpus PDFs and DLSA directory data, THEN THE System SHALL use Amazon S3
7. WHEN the Streamlit frontend is deployed, THEN THE System SHALL host it on Amazon EC2 (t3.micro instance for prototype)
8. WHEN Lambda functions execute, THEN THE System SHALL complete within AWS Lambda timeout limits (configured at 60 seconds, max 15 minutes)

---

## Additional Non-Functional Requirements

### Language Support
- **Primary:** Hindi and English
- **Mixed-language:** Hinglish (mixed Hindi-English) input handling without requiring language switching
- **Interface:** All UI labels, system messages, error messages, and disclaimers available in both Hindi and English
- **Extensibility:** Architecture designed to support additional Indian languages (Tamil, Telugu, Marathi, Bengali, Gujarati, Kannada) via Amazon Translate + Bedrock in future phases

### Cost Efficiency
- Uses Amazon Bedrock **pay-per-use** pricing — no dedicated infrastructure or reserved instances needed
- Optimized prompt engineering to minimize token usage per request
- Knowledge Base uses Amazon S3 (pennies per GB) for document storage
- AWS Lambda billed per invocation — zero cost when idle
- Estimated prototype cost: Under $5/month for hackathon demo usage

### Reliability
- System uptime target: **99% during hackathon demo**
- Graceful error handling: If Bedrock API fails, system shows a helpful error message in Hindi (not a crash or stack trace)
- Fallback chain: LLM failure → cached common responses → DLSA referral
- All AWS services selected for high availability within `ap-south-1` (Mumbai) region

---

## User Personas

### Persona 1: Sunita — Rural Woman Facing Domestic Violence
- **Age:** 28 | **Location:** Village in Uttar Pradesh | **Education:** Class 8 | **Language:** Hindi
- **Context:** Sunita's husband beats her regularly. Her in-laws took her gold jewelry (stridhan). She doesn't know she has legal protections or how to get help. She has never spoken to a lawyer.
- **Needs:** Understand her rights under DV Act and Dowry Prohibition Act, get help filing a complaint, find free legal aid at the nearest DLSA
- **Tech Access:** Husband's old Android smartphone, intermittent 2G/3G connectivity
- **Success Scenario:** Sunita types "Mere pati mujhe marte hain aur sasural waalon ne mera zewar le liya" → Nyaya Saathi explains her rights under PWDVA Section 17 (right to residence) and Section 12 (protection order), provides step-by-step guidance to file a complaint, and generates a draft complaint letter addressed to the Protection Officer.

### Persona 2: Ramesh — Daily-Wage Worker with Unpaid Wages
- **Age:** 35 | **Location:** Semi-urban town in Jharkhand | **Education:** Class 10 | **Language:** Hindi
- **Context:** Ramesh worked under MGNREGA for 45 days but received payment for only 20 days. He doesn't know he can file an RTI to check payment records or complain to the block office.
- **Needs:** Understand MGNREGA entitlements, draft an RTI application to check his payment records, know where and how to complain
- **Tech Access:** Basic Android smartphone, Common Service Centre (CSC) access in nearby town
- **Success Scenario:** Ramesh types "MGNREGA mein 45 din kaam kiya lekin 20 din ka hi paisa mila" → Nyaya Saathi explains his right to full payment under MGNREGA Section 3, generates a ready-to-submit RTI application addressed to the Block Development Officer, and provides step-by-step instructions for submission.

### Persona 3: Kavita — NGO Fieldworker
- **Age:** 32 | **Location:** Operates across rural Rajasthan | **Education:** Graduate | **Language:** Hindi + English
- **Context:** Kavita works with an NGO helping trafficking survivors and child marriage victims. She needs to quickly identify applicable laws and draft complaints for multiple people every week.
- **Needs:** Quick legal reference tool across multiple acts, efficient document drafting for varied scenarios, reliable source citations for court filings
- **Tech Access:** Smartphone with 4G, laptop at office
- **Success Scenario:** Kavita uses Nyaya Saathi to rapidly identify applicable provisions under POCSO, IPC Sections 370-373 (trafficking), and the Prohibition of Child Marriage Act for each case, then generates tailored FIR descriptions and complaint letters — cutting her legal research time from hours to minutes.

---

## Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| **Comprehensiveness** | System correctly identifies applicable legal provisions for ≥80% of common scenarios | Test with 20+ scenarios across DV, wage theft, caste discrimination, consumer complaints, property disputes |
| **Usability** | Non-technical user receives actionable output within 3 messages | User testing with target personas |
| **Document Quality** | Auto-generated documents require <20% manual editing before submission | Expert review by legal aid professionals |
| **Response Time** | Rights identification <15s, document generation <30s | Automated performance testing |
| **User Satisfaction** | Target users report that explanations are understandable and helpful | Feedback survey during demo |

---

## Scope Boundaries (What This Project Is NOT)

- **Not a replacement for lawyers:** This is a legal awareness and guidance tool, not legal representation. It does not appear in court or negotiate on behalf of users.
- **Not a case management system:** Does not track ongoing court cases, hearing dates, or legal proceedings.
- **Not a legal database search engine:** Users don't search by Act/Section — they describe situations in plain language and the AI identifies applicable laws.
- **Not real-time legal advice:** System provides general legal information based on central legislation, not situation-specific legal opinions accounting for all jurisdictional variations.
- **Not a replacement for DLSA:** When situations are complex, the system actively refers users to District Legal Services Authorities for in-person professional assistance.

---

## Future Roadmap (Post-Hackathon)

| Phase | Enhancement | Description |
|-------|-------------|-------------|
| **Phase 1** | Voice Input/Output | Integrate Amazon Transcribe (speech-to-text) and Amazon Polly (text-to-speech) for fully voice-based interaction — enabling illiterate users to access the system |
| **Phase 2** | Regional Languages | Expand to Tamil, Telugu, Marathi, Bengali, Gujarati, Kannada using Amazon Translate + Bedrock multilingual capabilities |
| **Phase 3** | State-Specific Laws | Add state-specific acts and rules (Maharashtra Rent Control, UP Revenue Code, etc.) beyond current central legislation focus |
| **Phase 4** | CSC Integration | Deploy as a tool for Common Service Centre (CSC) operators who assist villagers with government services |
| **Phase 5** | NGO Dashboard | Analytics dashboard for NGOs to track common legal issues in their operational areas and generate impact reports |
| **Phase 6** | Offline Mode | Cached common scenarios and rights information for zero-connectivity use in remote areas |
| **Phase 7** | WhatsApp Integration | Reach users on the messaging platform they already use daily — no app installation required |
