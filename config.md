# Project Configuration

## Project Name
[Your Project Name]

## Product Description
[1–3 sentences: what the product is, who it's for, and the key features/screens relevant to this research round.]

## Language
- Platform UI: [e.g. German, English]
- Interview language: [e.g. German, English — or "Mixed" if participants vary]
- Analysis output: [e.g. German, English, French, Spanish]

<!-- Set "Analysis output" to the language you want all agent outputs written in.
     The agent prompts are in English, but every output file (coded interviews,
     themes, insights, report) will be written in the language set here.
     Examples: English, German, French, Spanish -->

## Research Type
[e.g. Moderated usability testing (think-aloud protocol), Unmoderated remote testing]

<!-- Both options above test participants against a concrete UI, which this pipeline
     is built for (the Extractor parses actual website HTML; every observation maps
     to a UI element and a Nielsen severity rating). It is not suited for generative/
     exploratory interview studies that don't involve a concrete interface to test. -->

## Participants
- Target: [e.g. 3-5 participants per round]
- Demographics: [age range, relevant experience/background]
- Device: [e.g. Desktop, Mobile, Both]
- Format: [e.g. Remote or in-person]

## Severity Scale (Nielsen, 0-4)
- **0** — Not a usability problem
- **1** — Cosmetic problem only — fix if extra time is available
- **2** — Minor usability problem — low priority fix
- **3** — Major usability problem — important to fix, high priority
- **4** — Usability catastrophe — imperative to fix before release

## Additional Context
[Anything downstream agents should know: staging environment notes, known incomplete features, specific task areas the testing script covers, etc.]
