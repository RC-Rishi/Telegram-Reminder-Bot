# College AI Assistant - Project Context

## Specification authority

This file is the authoritative root specification and scope. The ten files under
`docs/` are authoritative detailed specifications for their owned areas and must
conform to this file. The clarifications integrated here were approved during the
documentation planning and implementation-gate research sessions on 02 October 2026.

If implementation reveals a product-level contradiction, do not silently change
behaviour. Obtain user approval and update this file and the affected detailed
specifications together. Implementation choices may be refined without changing
product behaviour; record them in the owning detailed specification.

The documentation ownership map and shared terminology are in
[docs/PRD.md](docs/PRD.md). Implementation sequencing is in
[docs/ROADMAP.md](docs/ROADMAP.md).

## 1. Project Overview

## Name
College AI Assistant

## Goal

Build a personal AI-powered college assistant that helps a student manage and understand college information.

The assistant receives college-related documents and information, converts them into structured knowledge, and allows the user to ask natural questions through Telegram.

The system should behave like a reliable personal assistant, not a generic chatbot.

---

# 2. Current Scope

## User Scope

This is a personal assistant for one user.

The current version is NOT a public product.

Do not design for:
- Multi-user support
- User registration systems
- Billing
- BYOK (Bring Your Own Key)
- WhatsApp direct integration
- Autonomous agent behaviour
- Unrelated personal-assistant functionality
- Enterprise infrastructure, Kubernetes, and unnecessary microservices

These are outside the current scope.

---

# 3. User Experience Philosophy

The user should have a simple experience:

```
Upload document
        |
Ask questions naturally
        |
Receive accurate answers
```

The complexity should exist internally, not in the user interface.

The user should not manually organize files, add metadata, or manage databases.

The AI system should automatically organize information.

---

# 4. Interface

## Telegram Bot

Telegram is the only interface.

The user interacts through one continuous chat.

The user can:

- Upload PDFs
- Upload images
- Upload documents
- Send text messages
- Ask questions naturally

Avoid forcing complicated commands.

Commands can exist only for system control.

Example:

```
/commands
```

---

# 5. Assistant Behaviour

## College Assistant Mode

The default mode is College Assistant mode.

The assistant should assume questions are related to the user's college unless clearly unrelated.

Priority information:

- Timetable
- Lectures
- Rooms
- Exams
- Exam rules
- Notices
- Academic documents

---

# 6. Accuracy Rules

The assistant must follow strict accuracy mode.

Rules:

- Never hallucinate information.
- Never invent missing details.
- Always use available knowledge before answering college-related questions.
- If information is unclear, explain the reason briefly.
- If information is unavailable, say it is unavailable.

Every important answer should include source information.

College answers that assert sourced facts include concise original filename and
upload date information. Multiple sources remain distinguishable. Missing or
conflicting facts must be identified rather than fabricated. General educational
answers must not imply that they describe the user's college.

Use the real current date/time when required, interpreted in Asia/Kolkata for
local college dates. Never substitute model training knowledge for a live clock.

Example:

```
Source:
CE_Exam_Timetable.pdf

Uploaded:
02 October 2026
```

---

# 7. Document Upload Philosophy

The user may add a short message with a file.

Example:

```
new timetable
```

or:

```
timetable for next 5 days
```

The message is only a hint.

It is NOT the truth.

Priority:

```
Document content
        >
Document metadata
        >
User caption/instruction
        >
AI interpretation
```

The AI must analyze the document itself.

This hierarchy concerns evidence about document contents. A caption is not proof
of a document's authority, dates, or applicability. If the document does not
establish a caption's validity window, obtain explicit confirmation before
applying it. Record confirmed local validity or correction decisions separately
from original evidence; never rewrite what the source document actually says.

---

# 8. Document Processing Pipeline

The system should not directly use raw PDFs for retrieval.

Final flow:

```
Original Document

        ↓

Native Parsing / Targeted Gemini Understanding Where Required

        ↓

Classification

        ↓

Information Extraction

        ↓

Structured Knowledge File (.md)

        ↓

Validation

        ↓

Embedding Generation

        ↓

Vector Database
```

