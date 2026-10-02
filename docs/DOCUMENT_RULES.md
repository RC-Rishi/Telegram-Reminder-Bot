# Document Rules

This document owns document authority, validity, conflicts, versions, duplicates, recovery and fixed reminder policy. It conforms to [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md). Terminology is defined in [PRD.md](PRD.md); representations are specified in [KNOWLEDGE_SYSTEM.md](KNOWLEDGE_SYSTEM.md).

## Evidence and user decisions

For claims about a document: document content > document metadata > caption/instruction > AI interpretation. Examine actual contents before accepting a caption. Upload recency, filenames and model confidence do not prove that a document supersedes another.

Source issuance dates and explicit amendment language are evidence; upload timestamps record receipt only. Extract scope such as course, semester, section and affected subject from evidence. If the intended scope cannot be distinguished from another plausible scope, ask before applying changes.

A user may explicitly submit knowledge, correct a fact or resolve a conflict. Preserve that instruction as a separate traceable text source/decision and show its relationship to the original claim. A decision controls the assistant's local accepted interpretation; it does not alter what the original document says or establish external institutional authority.

Captions that assert validity without documentary evidence require explicit confirmation of exact dates and scope. If a caption contradicts the document, show the discrepancy. A user's explicit subsequent resolution is a decision, not retrospective evidence that the caption was true.

Normal text conversation remains temporary. Explicit submissions/corrections may enter processing; ambiguous intent asks whether to save the proposed information. Do not extract permanent facts from ordinary questions or casual statements merely because they look useful.

## Dates, scope and temporary validity

- Keep upload time, source issuance date, event date/time and knowledge validity distinct.
- Use Asia/Kolkata for local dates and an injected current-time clock. Persist instants in UTC and keep original local dates/time precision.
- Resolve explicit complete dates deterministically. Do not invent missing years, start times, ends, sections or academic terms. Context from the same source or an applicable accepted source may establish them; otherwise ask if consequential.
- Confirm ambiguous expressions such as “next 5 days” with an explicit start and end. Convert a confirmed inclusive end date into an exclusive next-midnight boundary internally.
- Use half-open validity intervals `[valid_from, valid_until)`. A missing end means no evidenced expiry, not authority for all semesters. An explicit permanent local adoption is still scoped.
- Temporary validity requires a bounded window and an explicit target scope/fields. ACTIVE temporary knowledge overrides matching normal knowledge only in that window. Non-overlapping entities/fields remain unchanged.
- Accepted future-effective revisions can be ACTIVE but are ineligible before their window. Passing a validity end makes them ineligible immediately even if EXPIRED has not yet been materialized.

An event in the future may be known now. Event occurrence time is not the same as knowledge validity. A notice currently valid may describe a future exam; do not hide that exam until its start. For a temporary timetable query about a future day, evaluate the timetable's applicability on that day. Still require current publication, non-archived state and resolution of relevant conflicts.

### Override and restoration rules

An override masks only its declared targets. Preserve the underlying revision and relationship. Never destructively replace the underlying facts just to implement a temporary window.

Apply supersession/amendment relationships only within their declared scope and effective window. A future-effective permanent replacement must leave the currently applicable base usable until the boundary. Partial replacement preserves unaffected fields. Archive a revision only when it has no remaining normal applicability; do not globally archive it merely because a future revision was accepted.

At expiry, recompute effective knowledge from currently accepted, applicable revisions. Do not simply restore a saved pointer: a newer permanent revision, unresolved conflict or underlying expiry may prevent restoration. If no valid underlying facts remain, report unavailable information.

For overlapping temporary overrides, accept compatible/disjoint facts together. Incompatible facts with overlapping applicability require confirmation unless the new source unambiguously amends an existing override and the changed facts qualify for automatic updates below.

Scheduler cleanup updates status, event projections and cache generations. All reads and reminder sends independently enforce time boundaries; cleanup is not the correctness boundary.

## Lifecycle and conflict granularity

| Status | Meaning |
| --- | --- |
| ACTIVE | Accepted lifecycle state; publication, validity, masking and conflict checks still apply. |
| EXPIRED | Its bounded applicability has ended. Excluded from normal retrieval. |
| ARCHIVED | Superseded or explicitly retired revision, retained for explicit recovery. |
| CONFLICTED | No undisputed usable content remains at the represented scope pending resolution. |

TEMPORARY is a validity property, not a lifecycle status. Ingesting, validating and awaiting confirmation are processing states, not additional lifecycle statuses.

For partially disputed content, keep the revision ACTIVE and record a field/scope conflict mask. Exclude the disputed fields, not the entire unrelated document. Use CONFLICTED when the whole represented scope is disputed. Conflict explanations may display both claims as disputed evidence, even though neither is eligible as an answer fact.

## Automatic changes and confirmation

| Situation | Required action |
| --- | --- |
| Exact bytes submitted again | Reuse source/content processing; no duplicate knowledge/events. Reconsider new intent hints separately. |
| Same logical fact, same value and scope | Record corroborating evidence if useful; no behavioural change. |
| Clear room-only amendment for an identified lecture/event | Apply automatically only with reliable extraction, explicit applicability and an obvious replacement relationship. |
| New high-confidence, non-conflicting knowledge | Publish after validation without routine confirmation. |
| Exam date/time change, assignment/deadline change, cancellation, or consequential rule disagreement | Show both claims and sources; ask before applying. |
| Unclear scope, source relationship, caption validity, extraction or classification | Ask the specific missing question; do not guess. |
| Different courses/sections/terms with no overlapping applicability | Keep distinct; do not manufacture a conflict. |

