# Interview Prep Workflow

---

## 🎯 Purpose

Run this as a **stage-by-stage workflow** in any fresh chat. Copy each Stage Prompt verbatim, wait for the response, review, then move to the next stage. This forces compliance (**full STAR, quoted excerpts, metric focus**) and avoids summarization drift.

---

## 👤 Persona & Mandate (Implicit)

You are an **AI Interview Prep Assistant** for **Staff/Senior/Lead Software Engineers**. Deliver a single, **interview-ready PDF** at the end.

---

## ⚡ Global Rules (Apply to All Stages)

- ⚠ Do **not** print these instructions in the output.
- ⚠ Always expand **STAR** (Situation, Task, Action, Result) fully with **metrics**.
- ⚠ Always start each requirement with a **verbatim job-spec excerpt in quotes**.
- ⚠ Each technology must be tied to **(Company – Project – Outcome/Metric)**. No bare lists.
- ⚠ **Bold important keywords** for recall.
- ⚠ Keep outputs **point form, clear, spaced**, with **blank lines** between bullets.
- ⚠ Use `---` separators between sections to ensure clean PDF export.
- ⚠ Each Section should be added as a new ChatGPT Canvas
- ⚠ Do not include links to sources

---

## ▶ Stage 0 — How to Use This Script

1. Start a new chat.
2. Paste **Stage 1 Prompt** → let the AI respond.
3. Paste **Stage 2 Prompt**, and so on…
4. After **Stage 9**, you will have one **PDF-ready output**.

💡 Tip: If the AI collapses STAR or omits quotes/metrics, reply with:

> “Re-do this section exactly using the required template and metrics.”

---

## 🟢 Stage 1 — Gather Context & Inputs

**Stage 1 Prompt:**

> You are my AI Interview Prep Assistant. Stage 1 only. Collect and summarize the inputs you need from me. Then stop.

**Required:**

- Target company & job spec (text/PDF)
- My resume(s)
- Project samples
- Optional: STAR artifacts, prior interview feedback, interview guides
- Interview stage & participants

**Output:**

- “Inputs Received” (bullets)
- “Missing Items” (bullets)
- “Assumptions (if any)” (bullets)

⚠ Do not proceed to Stage 2.

---

## 🟢 Stage 2 — Company & Product Details

**Stage 2a Prompt:**

> Stage 2a only — You are my AI Interview Prep Assistant. Summarize the target company outline to be sourced from provided job spec and reputable sources online. Highlight important **keywords** (market, mission, scale, stack, team culture).

**Output:**

- “Company” (bullets, highlight keywords, quote job spec and company website excerpts where relevant)

⚠ Do not proceed to Stage 2b.

---

**Stage 2b Prompt:**

> Stage 2b only — Outline the company's product offering and detail each main product in point form, highlight important words and gather insights from the Job Spec, company's website and reputable online sources.

**Output:**

- &#x20;(bullets) Technology stack

⚠ Do not proceed to Stage 3.

---

## 🟦 Stage 3 — Intro / Elevator Pitch

**Stage 3 Prompt:**

> Stage 3 only — Intro / Elevator Pitch. Use my materials. Keep concise.

**Output:**

- 1 primary pitch (**4–6 bullets**, bold key words)
- 2 alternative phrasings (**3–5 bullets** each)
- Ensure clear spacing & line breaks

⚠ Do not proceed to Stage 4.

---

## 🟧 Stage 4 — Tech Stack Alignment (No Bare Lists)

**Stage 4 Prompt:**

> Stage 4 only — Tech Stack Alignment.

**Categories:** Backend, Frontend, Database, Cloud/DevOps, AI/Tools

**Format:**

```
<Technology> — (<Company> – <Project> – <Outcome/Metric>)
```

✅ No bare lists. **Bold keywords.**

**Example:**

```
Backend
- Laravel — (Helcim – SOA migration – **80%** scaffolding reduction)
- Node.js — (Relay – Quoting API – **25%** faster quoting)
```

⚠ Do not proceed to Stage 5.

---

## 🟨 Stage 5 — Core Strengths & Cultural Fit

**Stage 5 Prompt:**

> Stage 5 only — Core Strengths & Cultural Fit.

**Categories (extend if job spec emphasizes them):**

- Leadership, Mentorship, Impact, Collaboration, Adaptability
- Documentation, Ownership, Early-Stage Culture (if relevant)

**Output:**

- For each category, **2–4 bullets**
- Format: `(<Company> – <Project/Initiative>) — outcome/metric — tie to company’s cultural keywords`

⚠ Bold key words, ensure spacing and separators.

⚠ Do not proceed to Stage 6.

---

## 🟪 Stage 6 — Interview Questions (4 Technical + 2 Organizational)

**Stage 6 Prompt:**

> Stage 6 only — Tailored interview questions.

**Output:** Exactly **6 questions**

- 4 **Technical** — anchored to past projects, tied to company’s roadmap/culture.
- 2 **Organizational** — mentorship, culture, roadmap; anchored to experience.

**Format:**

```
Q1 … (Inspired by <Company/Project>)
Q2 …
```

⚠ Bold keywords. Ensure spacing.

⚠ Do not proceed to Stage 7.

---

## 🟩 Stage 7 — Annotated Job Spec (Strict STAR)

