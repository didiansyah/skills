# Plan: grill, spec, tickets, PR

Source: Matt Pocock's skills (`grilling`, `to-questionnaire`, `domain-modeling`, `to-spec`, `to-tickets`, `wayfinder`, `implement`, `tdd`, `code-review`, `pr`, `handoff`), adapted to craft.

**Contents:** 0 The flow · 1 Grilling · 2 Questionnaire for stakeholders · 3 Glossary and ADRs · 4 Spec · 5 Tracer-bullet tickets · 6 Wayfinder · 7 Implement loop · 8 PR template · 9 Handoff · 10 Feeding the UI modes

---

## 0. The flow

```
grill (+ glossary/ADRs) ──► can talk settle every question?
   │ no: one needs a runnable answer ──► explore.md (variants or logic prototype) ──► back with a one-line verdict
   ▼ yes
multi-session build? ── yes ──► spec ──► tickets ──► implement per ticket (fresh context each)
                     └─ no  ──► implement in this context
implement = tdd at agreed seams → review (two axes + Craft Review mode) → commit → PR
huge, foggy effort ──► wayfinder map first, joins at "spec"
```

- Keep grill, spec and tickets in ONE unbroken context. Don't compact or clear until the tickets exist.
- Decide on context only between phases: continue (default when the next phase needs this one as its source) > clear (context irrelevant) > handoff (new harness, repo or person) > subagent (AFK work such as review) > compact with an instruction.
- **Scope of "ask once".** SKILL.md §2.2 (one message of questions) governs Direct and Build. Plan mode is the exception: when the user asks to plan, spec, grill, or the decision tree is deep, run grilling rounds until the frontier is empty.

---

## 1. Grilling

Interview relentlessly until you and the user share one understanding.

- Map a **design tree**: each decision branches into the decisions hanging off it.
- Work in **rounds**. The **frontier** is every decision whose prerequisites are settled (askable now without guessing an unheard answer). Ask the **whole frontier in one round**, numbered, each with your **recommended answer**. Then wait.
- **Word each question so "yes" accepts the recommendation.**
- Each round's answers reshape the tree; recompute the frontier. A question that depends on one still open this round belongs to the next round.
- **Facts are your job; decisions are the user's.** Anything you can look up (files, tokens, routes, installed libraries, data shapes), look up yourself or dispatch a fresh subagent. Don't block: only questions downstream of a running lookup wait; ask the rest now.
- **Plain chat text**, never a harness question tool or picker. Pickers push toward pre-baked multiple choice; grilling wants answers in the user's own words.
- **No question cap.** Some plans need three questions, some need fifty. The user steers with words ("wrap up").
- Done when the frontier is empty: every branch visited, nothing silently assumed. **Do not act until the user confirms shared understanding.**
- A question you keep rephrasing because talk can't settle it: stop, build the throwaway (explore.md), look, answer in one line.

Round format:
```markdown
**Q1. <question title>**
<question body; may be several paragraphs and list choices>
Recommended: <your answer>

---

**Q2. <question title>**
<question body>
Recommended: <your answer>
```

UI frontier questions worth asking early: the one action the screen drives; who uses it and how often (frequency map); which states exist (empty, loading, error, partial, full, permission-denied); what real material exists (screenshots, numbers, quotes, logos; never invent); the tone extreme; what is explicitly out.

Success signals: answerable by number; later rounds ask things round 1 couldn't; many questions, few rounds; it stops and asks for confirmation instead of starting work.

---

## 2. Questionnaire for stakeholders

When a decision needs someone who isn't in the chat (client, PM, legal, a user). **Grill the send, not the subject:**
1. Who is it going to? (role, expertise, relationship). One exchange.
2. What do you need back? (the specific decisions and facts). One exchange.
3. Write `to-questionnaire-<slug>.md`, covering every needed item, most important first, grouped by theme under `##` once there are more than a handful.

```markdown
# <Questionnaire title>

**Purpose:** why this questionnaire exists and the decision riding on it.

**From:** <the user>, **To:** <the recipient>, **How your answers will be used:** <where they go>

## Context
One paragraph orienting a recipient who wasn't in the user's head.

## How to answer
Deadline and rough effort. Partial answers and "I don't know" are useful: flag anything you're unsure of rather than skipping it.

## <Theme heading>
### <One question, one idea>
_Why this matters: <only where the question could be misread>_
>

## Anything else?
A closing catch-all: anything we didn't ask that we should know?
```

Each question is one idea, never compound, with an answer stub beneath. Sending it is the user's action.

---

## 3. Glossary and ADRs (domain modeling)

An active discipline during grilling, not just reading.
- **Challenge against the glossary:** "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"
- **Sharpen fuzzy words:** "You say 'account': the Customer or the User?"
- **Force boundaries with scenarios:** invent edge cases that make a concept's edge precise.
- **Cross-check the code:** surface contradictions between stated behavior and the code.
- **Update `GLOSSARY.md` inline** as each term resolves; don't batch. No implementation details.

