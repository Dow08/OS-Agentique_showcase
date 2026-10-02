<p align="right"><b>English</b> · <a href="README.fr.md">Français</a></p>

# OS-Agentique

**A personal team of AI agents that runs on my own PC, talks with me by voice, and never acts without my approval.**

OS-Agentique turns a Windows 11 workstation into a small, well-run company of AI agents. I talk to one director, *Hermes*. He understands the request, hands the work to the right department, and every sensitive action waits for my click on **Approve**. Nothing counts as done until it has been checked.

![Jarvis interface: a voice conversation with Hermes](assets/jarvis-conversation.png)

> This repository is a **showcase**. The source code lives in a private repository; this page shows what the system does, how I designed it and why.

---

## At a glance

| | |
|---|---|
| **30 agents** in **6 departments** | Development · HR & job search · Defensive security · Red Cell · Digital commerce · Secretariat (+ a shared Coworking workspace) |
| **A kernel that keeps the team in check** | everything that runs starts through a named kernel that checks the caller's identity, grants **capability tokens** re-verified on every tool call, and can stop anything down to a single tool |
| **Brain of your choice** | switch in one click: a 25-billion-parameter local model on my own GPU (RTX 4080, **0 $** per request) or Claude (Opus 5.5, Sonnet 5, Haiku 4.5…) when a task needs more power |
| **Voice interface** | wake word “Hermès, …”, local speech recognition and synthesis, ~5 s per simple turn |
| **Human in the loop** | every medium- or high-risk action waits for my approval; a high-risk action checks my presence with **Windows Hello** |
| **1,296 unit tests passing** (0 failures, run on the day of this update) | + contract tests on every external tool and Pester tests on the PowerShell install |
| **98 written design decisions** (ADRs) | each one records the context, the choice, the rejected alternatives and the evidence |
| **Work in progress** | actively developed: new capabilities are added regularly |

---

## The idea

AI agents can now read files, write code, browse the web and run commands. That makes them useful, and it makes them risky. The tools that became popular in 2026 mostly *hand the agent your keys* and rely on its good behaviour.

I wanted the opposite: **a team I can trust because it is organised like a real company.**

- **One point of contact.** I only speak to the director. He decides and delegates; he never touches anything himself.
- **A clear org chart.** Managers plan, only designated workers are allowed to write.
- **Rules before power.** Every task is scored for risk; anything that can change something waits for me.
- **Proof, not promises.** *“The AI says it’s done”* is never accepted as proof: results are verified before a task is marked complete.
- **Memory that lasts.** The team keeps a journal and learns from what blocked it, independently of the AI model in use.
- **Private by design.** The brain can run entirely on my PC; my personal data (my CV, for example) is only ever handled by the local model or Claude, and is technically prevented from reaching GitHub.

➡️ The full story of how I designed it: [**Design & architecture**](docs/en/design.md)

---

## How it works, in one picture

```mermaid
flowchart LR
    A["🎙️ Me<br/>voice or keyboard"] --> B["Hermes<br/>the director"]
    B -->|delegates| C["Department<br/>manager"]
    C -->|assigns| D["Worker<br/>agent"]
    D --> E{"Risk check"}
    E -->|low| F["Runs"]
    E -->|medium / high| G["⏸ Waits for<br/>my approval"]
    G -->|Approve| F
    F --> H["✅ Verification"]
    H --> I["📁 Deliverable on<br/>my Desktop"]
    H --> J["🧠 Memory &<br/>lessons learned"]
```

| Asking | Approving |
|---|---|
| ![Home screen, waiting for “Hermès, …”](assets/jarvis-accueil.png) | ![An action waits for approval](assets/jarvis-approbation.png) |

---

## The team

```mermaid
flowchart TD
    H["<b>Hermes</b><br/>Director · my only contact"]
    H --> DEV["<b>Ada</b><br/>Development"]
    H --> RH["<b>Nora</b><br/>HR & job search"]
    H --> SEC["<b>Alix</b><br/>Defensive security"]
    H --> RED["<b>Strike</b><br/>Red Cell"]
    H --> TIK["<b>TIK</b><br/>Digital commerce"]
    H --> SEC2["<b>Margot</b><br/>Secretariat"]
    H -.-> IRIS["<b>Iris</b><br/>Web navigator"]
    DEV --> DEV1["Architect · Linus (developer)<br/>Grace (reviewer) · Tester<br/>ISO 27001 auditor"]
    RH --> RH1["Camille (CV) · Sacha (recruiter)<br/>Léo (job watch) · Inès (career analyst)"]
    SEC --> SEC1["Mira (detection & response)<br/>Elias (compliance) · Owen (pentest methodology)"]
    RED --> RED1["Spectre · Breach (analysts)<br/>Aegis (ethics advisor)"]
    TIK --> TIK1["Noé (design) · Jules (copy)<br/>Lina (SEO) · Maya (market research)<br/>Hugo (infra) · Clara (legal)"]
    SEC2 --> SEC21["Hélène (archivist)"]
```

