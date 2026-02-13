# Nyaya Saathi — Requirements Document

## Project Overview

### Problem Statement
India has over 1,800 central and state laws, yet access to legal awareness remains critically low among marginalized communities. According to NALSA (National Legal Services Authority) data, approximately 80 million Indians are eligible for free legal aid, but only about 15 million actually access it. The gap exists because:

- **Legal language is impenetrable:** Acts and sections are written in complex English legal jargon that even educated urban Indians struggle with.
- **Lawyers are expensive:** A basic legal consultation costs Rs 500–5,000, putting it out of reach for daily-wage workers earning Rs 300–500/day.
- **Awareness is near-zero:** A rural woman facing domestic violence may not know that the Protection of Women from Domestic Violence Act (2005) gives her the right to reside in her marital home. A Dalit facing caste-based discrimination may not know the SC/ST Prevention of Atrocities Act protects them. A worker with unpaid wages may not know they can file an RTI to check their MGNREGA payment status.
- **Free legal aid is underutilized:** District Legal Services Authorities exist in every district but are severely underutilized due to lack of awareness and accessibility.
- **Filing complaints is daunting:** Even when people know their rights, drafting an FIR, RTI application, or formal complaint requires literacy and legal formatting knowledge they don't have.

### Solution Summary
**Nyaya Saathi** (Justice Companion) is an AI-powered legal rights awareness and grievance drafting assistant that helps marginalized communities in India understand their legal rights and take action — in their own language, without needing a lawyer.

The system uses Amazon Bedrock's large language models with Retrieval-Augmented Generation (RAG) over Indian legal acts to:
1. Understand a user's situation described in plain Hindi/English
2. Identify applicable legal rights and provisions
3. Generate actionable outputs: plain-language rights explanations, step-by-step complaint filing guides, and auto-drafted legal documents (RTI applications, FIR descriptions, formal complaints)

### Target Track
- **Student Track — Problem Statement 3:** AI for Communities, Access & Public Impact

### Why AI Is Needed (Not Just Digitization)
A static FAQ or searchable database cannot solve this problem because:
- **Contextual reasoning required:** Matching a person's messy, emotional, real-world description ("My husband beats me and my in-laws took my gold jewelry") to specific legal provisions across 1,800+ laws requires natural language understanding and multi-hop reasoning — not keyword search.
- **Combinatorial complexity:** A single situation may invoke multiple acts simultaneously (DV Act + IPC Section 498A + Dowry Prohibition Act). An AI must reason across these to provide comprehensive guidance.
- **Document generation:** Auto-drafting a personalized complaint letter or RTI application requires text generation capabilities that go beyond template filling — every situation is unique.
- **Language barriers:** Translating legal concepts (not just words) between English legal text and Hindi/regional languages requires semantic understanding, not dictionary lookup.

---

## Functional Requirements

### FR-1: Conversational Situation Intake
**Description:** The system shall allow users to describe their legal situation in natural language (Hindi or English) through a text-based chat interface.

**Acceptance Criteria:**
- User can type a description of their problem in Hindi or English (e.g., "Mere pati mujhe marte hain aur sasural waalon ne mera zewar le liya" or "My employer hasn't paid my wages for 3 months")
- System acknowledges the input and asks clarifying follow-up questions if needed (e.g., "Are you currently living in your marital home?" or "Is your employer a government contractor or private?")
- System handles informal language, colloquialisms, and mixed Hindi-English (Hinglish) inputs
- Input field supports text of up to 2,000 characters per message

### FR-2: Legal Rights Identification via RAG
**Description:** The system shall use Retrieval-Augmented Generation (RAG) over a curated corpus of Indian legal acts to identify all applicable legal rights and provisions relevant to the user's described situation.

**Acceptance Criteria:**
- System maintains a knowledge base containing at least 15 major Indian legal acts including:
  - Indian Penal Code (IPC) / Bharatiya Nyaya Sanhita (BNS)
  - Code of Criminal Procedure (CrPC) / Bharatiya Nagarik Suraksha Sanhita (BNSS)
  - Protection of Women from Domestic Violence Act, 2005
  - Dowry Prohibition Act, 1961
  - SC/ST (Prevention of Atrocities) Act, 1989
  - Right to Information Act, 2005
  - Mahatma Gandhi National Rural Employment Guarantee Act (MGNREGA), 2005
  - Consumer Protection Act, 2019
  - Motor Vehicles Act, 1988 (for traffic challan grievances)
  - Payment of Wages Act, 1936
  - Minimum Wages Act, 1948
  - Maintenance and Welfare of Parents and Senior Citizens Act, 2007
  - Protection of Children from Sexual Offences (POCSO) Act, 2012
  - Right to Education Act, 2009
  - Legal Services Authorities Act, 1987