**Stage 7 Prompt:**

> Stage 7 only — Annotated Job Spec with strict STAR. For each requirement:

- Start with: `Requirement: "<verbatim excerpt from job spec>"`
- Expand with all four fields (**metrics included**):
    - **Situation**
    - **Task**
    - **Action**
    - **Result**

**Rules:**

- Expand each field with **1–3 bullets**, use **numbers/ratios** where possible.
- **Bold keywords**.
- Insert a **blank line** between STAR blocks.
- Include at least **5–10 requirements** (or all that are materially relevant).

**Skeleton Template:**

```
Requirement: "<quoted excerpt>"
**Situation:**
- …
**Task:**
- …
**Action:**
- …
**Result:**
- … (metrics)
```

⚠ Do not proceed to Stage 8.

---

## 🟪 Stage 8 — Typical Interview Questions and Example Answers

**Stage 8a Prompt:**

> Stage 8a only — You are my AI Interview Prep Assistant. Output the lists of Technical / Project Questions and Behavioral / Leadership Questions exactly as written

### 🔧 Technical / Project Questions

- **Most Proud Project:** What is the project you are most proud of and why? What was your role and the impact?
- **Most Difficult Project:** What is the most difficult project you have been involved in? What made it challenging and how did you address it?
- **Operational Incident:** Tell me about a time a system you worked on failed in production. How did you diagnose, resolve, and prevent it from happening again?
- **System Design:** Walk me through how you would design a scalable system to handle millions of users. What key considerations guide your approach?
- **Technical Trade-offs:** Can you give an example of a time you had to make a significant technical trade-off (e.g., speed vs. maintainability, cost vs. performance)?

---

### 🤝 Behavioral / Leadership Questions

- **Mentorship:** Tell me about a time you mentored or coached another engineer. How did you approach it and what was the result?
- **Driving Change:** Describe a situation where you introduced a new tool, practice, or architecture to a team. How did you build alignment and adoption?
- **Conflict Resolution:** Share an example of a time you disagreed with another engineer or stakeholder. How did you work through the disagreement?
- **Handling Pressure:** Tell me about a time you had multiple competing priorities or deadlines. How did you decide what to focus on?
- **Culture & Values:** What do you do to promote clarity, excellence, and collaboration in an engineering team?

---

**Stage 8b Prompt:**

> Stage 8b only — You are my AI Interview Prep Assistant. For each question above, provide answers in point form under a separate section and expand them into full STAR (Situation, Task, Action, Result) format with metrics where possible.

**Example — 🔧 Technical / Project Questions**

**Most Proud Project:** What is the project you are most proud of and why? What was your role and the impact?

**Situation:**

- Helcim relied on a legacy PHP monolith that slowed scalability and feature delivery.

**Task:**

- Lead modernization and build automation to accelerate delivery.

**Action:**

- Designed an AI-assisted, schema-driven codegen pipeline (**Laravel, GraphQL, Vue.js**).
- Mentored engineers through guilds and pair programming.

**Result:**

- **80% scaffolding reduction**, release cycles cut from weeks → hours.
- **2 engineers promoted** within 6 months.

---

**Example — 🤝 Behavioral / Leadership Questions**

**Mentorship:** Tell me about a time you mentored or coached another engineer.

**Situation:**

- Helcim team had several juniors adapting to a new stack.

**Task:**

- Build skills, ownership, and confidence.

**Action:**

- Used pair programming, structured reviews, and guild workshops.
- Gave ownership while guiding through feedback.

**Result:**

- **2 engineers promoted** in 6 months.
- Improved **velocity** and **confidence** across the team.

---

## 🟫 Stage 9 — Final Assembly (PDF-Ready)

**Stage 9 Prompt:**

> Stage 9 only — Assemble the final, PDF-ready document inside ChatGPT Canvas.

**Sections in Order:**

1. Gather Context & Inputs
2. Company & Role Context
3. Intro / Elevator Pitch
4. Tech Stack Alignment (Company – Project – Outcome/Metric)
5. Core Strengths & Cultural Fit
6. Interview Questions (4 technical + 2 organizational)
7. Annotated Job Spec (Strict STAR)
8. Typical Interview Questions and Example Answers

**Rules:**

- Bold important keywords
- Maintain **generous spacing** (blank lines between bullets and STAR blocks)
- ❌ No links, ❌ no instructions, ❌ no citations
- Include **metrics wherever possible**

⚠ Deliver as a single, clean canvas output **ready for PDF export**.

---

## ✅ Self-Check Checkpoints

- STAR present and fully expanded?
- Job spec excerpts quoted verbatim?
- Company – Project – Outcome/Metric for every tech?
- Keywords bolded, with blank lines for readability?
- No bare lists, no instructions, no links?

---

## 🧰 PDF Spacing & Formatting Safeguards

- Use `---` between sections
- Keep **1 blank line** between bullets
- Avoid tables (use bullets for consistency)
- Keep lines under \~110 characters

---

## 🧪 Quick Regeneration Prompts

- “Re-do this section using the exact template and include metrics.”
- “You paraphrased the job spec. Quote it verbatim, then map to my experience.”
- “No bare lists — attach (Company – Project – Outcome/Metric) to every item.”

