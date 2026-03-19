---
name: resume-generator
description: >
  Generate a fully tailored, ATS-optimized LaTeX resume from an uploaded PDF or DOCX resume
  and a job description. Use this skill whenever the user uploads a resume (PDF or DOCX) and
  wants to tailor it to a job, create a new version, improve ATS score, or generate LaTeX output.
  Also trigger when the user says things like "tailor my resume", "make my resume ATS friendly",
  "generate a resume for this JD", "rewrite my resume", "convert resume to LaTeX", or pastes a
  job description alongside a resume file. Always use this skill — don't try to wing resume
  generation without it.
---

# Resume Generator Skill

Produces a polished, one-page, ATS-optimized LaTeX resume tailored to a specific job description.
Input: uploaded PDF or DOCX resume + a job description pasted by the user.
Output: compiled PDF (downloadable) + raw LaTeX source (copyable).

**Before doing anything else, read `references/quality-rules.md` in full.**
Every rule in that file applies to every step in this skill.

---

## Step 1 — Extract Resume Content

```bash
ls /mnt/user-data/uploads/
```

### PDF — two-pass extraction

**Pass 1: Hyperlinks with position mapping**
PDF links are invisible to plain text extraction. Pull them separately:

```bash
pip install pymupdf --break-system-packages -q
python3 << 'PYEOF'
import fitz

doc = fitz.open("/mnt/user-data/uploads/FILENAME.pdf")
page = doc[0]

links = [(l["from"][1], l["from"][0], l["uri"]) for l in page.get_links() if l.get("uri")]
blocks = page.get_text("dict")["blocks"]

print("=== LINK MAPPING ===")
for link_y, link_x, link_url in links:
    candidates = []
    for b in blocks:
        for line in b.get("lines", []):
            for span in line.get("spans", []):
                if abs(span["bbox"][1] - link_y) < 30 and span["text"].strip():
                    candidates.append(span["text"].strip())
    print(f"URL: {link_url}")
    print(f"  Near: {' | '.join(candidates[:4])}")
    print()
PYEOF
```

**Pass 2: Plain text**
```bash
pdftotext -layout /mnt/user-data/uploads/FILENAME.pdf /tmp/resume_raw.txt
cat /tmp/resume_raw.txt
```

### DOCX
```bash
pandoc /mnt/user-data/uploads/FILENAME.docx -o /tmp/resume_raw.md && cat /tmp/resume_raw.md
```

---

## Step 2 — Parse Structured Data

Extract every field from the raw text and link mapping. Infer where possible.
Only ask the user for things that are genuinely missing and critical (CGPA, missing project links).

### 2a. Contact & Header Links
```
name:          full name, exact capitalisation
phone:         with country code
email:         exact
linkedin_url:  full URL from hyperlink extraction only
github_url:    full URL from hyperlink extraction only
portfolio_url: full URL from hyperlink extraction only
location:      city/country if present
```
Never fabricate a URL. If a label exists but no URL found, mark MISSING.

### 2b. Experience
Capture ALL jobs. Never drop any. Order: most-recent first.
```
title, company, start_date, end_date (or "Present"), location/remote
raw_bullets: [copy verbatim — do not rewrite yet]
```

### 2c. Projects
```
name, tech_stack (as written in source)
link_url:  the single URL extracted for this project (null if none)
           (label is always "Live Preview" regardless of whether it's GitHub or live)
raw_bullets: [copy verbatim]
```

**Missing project links:** If no URL was extracted for a project, ask the user:
"I didn't find a link for [Project Name]. Do you have a GitHub repo or live URL?"
If they provide one, use it. If no response or "no", omit. Never fabricate.

### 2d. Education
```
institution, degree, field, start_year, end_year, cgpa (if present), location
```
**CGPA:** If not in the document, ask once: "I didn't find your CGPA. Want to add it?"
If they provide it, include it. If not, omit silently.

### 2e. Skills
Extract all skills verbatim. Will be reorganised in Step 4.

---

## Step 3 — Analyze the Job Description

