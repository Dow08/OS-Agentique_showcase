<p align="right"><b>English</b> · <a href="../fr/construction.md">Français</a> · <a href="../../README.md">← Back</a></p>

# How it was built

Seven weeks, 788 commits, 148 written decisions, ≈ 253,000 lines of TypeScript, and not one line of code typed by hand: everything was produced by AI agents (Claude Code, occasionally Cursor) that I directed. This page tells **in which order** the bricks appeared, **why** each one was born, and **how** I kept the project on track.

Numbers refer to the ADRs (*Architecture Decision Records*) of the private repository: one decision = one page, with its context, the choice, the rejected alternatives and the evidence.

---

## Day 1 — 15 September: write before coding

The project starts with a decision of principle: **an agent OS, not a pile of scripts** (ADR-001). Before any feature, eight design documents:

| Document | What it fixes |
|---|---|
| Architecture map | eight planes (interaction, intent, planner, agents, providers, executors, coworking, self-improvement), each projected onto what already exists |
| Contracts | the interfaces between planes, validated before any code (ADR-006) |
| State machines | the life of a task, of an approval, of an executor |
| Threat model | what an agent can do wrong, and what stops it |
| Test plan | isolated tests, contract tests for every external tool |

Two founding choices: build **around** an open-source engine (Hermes Agent) rather than write one (ADR-003), and make the engine's native *hooks* my **event bus** (ADR-004). A unique task identifier correlates every log (ADR-005). The first of these choices held for six weeks: it was undone in week 6.

## Week 1 — the core, then the voice

- **The orchestration core**: a task is a state machine; execution only starts at the “ready” transition (ADR-007); the effective risk is the **maximum of what is declared and what the task can actually do** (ADR-015); “done” requires every declared verification to pass (ADR-016).
- **The executors**: Claude Code, Cursor, an allow-listed shell, and the engine itself; bounded output, process tree really killed (ADR-018).
- **Baseline security**: secret filter before any write, never a prompt as a shell argument, shape validation against command injection (ADR-014, ADR-019).
- **The planner** (a sequential chain, not an arbitrary graph, ADR-010), **Coworking** (ADR-012) and the **friction engine** that only learns from actually detectable signals (ADR-013).
- **One single contact**: the director, read-only (ADR-021, ADR-022).
- **Jarvis**: local voice conversation, through the same door as everything else (ADR-025), then one-click launch and voice acknowledgements (ADR-029).

## Week 2 — departments that do real work, and seeing the team

- **HR department**: the CV Creator as a work tool, personal data never on GitHub (ADR-026); local brain first, deterministic evaluation, automatic takeover by Claude when the answer is not good enough (ADR-027); the **CV pipeline** (Camille writes, Sacha reviews, delivery only without a blocking finding, ADR-038); then **job search** in five lots: criteria, watch, sorting, Notion mirror, automation (ADR-042 → 049), with the “web ⊕ personal data” invariant (ADR-045) and questions to the operator rather than inventions (ADR-047).
- **Defensive security department**: documentary knowledge, no offensive execution (ADR-031); the offensive department forced to the maximum risk level (ADR-032, ADR-033).
- **Seeing the team**: detailed live tracking (ADR-034), deterministic drift alerts (ADR-035), access matrix (ADR-036), profiling “who did what, with which tools, for how much” (ADR-039), orphan tasks settled (ADR-040).
- **Coworking** as a shared workshop (ADR-043), **action fingerprint**: an approval is bound to what was approved (ADR-044), then conversation + Coworking merged into one home (ADR-055).
- **Brain selector**: local or Claude, for the whole team, in one click (ADR-056); **Iris**, the only agent driving a browser (ADR-057).

## Week 3 — the kernel, the SOC, the Lounge

An **internal audit of the agentic runtime** (29 September) was the turning point: it listed the primitives missing for the rules to be *enforced* rather than *displayed*. The answer was the kernel:

