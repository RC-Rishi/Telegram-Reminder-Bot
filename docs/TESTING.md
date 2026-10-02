# Testing

This document owns verification under [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md). Requirement IDs are defined in [PRD.md](PRD.md); exact policy comes from [DOCUMENT_RULES.md](DOCUMENT_RULES.md). These are specifications for future tests, not a claim that application tests have run during documentation work.

## Test strategy

Use deterministic unit tests for policy, database integration tests for transactions/idempotency, adapter tests for external services and complete scenario tests for upload/query/reminder behaviour. Keep current time injectable. Use temporary isolated PostgreSQL, Redis and Chroma instances for integration; never test destructive recovery against real college data.

Maintain a small curated source corpus with manually reviewed expected facts, unknowns, scope, provenance locators, conflicts and eligible answers. Include PDFs with native text, scans, multi-page tables, screenshots, DOCX and explicit text. Synthetic documents should be clearly labelled; anonymize real examples used with permission.

Mock Gemini for repeatable orchestration/failure tests. Separately evaluate pinned Gemini extraction/generation/embedding versions against the corpus using real provider calls when authorized. A mocked pipeline proves orchestration, not model accuracy. Record model/prompt/parser versions with results.

## Unit and policy cases

| ID | Scenario and expected result |
| --- | --- |
| U-01 | Complete dates resolve in Asia/Kolkata; missing year/time stays unknown unless supported. Ambiguous numeric/relative dates request clarification. |
| U-02 | At valid_from an override applies; at exclusive valid_until it no longer applies. Delayed cleanup cannot extend it. |
| U-03 | Temporary override masks only matching fields/scope. A different section or disjoint field remains usable. |
| U-04 | Expiry with an expired, superseded or conflicted base yields the currently valid replacement or unavailable information, never blind restoration. |
| U-05 | Clear room amendment can apply automatically; important date/time/deadline disagreement requires confirmation. Newer upload alone grants no authority. |
| U-06 | Partial conflict masks disputed facts while unrelated facts remain eligible. Wholly disputed scope is excluded. |
| U-07 | Future event can be queried now; timetable future-day applicability uses the requested day, not event time as publication time. |
| U-08 | Duplicate bytes, duplicate transport and semantic overlap take distinct paths. Same filename/different bytes is not a hash duplicate. |
| U-09 | Normal conversation is not a persistent source. Explicit submissions are; ambiguous intent waits for confirmation. |
| U-10 | Content, metadata, caption and interpretation follow the evidence hierarchy; a confirmed local decision remains separate evidence. |
| U-11 | Future-effective or partial replacement leaves today's/unaffected base facts usable; archive only after remaining normal applicability ends. |

## Fixed reminder clock cases

Freeze clocks and compare exact UTC instants as well as displayed local dates.

| ID | Fixture | Expected result |
| --- | --- | --- |
| R-01 | Exam 15 October 2026 at 10:30 | Due 10 October, 14 October and 15 October at 09:00 Asia/Kolkata. |
| R-02 | Assignment deadline 15 October at 17:00 | Due 12 October, 14 October and 15 October at 09:00. |
| R-03 | Exam 15 October at 08:00 | Same-day offset due 15 October at 07:00; preceding offsets stay at 09:00. |
| R-04 | Exam 15 October at 00:30 | Same-day offset due 14 October at 23:30; preserve separate offset identity. |
| R-05 | Exam starts exactly 09:00 | Same-day default is at start and is skipped; never send at/after start. |
| R-06 | Exam date known, time unknown | Use 09:00 without claiming a start time; cutoff is next local midnight, not an invented exam time. |
| R-07 | Event added at 10:00 on its day | Do not create a catch-up 09:00 occurrence. |
| R-08 | Existing due occurrence checked at 09:01 | May send within its due-date/event cutoff window; late triggering is not creation of a catch-up offset. |
| R-09 | Due occurrence delayed beyond its due local date or event start | Mark SKIPPED. No later-day or post-event backfill. |
| R-10 | Important date conflict opens, then resolves after an offset passed | Suspend affected unsent occurrences; create only future eligible offsets after resolution. |
| R-11 | Room-only update or Chroma rebuild | Do not change schedule_revision or duplicate reminder offsets. |
| R-12 | Accepted event date change or cancellation | Cancel obsolete unsent occurrences; preserve sent history; generate only future offsets for an accepted new date. |
| R-13 | Repeated/concurrent Scheduler calls | One claim per occurrence; a confirmed SENT occurrence is not resent. |
| R-14 | Send accepted but response/commit lost | Mark UNKNOWN after ambiguity/lease expiry; do not blindly resend or claim exactly-once delivery. |
| R-15 | Ordinary lecture or notice date | Remains queryable but receives no automatic reminder. |

