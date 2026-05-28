---
name: tech-design
description: Draft, review, pressure-test, compare, and polish technical design documents for product, API, SDK, infrastructure, storage, migration, reliability, security, privacy, performance, data model, or architecture work. Use when Codex should help engineers make and defend a technical decision, turn rough notes into a review-ready TD, review an existing TD, compare alternatives, or plan rollout/testing/migration/rollback for a proposed technical change.
---

# Tech Design

Help engineers create clear, reviewable technical designs. Do not merely fill a template; help the author make and defend a technical decision.

## Operating Modes

Infer the mode from the user's request. Ask the user to choose only when the intent is genuinely ambiguous.

- **Draft mode**: Create a TD from an idea, issue, PRD, thread, or notes. Output a structured design with assumptions and open questions.
- **Review mode**: Review an existing TD. Output the design summary, strongest parts, unclear areas, missing risks/requirements, weak assumptions, suggested edits, and likely reviewer questions.
- **Sparring mode**: Help explore a design before it is settled. Output clarifying questions, possible approaches, trade-offs, and the recommended next step.
- **Decision mode**: Compare alternatives and recommend one. Output options, recommendation, rationale, what would change the recommendation, owner, deadline, reversibility, and follow-ups.
- **Compression mode**: Turn messy notes into a polished TD while preserving intent, marking assumptions, and making gaps explicit.

## Output

Whenever the deliverable is a TD (draft, compression, or a revised document), output the **complete** document, in full, every time -- never a summary, a diff, or only the changed sections, even on a one-line revision. The author copies the whole thing into Notion in one shot via `/copy`, so the document must stand alone.

- Render the TD as raw markdown directly in the response, in the dialect Notion understands on paste. Do **not** wrap the document in a code fence -- Notion pastes raw markdown natively, but treats a fenced block as a single code block.
- Keep the document as one contiguous block so a single `/copy` captures all of it. Put any out-of-document commentary (review findings, sparring questions, meta-notes) after it, below a `---` divider, so the copyable document stays clean.
- On revisions, re-emit the entire updated document, not just the parts that changed.

Stick to the markdown subset Notion parses on paste:

- **Headings:** only `#`, `##`, `###`. Notion has exactly three heading levels; `####` and deeper paste as plain text, not headings. Use `#` for the title, `##` for the numbered sections, `###` for subsections.
- **Tables:** GFM pipe tables -- they become Notion table blocks. Keep cells to inline formatting only (no block elements inside a cell).
- **Checkboxes:** `- [ ]` and `- [x]` become to-do blocks.
- **Code:** triple-backtick fenced blocks with a language hint.
- **Dividers:** `---` on its own line.
- `**bold**`, `*italic*`, `inline code`, and `[links](url)` all work. Avoid raw HTML, footnotes, definition lists, and nested blockquotes -- Notion drops or mangles them.

## Intake

Ask for missing information only when it materially changes the design. Prefer 3-5 high-leverage questions over a long questionnaire.

If enough context exists, proceed with clearly marked assumptions. Never invent facts silently; use `[TODO: ...]` placeholders where facts are unknown.

Try to identify:

- Problem statement
- Users, customers, internal teams, systems, or SDK consumers affected
- Current state
- Proposed change, if any
- Constraints
- Success criteria
- Scope and non-goals
- Known risks and open questions

Use questions like:

- What decision do reviewers need to approve?
- What breaks, slows down, or becomes painful if we do nothing?
- Who or what is affected?
- Is this reversible, partially reversible, or hard to undo?
- What is the riskiest assumption?
- What constraints matter most: time, compatibility, cost, latency, reliability, security, privacy, or migration complexity?
- What existing systems, APIs, SDKs, data models, or workflows does this touch?
- What would make this design unacceptable?

## Depth

Choose the smallest depth that fits the change.

- **Lightweight TD**: Small, reversible, low-risk changes. Use TL;DR, Problem, Proposed change, Alternatives, Risks, Rollout/validation, Open questions.
- **Standard TD**: Most product, API, SDK, and system changes. Use the full template.
- **Deep TD**: Migrations, infra, storage, protocols, security-sensitive work, or high-scale systems. Add failure modes, data lifecycle, operational plan, backfill/migration plan, cost model, load testing, rollback constraints, and support impact.
- **Critical TD**: Irreversible, high-risk, customer-visible, security-sensitive, or large-scale architecture changes. Add explicit decision record, staged decision points, formal risk register, compatibility matrix, incident response plan, and long-term ownership.

## TD Template

Use only the sections that fit the selected depth.

## 0. Document Metadata

- Status: Draft / In review / Approved / Deprecated
- Author:
- Reviewers:
- Decision owner:
- Target decision date:
- Related docs / issues / PRDs:
- Last updated:

## 1. TL;DR

Summarize in 5 bullets or fewer:

- Problem
- Proposed solution
- Biggest trade-offs
- Rollout plan
- Key risks

## 2. Context and Problem

Explain what happens today, who is affected, why it matters now, what breaks/slows down/becomes expensive, and prior art or related decisions.

Prefer concrete statements over vague claims. Replace "does not scale" with the specific scale, cost, latency, correctness, or workflow problem.

## 3. Goals and Non-Goals

Use measurable or falsifiable goals where possible.

Examples:

- Keep P95 latency below `[TODO: target]` for `[TODO: scale]`.
- Maintain backwards compatibility with existing clients.
- Reduce payload size by at least `[TODO: target]` for common cases.

Be explicit about non-goals, such as no arbitrary query language, no cross-tenant behavior change, no new authorization model, or no migration of historical data in v1.

## 4. Requirements

Include the categories that matter:

- Functional requirements
- Non-functional requirements: performance, scale, reliability, cost, latency, availability, durability
- Compatibility requirements: APIs, SDKs, clients, data formats, migrations, version gates
- Security and privacy requirements: authorization, data exposure, tenancy boundaries, auditability, PII, secrets
- Developer experience requirements: API clarity, validation errors, docs, SDK ergonomics, debuggability

## 5. Proposed Design

Explain the design at three zoom levels.

### 5.1 User-Facing Behavior

Include API surface, SDK usage, configuration, visible behavior, error states, limits, and examples.

### 5.2 System Architecture

Include components, responsibilities, data flow, read path, write path, concurrency model, and failure handling.

### 5.3 Data Model and Invariants

Include entities, schemas, relationships, invariants, idempotency rules, ordering guarantees, and data lifecycle.

Make invalid states hard to represent. State invariants as testable rules.

## 6. Alternatives Considered

For each alternative, include:

- Summary
- Pros
- Cons
- Why rejected
- When to reconsider it

Avoid weak strawman alternatives. Include at least one plausible option unless the change is trivial.

## 7. Recommendation

State the recommended option clearly.

Include:

- Chosen approach
- Why this is the best trade-off
- Assumptions it depends on
- Evidence supporting it
- What would change the recommendation
- Reversibility: reversible / partially reversible / hard to undo
- Decision owner
- Target decision date
- Follow-up decisions

## 8. Compatibility and Migration

Include backwards compatibility guarantees, feature flags, version gates, data migration, backfill plan, deprecation plan, and rollback constraints.

Treat public APIs, persisted data, user workflows, SDK behavior, and observable behavior as compatibility boundaries.

## 9. Rollout Plan

Include phases, guardrails, metrics, and rollback.

Use concrete stop conditions where possible:

- P95 latency regresses by more than `[TODO: threshold]`
- Error rate increases by more than `[TODO: threshold]`
- Data drift or correctness checks exceed `[TODO: threshold]`
- Support tickets or customer-visible failures exceed `[TODO: threshold]`

## 10. Testing Strategy

Include the relevant test types: unit, integration, contract, load, migration, failure injection, cross-version, security, and privacy tests.

Prefer tests that compare new behavior against a source of truth or existing behavior when possible.

## 11. Failure Modes and Mitigations

Use a table:

| Failure mode | Impact   | Detection | Mitigation |
| ------------ | -------- | --------- | ---------- |
| `[TODO]`     | `[TODO]` | `[TODO]`  | `[TODO]`   |

## 12. Security, Privacy, and Abuse

Include authorization checks, tenant isolation, data leakage risks, enumeration risks, PII handling, audit logs, rate limits, and abuse cases when relevant.

Designs must avoid revealing whether unauthorized resources exist through errors, timing, counts, pagination, or filtering behavior.

## 13. Operational Considerations

Include dashboards, alerts, runbooks, on-call ownership, support impact, cost impact, capacity planning, data retention, and cleanup jobs when relevant.

## 14. Documentation and Developer Experience

Include docs changes, SDK examples, migration guide, error messages, changelog, customer communication, and debuggability.

## 15. Open Questions

Use a table where possible:

| Question | Owner    | Needed by | Impact   |
| -------- | -------- | --------- | -------- |
| `[TODO]` | `[TODO]` | `[TODO]`  | `[TODO]` |

Avoid vague questions like "Need to figure out pagination." Name the decision and why it matters.

## 16. Decision Log

Track major decisions:

| Date     | Decision | Owner    | Reason   |
| -------- | -------- | -------- | -------- |
| `[TODO]` | `[TODO]` | `[TODO]` | `[TODO]` |

## Review Behavior

When reviewing a TD, act like a senior technical reviewer. Do not just polish wording.

Check for:

- Unclear problem statement
- Missing goals or success metrics
- Hidden scope creep
- Weak alternatives
- Unsupported recommendation
- Missing migration or rollback plan
- Missing security/privacy analysis
- Missing observability
- Missing test strategy
- Vague claims like "fast", "scalable", or "simple"
- Unclear ownership
- Decisions hidden inside implementation details

For every issue, provide:

- Severity: blocking / important / nice-to-have
- Why it matters
- Suggested fix

Lead with the most important risks. Separate blocking issues from polish. Include likely reviewer objections.

## Critique Pass

After drafting or recommending, include a short critique section:

- Weakest assumptions
- Riskiest parts of the design
- Missing data
- Likely reviewer objections
- What would change the recommendation

## Quality Bar

Prefer concrete, falsifiable statements.

- Replace "This will be scalable and fast" with latency, throughput, data volume, or cost targets.
- Replace "We will add an index" with key shape, read/write path impact, correctness model, and reconciliation strategy.
- Replace "Roll out gradually" with phases, eligibility, metrics, and stop conditions.

When drafting, start concise, mark assumptions, include concrete examples, make trade-offs visible, and end with open questions plus recommended next steps.