---

# 9. Supported Documents

The system should support:

- PDF
- Images
- Documents
- Text

The document processor should handle:

- Normal PDFs
- Tables
- Scanned documents
- Images/screenshots

---

# 10. Document Classification

Documents should automatically be classified.

Examples:

```
TIMETABLE

EXAM_TIMETABLE

EXAM_RULES

NOTICE

CIRCULAR

ASSIGNMENT

ACADEMIC_CALENDAR

OTHER
```

If confidence is high:

Process automatically.

If confidence is low:

Ask the user for confirmation.

Example:

"I think this is an exam timetable, but I am not completely sure. Please confirm."

---

# 11. Knowledge System

The AI creates human-readable structured knowledge files.

The original files are preserved.

Storage philosophy:

```
Original Document
        |
        ↓
Structured Knowledge
        |
        ↓
AI Retrieval
```

---

## Knowledge File Structure

Use small logical files.

Do NOT create page-based files.

Good:

```
knowledge/

exam/
    python_exam.md
    physics_exam.md

timetable/
    current_timetable.md

rules/
    exam_rules.md
```

Bad:

```
page1.md
page2.md
table_row_1.md
```

Split by logical information, not size.

---

# 12. Knowledge File Rules

Each knowledge file should contain:

- Title
- Type
- Status
- Source document
- Upload date
- Extracted information
- Validity information

Example:

```
Title:
Python Programming Exam

Type:
Exam Event

Status:
Active

Source:
Exam_Timetable.pdf

Date:
15 October 2026

Time:
10:30 AM

Room:
C301
```

---

# 13. RAG Design

The system uses Retrieval Augmented Generation.

Important:

Do NOT embed raw PDFs.

Final RAG flow:

```
PDF

↓

Structured Markdown

↓

Embedding Model

↓

Vector Database

↓

Retrieval

↓

Gemini Answer Generation
```

---

# 14. AI Pipeline

The assistant uses a custom AI pipeline.

The specified custom pipeline does not use LangChain.

Flow:

```
User Question

↓

Intent Router

↓

Information Planner

↓

Data Retrieval

↓

Context Builder

↓

Gemini Reasoning

↓

Response Validation

↓

Final Answer
```

This is the logical pipeline. Skip Gemini reasoning/generation when validated
structured data and deterministic rendering can answer reliably.

---

# 15. Intent Router

The system should identify what kind of request it is.

Examples:

```
"Where is my next lecture?"

Intent:
NEXT_LECTURE


"What are exam rules?"

Intent:
EXAM_RULE_QUERY


"When is Python exam?"

Intent:
EVENT_QUERY


"Explain recursion"

Intent:
GENERAL_KNOWLEDGE
```

---

# 16. Retrieval Strategy

Not every question needs AI.

Use the correct system.

## Timetable / Events

Use:

- PostgreSQL
- Redis cache

## Rules / Notices / Explanations

Use:

- Vector retrieval
- Structured knowledge files

## General questions

Use Gemini directly.

---

# 17. Gemini Usage

Gemini is used for:

- Document understanding
- Classification
- Extraction
- Reasoning
- Natural language responses

Gemini should NOT control:

- Database operations
- Reminder scheduling
- Cache management
- Document lifecycle

Use the Gemini API free tier only, with no paid fallback or automatic upgrade.
Deterministic functionality continues during Gemini quota exhaustion or outages
when its persistent dependencies are available. AI-dependent processing remains
durably pending or fails gracefully without fabricated results.

Initial configurable model identifiers: `gemini-3.8-flash` for generation and
extraction, `gemini-3.5-flash-lite` for lightweight routing/classification when
needed, and `gemini-embedding-2` for structured-Markdown embeddings. These are
replaceable implementation configuration, not permanent product requirements.
Gemini remains the approved provider; configurable provider identity does not
authorize a second provider. Replacement models require regression evaluation;
incompatible embedding changes require an index rebuild.

---

# 18. Storage Architecture

## Google Drive

Purpose:

Store actual files.

Contains:

