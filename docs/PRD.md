# College AI Assistant — Product Requirements

Status: approved scope with engineering defaults identified below. Updated: 02 October 2026.

## Authority and documentation map

[PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) is the authoritative root specification and scope. This document owns product requirements and shared terminology. The documents below own the detailed specifications for their areas. A product-level contradiction requires user approval and coordinated updates to the root and affected documents before implementation changes behaviour.

| Document | Owns |
| --- | --- |
| [DOCUMENT_RULES.md](DOCUMENT_RULES.md) | Evidence authority, validity, conflicts, versions, duplicates and recovery |
| [KNOWLEDGE_SYSTEM.md](KNOWLEDGE_SYSTEM.md) | Knowledge representation, granularity, provenance and retrieval eligibility |
| [AI_BEHAVIOR.md](AI_BEHAVIOR.md) | Assistant behaviour, uncertainty and source attribution |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Technical boundaries, deployment, security and recovery |
| [DATABASE_DESIGN.md](DATABASE_DESIGN.md) | Persistent entities, relationships and constraints |
| [DOCUMENT_PROCESSING.md](DOCUMENT_PROCESSING.md) | Upload-to-publication processing contract |
| [AI_PIPELINE.md](AI_PIPELINE.md) | Question routing, retrieval, context and response validation |
| [TESTING.md](TESTING.md) | Verification scenarios and release gates |
| [ROADMAP.md](ROADMAP.md) | Implementation order, dependencies and milestone acceptance |

## Product and problem

One student receives changing timetables, notices, exam information, rules and assignments in inconsistent formats. Finding the correct current information and remembering deadlines is difficult. The College AI Assistant organizes intentionally supplied college information automatically and answers natural questions through one private Telegram chat, preserving evidence and uncertainty.

Success means the user can upload a supported source, receive a clear processing outcome, ask a question and obtain an accurate current answer with concise source information. Important changes are confirmed; temporary information expires correctly; reminders follow fixed rules without an AI deciding when to send them.

## Shared terminology

| Term | Meaning |
| --- | --- |
| Source | Preserved original upload or explicitly submitted/confirmed text saved in Drive |
| Document | A source registered for processing, including a text source |
| Knowledge item | A stable logical entity such as one exam, a scoped timetable or a coherent rules topic |
| Knowledge revision | Immutable evidence-backed content for an item, represented as structured Markdown |
| Publication | PostgreSQL-controlled acceptance of a validated revision for use |
| Lifecycle status | ACTIVE, EXPIRED, ARCHIVED or CONFLICTED; separate from processing state |
| Temporary validity | A bounded applicability window, independent of lifecycle status |
| Eligible fact | A published ACTIVE fact valid for the requested scope/time, not masked by an override or unresolved conflict |
| Effective knowledge | Eligible knowledge selected after applying scoped overrides and decisions |
| Conflict | Incompatible claims about the same fact and overlapping applicability |
| Event | Structured timetable occurrence, exam, assignment/deadline or other college date derived from accepted knowledge |
| Decision | A structured user confirmation or resolution, preserved separately from source evidence |
| Archive | Retained revisions for explicit undo, recovery and debugging; excluded from normal retrieval |

## User experience

1. Send a PDF, image, document or an explicit text submission. Captions are optional hints.
2. Receive an acknowledgement, then a success, clarification or failure outcome. An acknowledgement alone never means the content is active.
3. Answer a concise question only when classification, extraction, validity, intended persistence or an important conflict cannot be resolved safely.
4. Ask naturally: “Where is my next lecture?”, “When is my Python exam?”, “What are the exam rules?”
5. Receive a concise answer or an explanation of what is missing/conflicting. College facts include original filename and upload date.
6. Receive fixed exam and assignment/deadline reminders. Undo or recovery is an explicit system-control action with a preview of consequences when ambiguous.

No manual folder management, routine tagging or complex commands are required. General educational questions such as “Explain recursion” are allowed; they do not create knowledge or expand the product into an unrelated personal assistant.

## Functional requirements