```markdown
# {Context Name}

{One or two sentences: what this context is and why it exists.}

## Language

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request
```

Rules: be opinionated (pick the best word, list the rest under `_Avoid_`); 1–2 sentence definitions of what it IS; project-specific terms only. Files are created lazily: root `GLOSSARY.md` + `docs/adr/`; multi-context repos use `GLOSSARY-MAP.md` → `src/<ctx>/GLOSSARY.md`.

**ADR** only when all three hold: hard to reverse, surprising without context, the result of a real trade-off. `docs/adr/NNNN-slug.md`, sequential:
```markdown
# {Short title of the decision}

{1-3 sentences: the context, what we decided, and why.}
```
Qualifies: architectural shape, integration patterns, lock-in tech choices, explicit scope no's, deliberate deviations from the obvious path, constraints invisible in code. UI examples: "No dark mode in v1", "Toasts via Sonner, not a custom stack", "One accent, no secondary brand color".

Consumer rule: before exploring, read `GLOSSARY.md` and relevant ADRs; if absent, proceed silently. Use glossary nouns in specs, tickets, test names, PRs and UI copy. When output contradicts an ADR, flag it: "_Contradicts ADR-0007 (…), but worth reopening because…_"

---

## 4. Spec

Synthesis, not interview: write from what was already decided. Invent nothing to fill a section.
1. Explore the repo if not done; use glossary vocabulary; respect ADRs.
2. **Sketch the test seams.** Prefer existing seams, the highest possible; the ideal number is one. Check the seams with the user.
3. Write the spec with the template; publish to the tracker (label `ready-for-agent`) or save locally as `.scratch/<feature-slug>/spec.md`.

```markdown
## Problem Statement
The problem that the user is facing, from the user's perspective.

## Solution
The solution to the problem, from the user's perspective.

## User Stories
A LONG, numbered list of user stories. Each user story should be in the format of:
1. As an <actor>, I want a <feature>, so that <benefit>
  e.g. 1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions
A list of implementation decisions that were made. This can include:
- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions
Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.
Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts.

## Testing Decisions
- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope
A description of the things that are out of scope for this spec.

## Further Notes
Any further notes about the feature.
```

UI additions under **Implementation Decisions** (when the feature has UI): the chosen direction and the variant verdict (explore.md, with the `prototype/<name>` branch pointer); the states list (§10); the motion budget per surface from the frequency gate; tokens reused; breakpoints that change layout.

Success signals: project nouns, not PM boilerplate; Out of Scope lists real refusals (often the most useful lines); the user remembers making every decision in it.

---

## 5. Tracer-bullet tickets

1. **Gather context** from the conversation; if given a spec path or issue ref, fetch the full body and comments.
2. **Explore** (optional). Glossary vocabulary; respect ADRs. Look for prefactoring: make the change easy, then make the easy change.
3. **Draft vertical slices:**
   - Each slice is a narrow but COMPLETE path through every layer (schema, API, UI, tests). Never a horizontal layer.
   - A finished slice is demoable or verifiable on its own: "what can I demo when this is done?" is answered with behavior.
   - Each slice fits one fresh context window.
   - Prefactoring first. Each ticket declares its **blocking edges**; no blockers = start now.
   - **Wide refactor** (one mechanical change fanning across the codebase, e.g. renaming a token or column): **expand–contract**. Expand (new form beside old) → migrate call sites in batches sized by blast radius, each its own ticket blocked by expand, CI green throughout → contract (delete old) blocked by every batch. Batches that can't stay green alone share an integration branch, all blocking a final integrate-and-verify ticket.
4. **Quiz the user** with a numbered list. Each item: **Title**, **Blocked by**, **What it delivers**. Ask: granularity right? edges only genuine gates? merge or split any? Iterate until approved. **Publish nothing until the user approves the breakdown.**
5. **Publish in dependency order, blockers first**, so edges reference real ids. Use native blocking and sub-issue links where the tracker has them (`gh issue create --parent … --blocked-by …`), else the `Blocked by` section. Label `ready-for-agent`. Never close or modify the parent issue. Then work the **frontier**: tickets whose blockers are all done.

Tracker issue template:
```markdown
## Parent
A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build
The end-to-end behaviour this ticket makes work, from the user's perspective, not layer-by-layer implementation.

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by
- A reference to each blocking ticket, or "None (can start immediately)". Omit this section when blockers were set as native edges.
```

Local ticket, one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (never one combined file):
```markdown
# <NN>: <Ticket title>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None (can start immediately)".

**Status:** ready-for-agent

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2
```

No file paths or code snippets (same prototype exception as the spec). UI acceptance criteria name observable states and checks, e.g.:
- [ ] Empty, loading, error and full states render at 375 and 1280.
- [ ] Keyboard-only: every action reachable, focus ring visible.
- [ ] Reduced motion keeps the opacity crossfade; no motion on keyboard actions.
- [ ] Worst-case data (harden.md) does not overflow.