Build this map in full before touching any content:

```
company_name:       (for summary personalisation)
role_title:         exact title
required_tech:      [languages, frameworks, tools explicitly listed as required]
preferred_tech:     [nice-to-have items]
domain_keywords:    [e.g. RAG, LLM, serverless, microservices, vector search, WCAG]
action_verbs:       [verbs the JD uses: architect, deploy, scale, ship, optimise]
seniority_signals:  [e.g. "production-grade", "ownership", "end-to-end", "infra"]
soft_skills:        [collaboration, curiosity, mentoring, ownership, etc.]
```

Cross-reference against what the candidate genuinely has. Never claim skills not present in source.

---

## Step 3.5 — Classify Role Type

Before writing a single word, classify the JD into exactly ONE of these role types.
This drives ALL framing decisions: which bullets lead, which projects surface first,
which skills are highlighted, and what tone the summary takes.

| Role Type | JD signals | Framing priority |
|---|---|---|
| **AI / ML Engineer** | LLM, RAG, vector DB, embeddings, prompt engineering, AI APIs | AI projects first, RAG/LLM verbs throughout, AI skills row at top |
| **Backend / Infra** | serverless, Lambda, microservices, Terraform, ECS, infra ownership | Architecture bullets lead, async/reliability/scale language |
| **Frontend** | React, component-driven, accessibility, WCAG, performance, bundle size | UI bullets lead, component systems, metrics tied to user experience |
| **Full-stack / Startup** | full ownership, end-to-end, product + backend + frontend, fast-moving | Breadth bullets lead, ownership language, shipped-to-production signals |
| **Mobile** | React Native, Expo, iOS, Android, cross-platform | Mobile projects lead, platform-specific metrics |

Write the chosen role type at the top of your planning notes. Every subsequent decision
must be consistent with this classification.

---

## Step 4 — Tailor All Content

Read `references/quality-rules.md` before writing any bullet. Every rule there applies here.

### 4a. Bullet Rewrites — XYZ format (every bullet, no exceptions)

Structure: "Accomplished [X] by doing [Y], resulting in [Z]"
- X = the outcome or deliverable
- Y = method, tech, approach used
- Z = measurable impact (%, count, scale, time saved)

**Verb rules:**
- Open with a strong past-tense verb
- Good verbs: Engineered, Architected, Built, Deployed, Designed, Implemented, Optimized, Reduced, Shipped, Scaled
- Check `references/quality-rules.md` Section 2 (Verb Discipline) before choosing a verb
- Never upgrade ownership beyond what the source supports

**Metric rules:**
- Every bullet needs at least one metric or scale signal
- Only use numbers that appear in the source resume
- Never invent a percentage or count not present in the original

**Formatting rules:**
- Max 210 characters per bullet (see quality-rules.md Section 4 for line budget)
- No em dashes. Use commas or periods.
- No filler phrases: "in order to", "with the goal of", "so as to"

**Bold with `\textbf{}`:**
```latex
\resumeItem{Engineered a \textbf{multi-tenant AI chatbot} in TypeScript and Node.js,
deployed across \textbf{1,000+ Shopify stores} in the first week, driving \textbf{40\%}
higher sales conversion.}
```
Max 2-3 bold tokens per bullet. Bold metrics and primary tech nouns only.
Never bold the opening verb. See quality-rules.md Section 5.

### 4b. Project Bullets — 2 to 3 per project (mandatory)

- **Bullet 1:** What it does + core technical approach + scale or impact metric
- **Bullet 2:** A specific engineering decision or challenge + its measurable outcome
- **Bullet 3 (if space):** A secondary integration, tool choice, or result worth highlighting

Single-line project summaries are not acceptable. A hiring manager needs to see:
what problem it solves, how it's built technically, and why it's non-trivial.

### 4c. Section and Bullet Ordering

Based on the role type from Step 3.5:
- Experience: most-recent job always first
- Within each job: lead with the bullet most relevant to the role type
- Projects: most role-relevant project first
- Skills: JD-matching skills listed first within each category row