```
Original Documents

Structured Knowledge Files

Backups

Logs
```

Google Drive is the source storage.

Drive preserves original documents, explicitly submitted text sources, structured
Markdown revisions, backups and sanitized log exports. PostgreSQL owns control
and publication state; the presence of a file in Drive alone never activates it.

---

## PostgreSQL

Purpose:

Store structured control data.

Stores:

- User profile
- Document metadata
- Knowledge metadata
- Events
- Settings
- System logs

Do NOT store:

- PDFs
- Images
- Full documents
- Embeddings
- Permanent conversation history

PostgreSQL also owns processing jobs, publication/eligibility state, conflicts,
confirmed decisions, reminder delivery records and recovery metadata. Failed or
stale Drive, ChromaDB or Redis updates cannot make invalid knowledge active.

Initial hosting: Aiven Free PostgreSQL with authenticated public TLS, certificate
and hostname verification (`sslmode=verify-full`).

---

## Redis

Purpose:

Fast temporary cache.

Stores:

- Today's timetable
- Tomorrow's timetable
- Active exams
- Frequently used information

Redis is NOT the source of truth.

Use Upstash Free over authenticated public TLS for rebuildable derived cache only.
Its persistence does not make cache authoritative. Temporary conversation context
belongs only in bounded process memory, never Redis.

---

## ChromaDB

Purpose:

Vector search.

Stores:

- Embeddings of structured knowledge files

ChromaDB runs persistently on a Google Compute Engine e2-micro VM with persistent
disk in us-central1. The Cloud Run backend initially runs in us-central1 too.
This placement supports the intended free-tier setup; it does not guarantee
that the complete application or all dependent services cost nothing.

Redis caches and Chroma-derived indexes are rebuildable and never independently
determine truth. Validate retrieved revision eligibility against PostgreSQL.
Original files and structured knowledge remain persistently stored in Drive.

---

# 19. Cache Philosophy

Cache is used for speed.

Example:

Without cache:

```
Question
↓
PostgreSQL eligibility and structured data
↓
Drive structured Markdown when needed
↓
Answer
```

With cache:

```
Question
↓
Redis
↓
PostgreSQL eligibility and validity check
↓
Answer
```

Cache must be rebuilt when knowledge changes.

Redis is optional for availability: if it is unavailable or stale, use persistent
PostgreSQL data and Drive knowledge. Do not re-extract original documents on
every cache miss. Bound cached results by source revisions and validity windows;
check authoritative eligibility before using cached facts.

---

# 20. Conflict Handling

The assistant must detect conflicts.

Example:

Old:

```
Python Exam:
15 October
Room B204
```

New:

```
Python Exam:
17 October
Room C301
```

The system should compare.

---

## Conflict Rules

Simple changes:

Automatically update.

Example:

Room change.

Automatic updates require high-confidence extraction, clear applicability and
an obvious replacement relationship. A newer upload alone is not proof of
authority. Ambiguous room changes also require confirmation.

Important conflicts:

Ask user.

Example:

Exam date changes.

The assistant should show:

```
Old information:

New information:

Source documents:
```

During an unresolved important conflict, suspend the disputed facts and their
affected reminders. Explain the conflict instead of giving a definitive answer
about those facts. Unrelated knowledge remains usable. Record the user's
resolution with source traceability before reactivating affected facts.

---

# 21. Knowledge Lifecycle Status

Knowledge items can have:

```
ACTIVE

EXPIRED

ARCHIVED

CONFLICTED
```

Normal retrieval only uses ACTIVE information.

Temporary validity is separate from lifecycle status. Temporary knowledge may
be ACTIVE during its validity window and override normal active knowledge only
within its applicable scope. ACTIVE is necessary but not sufficient: retrieval
also checks publication, validity, scope, overrides and unresolved conflicts.
Processing states for unpublished candidates are separate from these statuses.

Archive exists for:

- Undo
- Recovery
- Debugging

---

# 22. Temporary Information

Example:

User:

"Timetable for next 5 days"

