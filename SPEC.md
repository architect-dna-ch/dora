# DORA — Definition of Done (DoD) Specification

**Project:** DORA — Dynamic Overview & Retention Assistant
**Document type:** Engineering Specification / Definition of Done
**Audience:** Engineering team, reviewers, maintainers
**Status:** Living document — must be updated as the project evolves

---

## 1. Purpose

This document defines the **Definition of Done (DoD)** for every change shipped to
DORA. A change is only "done" when it satisfies **every** criterion below. The DoD
is the shared contract between developers, reviewers, QA, and the pipeline. If any
gate is not met, the change is **not done** — it stays in progress.

The DoD applies to all work products: features, bug fixes, refactors, docs,
infrastructure, and configuration changes.

---

## 2. How to use this document

- **Before starting work:** read the relevant sections so the acceptance criteria
  shape the implementation from the start (shift-left).
- **During development:** treat the checklist as a running task list.
- **Before opening a PR:** self-review against the full checklist.
- **During review:** the reviewer verifies the checklist, not just the diff.
- **In CI:** the automated gates (Section 6) enforce what can be enforced by machine.

> A PR that passes CI but fails a human-only criterion (e.g., accessibility review)
> is **blocked**, not done.

---

## 3. Core Definition of Done

A change is **Done** when **all** of the following are true:

### 3.1 Functional correctness
- [ ] The feature/bugfix behaves as specified in the linked issue/acceptance criteria.
- [ ] All acceptance criteria from the ticket are demonstrably met.
- [ ] Edge cases and error paths are handled (empty state, loading, failure, timeout).
- [ ] No regression in existing behavior (verified by automated + manual checks).

### 3.2 Code quality
- [ ] Code follows the project's style guide and lint rules (zero new lint errors).
- [ ] No dead code, commented-out code, or debug leftovers (`console.log`, TODO without ticket).
- [ ] Naming is clear and intent-revealing; no magic numbers without named constants.
- [ ] Complexity is appropriate; no copy-paste that should be a shared helper.
- [ ] No secrets, keys, or personal data committed (see Section 7.3).

### 3.3 Testing
- [ ] New/changed logic is covered by automated tests (see Section 4).
- [ ] Tests are meaningful (assert behavior, not implementation) and not flaky.
- [ ] Full test suite passes locally and in CI.
- [ ] Coverage thresholds defined in Section 4.4 are met.

### 3.4 Documentation
- [ ] User-facing behavior is reflected in the README / user docs if applicable.
- [ ] Public functions/components have clear doc comments where non-obvious.
- [ ] Any config/environment change is documented (see Section 5).
- [ ] ADR added for significant architectural decisions (see Section 5.4).

### 3.5 Review
- [ ] At least one reviewer (other than the author) has approved the PR.
- [ ] All review comments are resolved or explicitly acknowledged with rationale.
- [ ] The author has self-reviewed the diff before requesting review.

### 3.6 Pipeline & build
- [ ] All CI pipeline gates pass (Section 6).
- [ ] The change builds cleanly in a clean environment.
- [ ] No new dependency is added without justification and license check (Section 7.4).

### 3.7 Accessibility & UX (for UI changes)
- [ ] Keyboard navigable; visible focus states.
- [ ] Semantic HTML / ARIA used correctly; no contrast violations.
- [ ] Works at the supported viewport sizes (responsive).
- [ ] Screen-reader flow verified for the changed component.

### 3.8 Performance & security
- [ ] No obvious performance regression (bundle size, render, network).
- [ ] Inputs are validated/sanitized; no new injection or XSS vector (Section 7.2).
- [ ] No sensitive data exposed in logs, DOM, or network payloads.

### 3.9 Release readiness
- [ ] Changelog entry added (if the project maintains one).
- [ ] Version bump applied per semantic versioning if this is a release change.
- [ ] Rollback plan considered for anything that changes data or infra.

---

## 4. Testing Strategy

### 4.1 Test pyramid (target distribution)
```
        /  E2E (10%)   \     <- few, slow, high confidence
      /  Integration (30%) \  <- medium
    /   Unit (60%)          \ <- many, fast, cheap
```
Optimize for **fast, reliable unit tests** at the base.