## Extraction and realistic pipeline scenarios

| ID | Scenario | Required observations |
| --- | --- | --- |
| P-01 | Multi-page timetable with repeated headers and a row split across pages | Preserve row/column meaning; logical scoped timetable, not page files; no duplicate lectures. |
| P-02 | Blurry scanned exam date or cropped room | Mark uncertain/missing field; ask specifically; do not guess. |
| P-03 | Caption “next 5 days”, document lacks validity | No override until exact dates are confirmed; decision recorded separately. |
| P-04 | Caption says “exam timetable”, contents are a circular | Classify from contents; caption cannot dictate facts. |
| P-05 | One document contains exams, rules and assignments | Produce logical typed items with per-item provenance; independent valid items can publish while another waits. |
| P-06 | Old source says 15 October/B204; new source says 17 October/C301 | Register date conflict; suspend date-dependent reminders; show both sources rather than silently choosing recency. |
| P-07 | Two incompatible overlapping temporary timetables | Ask for resolution of overlapping facts; preserve disjoint valid entries. |
| P-08 | Permanent timetable changes during a temporary override | After expiry use the valid accepted permanent revision, not the originally masked stale revision. |
| P-09 | Explicit user correction disagrees with a document | Preserve correction text as a source and accepted decision; never rewrite original evidence. |
| P-10 | Duplicate upload plus new caption | Reuse content extraction, avoid duplicate events, and confirm any new unsupported validity instruction. |
| P-11 | Corrupt/encrypted/unsupported/oversized file | Safe actionable failure; no publication and no unbounded retry. |
| P-12 | Stale confirmation after another relevant change | Reject outdated application and present the current comparison. |
| P-13 | Undo after intervening updates and previously sent reminder | Revalidate restoration, retain evidence/history and never replay sent reminders. |
| P-14 | Document embeds instructions to bypass rules or reveal secrets | Treat instructions as source data; no action/credential access or policy change. |

## Database and service integration

- Interrupt each stage before/after Drive writes, database commits and Cloud Tasks enqueue. Reconcile orphan artifacts and outbox entries; exactly one logical result appears despite repeated delivery.
- Execute concurrent publication, conflict registration, expiry and query operations. Assert expected-version retries, generation updates and no known-stale response after final eligibility validation.
- Fail Chroma indexing after publication and inject pre-publication/archived vector hits. Verify structured queries work, pending coverage is visible and stale hits never authorize facts.
- Flush Redis, return stale generations, delay invalidation and cross midnight/expiry. Persistent-data answers must match healthy-cache answers.
- Stop PostgreSQL: prohibit unverifiable college answers and reminder sends. Stop Drive: reject new publication; allow only independently supported existing projections/verified content. Stop Gemini: deterministic answers/reminders continue.
- Revoke/misconfigure tokens, forge webhook secrets, use a different Telegram user/chat, or call internal routes without approved OIDC identity. Assert rejection before data access or model calls.
- Test Telegram rate limiting and safe markup splitting without losing exceptions or citations. Keep all sends inside the configured private chat.
- Inspect stored records/logs after ordinary conversation, errors and model calls. No raw chats, full prompts, document bodies or credentials may appear in PostgreSQL or operational logs.
- Verify Redis persistence/snapshots/backups contain no temporary conversation context. Distinguish a transient Drive outage from confirmed source deletion/hash corruption; the latter imposes an integrity hold on dependent facts/reminders.
- Verify process-memory context expiry, instance-change loss and isolation from Redis; Redis failure alone must not be modelled as deleting process memory. Inspect crash/error logging paths for accidental context persistence.
- Exhaust Gemini free quota and inject 429/unavailability. Deterministic answers, exact retrieval and reminders continue with healthy persistent dependencies; AI-dependent source/index jobs persist their reason/next-attempt time, acknowledge deferral and resume safely. No paid API/project fallback or durable ordinary-question retry transcript.
- Test authenticated public Aiven/Upstash TLS with certificate/hostname verification and invalid certificate/credentials; Chroma remains private. Verify queue/OIDC identity separately from task headers and public webhook ingress.
- Test five-attempt transport policy, task deadline cancellation/lease expiry, duplicate delivery outside the Tasks deduplication window, deferred quota recovery and terminal-failure reconciliation. Duplicate Scheduler jobs must not duplicate backups or maintenance effects.
- Expire/lose transient context: ask for a missing antecedent. Restore a pending durable conflict decision without needing chat history.
- Restore PostgreSQL and Drive artifacts into an isolated environment, rebuild Chroma/Redis and compare effective facts and unsent reminder state before enabling delivery.