- **Named kernel**, the task as a system object, real `deny` and `RETRYING` (ADR-077); **caller identity**: operator, local token or agent, with different rights (ADR-078); **tool-level kill switch** across all profiles (ADR-079); **child processes** with no inherited secret name (ADR-080); **capability tokens** checked on every tool call (ADR-081); **single scheduler** for every process (ADR-083).
- **Local SOC**: Security Control Plane backed by the kernel, catalogue broker, UAC elevation per action (ADR-061); high-risk approval under **Windows Hello** (ADR-063); chained audit, seal, emergency stop, safe mode (ADR-069); agent egress registry (ADR-071); Sysmon with a custom configuration (ADR-072).
- **The director delegates, it does not do**: no more web, files or terminal for it, an MCP server as the only door (ADR-070).
- **Agents' lounge**: visible thread, proposed memory notes (ADR-084), proposed projects (ADR-088), tickets and mentions on the local brain (ADR-090), weekly retro and unattended operation (ADR-096, ADR-097). **Pending memories** (ADR-086, ADR-089).
- **Digital commerce department**: frame and deterministic base (ADR-082), read-only hosting gateway (ADR-085), the department and its “web ⊕ privilege” boundary (ADR-087), site and pipeline (ADR-092), gated content (ADR-093), opportunities and legal advisor (ADR-095).

## Week 4 — industrialise, then redesign

- **Server administration panel**: administration shell by recipes, chained journal, SSH console (ADR-102); **autonomous market watch**, closed sources, nothing published without me (ADR-103).
- **Frontend gate**: a written design system, a design lint, Playwright on an isolated instance with fictional data, screenshots at three widths; nothing merges without a green gate (ADR-104).
- **Write transactions and policy as data**: isolated worktree, diff with a second approval, sealed and simulable risk rules (ADR-105).
- **Four new departments or roles** in one project: Sales (ADR-106), External audit (ADR-107), post-mortems (ADR-108), career mentor (ADR-109); **Back office in real conditions** (ADR-113).
- **Orbital redesign of Jarvis**: HTML prototypes validated first, then ring, hexagon, cockpits per department, unified bar, Team view, GRC cockpit, Systems view (ADR-112): seven lots delivered in two days, each built by one session and verified by another.
- **One incident, one post-mortem, one rule**: a kernel key overridden in a verification instance had signed the real audit log; the log now refuses a foreign key and any unexpected signature must be attested by me (ADR-114).

## Week 5 — 17 to 23 October: the pulse, the coach, and a smouldering failure

- **Career coach**: Basile moves from a roadmap to real follow-up in five sprints: objective plan and assumptions to confirm, official links checked by code *before* reaching me, a "This week" card, skills and quizzes, mock interview, applications and market by job, first-run experience (ADR-119 → 121, ADR-130, ADR-132 → 134).
- **The Pulse**: a registry of capped routines, a timer on the PC, a server reading (ADR-123); a nightly checker for links in documentation and memory (ADR-122); a morning digest written by the director and verified by code (ADR-125).
- **The Brain**: a visual map of agents and memory, in six views (ADR-124).
- **Measure before enabling**: a bench compares ways of injecting the map into the director. By default, it is not injected (ADR-126).
- **The smouldering failure**: an upstream engine update (17 October) routes turns through a shared gateway, isolated per profile rather than per task. Found on 19 October: a turn could inherit the identity and mode of another. A workaround restarts it for each turn (ADR-129), and a trial runs agents under a separate Windows account (ADR-131). They hold, but at one turn at a time.

## Week 6 — 26 to 29 October: leaving the engine

- **Brain v3**: the Dashboard becomes the home, with the Second brain behind it (ADR-138).
- **The eye**: an attached image is read by Lucie, an agent with no tools. The director receives a card, never the image. An instruction hidden in a screenshot stays data (ADR-139).
- **Agent bureaus**: 161 specialists imported from an open-source library (VoltAgent, MIT licence), in 9 bureaus with one manager each. Deterministic importer, pinned source, sealed files, no writing (ADR-140).
- **The forge**: a writer → bench → judge loop produces prompts and skills, scored on test cases, never deployed without my review (ADR-141).
- **The cost of a turn**: a measured breakdown, weight removed from the director's prompt (ADR-142).

On 27 October, everything stops. Activating the 152 profiles of eight bureaus brings the total to 211. The gateway then takes 3 to 5 minutes to start, while the engine only waits 60 seconds, with no setting to change it. No turn goes through, for any agent, from 14:15 to 20:33.

My conclusion: the engine was now only giving me the model ↔ tools loop, an MCP client, memory, skills and the session. Everything else was already mine. I decided to replace it (**ADR-144**): everything under my Windows account, Claude through its official client, an interchangeable brain, an unchanged interface.

The plan was written on the 27th, reviewed by an external AI, then built overnight by parallel sessions, lot by lot. By the morning of the 28th, the code no longer contained Hermes Agent. **Jarod stayed the director's name.**

