<p align="right"><b>English</b> · <a href="../fr/decisions.md">Français</a> · <a href="../../README.md">← Back</a></p>

# Key decisions

The project holds 98 written decision records (ADRs). Here are fourteen that best show how I think: each one starts from a real problem, often one I found by testing or auditing my own work.

---

### 1. Build around an existing engine instead of rewriting one
**Problem:** I had started writing my own agent engine: lots of code, few guarantees.
**Decision:** on day 2, I pivoted to an open-source engine (Hermes Agent) and focused on what it lacked: organisation, rules, verification, interface. Old code was frozen behind adapters rather than rewritten.
**Why it matters:** knowing when *not* to build is an engineering skill. It saved weeks and kept the system maintainable.

### 2. One single point of contact, who can only read
**Problem:** several assistants (voice, project manager, mail…) were starting to pile up.
**Decision:** one director only. He talks to me, delegates, and has read-only rights.
**Why it matters:** the agent most exposed to what I say is also the least powerful one. It shrinks the attack surface.

### 3. Risk can only go up
**Problem:** a cross-audit showed that a task's risk was whatever the requester *declared*. An agent could label a file-writing task “low risk” and skip my approval.
**Decision:** the effective risk is the highest of what is declared and what the task can actually do. Proven by tests.
**Why it matters:** I audit my own work. A rule only counts once a test proves it is enforced.

### 4. “The AI says it's done” is not proof
**Problem:** a task was considered complete as soon as the AI said so.
**Decision:** every task declares checks (file created, command succeeded, no secret leaked), and all must pass before it can be marked complete.
**Why it matters:** AI can be confidently wrong. Verification is what makes it reliable.

### 5. An approval is tied to what I approved
**Problem:** approving by ID alone did not guarantee that the action run was the one I had seen.
**Decision:** each approval is linked to a fingerprint of the action; any change after my click invalidates it.
**Why it matters:** the same principle as signing a document. It closes a classic “time-of-check / time-of-use” gap.

### 6. Never the web and my personal data in the same hands
**Problem:** an HR agent had both web access and access to my CV data. A malicious job ad could have instructed it to send my data out.
**Decision:** a strict split. Léo browses the web without ever seeing my data; Inès reads my data without any web access.
**Why it matters:** this is how you defend against *prompt injection*, one of the main risks of AI agents today.

### 7. A local brain, chosen by measurement
**Problem:** cloud models cost money and send data out; my laptop has no GPU.
**Decision:** one brain per machine. On my desktop, a local model chosen after benchmarking several on agent tasks (9/10, about 5 s per simple exchange). On the laptop, a free cloud model. Claude is used only as an explicit fallback for demanding CV work.
**Since then:** the brain became switchable from the interface, local or Claude (Opus 5.5, Sonnet 5, Haiku 4.5), for the whole team in one click, with some agents pinned to a fixed brain (the Red Cell always local).
**Why it matters:** cost, privacy and performance were weighed with data, not guesses, and the final choice stays mine, task by task.

### 8. Make the work visible
**Problem:** Hermes once answered “I'm launching a search…” without calling a single tool.
**Decision:** the interface shows every real step live, the reason why a task stopped, and flags any action that was only *announced*. Deliverables land in a dedicated folder on my Desktop.
**Why it matters:** trust comes from transparency, not from a spinning wheel.

### 9. Sensitive capabilities are always under manual control
**Problem:** an “Auto” mode that approves medium-risk tasks by itself is convenient, but must never cover offensive security actions.
**Decision:** every task in the Red Cell is forced to the highest risk level, whatever the entry point. It always waits for my click, and an ethics advisor reviews every plan.
**Why it matters:** convenience never outranks safety. The rule is enforced on the server side, not just in the interface.

### 10. Scripts before AI, and contracts on every dependency
**Problem:** using AI for mechanical tasks is slow, costly and unpredictable; a tool update can silently break an integration.
**Decision:** everything mechanical is a plain script. Every external tool the system relies on has a *contract test* that fails loudly if its behaviour changes.
**Why it matters:** a reliable system uses AI where it adds value, and only there.

### 11. Move security from the screen into the kernel
**Problem:** a runtime audit (doc 07) showed too many safeguards lived in the interface. Anything that started “beside” the orchestrator escaped the rules.
**Decision:** a named kernel that **everything** goes through, a single scheduler with state shared across processes, and a **caller identity** on every turn — the director himself talks to the kernel in his own name.
**Why it matters:** a security rule is only worth it if the system enforces it. At the kernel level it stops being a courtesy and becomes a guarantee.

### 12. A capability token re-checked on every tool call
**Problem:** checking rights at the start of a turn leaves the door open to an agent that widens its power mid-way.
**Decision:** the kernel grants a precise capability token (which tools, what scope) **re-verified on every tool call**, with an emergency stop that reaches down to a single tool, and child processes that inherit no secret names.
**Why it matters:** least privilege, verified continuously rather than once.

### 13. Memory is learned, but under my approval
**Problem:** letting agents decide on their own what to keep is letting them rewrite their own rules.
**Decision:** proposed memory notes go through a “pending” queue: I approve or reject, and it is the engine itself that applies my decision. Only what the operator asks to remember about himself is written directly.
**Why it matters:** the team can learn without ever drifting outside what I approved.

### 14. Let the agents talk to each other — within a frame
**Problem:** a strict delegation tree never lets coordination emerge between agents; but letting them chat freely is a risk.
**Decision:** a “Lounge” where agents talk on the **local** brain, bounded by a policy: tickets, mentions, a weekly retrospective, unattended operation, and opening a piece of work in Coworking **only when I decide so**.
**Why it matters:** you get the benefit of a team that coordinates, without giving up control or traceability.

---

Next: [**Design & architecture**](design.md) · [**Meet the team**](team.md) · [**A day with Hermes**](walkthrough.md)