The system examines the document first. If the document establishes the window,
use that evidence; otherwise ask for explicit confirmation of the exact dates
before creating an effective temporary override. Do not silently interpret an
ambiguous relative window such as "next 5 days."

After expiry:

- Deactivate temporary file
- Restore previous information only where it is still valid and applicable
- Update cache

Evaluate time validity on reads and before reminder delivery, even if a scheduled
expiry job is delayed. Expiry must not revive superseded, conflicted or otherwise
invalid knowledge. Overlapping incompatible temporary overrides require
resolution instead of choosing the latest upload silently.

A future-effective replacement does not retire currently valid knowledge early.
Apply replacement relationships only to their declared scope and effective
window; archive a revision when it has no remaining normal applicability.

---

# 23. Reminder System

Reminder system is deterministic.

AI proposes event information. Validated application logic creates events.

System manages reminders.

Flow:

```
Knowledge

↓

Event Extraction

↓

PostgreSQL Event

↓

Scheduler

↓

Telegram Reminder
```

---

## Reminder Rules

Fixed rules.

Do not make adaptive AI reminders.

Required offsets (local calendar days):

Exam:

```
5 days before
1 day before
Same day
```

Assignment/deadline:

```
3 days before
1 day before
Deadline day
```

- Reminder timezone is Asia/Kolkata; store execution timestamps in UTC.
- Default delivery is 09:00 local time on each offset date.
- For an earlier same-day event, use one hour before its start; this may fall on
  the previous calendar date for an event shortly after midnight.
- Never send at or after a known event start/deadline. An event exactly at 09:00
  therefore has no same-day reminder under these fixed rules.
- For a date-only event, preserve the missing time rather than inventing one;
  deliver at 09:00 on eligible dates and stop after its local event date.
- Do not create catch-up reminders for offsets already missed when an event is
  added or changed. Missed reminder offsets are never backfilled after the
  event/deadline. Delivery retries must also respect the expiry cutoff.
- Other event types remain queryable but do not receive automatic reminders.
- Revalidate conflicts, active revisions and validity immediately before sending.
- Cloud Scheduler triggers deterministic checks; AI does not execute reminders.

Date-only cutoffs and bounded retry-window handling are explicit engineering
defaults detailed in [docs/DOCUMENT_RULES.md](docs/DOCUMENT_RULES.md). They do not
permit inventing an event time or adapting the fixed offsets.

---

# 24. Conversation Memory

Do NOT store permanent chat history.

The assistant should remember:

YES:
- College knowledge
- Documents
- Events
- Settings

NO:
- Every conversation
- Random questions
- Old chat messages

Only temporary context is allowed.

Temporary context must not enter Redis persistence/snapshots or backups.

Keep it only in process memory: initially 30 minutes of inactivity, two hours
absolute lifetime and at most six turns. Instance changes may erase it; ask for
a missing antecedent rather than recovering chat history.

Normal text conversation is temporary. Persistent text knowledge is created only
from explicit submissions/corrections or confirmed changes; ambiguous intent
requires confirmation. Save accepted text as a traceable source in Drive, not as
a conversation transcript. Durable conflict decisions and system audit records
contain structured outcomes, not accumulated chat messages.

---

# 25. Security Rules

Required:

- Telegram user ID verification
- Verification of the configured single private Telegram chat
- API keys stored securely
- No secrets in code
- Limited Google Drive access
- Safe error messages
- No sensitive information in logs

---

# 26. Backend Architecture

Use a modular monolith.

Do NOT create many microservices.

Backend modules:

```
backend/

telegram/

ai/

documents/

knowledge/

retrieval/

reminders/

database/

cache/

scheduler/

utils/
```

---

# 27. Technology Stack

## Backend

Python + FastAPI

## AI

Gemini API

## Storage

Google Drive

## Database

PostgreSQL

## Cache

Redis

## Vector Database

ChromaDB

## Deployment

Google Cloud ecosystem

Preferred:

Google Cloud Run

## Scheduler

Google Cloud Scheduler

## Durable asynchronous processing