Treat uncertainty in evidence, interpretation and applicability separately. A model's self-reported confidence is a signal, never the sole authorization for publication or automatic replacement. Confidence thresholds require fixture calibration in [TESTING.md](TESTING.md).

Initial evidence gate for automatic acceptance: every material accepted field has source traceability; zero unresolved critical validation errors, material ambiguity or important conflicts; established scope and replacement relationship where relevant; and the extraction route passes representative acceptance fixtures. Important conflicts always require resolution regardless of a score. These checks do not prove arbitrary interpretation correct. Numeric classification/extraction/retrieval cutoffs require labelled test calibration; never hard-code a self-reported score such as 0.95 as a publication probability.

Once reliable evidence establishes an important conflict, atomically register the disputed fields and suspend dependent reminders before waiting for the user. An unreadable new upload alone is not sufficient evidence to suspend existing knowledge; request clarification about the unreadable candidate.

The conflict prompt shows old claim, new claim, affected date/scope and original filenames/upload dates. Accept keep-old, accept-new, keep-both-with-distinct-scopes or an explicit correction only when applicable. Store the decision and its exact revision references. If those references changed before the reply, show the updated comparison; do not apply a stale confirmation.

On resolution, update masks, accepted revisions, events and future unsent reminders consistently. Facts unrelated to the conflict remain usable throughout. A date/time conflict suspends event reminders that depend on that scheduling fact; a disputed room need not suspend a date-only reminder that omits the room, unless the event identity/applicability itself is disputed.

## Versioning, duplicate handling and recovery

Preserve immutable original sources and Markdown content revisions. Store content hashes, lineage, amendment/override relationships, extraction version and source locators. Reprocessing the same source under a changed extractor creates a candidate revision; it cannot silently overwrite accepted facts.

Distinguish duplicate Telegram delivery, duplicate original content and semantically overlapping documents. Telegram delivery deduplication is transport control. A content hash avoids repeated extraction. Semantic overlap uses logical identity/scope comparison and the conflict rules. The same filename with changed bytes is not a duplicate.

Repeated uploads keep the original canonical receipt date for original-source attribution; receipt metadata can record later submissions without chat text. A new caption on duplicate bytes may initiate a confirmation; it cannot silently alter existing validity.

Undo targets an identified publication or decision. Preview ambiguous or consequential reversals, including affected events/reminders. Execute a compensating decision/publication that preserves history; do not delete evidence or reuse an old revision as active without revalidation. Rebuild derived projections and caches from the resulting accepted state. Already delivered reminders cannot be unsent by undo and must not be automatically replayed.

Archive originals/revisions for recovery and debugging. Normal retrieval never broadens into archives when active results are insufficient. An explicit recovery request may inspect history, clearly labelled with status and dates. Retention/deletion policies are not automatic product features.

## Fixed reminders

This section is the policy owner; [ARCHITECTURE.md](ARCHITECTURE.md) and [DATABASE_DESIGN.md](DATABASE_DESIGN.md) define execution and records.

| Event type | Calendar-day offsets before local event date |
| --- | --- |
| Exam | 5, 1, 0 |
| Assignment/deadline | 3, 1, 0 |
| Lecture, notice date or other event | No automatic reminder |

Use Asia/Kolkata. Each offset is normally due at 09:00 on its local date. Only the same-day offset for a start earlier than 09:00 moves to one hour before the known start. This can fall on the preceding date. At exactly 09:00, do not move the reminder; the no-send-at-start rule skips it. Offsets use local calendar days, not a multiple of 24 hours from the event instant.

Never send at or after a known event start/deadline. Date-only events remain date-only: use 09:00 and stop at the next local midnight after the event date; do not claim a deadline of midnight. A missing event date blocks reminder creation and requires clarification.

Create only future due occurrences when publishing or revising an event. Do not synthesize catch-up reminders for offsets already elapsed. Engineering delivery default: an occurrence already scheduled before its due time can be dispatched/retried until the earlier of the end of its due local date and the event cutoff. After that it is SKIPPED. This allows normal Scheduler delay without converting stale offsets into later-day reminders.

An unresolved conflict suspends affected occurrences. After resolution, recompute and schedule future occurrences only; skip past suspended offsets. A late upload, revision or recovery never backfills after the event/deadline. Resolved accepted date changes recalculate future offsets; sent records remain immutable and auditable.

Every send uses current event/knowledge eligibility. AI can extract candidates but cannot select offsets, retry messages, alter delivery times or execute scheduling.

## Acceptance invariants

- No caption-only date window takes effect without confirmation.
- Temporary overrides, event projections, query answers and reminder checks use the same effective-knowledge resolver.
- An important conflict cannot leak disputed facts through a cache, vector hit, response or affected reminder.
- Expiry/undo cannot revive a newer-superseded or otherwise invalid revision.
- Original evidence and user decisions remain distinguishable after every update.
