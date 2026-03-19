# Resume Quality Rules

All generation and critique steps load this file. Every rule here is non-negotiable.

---

## 1. Banned Words

Never use these in ANY generated text — summary, bullets, skills, anywhere:

| Word / Phrase | Why banned |
|---|---|
| delve, delving | AI tell |
| leverage (as verb) | corporate filler |
| spearhead, spearheaded | overused AI verb |
| synergy, synergies | meaningless |
| cutting-edge | vague, unverifiable |
| passionate about | filler |
| dynamic | meaningless |
| results-driven | filler |
| game-changer | hyperbole |
| transformative | hyperbole |
| seamlessly | unverifiable |
| harness, harnessed | AI tell |
| empower, empowering | corporate filler |
| revolutionize | hyperbole |
| streamline (as general filler) | overused — ok only if specific e.g. "streamlined deploy pipeline from 20min to 4min" |
| robust (as vague descriptor) | ok only with a metric e.g. "robust pipeline handling 10K req/s" |
| innovative | vague |
| world-class | vague |
| detail-oriented | filler |
| team player | filler |
| successfully | never start a bullet with this |
| responsible for | weak — use an active verb instead |
| involved in | weak — use an active verb instead |
| worked on | weak — use an active verb instead |
| helped with | weak — unless ownership is genuinely partial (see verb discipline) |
| various | vague — name them |
| numerous | vague — count them |
| etc. | never use in a resume |

---

## 2. Verb Discipline — Anti-Fabrication

**Never upgrade a verb beyond what the source resume supports.**

Check the candidate's original bullet before rewriting. Match the ownership level:

| Original says | You may write | You may NOT write |
|---|---|---|
| "contributed to", "helped build" | "Contributed to", "Built components for" | "Led", "Architected", "Owned", "Drove" |
| "worked on", "assisted with" | "Implemented", "Developed" | "Designed", "Architected", "Spearheaded" |
| "built", "developed" | "Built", "Engineered", "Developed" | "Led", "Directed" |
| "led", "owned", "drove" | "Led", "Architected", "Owned" | (all fine) |

If the original has no ownership signal — use "Built" or "Implemented" as the safe default.

**Never invent a metric.** If no number exists in the original, use scale signals:
- ✅ "deployed across 1,000+ stores" (from source)
- ✅ "serving 1,500+ students" (from source)
- ❌ "improving performance by 35%" (invented — not in source)

---

## 3. Structural Anti-Patterns

Avoid these patterns — they are known AI fingerprints:

- **Parallel three-item lists everywhere** — "X, Y, and Z" formula in every bullet reads robotic. Vary sentence structure.
- **More than 40% of bullets starting with -ing verbs** — "Building", "Developing", "Implementing" is an AI pattern. Use past tense: "Built", "Developed", "Implemented".
- **Every bullet the same length** — vary between 1-line tight bullets and 2-line detailed ones.
- **Summary starting with "I am"** — never.
- **Bullet starting with "Successfully"** — never.
- **Em dashes in prose** — never. Use commas, colons, or periods.
- **Three-word noun stacks** — "AI-powered cloud-native microservices-based solution" — break them up.
- **Hedging phrases** — "in order to", "with the goal of", "so as to" — cut them.
- **Passive voice in bullets** — "was responsible for", "was tasked with" — always active voice.

---

## 4. Bullet Character Budgets

These match 11pt font at the template's margin settings. Bullets outside these ranges will
either waste space or overflow visually:

| Bullet lines | Character range | Action if over |
|---|---|---|
| 1-line bullet | < 95 characters | Fine |
| 2-line bullet | 95 – 210 characters | Fine |
| Would be 3 lines | > 210 characters | Cut — tighten wording, this is too long |

Count approximate characters (including spaces) before finalising each bullet.
If a bullet is > 210 chars, it must be shortened — not wrapped to 3 lines.

---

## 5. Bold Token Rules

`\textbf{}` is for directing the hiring manager's eye. Used correctly, it creates a visual
scanning path. Used wrong, it's noise.

**Bold these:**
- Key metrics: `\textbf{40\%}`, `\textbf{1,500+ students}`, `\textbf{sub-200ms}`
- Primary technology noun: `\textbf{RAG pipeline}`, `\textbf{React component library}`
- Named deliverable: `\textbf{XSAT}`, `\textbf{LMS}`

**Never bold these:**
- The opening verb of a bullet
- Prepositions, articles, conjunctions
- More than 3 tokens per bullet
- The same word in every bullet (e.g. bolding "Node.js" in every single bullet)

---

## 6. Summary Rules

The summary is 1-2 sentences, max 40 words, left-aligned.

**Must contain:**
- Who the candidate is (role/domain)
- Years of experience
- What they bring to THIS specific role/company

**Must NOT contain:**
- Any banned word from Section 1
- Em dashes
- "seeking", "looking for", "hoping to"
- "passionate", "enthusiastic", "excited to"
- Generic filler like "dynamic team environment"

---

## 7. Post-Generation Checklist (12 items)

Run this BEFORE compiling. Fix anything that fails.

- [ ] 1. No banned words from Section 1 appear anywhere in the document
- [ ] 2. No bullet starts with "Successfully" or "Responsible for"
- [ ] 3. No em dashes (—) anywhere in content text
- [ ] 4. Fewer than 40% of bullets open with an -ing verb
- [ ] 5. No verb was upgraded beyond what the source resume supports (verb discipline)
- [ ] 6. No metric was invented — every number traces back to the source resume
- [ ] 7. Every bullet has at least one metric or scale signal
- [ ] 8. Summary contains no banned words, no em dash, no "seeking"/"passionate"
- [ ] 9. No raw URLs appear as visible text anywhere
- [ ] 10. No placeholder text remains (e.g. [COMPANY], [TODO], FILENAME)
- [ ] 11. All links were actually extracted from the source — none fabricated
- [ ] 12. Role-type framing is consistent throughout (e.g. an AI role doesn't have frontend bullets leading)
