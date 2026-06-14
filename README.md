# lab_notebook

A Quarto-based wet lab notebook for protocols, experiment entries, and reading notes.

This is **not** a computational notebook — analysis code lives separately.
This is the record of what happened at the bench.

---

## What this repo is

A folder of `.qmd` files that together form a navigable lab notebook website.
Each file is one of four things:

| Type | Lives in | What it is |
|------|----------|------------|
| **Protocol** | `protocols/` | Interactive Shiny document — configure inputs, print to PDF, take to bench |
| **Experiment entry** | `experiments/` | Static record of one experiment run, filled in day-by-day |
| **Reading note** | `reading-notes/` | Structured annotation of a protocol-adjacent paper |
| **Aim** | `aims/` | Living fellowship/grant aim document |

The whole thing renders to a searchable HTML website with `quarto render`.
Git is the version control layer — every completed experiment gets a commit,
giving you a timestamped, immutable record of exactly what you did and when.

---

## Repo structure

```
lab_notebook/
├── _quarto.yml                          # site config, navbar, sidebar
├── index.qmd                            # homepage with recent experiments
├── styles.css                           # global styles + print CSS
│
├── protocols/
│   ├── index.qmd                        # protocol index page
│   ├── HiC_protocol.qmd                 # in situ Hi-C
│   ├── CnR_protocol.qmd                 # CUT&RUN
│   └── ENS_2D_protocol.qmd              # 2D ENS differentiation 
│
├── experiments/
│   ├── index.qmd                        # auto-generated experiment table
│   ├── _templates/
│   │   ├── HiC_entry_template.qmd       # Hi-C entry template (6-day)
│   │   └── CnR_entry_template.qmd       # CUT&RUN entry template (3-day)
│   └── YYYY-MM-DD_protocol-target-celltype.qmd   # your entries go here
│
├── reading-notes/
│   ├── index.qmd
│   └── Author-YYYY-Journal.qmd          # one file per paper
│
└── aims/
    ├── index.qmd
    └── *.qmd                            # fellowship and grant aims
```

---

## How it works

### Protocols — configure and print

Protocol QMDs are interactive Shiny documents. You open them, set your experiment
parameters in the sidebar (number of samples, cell count, PCR cycles, etc.), and
all master mix volumes update live. Then print to PDF and take to bench.

```bash
# Protocol calculators must be served from inside their own directory
cd protocols/_shiny
quarto serve HiC_protocol.qmd   # or CnR_protocol.qmd, ENS_2D_protocol.qmd

# In browser: set your parameters → File → Print → Save as PDF
# When done: Ctrl+C to stop, then cd ../.. to return to the repo root
```

Every volume calculation in the protocol is done by R so you never have to do the math.

Master mix tables scale automatically with your inputs and include a 10% excess
by default (to account for pipetting error). Each protocol also has
collapsible **math audit** sections that show the exact equations behind every
calculation, so if something looks wrong you can verify the formula rather than
reverse-engineering the output.

### Experiment entries — fill in as you go

Each experiment run is one `.qmd` file. Copy the relevant template to start:

```bash
# Start a new Hi-C entry
cp experiments/_templates/HiC_entry_template.qmd \
   experiments/2025-08-12_HiC-SOX10-EGC.qmd
```

Open it in Positron or RStudio. It already has:
- A metadata table (cell line, aim, protocol version, raw data path)
- One section per protocol day, pre-labeled with what happens that day
- QC tables pre-filled with expected values — you just fill in what you observed
- Collapsed protocol reminder callouts for each day
- Fields for deviations and notes

Fill in each day's section the evening after you run it. Don't commit until
the experiment is fully complete.

### Editing protocols in Positron

Open the repo as two separate Positron windows:

- **Window 1:** `lab_notebook/` — for editing experiment entries, reading notes, and aims
- **Window 2:** `lab_notebook/protocols/_shiny/` — for editing protocol calculators

This is necessary because Positron inherits Quarto's project context. If you open a
Shiny `.qmd` while the project root is a website project, the Run button throws an
error. Opening `_shiny/` as its own project in a second window makes Positron see
the local `_quarto.yml` (`type: default`) instead, and everything works.

To open a second window in Positron: **File → New Window**, then open the `_shiny/` folder.

### Committing — locking the record

```bash
# When the experiment is done and the entry is written:
git add experiments/2025-08-12_HiC-SOX10-EGC.qmd
git commit -m "Hi-C SOX10 EGC pilot — good library 3.2 ng/uL, submitting"
git push
```

The git timestamp is your permanent record. The one-line commit message
(result summary) shows up as the entry description on the experiments index page.

### Building the site

```bash
quarto render        # builds _site/ from all .qmd files
open _site/index.html
```

The site has a searchable experiments table, auto-sorted by date, with tags
and result summaries visible at a glance.

---

## Naming conventions

| File type | Convention | Example |
|-----------|-----------|---------|
| Experiment entry | `YYYY-MM-DD_protocol-target-celltype.qmd` | `2025-08-12_HiC-SOX10-EGC.qmd` |
| Reading note | `FirstAuthor-YYYY-Journal.qmd` | `Barber-2019-NatProtoc.qmd` |
| Aim | `grant-aim-number.qmd` | `F32-aim1.qmd` |

Experiment entries use `YYYY-MM-DD` so they sort chronologically in the sidebar
without any configuration.

---

## Adding a new protocol

Give Claude the protocol source (PDF, `.docx`, or paste the methods section) and say
**"make a QMD for this protocol."** The `protocol-to-qmd` Claude skill handles the
conversion — it knows this repo's structure, the correct Shiny architecture, and the
callout block conventions.

The output goes directly into `protocols/`. Then add it to `protocols/index.qmd`
and the `_quarto.yml` sidebar.

## Starting a new experiment entry

Say **"make me an entry for a [protocol] run starting today"** to Claude.
The `experiment-entry` Claude skill generates a protocol-specific entry pre-filled
with the right day sections, QC expected values, and metadata fields.

---

## Requirements

- [Quarto](https://quarto.org/) ≥ 1.4
- R ≥ 4.3 with packages: `shiny`, `tidyverse`, `knitr`, `kableExtra`, `fs`
- Git

```bash
# Install R packages
install.packages(c("shiny", "tidyverse", "knitr", "kableExtra", "fs"))

# Verify Quarto
quarto check
```

---

## Copying this system for your own lab

1. Fork this repo (or wait for the template version — coming soon)
2. Update `_quarto.yml`: change the site title and GitHub link
3. Replace `protocols/` with your own lab's protocols
4. Delete the example entries in `experiments/`
5. Update the templates in `experiments/_templates/` to match your protocols
6. Run `quarto render` and check `_site/index.html`

The only lab-specific things are the protocol QMDs and the entry templates.
Everything else (`_quarto.yml`, `styles.css`, `index.qmd`, the folder structure)
is generic and reusable as-is.

---

## Philosophy

- **One file per run.** Granular entries are annoying to create and invaluable
  to read six months later when you're debugging.
- **Commit when done, not daily.** The file accumulates across bench days;
  the git commit is the "experiment complete" marker.
- **Math lives in R, not in your head.** Protocol QMDs do all volume calculations.
  You configure inputs; R produces the numbers.
- **Print CSS, not PDF rendering.** Shiny and PDF rendering are incompatible in
  Quarto. The protocols render to HTML with CSS that hides the sidebar on print —
  Cmd+P → Save as PDF is the workflow.
- **Not a computational notebook.** Analysis code, figures, and downstream
  processing live in a separate repo. This is purely the bench record.

---