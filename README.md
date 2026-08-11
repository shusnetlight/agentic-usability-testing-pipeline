# Usability Testing Pipeline

A folder-based multi-agent setup for analyzing moderated usability tests (think-aloud protocol) against an existing website or app, and generating structured UX insights. Drop in your interview data and website export(s), run agents step by step, review outputs at each stage, and end up with a traceable, stakeholder-ready research report.

**Scope**: This pipeline is built for usability testing against a concrete UI (the Extractor parses actual website HTML, and every observation is mapped to Nielsen severity + a specific UI element). It is not suited for generative/exploratory research without a concrete interface to test against (e.g. needs interviews, foundational research).

**No code, no scripts.** Everything runs through Claude Code by invoking named agents.

---

## What this does

Takes raw interview transcripts, observer notes, and the website's HTML export as input and produces:

1. **UI context document** — structured reference of every page, component, and interactive element, extracted from the actual website code
2. **Coded interviews** — every meaningful observation tagged with a code label, severity, and verbatim quote
3. **Reconciled codes** — consistent labels across all interviews, with a mapping table you review
4. **Theme map** — patterns that appear across multiple participants, with frequency and evidence
5. **Devil's advocate challenges** — inline annotations questioning your themes and insights before they become recommendations
6. **Insight report** — prioritized findings with the "observed / because / which means" structure and concrete UI recommendations
7. **Final report** — stakeholder-ready document with executive summary, task-organized findings table, and full citation appendix

---

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and running
- Interview data in `.md`, `.txt`, or `.vtt` (WebVTT) format
- Each page participants will interact with saved as a complete HTML file (File → Save As in browser, once per page). A single homepage export only covers what's on that page — if your test tasks send participants to other pages, export those too.

---

## Setup for a new project

1. **Duplicate this folder** — copy the entire `agentic-usability-testing-pipeline/` folder and rename it for your project
2. **Edit `config.md`** — set your project name, product description, and analysis output language
3. **Add your website to `input/ui-mockup/`** — save each relevant page as a complete HTML page and drop them all in; replace any existing files
4. **Say `"Extractor, analyze UI"`** — generates `input/ui_context.md` from the actual website code; review and correct if needed
5. **Replace `input/interview_guide.md`** — paste in your interview protocol or testing script
6. **Add interview data** to `input/interviews/` — one subfolder per participant:
   ```
   input/interviews/
   ├── participant_01/
   │   ├── transcript.vtt   (or .md / .txt)
   │   └── notes.md
   ├── participant_02/
   │   └── ...
   ```
7. **Clear `output/`** — delete any files from a previous analysis run
8. Open the folder in Claude Code — it reads `CLAUDE.md` automatically

---

## The seven agents

Pipeline status and "what's next" don't require a dedicated agent — `CLAUDE.md` checks project state automatically every time you open the folder, and again whenever you ask "what's the status?" or "what's next?". No invocation needed.

| Agent | What it does | Invocation |
|---|---|---|
| **Extractor** | Reads the website HTML from `input/ui-mockup/` and writes the UI reference document | `"Extractor, analyze UI"` |
| **Coder** | Open-codes a single interview (inductive, no predefined categories) | `"Coder, code all interviews"` |
| **Reconciler** | Normalizes code labels across all interviews; flags severity discrepancies | `"Reconciler, normalize codes"` |
| **Synthesizer** | Clusters codes into themes via affinity mapping | `"Synthesizer, find themes"` |
| **Advocatus** | Challenges the UI context, themes, and insights with inline devil's advocate annotations | `"Advocatus, challenge UI context"` / `"Advocatus, challenge themes"` / `"Advocatus, challenge insights"` |
| **Interpreter** | Transforms themes into prioritized insights with UI recommendations | `"Interpreter, generate insights"` |
| **Reporter** | Assembles the final stakeholder report with citation appendix | `"Reporter, write the report"` |

---

## Running the pipeline — step by step

### Step 0 — Extract UI context

**Say:** `"Extractor, analyze UI"`

Reads **every** HTML file in `input/ui-mockup/` — one per page you exported — parses the navigation structure across all of them, plus each page's components and interactive elements, and writes `input/ui_context.md`. This gives all downstream agents a shared vocabulary — consistent component names across every coded observation, theme, insight, and report.

**Output:** `input/ui_context.md`