### 4d. Tailored Summary

1-2 sentences, max 40 words, left-aligned below the header.
Structure: [who you are] + [years + domain] + [what you bring to THIS role at THIS company]

Check summary against quality-rules.md Section 6 before finalising.
No banned words. No em dashes. No "seeking"/"passionate"/"dynamic".

---

## Step 5 — Generate LaTeX

Read `references/jake-template.tex` for all macro definitions before writing any LaTeX.

### 5a. Page Margins (tighter than default — more content space)

```latex
\addtolength{\oddsidemargin}{-0.6in}
\addtolength{\evensidemargin}{-0.6in}
\addtolength{\textwidth}{1.2in}
\addtolength{\topmargin}{-.6in}
\addtolength{\textheight}{1.2in}
```

### 5b. Header

```latex
\begin{center}
    \textbf{\Huge \scshape First Last} \\ \vspace{1pt}
    \small +91 XXXXXXXXXX $|$
    \href{mailto:user@email.com}{\underline{user@email.com}} $|$
    \href{https://linkedin.com/in/handle}{\underline{LinkedIn}} $|$
    \href{https://github.com/handle}{\underline{GitHub}} $|$
    \href{https://portfolio.com}{\underline{domain.com}}
\end{center}
```

- LinkedIn → display `LinkedIn` (never the raw URL)
- GitHub → display `GitHub`
- Portfolio → display the short domain (e.g. `tanbir.in`)
- All wrapped in `\underline{}`
- Phone: plain text, no href
- Only include links actually extracted in Step 1

### 5c. Summary

```latex
\vspace{2pt}
\small\textit{Summary sentence here. No em dashes. No line breaks.}
\vspace{-6pt}
```

Never `\begin{center}` for the summary. No `\\` inside it. No em dashes.

### 5d. Experience Entries

```latex
\resumeSubheading
  {Job Title}{Start Mon. Year -- End Mon. Year}
  {Company Name}{City / Remote}
  \resumeItemListStart
    \resumeItem{Verb \textbf{key outcome} by doing Y, resulting in \textbf{metric}.}
    \resumeItem{...}
  \resumeItemListEnd
```

### 5e. Project Entries

**No year in the right column. The extracted link goes there as "Live Preview".**

```latex
% Link found (GitHub OR live — label is always "Live Preview"):
\resumeProjectHeading
  {\textbf{Project Name} $|$ \emph{Tech, Stack}}
  {\href{https://extracted-url.com}{\underline{Live Preview}}}

% No link found:
\resumeProjectHeading
  {\textbf{Project Name} $|$ \emph{Tech, Stack}}
  {}
```

2-3 bullets per project. No single-line summaries.

### 5f. Education

```latex
% With CGPA:
\resumeSubheading
  {Institution}{City, Country}
  {B.Tech in Computer Science, CGPA: 8.4}{2020 -- 2024}

% Without CGPA:
\resumeSubheading
  {Institution}{City, Country}
  {B.Tech in Computer Science}{2020 -- 2024}
```

### 5g. Skills Section

```latex
\section{Technical Skills}
 \begin{itemize}[leftmargin=0.15in, label={}]
    \small{\item{
     \textbf{Category}{: JD-matching skills first, then others} \\
    }}
 \end{itemize}
```

For AI/ML roles: add an "AI \& ML" row first with LLM APIs, RAG, vector DBs.
For Frontend roles: lead with a "Frontend" row, list React/TS/CSS first.
For Backend/Infra: lead with "Backend \& APIs", then Cloud \& DevOps.

### 5h. LaTeX Escaping

Escape in ALL content: `&` → `\&`, `%` → `\%`, `$` → `\$`, `#` → `\#`, `_` → `\_`
`\textbf{}` IS allowed inside `\resumeItem{}`.
No `\\` inside `\resumeItem{}`.
No em dashes (— or ---) in content text.

### 5i. One-Page Trimming (strict order — never skip steps)