![Team map: 30 agents, from the CEO to the development, HR, security, Red Team, commerce and secretariat departments](assets/team-map-30.png)

> The live map, as Hermes reads it before every decision: **30 agents**, their department, their role, and the “writes” badge for the few workers allowed to change files.

| Department | What it does for me |
|---|---|
| **Development** — Ada | designs, writes, reviews and tests code; can hand coding jobs to Claude Code or Cursor, always under approval |
| **HR & job search** — Nora | finds and sorts job offers (real sources: JobSpy, France Travail), turns a job ad into a tailored CV that is **reviewed before delivery**, drafts cover letters and interview prep |
| **Defensive security** — Alix | detection rules, incident analysis, ISO 27001 compliance, pentest methodology and reports, backed by a **local SOC** (see below) |
| **Red Cell** — Strike | adversary emulation for defensive purposes. Every task in this department is **forced to the highest risk level**: always a manual approval, never automatic, with a built-in **ethics advisor** (Aegis) who checks authorisation, scope and legality |
| **Digital commerce** — TIK | a full product department (design, copy, SEO, market research, infra, legal) to build and run a real website, behind a strict “web ⊕ privilege” boundary with sourced facts |
| **Secretariat** — Margot | recurring admin work (invoices, documents), local read only, aggregates only |
| **Web navigator** — Iris | the **only** agent that drives a browser, in the operator's sight, behind a deterministic guard |
| **Coworking** *(mode)* | a shared workspace where the team and I work on a project together, with a readable activity feed and a tamper-evident log |

➡️ Who each agent is and what it does: [**Meet the team**](docs/en/team.md)

---

## Proof of concept: see it working

Everything below was captured on the real interface, with real answers from the AI model.

| Quick stats in the side panel | Self-knowledge map |
|---|---|
| ![Profiling: sessions, tool calls, tokens, cost](assets/jarvis-profilage.png) | ![Map of the team, skills and executors](assets/jarvis-carte.png) |
| sessions, tool calls, tokens and cost read from the engine's own database, nothing invented | the director is given a real map of his team and tools before every decision |

<p align="center"><img src="assets/jarvis-mobile.png" alt="Jarvis on mobile" width="260"><br/><i>Responsive: the same interface on a phone. Here the action was refused, so nothing ran.</i></p>

➡️ A full step-by-step walkthrough: [**A day with Hermes**](docs/en/walkthrough.md)

---

## Live profiling: see what every agent does

Managing a team means knowing who is working, on what, for how long and at what cost. OS-Agentique has a full-screen **profiling console** that answers those questions for every agent. The figures are read from the engine's own records, not estimated by an AI.

**The week at a glance:** requests, typical turn time, tokens, real cost, tool calls, approvals and drift alerts.

![Profiling summary: requests, turn time, tokens, cost, approvals, alerts](assets/profiling-summary.png)

**Who worked when:** one line per agent over the last 24 hours. Blue: a task started from the interface; grey: a session run outside the interface (sub-agent, command line); red: a failure.

<p align="center"><img src="assets/profiling-timeline.png" alt="Activity timeline per agent over 24 hours" width="720"></p>

**Per agent:** number of requests, failures, working time, model thinking time, tool calls, tokens, cost and favourite tools. Clicking an agent filters every view.

![Per-agent activity table](assets/profiling-agents.png)

**Per tool:** every tool and MCP server used, how often, how fast, and by which agents. That shows at a glance which agent touches the file system, the terminal or the web.

![Tools and MCP servers: calls, latency, agents](assets/profiling-tools.png)

> **Why it matters:** observability is what turns a black box into a team you can manage. It shows what slows the system down, what costs money, and whether an agent is stepping outside its role.
>
> *The interface is in French: I designed it for my own daily use.*

---

## Live tracking: see every agent at work, as it happens

Profiling tells me what happened. **Live tracking** shows me what is happening *right now*, even when several tasks run in parallel. The goal: never lose sight of the team, and spot a drift before it becomes a problem.

