<p align="right"><b>English</b> · <a href="../fr/decisions.md">Français</a> · <a href="../../README.md">← Back</a></p>

# Key decisions

The project holds 148 written decision records (ADRs). Here are twenty that best show how I think: each one starts from a real problem, often one I found by testing or auditing my own work.

---

### 1. Build around an existing engine, then know how to leave it
**Problem:** I had started writing my own agent engine: lots of code, few guarantees.
**Decision:** on day 2, I pivoted to an open-source engine (Hermes Agent) and focused on what it lacked: organisation, rules, verification, interface. Old code was frozen behind adapters rather than rewritten.
**Since then:** six weeks later, I removed that engine (decision 15). The initial choice still stands: it let me build everything else first.
**Why it matters:** knowing when *not* to build is an engineering skill. Knowing when a choice has stopped being right is another.

### 2. One single point of contact, who can only read
**Problem:** several assistants (voice, project manager, mail…) were starting to pile up.
**Decision:** one director only. It talks to me, delegates, and has read-only rights.
**Why it matters:** the agent most exposed to what I say is also the least powerful one. It shrinks the attack surface.

### 3. Risk can only go up
**Problem:** a cross-audit showed that a task's risk was whatever the requester *declared*. An agent could label a file-writing task "low risk" and skip my approval.
**Decision:** the effective risk is the highest of what is declared and what the task can actually do. Proven by tests.
**Why it matters:** I audit my own work. A rule only counts once a test proves it is enforced.

### 4. "The AI says it's done" is not proof
**Problem:** a task was considered complete as soon as the AI said so.
**Decision:** every task declares checks (file created, command succeeded, no secret leaked), and all must pass before it can be marked complete.
**Why it matters:** AI can be confidently wrong. Verification is what makes it reliable.

### 5. An approval is tied to what I approved
**Problem:** approving by ID alone did not guarantee that the action run was the one I had seen.
**Decision:** each approval is linked to a fingerprint of the action; any change after my click invalidates it.
**Why it matters:** the same principle as signing a document. It closes a classic "time-of-check / time-of-use" gap.

### 6. Never the web and my personal data in the same hands
**Problem:** an HR agent had both web access and access to my CV data. A malicious job ad could have instructed it to send my data out.
**Decision:** a strict split. Léo browses the web without ever seeing my data; Inès reads my data without any web access.
**Since then:** the same rule holds in the Store: Ombeline has the web and no staff data; Thomas and Claire have the named schedule and no web.
**Why it matters:** this is how you defend against *prompt injection*, one of the main risks of AI agents today.

### 7. A local brain, chosen by measurement
**Problem:** cloud models cost money and send data out; my laptop has no GPU.
**Decision:** on my desktop, a local model chosen after benchmarking several on agent tasks (9/10). Claude when the work calls for it.
**Since then:** the brain is chosen from the interface, for the team, a department or an agent, from a closed list of verified models. The laptop now goes through Claude; the free provider left with the old engine. The Red Team, Hugo, Hélène and Aurèle are locked to local.
**Why it matters:** cost, privacy and performance were weighed with data, not guesses, and the final choice stays mine.

### 8. Make the work visible
**Problem:** Jarod once answered "I'm launching a search…" without calling a single tool.
**Decision:** the interface shows every real step live, the reason why a task stopped, and flags any action that was only *announced*. Deliverables land in a dedicated folder on my Desktop.
**Why it matters:** trust comes from transparency, not from a spinning wheel.

### 9. Sensitive capabilities are always under manual control
**Problem:** an "Auto" mode that approves medium-risk tasks by itself is convenient, but must never cover offensive security actions.
**Decision:** every task in the Red Team is forced to the highest risk level, whatever the entry point. It always waits for my click, and an ethics advisor reviews every plan. This department runs on the local brain, locked by a sealed policy.
**Why it matters:** convenience never outranks safety. The rule is enforced on the server side, not just in the interface.

### 10. Scripts before AI, and contracts on every dependency
**Problem:** using AI for mechanical tasks is slow, costly and unpredictable; a tool update can silently break an integration.
**Decision:** everything mechanical is a plain script. Every external tool the system relies on has a *contract test* that fails loudly if its behaviour changes.
**Why it matters:** a reliable system uses AI where it adds value, and only there.

### 11. Move security from the screen into the kernel
**Problem:** a runtime audit (doc 07) showed too many safeguards lived in the interface. Anything that started "beside" the orchestrator escaped the rules.
**Decision:** a named kernel that **everything** goes through, a single scheduler with state shared across processes, and a **caller identity** on every turn — the director itself talks to the kernel in its own name.
**Why it matters:** a security rule is only worth it if the system enforces it. At the kernel level it stops being a courtesy and becomes a guarantee.

### 12. A capability token re-checked on every tool call
**Problem:** checking rights at the start of a turn leaves the door open to an agent that widens its power mid-way.
**Decision:** the kernel grants a precise capability token (which tools, what scope) **re-verified on every tool call**, with an emergency stop that reaches down to a single tool, and child processes that inherit no secret names.
**Why it matters:** least privilege, verified continuously rather than once.

