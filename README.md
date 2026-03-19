# resume-generator

A Claude skill that takes your resume (PDF or DOCX) and a job description, and produces a fully tailored, ATS-optimized, one-page LaTeX resume — with a compiled PDF to download and raw LaTeX to copy into Overleaf.

Built for job seekers who apply to multiple roles and need each application properly framed for the specific position, not just a generic rewrite.

---

## What it does

1. Extracts all text and hyperlinks from your uploaded PDF or DOCX (including hidden embedded URLs)
2. Analyzes the job description and classifies the role type (AI/ML, Backend, Frontend, Full-stack, Mobile)
3. Rewrites every bullet in XYZ format: _Accomplished X by doing Y, resulting in Z_
4. Tailors framing, section ordering, and skills to match the role type
5. Runs a 12-item quality gate (banned words, verb discipline, metric verification, AI fingerprint check)
6. Compiles a clean one-page PDF and returns both the PDF and raw LaTeX source

---

## How to install

1. Go to **Claude.ai → Settings → Skills**
2. Upload `resume-generator.skill`
3. Done — the skill activates automatically whenever you upload a resume + paste a JD

---

## How to use

Upload your resume PDF or DOCX, paste the job description in the same message, and send. Claude handles the rest.

```
[attach: your_resume.pdf]

<paste job description here>
```

Claude will ask you two questions before generating:

- **CGPA** — if not found in your resume, it'll ask once. Say no to skip.
- **Missing project links** — if a project has no GitHub or live URL in the PDF, it'll ask. Say no to omit.

---

## What you get

- **Downloadable PDF** — compiled with pdflatex, one page, tight margins, ready to send
- **Raw LaTeX source** — paste into [Overleaf](https://overleaf.com) to edit, or compile locally with any TeX distribution

---

## What makes it different

### Role-type classification

Before writing a word, the skill classifies the JD into one of five role types and frames the entire resume around it:

| Role type            | What it does                                                                   |
| -------------------- | ------------------------------------------------------------------------------ |
| AI / ML Engineer     | AI projects lead, RAG/LLM/vector language throughout, AI & ML skills row first |
| Backend / Infra      | Architecture bullets lead, async/reliability/scale framing                     |
| Frontend             | UI bullets lead, component systems, user-experience metrics                    |
| Full-stack / Startup | Breadth bullets lead, ownership language, shipped-to-production signals        |
| Mobile               | Mobile projects lead, platform-specific metrics                                |

### Anti-fabrication rules

The skill checks verb ownership against your source resume before rewriting. It will never upgrade "contributed to" into "led" or "architected" without evidence. It will never invent a metric not present in your original.

### AI fingerprint avoidance

A dedicated quality rules file bans 30+ known AI writing patterns (delve, leverage, spearhead, seamlessly...), enforces past-tense active verbs, limits em dashes to zero, and checks that fewer than 40% of bullets start with -ing verbs.

### 12-item quality gate

Every generated resume is checked before compilation:

- No banned words
- No invented metrics
- No verb overclaiming
- No em dashes
- No placeholder text remaining
- Role-type framing is consistent throughout

---

## File structure

```
resume-generator/
├── SKILL.md                        # Main skill — 9-step pipeline
├── README.md                       # This file
├── references/
│   ├── quality-rules.md            # Banned words, verb discipline, bullet budgets, 12-item checklist
│   └── jake-template.tex           # Jake Gutierrez LaTeX template with all macros
└── assets/                         # (empty — reserved for future template assets)
```

---

## LaTeX template

Uses the [Jake Gutierrez resume template](https://github.com/jakegut/resume) (MIT license) — the most widely used single-page resume template for software engineers.

To compile the raw LaTeX output locally:

```bash
pdflatex resume_output.tex
```

Or paste into [Overleaf](https://overleaf.com) — no local install needed.

---

## Limitations

- **One page only** — the skill enforces a strict one-page limit. Content is trimmed in a defined order (project bullets first, then older job bullets).
- **PDF hyperlinks only** — project URLs are extracted from embedded hyperlinks in the PDF, not from visible text. If your resume PDF doesn't have clickable links, the skill will ask you to provide them.
- **No CGPA auto-fill** — if your CGPA isn't in the resume, the skill asks once. If you skip, education shows without it.
- **English only** — the template and quality rules are written for English-language resumes and job descriptions.

---

## Tips

- The more metrics in your original resume, the better the output. Scale signals like "1,500+ students" or "1,000+ stores" are always used — even when no percentage is present.
- If you want a different framing for the same resume, just paste a different JD. The role-type classification will shift the entire resume automatically.
- The raw LaTeX is clean and fully editable. Paste it into Overleaf to tweak wording, adjust spacing, or add sections.
- CGPA matters for early-career roles in India — add it when prompted if you have a strong one (above 8.0).

---