**Who is working, on what, since when.** Refreshed every 3 seconds. Here, three agents work at the same time: the Architect designs an application, Mira analyses a brute-force attack, Léo runs a web watch. Each one shows its task and how long it has been running, and a click opens the detail.

![Three agents working in parallel, followed live](assets/live-tracking.png)

**Every step, as it happens.** In the conversation, each real action appears live: the tool used, how long it took, the start of what it returned, and the agent a task is delegated to.

![Live steps of a task: delegations, tool calls, review rounds](assets/step6-live.png)

**Automatic drift alerts.** Deterministic rules, not an AI, watch every running task and raise an alert when something leaves its frame: an attempt to write during a read-only task, a delegation to a sensitive department, a loop, repeated failures.

> **Why it matters:** with several agents working at once, control only exists if you can *see*. Live tracking keeps a human eye on the whole team.

---

## Choose the brain, keep the team

Hermes stays the same; only the brain behind him changes. From the **Systems** tab, I switch the whole team from a local model running on my GPU to Claude, and back, in about fifteen seconds. The conversation continues without losing its history.

<p align="center"><img src="assets/brain-selector.png" alt="Brain selector: local model or Claude, with sub-model" width="380"></p>

- **Local** when I want privacy and zero cost; **Claude Opus 5.5** for demanding work; **Haiku** when speed matters more than depth.
- **Some agents keep a fixed brain**, whatever the team uses: the director and part of the development team (Ada, Linus, the Architect) run on Claude Sonnet 5, and the **Red Cell always stays local**.
- **Guard-rails included:** only models from a strict, tested list can be chosen (no free input), and the HR agents that handle my personal data refuse any provider other than the local model or Claude.

> **Why it matters:** the right tool for each job. Power, speed, cost and privacy become a choice, not a constraint.

---

## Under the hood: a kernel for agents

At first, the interface was the safety layer. Now the safety is **in the kernel**. Everything that runs — a conversation turn, a task, a sub-agent, a scheduled script — starts through a **named kernel**, a single scheduler that shares its state across processes. Nothing runs “on the side” any more.

- **Caller identity.** Every turn knows *who* asked for it. The director himself talks to the kernel in his own name; nothing acts anonymously.
- **Capability tokens.** The kernel grants a turn a precise set of rights (which tools, what scope), and that token is **re-checked on every tool call** — not just at the start. An agent cannot widen its own power mid-way.
- **Kill switch down to the tool.** The emergency stop doesn't just cancel a task: it applies at the level of a single tool, across all profiles, and the browser guard enforces it itself.
- **Bounded child processes.** When an agent launches Claude Code, Cursor or a shell, the child inherits **no secret names**, its output is bounded, and it is genuinely interruptible.
- **A single, chained, signed journal.** Every event is written to a hash-chained, signed journal: any tampering shows.

> **Why it matters:** a security rule is only worth it if the system enforces it itself. Moving the boundary from the screen into the kernel turns a courtesy into a guarantee.

---

## A local SOC for defensive security

The defensive-security department does more than give advice: it is backed by a **security operations centre (SOC) that runs on the machine**, wired to the kernel.

- **Real telemetry** (including Sysmon with a home-grown config), normalised, where every network egress is tied back to the agent that caused it.
- **A register of agent egress**: you see, and classify, what each agent attempts outbound; a learning firewall tells normal from abnormal.
- **A catalogue broker**: privileged actions go through a closed list, with **per-action UAC elevation** — never a blank cheque.
- **Chained audit, seal, emergency stop and safe mode** to keep control even during an incident.

> Presented here in outline only: this repository shows the intent and the safeguards, not the operational detail.

---

## The agents' lounge: a team that talks to itself

A team isn't just a delegation tree. OS-Agentique has a **Lounge**: a visible thread where agents talk to each other, on the **local** brain and within a policy-bounded frame.