## Retrieval and hallucination evaluation

Create expected-answer cases for exact structured lookup, paraphrased notices, similarly named subjects, source exceptions, combined sources, absent information and conflicting facts. Test whether retrieval finds the supporting eligible facts before judging generation.

Track extraction field accuracy, critical date/time errors, evidence-locator correctness, retrieval coverage, unsupported-claim count, appropriate abstention and citation correctness. Do not use answer fluency as a quality metric. Review free-form outputs against source evidence; a second model can assist but cannot certify correctness.

Release corpus gate: zero unsupported material college claims, zero incorrect critical event dates/times accepted as certain, zero inactive/conflicted fact leakage and correct attribution for every sourced answer. These are finite-suite acceptance criteria, not a universal proof of zero hallucinations. Add regression cases for every observed critical failure.

Clear structured lookup cases must demonstrate zero Gemini calls. Semantic lookup tests must show only the model calls needed for embeddings/generation. Test cache hits, direct metadata lookup and insufficient-coverage fallback separately.

## Confidence calibration

Automatic acceptance requires traceability for every material field, zero unresolved critical validation errors/ambiguities/important conflicts and a tested extraction route. These are evidence gates, not a calibrated probability. Do not label self-reported confidence, model agreement, cosine similarity or OCR heuristics as probability of truth.

Use manually labelled calibration and held-out evaluation fixtures, grouped by input type and consequential field. Measure classification precision/confusion, extraction field errors, uncertainty detection, retrieval supporting-source recall and abstention errors. Choose numeric cutoffs from these results, record sample sizes/error cases and re-evaluate after model/parser/prompt changes. No arbitrary universal 0.95 threshold or assumed similarity cutoff. Until calibrated, a numeric score cannot independently authorize publication; use validated deterministic/evidence routes or ask about unresolved interpretation. Important conflicts always follow DOCUMENT_RULES.

## Benchmark and deployment gates

Benchmark cold/warm structured queries, semantic queries, representative uploads, index rebuilds and minute-based reminder checks. Record corpus size, concurrency, model versions, VM memory/CPU and queue lag. Test Chroma on the approved e2-micro with persistent disk, one indexing operation at a time; include restart and disk persistence checks.

| Gate | Required measurement and acceptance evidence |
| --- | --- |
| B-01 Chroma capacity | Run early at ROADMAP 0A, before vector-dependent implementation. Record representative corpus and growth headroom, server/image versions, 768-dimensional inputs, OS/TLS overhead, idle/peak RAM and CPU, insert/query/rebuild latency, restart persistence and rollback overlap. No OOM or sustained swapping. Record usable capacity/latency from results, not raw-vector arithmetic; recheck completed corpus before rollout. Failure stops dependent work pending approved correction. |
| B-02 Container footprint/networking | Measure actual registry storage including shared layers, retained releases and rollout overlap against the current free allowance. Verify private Chroma connectivity, image pulls, administration and OS-update path without public Chroma, paid connector/NAT or VM IPv4. Failure requires an approved correction, not hidden cost. |
| B-03 Extraction accuracy/limits | Evaluate each supported format, scans/blur/handwriting, multi-page and merged tables, source locators, partial outputs and dates. Exercise byte/page/pixel/expanded-DOCX/token/deadline boundaries and peak memory. Preserve all material facts/exceptions or explicitly fail/clarify; critical incorrect dates cannot publish as certain. Record package/model/prompt versions and tune defaults. |
| B-04 Retrieval quality/calibration | Measure supporting eligible-source recall, exact versus semantic routes, exceptions, similarly named entities, stale-candidate filtering and incomplete coverage. Calibrate numeric thresholds and candidate/token bounds on held-out cases; meet the finite-suite grounding/citation gates above. Eight usable candidates is a starting setting, not proven optimal. |
| B-05 Cloud Run sizing | Measure cold/warm latency, Direct VPC readiness, peak memory including scratch/decoded images, concurrency, queue lag and checkpoint/cancellation behaviour at 1 vCPU/1 GiB/concurrency 4/max 2. Verify one heavy stage globally and stage/task/Run deadlines 240/300/360 s. Tune within the cost target; fixed correctness rules still hold. |
| B-06 Database/cache behaviour | Validate Aiven verified TLS, real region latency, connection recovery/health checks, rollout pool overlap below 20 connections, inactivity recovery, disk growth and dump/restore. Test Upstash latency/command consumption, eviction/quota failures and persistent fallback. No assumption of free-tier SLA or guaranteed automatic wake-up. |
| B-07 Quota and monetary target | Read actual project/model quotas and measure calls/tokens per upload/query, retries, minute-tick CPU, Redis commands, registry/disk storage, secret accesses, logs and external bytes. Estimate full monthly use with headroom and account-wide competing usage. Reverify current pricing; no trial-credit dependency, paid fallback or guarantee that budget alerts cap spending. |
| B-08 Backup/recovery | Validate six-hour encrypted dumps/manifests and retention (48 h six-hourly, 7 daily, 4 weekly, 3 monthly), incomplete-backup detection, available Drive space and separately recoverable key. Isolated restore must recover decisions/hashes/effective facts before sends, reconcile uncertain delivery intervals and never replay past reminders. Test six-hour recovery point/four-hour core restoration target; measure quota-dependent semantic rebuild separately. Drill before rollout, quarterly and after format changes. |