| Measure | Hermes Agent | Native engine |
|---|---|---|
| Same question, 5 parity questions | 98,267 tokens, 58 s | 4,509 to 9,406 tokens, ~5.5 s |
| Director's turn, delegation included | 25,594 tokens, 66 s | 19,330 tokens, 9 s |
| Engine security gate | — | 36 cases out of 36 |
| Real turn per department | — | 9 out of 9 |

What it cost: upstream updates and the laptop's free provider. The separate account for agents and its runner, tried the week before, were removed: **everything runs under my account**, and the real bound on a write is the worktree and the approved diff. The Linux code is written and tested under Windows, but **nothing has run on a real Linux machine yet**.

In the same stretch: a work-in-progress mode for the seal, with a single re-seal at validation (ADR-146).

## Week 7 — 2 to 4 November: the Store, and the audit that caught it

- **Store department**: running a local food shop, inside the Sales cockpit, with six read-only agents (ADR-148). Staff schedule and sales per paid hour (VHP), safety register and HACCP, document reconciliation, local watch, weather.
- **A product and UX audit, the same day**: the first version was declared validated while several integrations were fake or unreachable. The audit listed 11 points "declared delivered but false". I had the department's document rewritten to state what is real, then had those points rebuilt: a single scheduling engine, VHP computed on real hours, contracts shared between server and interface, reactivity, a Dashboard as a wall of widgets, a command palette, a responsive kit.
- **The schedule, reworked on evidence**: it could not find a solution on a real week. The code now explores with backtracking, on a fixed seed. Thomas and Claire then review it, and the weeks I validate serve as models.
- **Departments consult the bureaus**: any department agent can ask a specialist, except private HR and External audit (ADR-149).
- **Fewer tokens**: routing by tier (watch on Haiku, code on Sonnet, scheduling and architecture on Opus), enabled only by my choice (ADR-150); a bureau mission in one turn in the common case, with one in five reviewed by the manager (ADR-151). Replaying 83 real turns gives −15%: a projection, not a measurement.
- **The local brain**: waiting follows the model's actual progress, 480 s at most, and the stable prefix is read ahead (ADR-153).

---

## What changed course along the way

| What I believed | What I learned | The trace |
|---|---|---|
| Security can live in the UI | a rule not enforced by the system is a courtesy | audit of 29 September → kernel (ADR-077 → 083) |
| A local model is enough everywhere | it is enough for the Lounge, the Red Team and the back office; the director and the development department need more | brain selector (ADR-056), tiers (ADR-150) |
| One approval is enough | an approval without a fingerprint approves anything; a write without a transaction is irreversible | ADR-044, ADR-105 |
| Agents can write their own memory | they propose, I validate; otherwise memory drifts | ADR-064, ADR-086 |
| The director can “do a little” | as soon as it does, it works without the competent agent's context | ADR-070 |
| Screenshots prove the UI | only if taken on fictional data and regenerated by the gate | ADR-104 |
| You should not rewrite an engine | true on day 2; after six weeks, the engine only added its loop, and it was blocking everything | ADR-003 → ADR-144 |
| A separate Windows account would bound the agents | the trial worked, at the price of repeated permission failures; I chose the single account, an accepted limit | ADR-131 → ADR-144 |
| "Tests green" means "delivered" | a feature can pass its tests on a fake and do nothing for real | audit of 2 November, ADR-148 |

---

## The method, concretely

1. **A brief**: what I want, in plain language, by voice or keyboard.
2. **A gate**: WHAT · WHY · INTERFACE · SECURITY · TEST · ROLLBACK, written by the agent, reviewed by me. No code before.
3. **A design document** for big projects, split into **lots**; each lot is delivered green (tests, lint, gate) before the next.
4. **One session per lot, in its worktree**; two sessions never write the same files. A **separate verifier** reviews, runs the gate, measures on screen on an isolated instance.
5. **One ADR** per decision, written in the same lot as the code.
6. **An hourly checkpoint** that commits and pushes, with a secret and private-data detector.
7. **Regular audits** (application, runtime, UX) and a **post-mortem** per incident.

What I take away for an employer: vibe coding does not replace method, it makes it **indispensable**. The clearest proof is week 7: without an audit, a "validated" version would have stayed in place. The system survived an engine change in one night because the contracts, the tests and the kernel did not depend on it.

---

Next: [**Design & architecture**](design.md) · [**The team**](team.md) · [**Key decisions**](decisions.md)