---

## 6. Wayfinder (huge, foggy efforts)

For work bigger than one session can hold, where the way isn't clear yet. **Plan, don't do:** tickets resolve decisions, not deliverables. The pull to just build marks the map's edge: hand off to §4.
- Name the **destination** first (a spec, a decision, or an in-place change).
- One map issue labelled `wayfinder:map`, an index not a store:
  ```markdown
  ## Destination
  ## Notes
  ## Decisions so far
  - [<closed ticket title>](link): <one-line gist>
  ## Not yet specified
  ## Out of scope
  ```
- Child tickets: `## Question` + the decision, sized to one session, labelled `wayfinder:research | prototype | grilling | task`. Prototype tickets ("how should it look or behave") run explore.md. Claim by assigning before work.
- **Fog of war:** ticket a question only when it can be stated precisely now; otherwise it waits in "Not yet specified". Don't pre-slice fog.
- Resolve **one ticket per session** (research may run in parallel). Refer to tickets by linked title, never bare numbers.

---

## 7. Implement loop

Per ticket, in a fresh context: state the ticket title, then:
1. **TDD at pre-agreed seams.** Write the seams down and confirm them before any test. One red test → minimal green → next. Test behavior through public interfaces; mock only system boundaries (network, time, payment, email). Refactoring belongs to review, not the loop.
2. **UI pass.** Run the modes the ticket touches: Build → Polish → Animate → Harden (SKILL.md §1), verifying by looking.
3. **Review on two axes, in parallel fresh subagents, never merged:** Standards (documented rules + smell baseline) and Spec (missing requirements, unrequested behavior, wrong-looking behavior). For UI diffs add a Craft **Review** pass (review.md) with its findings table.
4. **Commit** to the current branch; typecheck regularly, single tests often, full suite once at the end.

---

## 8. PR template

Skip preambles; brief prose; glossary nouns.

```markdown
## Summary

<smallest visual that makes the point: component tree for UI structure, a diff block, call tree, or Mermaid>

## Evidence

- **Before:** <screenshot at 1280 and 375 / failing test run>
  **After:** <screenshot at 1280 and 375 / passing test run>

## Merge Danger

**Door:** <one-way or two-way>

<optional: description>

**Blast Radius:** <one word: layout-shift | mobile | consumers | none …>

<optional: potential ramifications of merge>
```

- **Summary for UI:** a component tree with the state and module boundaries that matter, or a component diff:
  ```diff
    <SessionPage>
      useSessionEvents()
      <SessionToolbar>
  +     <RunSkillButton />   (packages/ui)
  ```
- **Evidence:** screenshots are S-tier (before/after, same viewport, same data; add dark mode and a short clip for motion). Execution evidence (tests, console) is A-tier: show the exact test failing, then passing.
- **Merge danger for UI:** name layout shift, mobile responsiveness, token or theme changes that ripple, and z-index or focus-order changes. One-way door = destructive or hard to reverse (data migration, removed route, public token rename).

---

## 9. Handoff doc

When moving to a new harness, repo, person or a side task: write a compact doc in the OS temp dir (not the workspace). Include the goal, decisions so far, open questions, a "suggested skills/modes" section (e.g. "craft: Polish → Review"), and pointers to specs, ADRs, tickets and commits by path or URL rather than copying them. Redact secrets and personal data. Tailor it to the next session's focus.

---

## 10. Feeding the UI modes

| Plan artifact | Becomes | Used by |
|---|---|---|
| Problem + Solution | The one action the screen drives; the page's job | Direct (`direction.md`) |
| User stories (actors) | Audience + frequency map (who hits which surface how often) | Direct, Animate (frequency gate, SKILL.md §5) |
| User stories (each "I want…") | **States list**: per story, the screen states it implies | Build, Harden |
| Implementation decisions: variant verdict | The **direction**: chosen variant, its axis, what was rejected | Direct → Build |
| Glossary | UI copy nouns, labels, empty-state wording | Build, Review (writing) |
| Out of scope | Things not to design (no dark mode, no mobile app) | All modes |
| Testing decisions + seams | Where UI behavior is asserted | Implement loop |
| Ticket acceptance criteria | The Verify checklist for that slice | Build, Review |

**States list** (write it before building any component the stories touch):

| Story # | Surface | Empty | Loading | Error | Partial | Full | Permission / offline |
|---|---|---|---|---|---|---|---|
| 3 | Invoice list | "No invoices yet" + create action | Skeleton at row geometry | Inline retry, keeps filters | 1 row | 50+ rows, paginated | Read-only banner |

Path: **spec user stories → states list → direction (one recommendation, or explore.md when it must be seen) → build with house values → harden against worst-case data → review → PR with before/after.**