**You review:** Check that navigation labels, page names, and component names match what you see in the actual interface. Look for a "Missing Page Exports" section — if a page referenced by your test tasks has no HTML export, add it to `input/ui-mockup/` and re-run before starting Coding. You can also manually add context the HTML alone can't reveal — known bugs, staging-only limitations, or components not visible in the saved export.

Re-run with the same command, `"Extractor, analyze UI"`, if the website changes or you add/remove a page export between research rounds — it overwrites `input/ui_context.md` completely rather than appending, re-scanning all current HTML files from scratch.

---

### Step 1 — Code the interviews

**Say:** `"Coder, code all interviews"`

One sub-agent per participant runs in parallel. Each reads the interview guide, the UI context, and the participant's files. It detects the interview language automatically and codes in that language — so a German interview is coded in German, an English one in English.

**Output:** `output/coded/participant_XX.md` — one file per participant.

Each observation looks like this:

```markdown
### [P01-C03] · opportunities-label-fehlinterpretiert

**Task**: Aufgabe 6 — Opportunities
**UI Element**: Hauptnavigation > Opportunities-Tab
**Severity**: 3 — Schwerwiegend
**Sentiment**: negativ

> „Ich hab auf Workshop geklickt, aber das war falsch."
```

**You review:** Check that quotes are verbatim, code labels are descriptive, and nothing was missed. You can ask for a re-run on a single participant:

> `"Coder, re-code participant 03. The section where they interact with the calendar around minute 18 wasn't captured."`

---

### Step 1b — Challenge UI context (devil's advocate)

**Say:** `"Advocatus, challenge UI context"`

`input/ui_context.md` was written once by the Extractor from a static HTML export, before any participant touched the interface. The Coder is the only agent (besides Interpreter/Reporter at the very end) that actively reads it — copying UI Element names into every coded observation. Reconciler and Synthesizer never read `ui_context.md` again; they just pass that field through unchanged. So this step — right after coding — is the earliest point where real participant data exists to check `ui_context.md` against, and the last point where a fix is still cheap rather than requiring a re-code.

**Output:** Appends a single `⚠️ Advocatus — UI Context Review` section to the end of `input/ui_context.md`, flagging:
- **Missing elements** — things participants clearly interacted with that aren't described anywhere in `ui_context.md`
- **Naming mismatches** — the same element called different names across coded files and the reference doc
- **Undocumented states** — error/empty/loading/staging-only states participants ran into that aren't listed under Known UI States (important: this is often the difference between a real design flaw and a staging artifact)
- **Orphaned context** (informational) — pages/components in `ui_context.md` no one ever interacted with

**You review:** For each flagged line, either fix `ui_context.md` and delete the line, or delete the line if you disagree. When done, no `⚠️ Advocatus — UI Context Review` section remains in the file.

A gap caught here is a five-minute fix; a gap caught in the final report means re-coding and re-running most of the pipeline.

---

### Step 2 — Normalize codes

**Say:** `"Reconciler, normalize codes"`

Reads all coded files, identifies labels that describe the same phenomenon across participants, and produces a mapping table. Code labels from different-language interviews are translated to the global output language (set in `config.md`) at this step.

**Output:**
- `output/code_mapping.md` — mapping table with rationale and ⚠️ severity discrepancy flags
- `output/reconciled/participant_XX.md` — coded files with canonical labels applied

Each canonical code block looks like this:

```markdown
### `suche-nicht-funktional`

**Original labels**: `suche-kaputt` (P01), `search-not-working` (P03), `suchfeld-reagiert-nicht` (P04)
**Citation IDs**: [P01-C06], [P03-C09], [P04-C03]
**Rationale**: All describe the search field yielding no response to user input.
**Severity discrepancy**: ⚠️ Yes — P01=3, P03=4, P04=3 → please review before synthesis
```

**You review:** Approve or adjust the mappings. Resolve any ⚠️ severity discrepancies — you were in the room, you decide. After editing `code_mapping.md` by hand, say `"Reconciler, re-apply mapping"` to mechanically propagate your corrected groupings into `output/reconciled/` without re-clustering from scratch. (Only say `"Reconciler, normalize codes"` again if you actually want a full re-cluster from the raw data — that overwrites your edits.)

---

### Step 3 — Find themes

**Say:** `"Synthesizer, find themes"`