- For any given user situation, system identifies applicable legal provisions with specific Act name, Section number, and relevance explanation
- System retrieves and cites source passages from the actual legal text (source attribution)
- Response time for rights identification is under 15 seconds

### FR-3: Plain-Language Rights Explanation
**Description:** The system shall explain identified legal rights in simple, non-legal language that a person with basic literacy can understand, in Hindi or English.

**Acceptance Criteria:**
- Every legal right identified is explained in 2–4 simple sentences at a Class 8 reading level
- Explanations include: what the right means in practical terms, what the user can do about it, and what protection it offers
- Example: Instead of "Section 17 of PWDVA confers right of residence," the system says: "कानून कहता है कि आपको अपने ससुराल के घर में रहने का पूरा अधिकार है। कोई भी आपको घर से निकाल नहीं सकता।" (The law says you have full right to live in your marital home. No one can throw you out.)
- Each rights explanation cites the specific Act and Section number for verification

### FR-4: Actionable Step-by-Step Guidance
**Description:** For each identified right, the system shall provide a clear, numbered, step-by-step guide on how the user can take action.

**Acceptance Criteria:**
- Steps include: where to go (nearest police station, District Legal Services Authority, block office, etc.), what to say, what documents to carry, what to expect, and approximate timelines
- Steps are sequenced logically (e.g., "Step 1: Go to the nearest police station with your Aadhaar card. Step 2: Tell the officer on duty that you want to file an FIR under Section 498A...")
- System provides information about free legal aid availability under the Legal Services Authorities Act
- System specifies which authority/office to approach (police, magistrate, labor commissioner, etc.) based on the specific legal remedy

### FR-5: Auto-Drafted Legal Documents
**Description:** The system shall automatically generate draft legal documents relevant to the user's situation, ready for submission with minimal modification.