1. Trim project bullets to 2 each (minimum — never go below 2)
2. Trim oldest job to 2 bullets
3. Trim second-oldest job to 3 bullets
4. Never trim most-recent job below 2 bullets
5. Never remove any job from Experience
6. Tighten wording on long bullets (cut filler, reduce to budget)
7. Last resort only: remove least-relevant project entirely

---

## Step 6 — Pre-Compile Quality Gate

**Run the 12-item checklist from `references/quality-rules.md` Section 7 now — before compiling.**

Go through each item explicitly. If any item fails, fix the LaTeX before proceeding.
This is not optional. Do not skip to compile with known issues.

Common fixes:
- Banned word found → replace with a concrete specific word
- Invented metric → replace with a scale signal from the source
- Em dash found → replace with comma or period
- Verb overclaimed → downgrade to match source ownership level
- Bullet > 210 chars → cut a clause, tighten wording

---

## Step 7 — Compile

```bash
cd /tmp && pdflatex -interaction=nonstopmode resume_output.tex 2>&1 | grep -E "(^!|Output written|Error)"
pdflatex -interaction=nonstopmode resume_output.tex 2>&1 | grep "Output written"
```

Fix compilation errors (unescaped chars, unclosed environments), recompile until clean.

---

## Step 8 — Visual Verification

```bash
pdftoppm -jpeg -r 150 /tmp/resume_output.pdf /tmp/resume_check
# View /tmp/resume_check-1.jpg
```

Check ALL of the following. If anything fails, fix LaTeX and recompile.

**Layout:**
- [ ] Fits on exactly one page
- [ ] Margins are tight — content reaches close to edges
- [ ] No awkward whitespace gaps between sections

**Header:**
- [ ] All links show as underlined display text (LinkedIn, GitHub, domain) — not raw URLs
- [ ] Phone is plain text

**Summary:**
- [ ] Left-aligned, not centered
- [ ] Clean single block, not wrapping oddly

**Experience:**
- [ ] All jobs present — count against Step 2 parsed data
- [ ] Most-recent job is first

**Projects:**
- [ ] No year in right column — link ("Live Preview") or empty
- [ ] Each project has 2-3 bullets

**Content:**
- [ ] Metrics and key tech are visibly bolded
- [ ] No em dashes visible anywhere
- [ ] Education shows CGPA if provided

```bash
cp /tmp/resume_output.pdf /mnt/user-data/outputs/resume_tailored.pdf
```

---

## Step 9 — Deliver

1. Call `present_files` with `/mnt/user-data/outputs/resume_tailored.pdf`
2. Show the full LaTeX source in a code block
3. Add a brief note (4-6 lines) covering:
   - Role type classification and how it shaped framing
   - Top JD keywords woven in
   - Any metrics that were inferred from context (user should verify accuracy)
   - Any project links or CGPA that weren't found (ask user to provide if still needed)
   - Anything trimmed to fit one page

---

## ATS Compliance (always enforce)

- No multi-column layout, text boxes, or content tables
- No images, icons, or decorative elements
- Section headers as plain uppercase words
- Single font family throughout
- PDF with embedded fonts (pdflatex default)
- No headers/footers or page numbers
- Contact info as plain text + hyperlinks only

---

## Edge Cases

| Situation | Action |
|---|---|
| Project link not found | Ask user before generating. If no answer, omit. |
| CGPA not in resume | Ask user once. If declined or no answer, omit silently. |
| PDF label present but URL not extractable | Ask user for the URL |
| DOCX links missing via pandoc | Try python-docx: `from docx import Document` to iterate hyperlinks |
| Job dates missing | Infer from context or use year only |
| Scanned PDF (no text) | Rasterize with pdftoppm, read pages visually |
| Still over one page after all trimming | Remove least-relevant project, then tighten bullets further |
| JD has no explicit tech stack | Anchor to role-level signals: ownership, scale, production, reliability |
| Source has no metrics at all | Use scale signals only: users served, stores deployed, team size |
| Verb in source is weak ("worked on") | Use "Implemented" or "Built" — do NOT upgrade to "Led" or "Architected" |