Reads all reconciled files. Clusters codes by underlying phenomenon, names each theme descriptively, counts how many participants it affected, and derives an emergent category taxonomy from the data.

**Output:** `output/themes.md`

Each theme card looks like this:

```markdown
### T-02 · Suchfunktion auf allen Seiten nicht funktional

**Kategorie**: Interaktive Funktionen
**Häufigkeit**: 4 von 5 Teilnehmenden
**Max. Schweregrad**: 3 — Schwerwiegend
**Zugehörige Codes**: `suche-nicht-funktional`, `suchbutton-ohne-reaktion`

**Muster**: Vier von fünf Teilnehmenden versuchten mindestens einmal,
die Suchfunktion zu verwenden, und erhielten keine Ergebnisse.
Die Suche ist auf News-, Opportunities- und Startups-Seiten betroffen.

#### Belege

**[P01-C06]** — Teilnehmende 01 · Schweregrad 3

> „Klick auf Suchbutton funktioniert nicht."

**[P03-C09]** — Teilnehmende 03 · Schweregrad 4

> „I'm typing here but nothing happens at all."
```

**You review:** Do the groupings hold up? Merge, split, or adjust as needed before continuing.

---

### Step 3b — Challenge themes (devil's advocate)

**Say:** `"Advocatus, challenge themes"`

Goes back to the raw evidence and interrogates each theme. Inserts inline `⚠️ Advocatus` annotation blocks after each theme card — directly in the file you're already reading.

```markdown
> ⚠️ **Advocatus — Challenge T-02**
>
> **Evidence strength**: 4/5 is a strong signal. However, note that all four
> observations come from the search-button click — this may reflect a staging
> gap rather than a permanent design problem. Consider flagging this as
> "validate in production" rather than a design recommendation.
>
> **Alternative explanation**: The non-functional search may be an
> intentional staging limitation, not a usability failure. Confirm with
> the development team before prioritizing this as a design fix.
>
> **Contradictory evidence**: None found.
```

**You review each annotation:**
- If valid → edit the theme above it, then delete the annotation
- If you disagree → delete the annotation without changing anything
- Goal: no `⚠️ Advocatus` blocks remain when you move on

---

### Step 4 — Generate insights

**Say:** `"Interpreter, generate insights"`

Transforms each theme into a structured insight with a concrete UI recommendation. Prioritizes using a severity × frequency matrix.

**Output:** `output/insights.md`

Each insight looks like this:

```markdown
### I-02 · Navigationsbezeichnung „Opportunities" verhindert direkten Zugang

**Priorität**: Kritisch
**Thema**: T-01 — Opportunities-Navigation unklar
**Häufigkeit**: 3/5 Teilnehmende · **Max. Schweregrad**: 3

---

Wir beobachten, dass **3 von 5 Teilnehmenden** bei der Suche nach einem
Hackathon zuerst „Workshop" oder „Termine" anklicken und „Opportunities" übersehen,
weil der Begriff im deutschen Kontext primär mit Karrierechancen assoziiert wird
*(direkt geäußert von P02 und P04; für P01 aus Navigationsverhalten erschlossen)*,
was bedeutet, dass die Bezeichnung oder die Navigationshierarchie angepasst
werden muss.

---

#### Belege

**[P01-C11]** — Teilnehmende 01

> „Ich hab auf Workshop geklickt, aber das war falsch."

**[P02-C08]** — Teilnehmende 02

> „Opportunities? Ich dachte das wären Jobangebote."

#### Empfehlung

**Was ändern**: Bezeichnung „Opportunities" in der Hauptnavigation
**Wie**: Option A — umbenennen zu „Hackathons & Calls"; Option B — Untertitel hinzufügen
**Erwartetes Ergebnis**: Teilnehmende finden Hackathon-Events ohne Moderatorhinweis
```

---

### Step 4b — Challenge insights (devil's advocate)

**Say:** `"Advocatus, challenge insights"`

Challenges the logic of each insight — particularly the "because" clause (is the mental model inference actually supported by quotes, or is it researcher assumption?) and whether the recommendation addresses the root cause.

Same review process as Step 3b: address valid challenges, delete the rest.

---

### Step 5 — Write the report

**Say:** `"Reporter, write the report"`

Assembles all outputs into a polished stakeholder document.

**Output:** `output/report.md`