### 4.2 Test levels for DORA
| Level | Scope | Tooling (suggested) | Runs in |
|-------|-------|---------------------|---------|
| **Unit** | Pure logic: state machine, countdown, response aggregation, validation | Vitest / Jest | PR + main |
| **Component** | DOM behavior: overlay open/close, submit flow, progress rendering | Testing Library + jsdom | PR + main |
| **Integration** | Teacher → student → progress data flow | Testing Library | PR + main |
| **E2E** | Full user journeys in a real browser | Playwright | main + release |
| **A11y** | Automated accessibility checks | axe-core / Playwright a11y | main |

### 4.3 What must be tested
- [ ] State transitions (idle → active → complete; timer expiry).
- [ ] Countdown logic (start, tick, expiry, cancel).
- [ ] Response aggregation and progress percentage math.
- [ ] Validation (empty answer, out-of-range duration).
- [ ] Overlay open/close and backdrop behavior.
- [ ] All-students-responded completion path.

### 4.4 Coverage thresholds (enforced in CI)
- [ ] Lines: **≥ 80%**
- [ ] Functions: **≥ 80%**
- [ ] Branches: **≥ 70%**
- [ ] Statements: **≥ 80%**

> Thresholds are a floor, not a target. Critical paths (state machine, timer,
> validation) should aim for **100% branch coverage**.

### 4.5 Test quality rules
- Tests must be **deterministic** (no sleeps, no reliance on wall-clock where avoidable — use fake timers).
- Tests must not depend on execution order or shared mutable global state.
- Each test asserts one clear behavior; failure messages are descriptive.
- No testing implementation details (e.g., internal variable names) unless essential.

---

## 5. Documentation Requirements

### 5.1 README (root)
Must contain: project overview, quick start, prerequisites, install/run/test
commands, project structure, and a link to this DoD.

### 5.2 User / feature docs
- [ ] Any user-visible feature has a short usage note.
- [ ] Screenshots or GIFs for UI flows where helpful.

### 5.3 Developer docs
- [ ] `CONTRIBUTING.md` — how to branch, commit, open PRs, run checks.
- [ ] Local dev setup is reproducible from a single documented command.

### 5.4 Architecture Decision Records (ADR)
Significant decisions (framework choice, state management, data model, pipeline
design) require an ADR in `docs/adr/` capturing: **Context → Decision → Consequences**.

### 5.5 API / contract docs
- [ ] Any public interface (function, component props, endpoint) is documented.
- [ ] Breaking changes are flagged before merge.

---

## 6. CI/CD Pipeline Gates

The pipeline is the **automated enforcer** of the DoD. A merge is blocked until all
gates pass.

### 6.1 Gate order (fail fast)
1. **Format / Lint** — style + static analysis. *Fastest, run first.*
2. **Type check** (if typed) — compile-time correctness.
3. **Unit + component tests** — with coverage thresholds.
4. **Build** — production build succeeds in a clean environment.
5. **Integration / E2E** — full user journeys.
6. **A11y scan** — automated accessibility checks.
7. **Security scan** — dependency + secret scanning (Section 7).
8. **Bundle-size check** — fail if budget exceeded.

### 6.2 Branch protection (GitHub)
- [ ] `main` is protected: no direct pushes.
- [ ] Required status checks = all gates in 6.1.
- [ ] At least **1** approving review required.
- [ ] PR must be up to date with `main` before merge.
- [ ] Conversations must be resolved before merge.

### 6.3 Environments
| Environment | Trigger | Purpose |
|-------------|---------|---------|
| **Preview** | every PR | isolated demo of the change |
| **Staging** | merge to `main` | pre-release validation |
| **Production** | tagged release | live deployment |

### 6.4 Release pipeline
- [ ] Semantic version tag (`vX.Y.Z`) triggers production deploy.
- [ ] Changelog generated from conventional commits.
- [ ] Rollback is a single, documented action (revert tag / redeploy previous artifact).

---

## 7. Security & Compliance

### 7.1 Threat model (for DORA)
- **XSS** via student/teacher free-text input → must be escaped/rendered as text.
- **Injection** via any server-side query building.
- **Data privacy** — student responses are personal data (see 7.3).

### 7.2 Required controls
- [ ] All dynamic content rendered safely (no `innerHTML` with unsanitized input).
- [ ] Input length/type validation on all free-text fields.
- [ ] No secrets in client code or committed files.

### 7.3 Data privacy
- [ ] Student data handled per applicable regulation (e.g., GDPR).
- [ ] Data minimization: only collect what is needed.
- [ ] Retention/deletion policy defined for session data.
- [ ] No PII in logs, analytics, or error reports.