Cloud Tasks dispatches document-processing jobs with retries and deduplication.
PostgreSQL stores durable job state and application-level idempotency records;
task retries or duplicate deliveries must not duplicate knowledge or events.
Processing handlers remain modules of the same FastAPI application.

## Deployment constraints

Cloud Run backend: us-central1 initially.

ChromaDB: persistent Google Compute Engine e2-micro VM with persistent disk,
also in us-central1. Never use Cloud Run's disposable local filesystem as the
durable Chroma store. Test e2-micro capacity at an explicit early feasibility gate
before vector-dependent implementation: its 1 GB RAM may be insufficient. Recheck
before deployment; do not silently change the approved machine type.

Target zero monetary cost for the initial deployment. Check the full stack's
costs, quotas, storage and networking before provisioning; zero cost is not
guaranteed. Never silently add paid services, enable paid fallback or upgrade
plans to evade limits. Keep Gemini in an unbilled project separate from the
billing-enabled infrastructure project.

Aiven and Upstash may use public authenticated TLS. Chroma remains private through
Direct VPC egress and restricted firewall rules. Avoid paid VPC connectors, Cloud
NAT and a chargeable external IPv4 on the Chroma VM; validate private image pulls
and the OS-update path before rollout.

Three Cloud Scheduler jobs invoke the monolith: an operational tick every minute,
a backup trigger every six hours and daily maintenance at 03:00 Asia/Kolkata.
Long work uses resumable Cloud Tasks stages; PostgreSQL owns job state.

## Approved engineering defaults and validation gates

Defaults are adjustable implementation settings, not permanent product rules.
Detailed owners:

- [ARCHITECTURE](docs/ARCHITECTURE.md): request-based Cloud Run, 1 vCPU/1 GiB RAM,
  concurrency 4, minimum 0/maximum 2 instances, one heavy stage globally;
  stage/task/Run deadlines 240/300/360 seconds; five transport attempts with
  10-second initial/10-minute maximum backoff; five-minute derived-cache TTL.
- [DOCUMENT_PROCESSING](docs/DOCUMENT_PROCESSING.md): native parsing first using
  pdfplumber, pypdfium2, python-docx and Pillow, with targeted Gemini visual analysis;
  20 MB upload, 100 PDF pages, 20 megapixel images, 100 MB expanded DOCX, five-page
  visual batches at approximately 200 DPI, 32,000 input/8,192 output tokens per
  extraction call and 120-second Gemini request timeout.
- [KNOWLEDGE_SYSTEM](docs/KNOWLEDGE_SYSTEM.md) and
  [AI_PIPELINE](docs/AI_PIPELINE.md): 768-dimensional text embeddings, cosine
  similarity, eight usable retrieval candidates with bounded additional retrieval.
- Encrypted PostgreSQL backups to Drive every six hours; retention six-hourly for
  48 hours, daily for seven days, weekly for four weeks, monthly for three months.
  Recovery targets: six-hour PostgreSQL recovery point and four-hour core restoration
  after recovery begins. Semantic-index recovery is measured separately and may
  depend on free embedding quota. ARCHITECTURE owns recovery details.

Confidence is evidence-gated: every material accepted field needs source traceability,
with zero unresolved critical validation errors, material ambiguity or important
conflicts. Automatic extraction routes must pass representative acceptance tests.
Model self-reported confidence is not calibrated probability. Numeric extraction,
classification and retrieval thresholds require labelled test calibration.

Implementation must benchmark Chroma capacity, container footprint, extraction
accuracy, retrieval quality, Cloud Run sizing, database behaviour, quota consumption
and recovery. Tune defaults with recorded evidence without changing fixed product
policy; failed feasibility does not authorize scope changes or paid upgrades.

---

# 28. Development Philosophy

Do not jump directly into coding.

Order:

```
Documentation

↓

Architecture Review

↓

Implementation Plan

↓

Development

↓

Testing

↓

Deployment
```

The system should be built intentionally, not through random AI-generated code.

---

# Final Principle

This project is:

NOT:

"A PDF chatbot"

It is:

"A personal AI knowledge assistant that converts unstructured college information into structured, searchable, reliable knowledge and provides context-aware answers."
