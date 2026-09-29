<p align="right"><b>English</b> · <a href="../fr/parcours.md">Français</a> · <a href="../../README.md">← Back</a></p>

# A day with Hermes

A step-by-step walkthrough of the real system. Every screenshot comes from the actual interface with real answers from the local AI model. Nothing is mocked up. Demo data is fictional wherever personal information would otherwise appear.

---

## Step 1 · Wake it up

One click on the desktop shortcut starts everything: the AI model, the voice engine, the interface. Hermes waits for its wake word.

![Home screen: “Say ‘Hermès, …’ followed by your request”](../../assets/jarvis-accueil.png)

> **What it shows:** a single entry point for everything. Suggested requests help a first-time user get started.

---

## Step 2 · Just talk

I ask for help structuring a project: here, a website for a bakery. Hermes answers in about 15 seconds with a structured plan, then offers to **delegate** the next step to the right department (development or HR).

![A real conversation with Hermes](../../assets/jarvis-conversation.png)

> **What it shows:** the director reasons, then routes the work. He knows which department does what.

---

## Step 3 · An action waits for me

I ask him to *create a file*. That changes something on my PC, so the task is rated **medium risk** and stops. The orb turns amber: **approval required**.

![The action waits for my approval](../../assets/jarvis-approbation.png)

> **What it shows:** human in the loop. Nothing that modifies my machine runs without my explicit click.

---

## Step 4 · I stay in control

I refused this action (here, the same interface on a phone). Hermes confirms: **“Action refused.”** Nothing ran.

<p align="center"><img src="../../assets/jarvis-mobile.png" alt="Action refused, on mobile" width="300"></p>

> **What it shows:** a refusal is final, and the interface works just as well on a phone.

---

## Step 5 · Check the health of the system

Everything the team does is measured, read from the engine's own records: who worked when, for how long, with which tools, how many tokens and at what cost. Only Claude, the cloud fallback, costs anything: the local brain is free.

![Who worked when, per agent](../../assets/profiling-timeline.png)

![Activity per agent: time, tools, tokens, cost](../../assets/profiling-agents.png)

> **What it shows:** observability. Nothing is invented; every figure can be traced back.

---

## Step 6 · The system knows itself

Before each decision, the director receives a **real map** of his team, his skills and his tools, generated from what is actually installed.

![Map of the team and tools](../../assets/jarvis-carte.png)

> **What it shows:** the agent does not guess what it can do; it is told, from the real state of the machine.

---

## Behind the scenes

- every hour, an automatic backup of the whole system;
- a scan for secrets before each backup: a password or key **blocks** it;
- every morning, a job watch that runs by itself and prepares its summary.

---

Next: [**Meet the team**](team.md) · [**Design & architecture**](design.md) · [**Key decisions**](decisions.md)