| ID | Requirement | Acceptance |
| --- | --- | --- |
| FR-01 | Accept only the configured Telegram user and private chat. | Other senders/chats cannot cause processing, retrieval or changes. |
| FR-02 | Accept PDFs, images, documents and explicit text sources; support tables and scanned content. | A supported fixture for each category reaches publication or a specific actionable clarification. |
| FR-03 | Classify and organize automatically without mandatory tags. | Clear inputs need no confirmation; uncertain classification asks once with context. |
| FR-04 | Preserve originals and produce structured Markdown by logical entity. | Every published fact has source evidence; no page-based knowledge files or raw-PDF embeddings. |
| FR-05 | Apply evidence hierarchy and confirmed local decisions. | Misleading captions cannot silently establish facts or validity. |
| FR-06 | Resolve lifecycle, temporary validity, scope and conflicts consistently. | Temporary overrides expire, disputed facts suspend and valid underlying knowledge resumes where appropriate. |
| FR-07 | Route structured queries deterministically and semantic questions through grounded retrieval. | Known timetable/event lookups need no generation call; notices/rules use eligible knowledge. |
| FR-08 | Enforce strict accuracy and concise attribution. | No unsupported college claim is accepted in the release scenario suite; missing/conflicting information is explained. |
| FR-09 | Extract events and execute fixed reminders. | Exam and assignment/deadline schedules match DOCUMENT_RULES; affected conflicts cancel/suspend pending delivery. |
| FR-10 | Preserve revisions for duplicates, undo and recovery. | Repeated input creates no duplicate events; recovery cannot silently revive invalid content. |
| FR-11 | Retain useful knowledge, settings, decisions and operational records without permanent chat history. | Ordinary questions and conversational context never enter persistent knowledge/log payloads. |
| FR-12 | Work without Redis and rebuild derived indexes. | Cache loss does not change answers; Chroma can be rebuilt from approved Drive revisions and PostgreSQL control records. |

## Non-functional requirements

| ID | Requirement and verification |
| --- | --- |
| NFR-01 Accuracy | Missing values remain missing. Validators, fixture comparisons and evidence review enforce abstention; no claim that model confidence proves correctness. |
| NFR-02 Reliability | Retry processing safely, maintain durable job state, gate publication in PostgreSQL and recheck eligibility before response/reminder use. Test interrupted stages and duplicate delivery. |
| NFR-03 Security/privacy | Verify Telegram identity/chat and internal invokers; secure secrets; restrict Drive access; exclude secrets, raw messages and document contents from operational logs. |
| NFR-04 Performance | Cache derived information; avoid unnecessary Gemini calls. Measure cache hit/miss, extraction and query latency against a representative corpus before declaring service targets. |
| NFR-05 Maintainability | Python/FastAPI modular monolith, custom pipeline, no LangChain. External services have narrow adapters and recoverable failure contracts. |
| NFR-06 Time correctness | Obtain current time from the runtime clock; local calendar semantics use Asia/Kolkata. Test midnight, missing times, future validity and delayed scheduled checks. |
| NFR-07 Deployment | Target zero monetary cost; never guarantee it. Cloud Run and private persistent Chroma e2-micro initially co-located in us-central1. Early Chroma viability and final capacity/quota/cost gates are mandatory; no silent paid upgrade. |

No numerical latency, corpus capacity or uptime promise has yet been approved. Measurements and limits belong to the implementation/deployment gates, not invented product guarantees.

## Fixed scope and non-goals

Required stack: Python, FastAPI, Gemini, Google Drive, PostgreSQL, Redis, ChromaDB, Cloud Run, Cloud Scheduler and Cloud Tasks. Redis is disposable; Chroma is a rebuildable index; neither determines truth independently.

Excluded: multi-user support, public registration, billing, BYOK, WhatsApp, autonomous agents, unrelated personal-assistant functionality, enterprise infrastructure, Kubernetes and unnecessary microservices. No permanent conversation database, direct PDF RAG, adaptive reminders or manual knowledge-management UI.

## Approved clarifications and engineering defaults

Approved: temporary validity is independent of status; caption-only validity needs explicit confirmation; unresolved important conflicts suspend affected facts/reminders; only explicit or confirmed text becomes persistent; fixed reminder rules use Asia/Kolkata; Chroma runs on persistent e2-micro in us-central1; Cloud Tasks handles durable processing with PostgreSQL idempotency/publication control.

Approved implementation-gate decisions: Aiven Free PostgreSQL with verified public TLS; Upstash Free Redis over authenticated TLS for derived cache only; temporary conversation context exclusively in bounded process memory; Gemini free tier only, no paid fallback, with deterministic functionality preserved during quota outages. Chroma remains private and its 1 GB VM feasibility is tested early. Native parsing precedes targeted Gemini visual understanding. Scheduler has operational, backup and maintenance jobs. Evidence gates, not self-reported model probability, govern automatic acceptance.

Engineering defaults are labelled in their owning documents: model identifiers/embedding dimensions, parser packages, resource and format limits, timeouts/retries, retrieval/cache budgets, process-context bounds, backup/retention and recovery targets. ARCHITECTURE owns deployment and cost baselines; AI_PIPELINE, KNOWLEDGE_SYSTEM and DOCUMENT_PROCESSING own model/retrieval/extraction settings. Numeric thresholds require TESTING calibration. Hosting decisions are approved; capacity, compatibility, quota/cost and recovery claims still need measurement. Tune engineering defaults without changing fixed behaviour; new scope or changed product policy requires approval and coordinated root/detail updates.