![The agents' Lounge: a retrospective between the Architect, Linus and Grace; on the right, the proposed piece of work and the memory notes awaiting approval](assets/salon.png)

> Real capture, on an isolated demo (local brain gemma4-hermes). In the centre, the agents reply to each other; on the right, a **proposed piece of work** (which I open, or not, as a Coworking space) and the **lessons to keep**: each note waits for my “Keep” or “Don't keep”.

- An agent can **open a ticket**, mention another, and the thread follows the piece of work it spawned.
- Agents **propose memory notes**; nothing is kept without my approval (see below).
- A **weekly retrospective** runs on its own, and the lounge can **run for weeks unattended** (an unbreakable dispatcher, inactive tickets closed, bounded history, a stuck-signal).
- A piece of work proposed in the lounge only opens as a Coworking space **when I decide so**.

> **Why it matters:** this is where “a team” stops being a metaphor. Collective work becomes legible — and stays under control.

---

## Memory, but under approval

Agents learn, but they don't decide on their own what to keep. Jarvis has a **“Pending memories”** queue: each proposed note is shown to me to **Approve** or **Reject**, and it is Hermes Agent itself that applies my decision. What the operator asks to remember **about himself** is written during his own turns; everything else waits for my go-ahead.

---

## A real product department: digital commerce (TIK)

Beyond code and job search, the team can carry a **commercial project** end to end. The TIK department brings together design, copy, SEO, market research, infrastructure and legal around a real website.

- **A “web ⊕ privilege” boundary:** an agent may have the web **or** a privilege on the server, never both in the same move.
- **Facts with provenance:** nothing is asserted without a source; decisions rest on dated, traceable facts.
- **Missions and budgets:** work is framed by goals and ceilings (including a **publication cap**), with a legal agent for the legal groundwork.
- **A hosting gateway, read-first**, writes later, token in the vault.

---

## What it can do

- **Talk.** Continuous voice conversation (“Hermès, stop” interrupts him), or typing.
- **Delegate real work.** Code, documents, research: split across the right agents, followed live on screen step by step.
- **Build a CV from a job ad.** Camille writes, Sacha reviews, delivery happens only if no blocking issue remains. Missing information becomes a question to me, never an invention.
- **Hunt for jobs.** Daily watch, sorting, a Notion mirror, follow-ups. The final “apply” click is always mine.
- **Support security work.** Detection rules, compliance checks, methodology and reports, backed by a local SOC.
- **Run a commercial project.** The TIK department designs, writes, optimises for SEO and frames a real website, budgets and legal groundwork included.
- **Work as a team, visibly.** The Lounge lets agents coordinate, open tickets and hold a retro — under my control.
- **Learn under approval.** Memory notes are proposed to me; I approve or reject.
- **Know itself.** A health check (`doctor`), a live map of the team, profiling of who did what, when and at what cost.
- **Protect itself.** Automatic hourly backups with a secret scanner that blocks any leak.
- **Propose improvements.** It suggests the agents or skills it lacks, but never activates them on its own.

---

## Built with

| Layer | Technology |
|---|---|
| Agent engine | [Hermes Agent](https://github.com/NousResearch/hermes-agent) (Nous Research): profiles, skills, memory |
| AI brain | switchable from the interface: local models served by Ollama (`gemma4-hermes` on an RTX 4080) or Claude (Opus 5.5, Fable 5.1, Sonnet 5, Haiku 4.5) through a subscription |
| Kernel | single scheduler, caller identity, capability tokens, chained signed journal, tool-level kill switch |
| Orchestration core | Node.js / TypeScript, **zero runtime dependency** |
| Interface | React, TypeScript, Tailwind, Vite |
| Voice | Whisper (speech-to-text) and Kokoro (text-to-speech), 100 % local |
| External tools | MCP servers (including CV Creator, the “hermes-os” gateway), with one contract test per tool |
| SOC | telemetry (including Sysmon, home-grown config), catalogue broker, per-action UAC elevation |
| Coding executors | Claude Code, Cursor |
| Scripts | PowerShell 7: one-command, repeatable install (Pester tests) |

➡️ The engineering choices I am most proud of: [**Key decisions**](docs/en/decisions.md)

---

## Where it sits among “agentic OS” projects

| | Who keeps the agent in check? |
|---|---|
| Research agentic OS (e.g. AIOS) | the operating system itself, rebuilt for agents |
| Windows 11 agentic features | Windows: a separate account and workspace for each agent |
| Desktop assistants (OpenClaw, Manus…) | mostly the agent’s own good behaviour |
| **OS-Agentique** | **explicit rules, a risk policy, verification, and my approval** |

OS-Agentique is an **agent orchestration layer**: an “agentic OS” in the business sense, running on top of Windows. The next milestone is to run the agents that write files under a dedicated, restricted Windows account, so that the rules are also enforced by the operating system.

---

<p align="center"><i>Designed and built by <a href="https://github.com/Dow08"><b>Dow08</b></a></i></p>