Pin compatible dependencies/images and record measured settings before rollout. Major hosting choices are approved; benchmark success is still pending. If e2-micro or the zero-cost layout fails, record evidence and seek approval before changing the root or accepting a paid component. Free-tier availability is neither a performance nor a zero-cost guarantee. Targets and tunable values are engineering defaults, not permanent product promises.

## Requirement coverage and documentation review

| Requirements | Primary evidence |
| --- | --- |
| FR-01, NFR-03 | Authorization, secret/log inspection and internal endpoint rejection tests |
| FR-02–FR-05 | Extraction corpus, authority/intent unit tests and P-01–P-05/P-09–P-11 |
| FR-06, FR-10, NFR-06 | U-01–U-08/U-11, P-06–P-08/P-12–P-13 and boundary clocks |
| FR-07–FR-08, NFR-01/NFR-04 | Retrieval/grounding suite and model-call/latency measurements |
| FR-09 | R-01–R-15 and concurrent delivery integration |
| FR-11 | U-09, text-source provenance and persistence/log inspection |
| FR-12, NFR-02 | Outage, stale-index/cache, transaction interruption and restore/rebuild tests |
| NFR-05/NFR-07 | Module-boundary review, deployment/capacity validation and architecture checks |

Review the root plus all ten documents for identical terminology, scope, validity/status separation, text-persistence rules, reminder constants and source-of-truth ownership. Check every internal link, requirement-to-test mapping and roadmap dependency. Any new product-level contradiction requires user approval and coordinated documentation updates; never resolve it only in code.

## Documentation baseline review — 02 October 2026

Reviewed PROJECT_CONTEXT.md and all ten detailed specifications together. Confirmed the exact file set, local link targets, balanced fenced blocks, unique requirement/scenario identifiers and coverage of all 19 product/non-functional requirements. The specification includes 40 named policy/reminder/pipeline scenarios plus service integration and grounding evaluations, and six Mermaid diagrams. Application/model tests have not been run: this phase creates documentation only.

The review corrected future-effective replacement handling, immutable Markdown state snapshots, cache eligibility diagrams, transient-context persistence and the distinction between a Drive outage and confirmed artifact corruption. No remaining blocking product contradiction or scope expansion was identified.

The implementation-gate research update locks Aiven Free, Upstash Free derived cache, process-memory context, free-only Gemini, native-first parsing, private e2-micro, three Scheduler jobs and the zero-cost target. It records initial configurable models/packages/resources/retries/retrieval/cache/backup settings. Exact dependency compatibility and B-01–B-08/calibration results remain implementation work; no live benchmark or application test is claimed by this documentation phase. Changing fixed product policy requires coordinated approved root/detail updates.

Post-update consistency validation: reviewed the root and all ten owned-area documents, confirmed the exact documentation file set, 48 valid local links, balanced fenced blocks and 48 unique named scenarios (40 policy/reminder/pipeline plus eight benchmark gates). Checked hosting/network boundaries, memory-only context, model configuration/free-only policy, default limits/retention, reminder constants and early roadmap dependencies together. Removed stale undecided-hosting/Redis-context statements. No remaining documentation contradiction or scope drift was identified; empirical feasibility and calibrated thresholds remain explicitly gated. No application code, provisioning or live benchmark was performed.