Sections (from `templates/report_template.md`, headers translated into the Analysis output language, structure unchanged):
- Executive summary (blockquote callout)
- Methodology
- Findings organized by task — one table per task with `#, Finding, Screenshot, Severity 🔴🟠🟡🟢, Frequency, Recommendation`
- Post-Session Questions — post-session topics (brand perception, NPS, etc.)
- Detailed Findings — full insight cards with verbatim quotes for Critical and High findings
- Participants × Themes Matrix
- Appendix A: Emergent codebook
- Appendix B: Citation index (every `[PXX-CYY]` traced to source file)
- Appendix C: Code reconciliation map

---

## How to iterate

| What you changed | Re-run from |
|---|---|
| Website HTML in `input/ui-mockup/` | Extractor → review `ui_context.md` → Coder onward if names changed |
| One coded file | Reconciler onward |
| Added a new participant | Coder (just that one) → Advocatus UI context check → Reconciler onward |
| Edited `ui_context.md` after Advocatus review | Reconciler onward (only if canonical names changed) |
| Edited `code_mapping.md` | Reconciler (re-apply) → Synthesizer onward |
| Edited a reconciled file | Synthesizer onward |
| Edited `themes.md` after Advocatus review | Interpreter onward |
| Edited `insights.md` after Advocatus review | Reporter |

**Example iteration commands:**
- `"Coder, re-code participant 05. The moderator's questions are being coded — exclude those."`
- `"Reconciler, re-apply mapping. I split one code group in the mapping."`
- `"Interpreter, regenerate I-03. We can't rename the nav item due to brand guidelines — suggest alternatives."`

---

## Language handling

- **Coding language**: Detected automatically from the transcript. A German interview is coded in German, an English interview in English. Verbatim quotes always stay in the original spoken language.
- **Analysis output language**: Set in `config.md` under `Analysis output`. The Reconciler translates all code labels into this language when forming canonical labels. Everything from Reconciler onwards is in this language.
- **UI context language**: Written in the analysis output language by the Extractor.

---

## File reference

```
agentic-usability-testing-pipeline/
├── README.md                    ← you are here
├── CLAUDE.md                    ← loaded automatically by Claude Code; pipeline instructions
├── config.md                    ← edit per project: name, description, output language
├── agents/
│   ├── extractor.md             ← UI context extraction prompt
│   ├── coder.md                 ← open coding prompt
│   ├── reconciler.md            ← label normalization prompt
│   ├── synthesizer.md           ← affinity mapping prompt
│   ├── advocatus.md             ← devil's advocate prompt
│   ├── interpreter.md           ← insight synthesis prompt
│   └── reporter.md              ← report assembly prompt
├── input/
│   ├── ui-mockup/                ← website HTML exports ("User Interface Mockup") — one .html per page tested
│   │   ├── [home-page].html     ← save each relevant page here (File → Save As)
│   │   ├── [home-page]_files/   ← static assets (created automatically on save)
│   │   ├── [sub-page].html      ← e.g. a product page, checkout, etc. — only if participants navigate there
│   │   └── [sub-page]_files/
│   ├── interview_guide.md       ← your testing script / interview protocol
│   ├── ui_context.md            ← auto-generated by Extractor; can be manually edited
│   └── interviews/
│       └── participant_XX/      ← one folder per participant
│           ├── transcript.vtt   ← supports .vtt, .md, .txt
│           └── notes.md
├── output/                      ← generated during analysis; clear between projects
│   ├── coded/                   ← Coder output
│   ├── reconciled/              ← Reconciler output
│   ├── code_mapping.md          ← Reconciler mapping table
│   ├── themes.md                ← Synthesizer output
│   ├── insights.md              ← Interpreter output
│   └── report.md                ← final report
└── templates/
    └── report_template.md       ← structure template used by Reporter
```

---

## Reusing for a different project

1. Duplicate this folder
2. Edit `config.md` (project name, product description, output language)
3. Replace the HTML in `input/ui-mockup/` with the new project's website export(s) — one file per page participants will interact with
4. Say `"Extractor, analyze UI"` to regenerate `input/ui_context.md`
5. Replace `input/interview_guide.md` with the new interview protocol
6. Add participant folders to `input/interviews/`
7. Clear everything in `output/`
8. Open in Claude Code — the agents in `agents/` work for any usability testing project without changes

The agent prompts are project-agnostic. Only `config.md`, `input/ui-mockup/`, and `input/` change between projects.