### 7.4 Dependencies
- [ ] Every dependency is justified and pinned/locked.
- [ ] License is compatible with the project.
- [ ] Automated dependency vulnerability scan passes (e.g., `npm audit`, Dependabot).

### 7.5 Secret scanning
- [ ] CI runs a secret scanner (e.g., gitleaks) on every push.
- [ ] `.gitignore` excludes all secret/config files; `.env*` never committed.

---

## 8. Code Review Standards

### 8.1 Author responsibilities
- [ ] Self-review the diff before requesting review.
- [ ] Keep PRs small and focused (one logical change; < ~400 lines preferred).
- [ ] Write a clear PR description: **what, why, how tested, screenshots**.
- [ ] Link the issue and note any follow-ups as separate tickets.

### 8.2 Reviewer responsibilities
- [ ] Verify the DoD checklist, not just the code.
- [ ] Check for correctness, security, a11y, performance, and test quality.
- [ ] Approve only when genuinely satisfied; request changes with specific, actionable feedback.
- [ ] Distinguish **blocking** issues from **nitpicks**; label them clearly.

### 8.3 Review etiquette
- [ ] Be respectful and specific; suggest, don't dictate.
- [ ] Author responds to every comment (fix or explain).
- [ ] No silent dismissals; unresolved threads block merge.

---

## 9. Git & Branching Conventions

### 9.1 Branching model
- `main` — always releasable.
- Feature branches: `feature/<dora-id>-<slug>` (e.g., `feature/DORA-42-checkpoint-timer`).
- Fix branches: `fix/<dora-id>-<slug>`.
- Release branches/tags: `release/vX.Y.Z`, tag `vX.Y.Z`.

### 9.2 Commit conventions (Conventional Commits)
```
<type>(<scope>): <subject>

feat(teacher): add checkpoint launch
fix(student): correct countdown expiry
docs: add definition of done
test(progress): cover aggregation math
```
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.

### 9.3 Pull request template (required fields)
- **Summary** — what and why.
- **Acceptance criteria** — how this maps to the ticket.
- **Testing** — what was run and results.
- **Screenshots** (UI changes).
- **DoD checklist** — self-verified.

---

## 10. Definition of Ready (DoR) — before work starts

A task is only *ready* to be picked up when:
- [ ] Clear, testable acceptance criteria exist.
- [ ] Scope is understood and estimated.
- [ ] Dependencies/blockers are identified.
- [ ] Relevant context (design, ADR, related code) is linked.

> DoR prevents half-defined work from entering the pipeline and keeps the DoD achievable.

---

## 11. Definition of Shipped (DoS) — after release

A release is only *shipped* when:
- [ ] Deployed to production from a tagged, CI-green artifact.
- [ ] Smoke test passed in production.
- [ ] Monitoring/alerting confirms healthy (Section 12).
- [ ] Changelog and release notes published.
- [ ] Rollback runbook verified.

---

## 12. Observability & Monitoring (for production)

- [ ] Error tracking configured (e.g., Sentry) with source maps.
- [ ] Key metrics defined: session starts, responses, completion rate, error rate, latency.
- [ ] Alerts on error-rate and availability thresholds.
- [ ] Structured logging; no PII in logs.

---

## 13. Definition of Done — Quick Reference Checklist

> Copy this into every PR description and tick it off.

```
[ ] Acceptance criteria met
[ ] No lint/type errors; code follows style guide
[ ] Automated tests added/updated and passing
[ ] Coverage thresholds met (≥80% lines/functions, ≥70% branches)
[ ] Manual + a11y check done for UI changes
[ ] Docs updated (README / user docs / ADR as applicable)
[ ] Security: no secrets, inputs validated, no XSS
[ ] Performance: no regression; bundle within budget
[ ] CI pipeline fully green
[ ] Reviewed and approved by ≥1 reviewer; comments resolved
[ ] Changelog/version updated if releasing
```

---

## 14. Automated AI Workflow — GitHub Push & Pull Requests

This section defines the **end-to-end automated workflow** that governs how changes
(especially AI-agent-produced changes) move from a local edit to a merged, shipped
state on GitHub. It is the operational companion to the DoD: the DoD says *what*
"done" means; this section says *how* a change travels through automation.

### 14.1 Workflow overview

```
[Edit] → [Verify locally] → [Branch] → [Commit] → [Push]
    → [Open PR] → [CI gates] → [AI + human review] → [Merge] → [Deploy]
```

Every step is automated where possible; humans and AI agents only intervene where
judgment is required.

