
# Product Requirements Document: Detroit Legislative Intelligence (DLI)

| **Project Name** | Detroit Legislative Intelligence (DLI) |
| :--- | :--- |
| **Version** | 1.0 (MVP Scope) |
| **Status** | **Draft** / In Review |
| **Product Owner** | Christopher G. Davis |
| **Target Launch** | Q2 2026 (Internal Alpha: Month 2) |
| **Document Date** | December 22, 2025 |

---

## 1. Problem Statement
Detroit’s legislative data is siloed, unstructured, and often invisible. While the City Council uses Municode, powerful quasi-public agencies (e.g., Housing Commission, Land Bank) publish critical spending decisions as static PDFs on isolated websites. There is no central index to track contracts, resolutions, or vendor history across the city’s 15+ governing bodies.

**Impact:**
*   **External:** Developers and lobbyists miss RFPs; advocates miss voting records.
*   **Internal:** Outlier reporters spend hours manually searching PDFs and transcripts to find quotes or verify facts.
*   **Missed Revenue:** We are sitting on valuable data (Documenters notes + Otter transcripts) that is currently unmonetized "dark data."

## 2. Product Vision
A unified, searchable intelligence platform that ingests legislative artifacts (PDFs, Transcripts, Minutes) from all Detroit agencies, indexes them for semantic search, and provides automated monitoring/alerting for professional subscribers.

**Core Value Prop:** "Google for Detroit Government."

---

## 3. User Personas

| Persona | Role | User Goal | Pain Point |
| :--- | :--- | :--- | :--- |
| **The Hunter** (Primary Revenue) | Lobbyist, Developer, Vendor | Track opportunities (RFPs) and threats (regulations). | "I have to check 5 different websites every week to see if 'Zoning' is discussed." |
| **The Gatherer** (Internal User) | Outlier Reporter / Editor | Find quotes, verify facts, spot trends. | "I know Council discussed 'ShotSpotter' last year, but I can't find the video timestamp." |
| **The Watchdog** (Mission User) | Resident / Advocate | Hold officials accountable. | "I can't read the DHC resolution because it's buried in a 100-page PDF." |

---

## 4. Functional Requirements

### 4.1 Data Ingestion (The Pipeline)
| ID | Priority | Feature | Description |
| :--- | :--- | :--- | :--- |
| **ING-01** | **P0** | **Multi-Source Scraper** | System must ingest artifacts from: <br>1. Municode/Legistar (City Council)<br>2. WordPress/Static Sites (DHC, DLBA, BZA)<br>3. Documenters.org (Notes) |
| **ING-02** | **P0** | **PDF OCR** | System must perform Optical Character Recognition on scanned PDFs (specifically DHC resolutions) to extract indexable text. |
| **ING-03** | **P1** | **Transcript Sync** | System must ingest Otter.ai text files and link them to the specific meeting date/board. |
| **ING-04** | **P2** | **Change Detection** | Scrapers must run on a schedule (Daily/Weekly) and only ingest *new* or *modified* artifacts. |

### 4.2 Intelligence & Processing
| ID | Priority | Feature | Description |
| :--- | :--- | :--- | :--- |
| **INT-01** | **P0** | **Entity Extraction** | LLM pipeline to extract structured data from unstructured text: <br>- **Vendor Name** (e.g., "MDG Connected Solutions")<br>- **Dollar Amount** (e.g., "$786,690")<br>- **Vote Status** (Pass/Fail) |
| **INT-02** | **P1** | **Summarization** | Generate a 1-sentence "Human Readable" summary of complex resolutions (e.g., "Approves $786k for cameras at 6 housing sites"). |
| **INT-03** | **P1** | **Documenter Link** | Automatically associate the official artifact (Resolution) with the corresponding Documenters Note based on Date + Agency. |

### 4.3 Search & Retrieval
| ID | Priority | Feature | Description |
| :--- | :--- | :--- | :--- |
| **SRCH-01** | **P0** | **Universal Search** | Single search bar querying all indices. Support for boolean operators (AND, OR, "Exact Phrase"). |
| **SRCH-02** | **P0** | **Faceted Filtering** | Filter results by: Agency, Date Range, Artifact Type (Resolution, Contract, Transcript), Money ($). |
| **SRCH-03** | **P1** | **Highlighting** | Search results must highlight the keyword in context (KWIC) within the source text. |

