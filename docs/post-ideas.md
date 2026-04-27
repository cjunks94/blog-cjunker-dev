# Blog Post Ideas

Working notes for upcoming posts. Each section has the angle, the personal connection, and open questions to answer before writing.

---

## 1. The migration that never ends

**Angle:** Managing long-running migrations when scope shifts and you can't stop the train.

**Your experience:**
- Multi-org Salesforce migration with weekly cadences
- Partner dependencies (70+ partners relying on export jobs)
- Team handoff that drifted from advisory to ownership gap
- Changes landing faster than review capacity

**Questions to answer:**
- What decisions would you make differently about the handoff?
- How do you scope a migration that has a moving target?
- What's the right cadence for check-ins vs. just doing the work yourself?
- When do you escalate vs. absorb?

**Draft status:** Not started

---

## 2. When your test suite is the bottleneck

**Angle:** Slow CI changes team behavior before it changes deploy frequency. The fix was technical but the problem was organizational.

**Your experience (4 phases):**

Phase 1 — Custom parallel Cucumber on Jenkins
- First bottleneck was the Jenkins instance itself
- Wrote custom code to run Cucumber tests in parallel
- Spun up a dedicated Jenkins instance for tests (separated from deploys, non-prod jobs)
- Result: Times came down, but introduced inter-job DB pollution in parallel tests — consistency suffered

Phase 2 — Replaced custom parallel with a gem
- Swapped homegrown parallelization for an established gem
- Still not enough — flaky tests became the new problem
- Cut runtime in half but needed reruns more often than before — net improvement was questionable

Phase 3 — Selective parallelization
- Stopped trying to parallelize everything
- Moved only lightweight, robust RSpec tests to parallel execution
- Left flaky Cucumber tests running serially — accepted the tradeoff

Phase 4 — Jenkins → GitHub Actions with self-hosted runner
- Migrated CI off Jenkins entirely into GHA
- Self-hosted runner for performance
- Huge runtime reduction + reliability gains
- Also moved toward the strategic goal of deprecating Jenkins

**Questions still to answer:**
- What did people actually do differently when CI was 105 min? (skip tests locally? batch PRs? deploy blind?)
- What broke first — confidence or speed?
- At each phase, what was the team's reaction? Relief, skepticism, "not again"?
- Did faster CI change how people wrote tests?
- What finally killed Jenkins? Was it this effort or something else?

**Draft status:** Notes captured, ready to outline

---

## 3. Owning code you didn't write

**Angle:** Leadership post disguised as a technical one. Advisory relationships that drift, inheriting antipatterns, the politics of telling another team their approach is wrong.

**Your experience:**
- Export tool handoff (connects directly to post 1)
- Advisory role that became an ownership vacuum
- Having the right instinct but not the right words (post 1's "principle I needed")

**Questions to answer:**
- How do you give technical feedback to a team that doesn't report to you?
- When does advisory become accountability?
- What's the line between "their problem" and "your system"?
- How do you handle it when your advice doesn't land?

**Draft status:** Not started

---

## 4. Secrets rotation at scale without downtime

**Angle:** Concrete, operational, something people Google for. PCI-compliant secrets rotation with real constraints.

**Your experience:**
- AWS Secrets Manager + K8s sidecars
- Custom wrapper with pre-cutover validation
- 100+ secrets synchronized rotation
- Zero downtime requirement

**Questions to answer:**
- What was the rotation flow step by step?
- What failed during testing? What almost failed in prod?
- Why sidecars and not init containers or mounted volumes?
- What does "pre-cutover validation" actually check?

**Draft status:** Not started

---

## 5. What observability actually means when you're small

**Angle:** Contrarian take — observability at one-team scale, not Datadog-the-company scale. What works when you don't have a platform team.

**Your experience:**
- Datadog dashboards, CloudWatch, AWS pod metrics
- Built the SQS monitoring UI because existing tools weren't enough
- Real incident debugging with real constraints

**Questions to answer:**
- What's the minimum viable observability stack?
- What did you build vs. buy vs. skip entirely?
- When did "just check the logs" stop working?
- What's the first alert that actually saved you?
- How does this connect to the worker state problem from post 1?

**Draft status:** Not started

---

## 6. When your source of truth isn't

**Angle:** What happens when two systems both think they own the data. Reconciliation as an ongoing architectural pattern, not a one-time migration.

**Your experience:**
- Salesforce sync with internal systems — bidirectional data flow
- Sidekiq jobs polling and reconciling drift in sensitive partner data
- Orchestration-like components: scheduled reconciliation, conflict detection
- Partner data (70+ partners) where inconsistency has real business consequences

**Questions to answer:**
- What's the reconciliation loop? Poll, compare, resolve — or something more nuanced?
- How do you detect drift? Checksums, timestamps, field-level diffing?
- What happens when both sides changed? Who wins and how do you decide?
- How often does reconciliation run and what triggers it?
- What's the blast radius when reconciliation gets it wrong?
- How does this relate to the "stateful jobs / stateless workers" principle from post 1?

**Draft status:** Not started

---

## 7. Migrating a payments database (twice)

**Angle:** Two different migrations, two different target databases, same source. What you learn about data migration when the data is payments and you can't afford to lose a row.

**Your experience:**
- Mongo → MySQL migration
- Mongo → DocumentDB migration
- Payments data — PCI sensitivity, can't tolerate data loss or corruption
- Two migrations means you learned from the first one (or didn't)

**Questions to answer:**
- Why two migrations? What drove each decision? (MySQL for X, DocDB for Y?)
- What was the cutover strategy? Dual-write? Shadow reads? Big bang?
- How did you validate data integrity post-migration? Row counts, checksums, reconciliation?
- What broke or almost broke?
- What's different about migrating payments data vs. anything else? (PCI, audit trails, idempotency)
- Did the first migration inform the second? What did you do differently?
- How long did each take, and what determined the timeline?

**Draft status:** Not started