### 14.2 Branch creation (automated)
- [ ] Every change starts from an up-to-date `main`.
- [ ] Branch name is auto-derived from the issue: `feature/DORA-<id>-<slug>`,
      `fix/DORA-<id>-<slug>`, or `docs/...` / `ci/...` for non-code changes.
- [ ] The agent confirms the branch does not already exist before creating it.

### 14.3 Commit discipline (automated checks)
- [ ] Commits follow **Conventional Commits** (see Section 9.2).
- [ ] A commit-hook / CI check validates the message format and fails fast on
      non-conforming messages.
- [ ] No secrets are committed — a pre-commit hook runs a secret scanner.
- [ ] Large or unrelated changes are split into logical commits.

### 14.4 Push
- [ ] The agent pushes the branch to the remote (`origin`).
- [ ] Push triggers the CI pipeline automatically (no manual trigger).
- [ ] If a push fails (e.g., rejected due to protection), the agent pulls latest
      `main`, rebases, and retries — never force-pushes over others' work without
      confirmation.

### 14.5 Pull request creation (automated)
When the branch is pushed, an automated PR is opened (or updated) with:
- [ ] **Title** derived from the conventional commit / issue.
- [ ] **Description** populated from the PR template (Summary, Acceptance criteria,
      Testing, Screenshots, DoD checklist — see Section 9.3).
- [ ] **Linked issue** referenced (e.g., `Closes #DORA-42`).
- [ ] **Labels** applied (e.g., `feature`, `bug`, `docs`, `ai-generated`).
- [ ] **Reviewers** auto-assigned per the project's CODEOWNERS.

### 14.6 CI gates on the PR (automated, blocking)
The PR is **blocked from merge** until every gate in Section 6.1 passes:
1. Lint / format
2. Type check
3. Unit + component tests (coverage thresholds)
4. Build
5. Integration / E2E
6. Accessibility scan
7. Security + secret scan
8. Bundle-size check

- [ ] Status checks are **required** on `main` (branch protection).
- [ ] The PR must be up to date with `main`; a stale PR triggers an automated
      update or a "needs rebase" block.

### 14.7 AI-assisted review loop
- [ ] An AI reviewer (e.g., a bot) performs a first-pass review: checks the DoD
      checklist, flags obvious bugs, security issues, and missing tests.
- [ ] The AI reviewer **does not** approve alone — it reports findings for a human.
- [ ] The authoring agent addresses AI findings, re-runs verification, and pushes
      follow-up commits (which re-trigger CI).
- [ ] Human reviewer(s) then review with full context; approval is required.

### 14.8 Merge (automated, gated)
- [ ] Merge is only possible when: all CI gates green, ≥1 human approval, all
      conversations resolved, branch up to date.
- [ ] Preferred strategy: **squash merge** with a Conventional Commit message, or
      **rebase merge** for multi-commit features — decided per repo.
- [ ] Merging to `main` triggers the **staging deploy** automatically.

### 14.9 Post-merge automation
- [ ] `main` is always releasable (DoD enforced at merge time).
- [ ] A tagged release (`vX.Y.Z`) triggers the **production deploy**.
- [ ] Changelog is generated from merged Conventional Commits.
- [ ] The linked issue is auto-closed on merge.
- [ ] Monitoring/alerting is active post-deploy (see Section 12).

### 14.10 AI-agent guardrails in the workflow
- [ ] The agent never merges, tags, or deploys without explicit human approval.
- [ ] The agent never force-pushes to `main` or shared branches.
- [ ] The agent never bypasses required status checks.
- [ ] If a step fails, the agent reports the failure and requests direction rather
      than working around the gate.
- [ ] All AI-generated changes are clearly labeled for human review.

### 14.11 Workflow automation tooling (reference)
| Concern | Suggested tooling |
|---------|-------------------|
| CI orchestration | GitHub Actions |
| Branch protection | GitHub branch rules |
| Secret scanning | gitleaks (pre-commit + CI) |
| Dependency vulns | Dependabot / `npm audit` |
| AI code review | Repo AI reviewer bot (human-gated) |
| Changelog | conventional-changelog / release-please |
| Deploy | GitHub Actions + environment protection rules |

---

## 15. Versioning & Maintenance of this document

- This DoD is versioned alongside the code.
- Any change to the DoD itself requires a PR and review (dogfooding).
- Review the DoD quarterly to keep it aligned with current tooling and standards.

**Document version:** 1.1.0
**Last reviewed:** 2026-09-04
