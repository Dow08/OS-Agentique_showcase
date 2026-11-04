<p align="right"><b>English</b> · <a href="../fr/parcours.md">Français</a> · <a href="../../README.md">← Back</a></p>

# A day with Jarod

> All the screenshots on this page come from the current Jarvis build (November 4, 2026), on a test instance, and the interface is in French, as I use it. The agents' names and roles are real; everything else is fictional, apart from the bench figures quoted elsewhere.

A step-by-step walkthrough of the real system. Nothing is mocked up. Demo data is fictional wherever personal information would otherwise appear.

---

## Step 1 · Wake it up

One click on the desktop shortcut starts everything: the AI model, the voice engine, the interface. The home is the **Dashboard**: the sphere of what the team knows, the ring of departments, the bureaus, what awaits my decision, and each department's status.

![Dashboard: sphere, ring of departments, bureaus, items to decide](../../assets/dashboard.png)

> **What it shows:** a single entry point. I turn on the PC and see where things stand.

---

## Step 2 · Just talk

The Jarvis orb opens the conversation, by voice or keyboard. Jarod waits for its wake word. I ask where the team stands: it answers from what the departments report, and offers to **delegate** as soon as there is work to do.

![A sample conversation with Jarod](../../assets/conversation.png)

> **What it shows:** the director reasons, then routes the work. It knows which department does what.

---

## Step 3 · An action waits for me

I ask it to *create a file*. That changes something on my PC: the task is classed **medium risk** and stops. The orb turns amber: **approval required**.

<p align="center"><img src="../../assets/a-traiter.png" alt="To handle: what awaits my decision, with the Approve button" width="380"></p>

> **What it shows:** the human in the loop. Nothing that changes my machine runs without my explicit click.

---

## Step 4 · I stay in control

I can approve or refuse from **To handle**, from the Tasks cockpit or from my phone. If I refuse, nothing runs.

<p align="center"><img src="../../assets/mobile.png" alt="Jarvis on a phone: what awaits a decision" width="300"></p>

> **What it shows:** a refusal is final, and the interface also works on a phone.

---

## Step 5 · Watch the team work

I ask for real work: search the web for official password recommendations, then write a 5-point summary. I approve, and each real step shows live: the web search, its duration, even the start of what the tool returned.

![The Tasks cockpit: to decide, in progress, failures, scheduled, and the detail of a task](../../assets/cockpit-taches.png)

Two minutes later, Jarod reports what it did. Deliverables are listed with an **Open folder** button. A task that wrote something goes through this card: it says why it waits, with **Integrate** or **Discard**.

![A task prepared in an isolated copy, ready to be integrated or discarded](../../assets/taches-diff.png)

Today, the **Second brain** shows these links too: who delegates to whom, which agent calls which tool, live.

![Second brain: "Live" column, active paths between Jarod, departments and tools](../../assets/second-cerveau.png)

> **What it shows:** transparency. I see *what* is being done, step by step, not a spinning wheel. And the result is a real file, not a promise.

---

## Step 6 · A CV built from a job ad (fictional candidate)

For this demo, the CV tool runs on an isolated copy with a **fictional candidate**, Alex Martin. I paste a (fictional) junior SOC analyst ad and ask: *"Create my CV for this offer."*

Nora, the HR manager, starts the CV pipeline: Camille writes, Sacha reviews. Round 1 was not validated, so Camille revised the CV, and Sacha validated it in round 2.

![The Jobs cockpit: the scored offer, the diagnosis, the button that starts the CV and letter](../../assets/cockpit-emploi.png)

The validated CV is delivered (design PDF, PDF readable by recruitment software, Word, and a provenance file linking each line to its source). And because the offer asks for things the profile does not show, **Nora asks the candidate** rather than making them up. A task that wrote something goes through this card: it says why it waits, with **Integrate** or **Discard**.

![The application tracked in the Jobs cockpit](../../assets/candidatures.png)

<p align="center"><img src="../../assets/step6-cv-fictif.png" alt="The CV generated for the fictional candidate" width="520"><br/><i>The CV produced for the fictional candidate: one page, tailored to the offer, each line traceable.</i></p>

> **What it shows:** a multi-agent pipeline with built-in quality control, and an AI that asks rather than invents.

---

## Step 7 · Ask a specialist

A question about debugging, architecture or law? I hand it to a **bureau**. Code preselects candidates, the manager picks, the agent answers, the manager checks. Department agents can do the same when they hit a topic outside their trade.

![The quality & security bureau: team, manager, agents in service or on standby, request to hand over](../../assets/bureau.png)

> **What it shows:** a fleet of specialists, without power. They advise; they neither write nor execute.

---

## Step 8 · Run a store

In the morning, the **Store** dashboard says what needs action: a temperature reading out of range, a schedule to prepare, a figure to enter, a mandatory visit coming up. Everything is computed by code; nothing is sent.

![Store dashboard: weekly brief, items to look at, one action per item](../../assets/magasin-tableau.png)

The weekly schedule is proposed by code from opening hours, contracts, rest days and absences. Thomas adjusts it, Claire reviews it, and nothing is saved without my click.

![Staff schedule: weekly grid, coverage and rest alerts](../../assets/magasin-planning.png)

> **What it shows:** the system serves a real trade, with rules that affect people. Code computes, agents review, I validate.

---

## Step 9 · Check the system's health

Everything the team does is measured: who worked when, for how long, with which tools, how many tokens and at what cost. Today this lives in **Systems › Performance**.

![Systems › Performance: who is working, who worked when](../../assets/systemes-performance.png)

![The detailed profiler: requests, time, tokens, tool calls](../../assets/profilage.png)

> **What it shows:** observability. Nothing is made up; every figure is traceable.

---

## Step 10 · The system knows itself

The map of the team, its skills and its tools is generated from what is actually installed. I read it in the Second brain. The director knows its team through its own card. Injecting the full map on every turn is expensive, so that is off by default, pending measurement.

![The team: all 47 agents, by department, with their status](../../assets/equipe.png)

> **What it shows:** a map produced by code, never copied by hand.

---

## Behind the scenes

- every hour, an automatic backup of the whole system;
- a scan for secrets and real names before each backup: a password, a key or a name **blocks** it;
- routines bounded by caps, with a log: nightly link check, Brain rebuild, self-check;
- a scheduled job-offer watch that prepares its summary.

---

Next: [**Meet the team**](team.md) · [**Design & architecture**](design.md) · [**Key decisions**](decisions.md)