### 4.4 Alerts & User Accounts
| ID | Priority | Feature | Description |
| :--- | :--- | :--- | :--- |
| **USR-01** | **P0** | **Keyword Alerts** | Users can save a search (e.g., "Tax Abatement") and receive an email summary when new matching artifacts are ingested. |
| **USR-02** | **P1** | **Saved Collections** | Users can "bookmark" resolutions into folders (e.g., "Project Parkside Research"). |

---

## 5. Technical Architecture

### 5.1 High-Level Stack
*   **Frontend:** Next.js (React) + Tailwind CSS (hosted on Vercel or Render).
*   **Backend API:** Python (FastAPI). Python is required for robust scraping libraries (`BeautifulSoup`, `Playwright`) and Data/AI processing (`LangChain`, `Pandas`).
*   **Database:**
    *   **PostgreSQL:** Relational data (Users, Alerts, Agency Metadata, Resolution Metadata).
    *   **Search Engine:** Typesense or Algolia (for fast, typo-tolerant full-text search).
*   **AI/ML:** OpenAI API (`gpt-4o-mini`) for entity extraction and summarization.

### 5.2 Data Model (Simplified)
*   **`Agencies`**: `id`, `name`, `scraper_config_json`
*   **`Artifacts`**: `id`, `agency_id`, `source_url`, `title`, `full_text` (OCR), `doc_type` (Resolution, Minutes), `ingested_at`
*   **`Entities`**: `id`, `artifact_id`, `type` (Vendor, Amount), `value`
*   **`Documenter_Notes`**: `id`, `meeting_date`, `agency_id`, `summary_text`, `author`

---

## 6. UI/UX Requirements
*   **The "Rosetta Stone" View:** The Detail Page must display the **Official Text** and **Human Context** (Documenter Notes) side-by-side.
*   **Speed:** Search results must load in <200ms.
*   **Mobile:** Interface must be fully responsive (Documenters often check facts from their phones during meetings).

---

## 7. Success Metrics (KPIs)

| Metric | Definition | Goal (Month 6) |
| :--- | :--- | :--- |
| **Coverage** | % of target agencies successfully indexed. | 100% of Top 5 (Council, DHC, Police, Zoning, Land Bank) |
| **Efficiency** | Time saved per internal research task. | >50% reduction reported by Editorial |
| **Revenue** | ARR from Professional subscriptions. | $25,000 (100 Pro users @ $20/mo or 25 Enterprise @ $1k/yr) |
| **Alert Volume** | # of automated emails sent. | >500 / week (High engagement) |

---

## 8. Risks & Mitigation

| Risk | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **Scraper Fragility** | High. Agencies change website layouts, breaking scrapers. | Build "Canary" tests that run daily. If a scraper fails to find known text, alert Admin immediately. |
| **OCR Accuracy** | Medium. Old scanned PDFs may result in "garbage text." | Implement a "Confidence Score." If OCR confidence is low, flag for human review or display "image only" warning. |
| **LLM Hallucination** | Medium. AI might extract wrong dollar amounts. | Use strict JSON schemas for extraction. Display "AI Generated" disclaimer on summaries. Link directly to source PDF for verification. |

---

## 9. Phasing / Roadmap

### Phase 1: The "Internal Engine" (Months 1-2)
*   Build Scrapers for **DHC** and **City Council**.
*   Setup Postgres & Search Index.
*   Basic Search Interface (Internal access only).
*   *Goal:* Give Reporters the tool.

### Phase 2: The "Beta" (Months 3-4)
*   Add **Alerts** system (Email).
*   Add **Documenters Notes** integration.
*   Invite 10 Beta Partners (External).
*   *Goal:* Validate the "Willingness to Pay."

### Phase 3: Commercial Launch (Months 5-6)
*   Stripe Integration (Subscription Billing).
*   User Management / Teams.
*   Public Launch.