### 13. Memory is learned, but under my approval
**Problem:** letting agents decide on their own what to keep is letting them rewrite their own rules.
**Decision:** proposed memory notes go through a "pending" queue: I approve or reject, and my action applies the note. Anything about the department or about me always waits. A note written during a turn that had the web waits too.
**Why it matters:** the team can learn without ever drifting outside what I approved.

### 14. Let the agents talk to each other — within a frame
**Problem:** a strict delegation tree never lets coordination emerge between agents; but letting them chat freely is a risk.
**Decision:** a "Lounge" where agents talk on the **local** brain, bounded by a policy: tickets, mentions, a weekly retrospective, unattended operation, and opening a piece of work in Coworking **only when I decide so**.
**Why it matters:** you get the benefit of a team that coordinates, without giving up control or traceability.

### 15. Replace the agent engine (ADR-144)
**Problem:** an upstream update routed turns through a gateway isolated per profile, not per task: a turn could inherit another's identity. My workarounds limited the system to one turn at a time. On 27 October, at 211 profiles, the gateway took 3 to 5 minutes to start against a fixed 60-second timeout: no turn went through any more.
**Decision:** an agent loop in the repository, with two families of brains (API and the official Claude client), one single guard chain for all tool calls, brains and engine described as data.
**Proof:** same question: 98,267 tokens and 58 s before, 4,509 to 9,406 tokens and ~5.5 s after. Director's turn: 25,594 tokens and 66 s before, 19,330 and 9 s after. Security gate 36/36, one real green turn per department (9/9).
**Accepted price:** no more upstream updates; everything runs under my Windows account, and the terminal is bounded but this is not OS isolation. The real bound is the worktree and the approved diff.
**Why it matters:** the contracts, the tests and the kernel did not depend on the engine. That is what made the change possible overnight.

### 16. Bureaus of specialists, without power (ADR-140, ADR-149)
**Problem:** I wanted a fleet of specialists without writing one prompt per need and without exploding token use.
**Decision:** 161 open-source prompts (VoltAgent, MIT licence) imported by a deterministic program from a pinned source, in 9 bureaus with one manager each. Every file is sealed: if modified, it is refused. Code preselects candidates, the manager picks and checks. These agents neither write, execute nor delegate, whatever their token. Since 3 November, department agents can consult them, except private HR and External audit.
**Proof:** a full delegation goes from 95,100 to ≈ 63,400 tokens, on the agents' side from 43,500 to ≈ 11,500.
**Why it matters:** an imported prompt is data that has been reviewed. It decides no right.

### 17. The forge: a prompt is judged on test cases (ADR-141)
**Problem:** every prompt was hand-written and judged by taste. A model scoring itself converges on pretty, not on right.
**Decision:** three agents with no tools: Anselme writes the test cases then the versions, Colin plays the candidate, Honorine scores on a fixed grid. Code decides, and always adds at least one injection case: obeying the hidden instruction caps the score. Nothing is deployed without my review, and it lands as a draft.
**Why it matters:** a prompt gets what code gets: a replayable test rather than an opinion.

### 18. The eye sees, the brain reasons (ADR-139)
**Problem:** an attached image reached the director, who holds the delegation tools. Text hidden in a screenshot became an instruction, and no prompt reliably detects that: the model reads before it can judge.
**Decision:** Lucie reads the image, with no tools, and returns a card in an enforced schema. The director receives the card as untrusted data, never the image, and has no tool to have it looked at again. Secrets are masked before any output.
**Limits:** a QR code is not decoded. If the eye follows a hosted brain, the image goes to that provider, as in a Claude conversation; a setting keeps it local.
**Why it matters:** the boundary is not "is this agent careful", it is "where does authority come from". My instruction is an order; the content of an image is data.

### 19. One model per difficulty, chosen by code (ADR-150)
**Problem:** a web watch cost the same model as an architecture decision. Measuring 83 real turns corrected my intuition: the real cost item was not the director, it was Léo (28%).
**Decision:** an "Automatic" mode: watch on Haiku, code and director on Sonnet, scheduling, architecture and review on Opus. The tier depends on the role, with no model call, and can only go up. An answer that is too weak is retried once, one tier higher. Nothing changes until I choose it.
**Honesty:** the replay gives −15% over those two weeks. That is a projection at public prices, not a measurement, and the quality of the watch on Haiku remains to be judged.
**Why it matters:** measuring first avoided a mistake: my first setting would have increased consumption.

### 20. The Store: code computes, agents review, I validate (ADR-148)
**Problem:** running a local food shop, with real schedules subject to labour law. A mistake affects people.
**Decision:** schedule, hours and VHP (sales per paid hour) are computed by code (8 rules, plus register consistency). Six read-only agents propose and review; a schedule is only validated by my action, with no blocking alert. No real staff data is in the repository, and the checkpoint blocks real names. Agents reading a named schedule have no web and only run on the local model or Claude.
**What went wrong:** the first version was declared validated with several fake integrations. An audit caught it, the department's document now says what is not connected, and automatic extraction from the store management software does not exist: it goes through an import of statements.
**Why it matters:** in a real trade, saying what does not work is worth more than announcing what does.

---

Next: [**Design & architecture**](design.md) · [**Meet the team**](team.md) · [**A day with Jarod**](walkthrough.md)
