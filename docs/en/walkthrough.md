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

## Step 5 · Watch the team work

I switch to **Execute** mode and ask for real work: search the web for the official French recommendations on passwords (ANSSI), then write a 5-point summary file. I approve, and every real step appears live: the web search, how long it took, even the start of what the tool returned.

![A task running, each step shown live](../../assets/step5-live.png)

Two minutes later, Hermes reports what he did. The deliverables are listed with an **Open folder** button. *(The local folder path is blurred.)*

![The finished task and its deliverables](../../assets/step5-done.png)

> **What it shows:** transparency. I see *what* is being done, step by step, not a spinning wheel. And the result is a real file, not a promise.

---

## Step 6 · A CV built from a job ad (fictional candidate)

For this demo, the CV tool runs on an isolated copy with a **fictional candidate**, Alex Martin. I paste a (fictional) job ad for a junior SOC analyst and ask: *“Create my CV for this offer.”*

Nora, the HR manager, starts the CV circuit: Camille writes, Sacha reviews. Round 1 was not validated, so Camille revised the CV and Sacha validated it at round 2. When the local model struggled, a step was automatically handed over to Claude: that is the fallback visible in the task list.

![The CV circuit running: delegations, CV tool calls, review rounds](../../assets/step6-live.png)

The validated CV is delivered (design PDF, ATS-friendly PDF, Word, plus a provenance file tracing every line back to its source). And because the offer asks for things the profile does not show (technical English, Microsoft Sentinel…), **Nora asks the candidate** instead of inventing them. *(The local folder path is blurred.)*

![CV validated at round 2, deliverables and Nora's questions](../../assets/step6-done.png)

<p align="center"><img src="../../assets/step6-cv-fictif.png" alt="The generated CV of the fictional candidate" width="520"><br/><i>The CV produced for the fictional candidate: one page, tailored to the offer, every line traceable.</i></p>

> **What it shows:** a multi-agent pipeline with built-in quality control, and an AI that asks instead of inventing.

---

## Step 7 · Check the health of the system

Everything the team does is measured, read from the engine's own records: who worked when, for how long, with which tools, how many tokens and at what cost. Only Claude, the cloud fallback, costs anything: the local brain is free.

![Who worked when, per agent](../../assets/profiling-timeline.png)

![Activity per agent: time, tools, tokens, cost](../../assets/profiling-agents.png)

> **What it shows:** observability. Nothing is invented; every figure can be traced back.

---

## Step 8 · The system knows itself

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