**Acceptance Criteria:**
- System can generate the following document types:
  - **RTI Application:** Formatted per RTI Act requirements with proper addressing, subject, and specific information sought
  - **FIR Description:** A structured narrative suitable for police complaint registration
  - **Formal Complaint Letter:** Addressed to appropriate authority (District Collector, Labor Commissioner, Women's Commission, etc.)
  - **Legal Aid Application:** Application to District Legal Services Authority for free legal representation
- Each generated document:
  - Has proper formatting (address, date, subject line, body, signature block)
  - Is in Hindi or English (user's choice)
  - Has placeholder markers for personal details (name, address, Aadhaar number) clearly marked as [YOUR NAME], [YOUR ADDRESS], etc.
  - Includes the relevant legal provisions and sections
- Documents can be copied to clipboard or downloaded as text files

### FR-6: Nearest Free Legal Aid Center Finder
**Description:** The system shall help users locate the nearest free legal aid center based on their district or state.

**Acceptance Criteria:**
- System maintains a directory of District Legal Services Authorities (DLSAs) across India
- User can specify their state and district to find the nearest DLSA
- Information provided includes: DLSA name, address, phone number, and eligibility criteria for free legal aid
- System explains who is eligible for free legal aid (women, SC/ST, persons with disabilities, victims of trafficking, industrial workers, persons in custody, those below income threshold, etc.)

### FR-7: Responsible AI Safeguards
**Description:** The system shall implement responsible AI practices including clear disclaimers, source attribution, and safety measures.

**Acceptance Criteria:**
- Every response includes a prominent disclaimer: "यह जानकारी केवल मार्गदर्शन के लिए है, यह कानूनी सलाह नहीं है। कृपया किसी वकील या जिला विधिक सेवा प्राधिकरण से संपर्क करें।" (This information is for guidance only, it is not legal advice. Please contact a lawyer or District Legal Services Authority.)
- All legal citations include Act name and Section number for independent verification
- System refuses to provide guidance on illegal activities
- System does not store any personally identifiable information (PII) from user conversations
- System acknowledges limitations when a situation is too complex or ambiguous and recommends consulting a lawyer
- System does not provide sentencing predictions or case outcome guarantees

---

## Non-Functional Requirements

### NFR-1: Performance
- Response generation time: Under 15 seconds for rights identification, under 30 seconds for document generation
- Concurrent users supported: At least 10 simultaneous sessions

### NFR-2: Accessibility & Inclusiveness
- Mobile-responsive web interface (majority of target users access internet via smartphones)
- Low-bandwidth optimized: Text-first interface, minimal images/JavaScript, total page load under 500 KB
- Readable font sizes (minimum 16px) for users with limited vision
- High contrast text for outdoor/bright light readability

### NFR-3: Language Support
- Primary: Hindi and English
- Interface labels and system messages available in both Hindi and English
- Mixed-language (Hinglish) input handling
- Future extensibility: Architecture supports adding more Indian languages

### NFR-4: Data Privacy & Security
- No user PII stored permanently — conversations are session-based
- No login or registration required for basic use
- Legal act corpus is public domain data (from indiacode.nic.in)
- All API communications over HTTPS
- AWS IAM-based access control for backend services

### NFR-5: Reliability
- System uptime target: 99% during hackathon demo
- Graceful error handling: If Bedrock API fails, system shows a helpful error message (not a crash)
- Fallback: If RAG retrieval returns low-confidence results, system states "I'm not confident about this — please consult a legal aid center" rather than guessing

### NFR-6: Cost Efficiency
- Uses Amazon Bedrock pay-per-use pricing (no dedicated infrastructure)
- Optimized prompt engineering to minimize token usage
- Knowledge Base uses Amazon S3 (pennies per GB) for document storage

---

## User Personas

### Persona 1: Sunita — Rural Woman Facing Domestic Violence
- **Age:** 28 | **Location:** Village in Uttar Pradesh | **Education:** Class 8 | **Language:** Hindi
- **Context:** Sunita's husband beats her regularly. Her in-laws took her gold jewelry (stridhan). She doesn't know she has legal protections or how to get help.
- **Needs:** Understand her rights under DV Act, get help filing a complaint, find free legal aid
- **Tech access:** Husband's old smartphone, intermittent 2G/3G connectivity

### Persona 2: Ramesh — Daily-Wage Worker with Unpaid Wages
- **Age:** 35 | **Location:** Semi-urban town in Jharkhand | **Education:** Class 10 | **Language:** Hindi
- **Context:** Ramesh worked under MGNREGA for 45 days but received payment for only 20 days. He doesn't know he can file an RTI to check payment records or complain to the block office.
- **Needs:** Understand MGNREGA entitlements, draft an RTI application, know where to complain
- **Tech access:** Basic Android smartphone, CSC (Common Service Centre) access

### Persona 3: Kavita — NGO Fieldworker
- **Age:** 32 | **Location:** Operates across rural Rajasthan | **Education:** Graduate | **Language:** Hindi + English
- **Context:** Kavita works with an NGO helping trafficking survivors and child marriage victims. She needs to quickly identify applicable laws and draft complaints for multiple people every week.
- **Needs:** Quick legal reference tool, bulk document drafting, multi-scenario handling
- **Tech access:** Smartphone with 4G, laptop at office

---

## Success Metrics
- **Comprehensiveness:** System correctly identifies applicable legal provisions for at least 80% of common scenarios (domestic violence, wage theft, caste discrimination, consumer complaints, property disputes)
- **Usability:** A non-technical user can describe their situation and receive actionable output within 3 interactions (messages)
- **Document Quality:** Auto-generated legal documents require less than 20% manual editing before submission
- **User Satisfaction:** Target users report that the explanation was understandable and helpful

---

## Scope Boundaries (What This Project Is NOT)
- **Not a replacement for lawyers:** This is a legal awareness and guidance tool, not legal representation
- **Not a case management system:** Does not track ongoing court cases or legal proceedings
- **Not a legal database search engine:** Users don't search by Act/Section — they describe situations in plain language
- **Not real-time legal advice:** System provides general legal information, not situation-specific legal opinions accounting for all jurisdictional variations

---

## Future Roadmap (Post-Hackathon)
1. **Voice Input/Output:** Integrate Amazon Transcribe for speech-to-text and Amazon Polly for text-to-speech, enabling fully voice-based interaction for illiterate users
2. **Regional Languages:** Expand to Tamil, Telugu, Marathi, Bengali, Gujarati, Kannada using Amazon Translate + Bedrock
3. **State-Specific Laws:** Add state-specific acts and rules (currently focused on central legislation)
4. **CSC Integration:** Deploy as a tool for Common Service Centre operators who assist villagers
5. **NGO Dashboard:** Analytics dashboard for NGOs to track common legal issues in their areas
6. **Offline Mode:** Cached common scenarios and rights information for zero-connectivity use
7. **WhatsApp Integration:** Reach users on the platform they already use daily
