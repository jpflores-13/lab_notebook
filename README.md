# lab_notebook

A Quarto-based wet lab notebook for protocols and experiment entries, hosted publicly
on GitHub Pages at **[jpflores-13.github.io/lab_notebook](https://jpflores-13.github.io/lab_notebook)**.

This is **not** a computational notebook — analysis code lives separately.
This is the record of what happened at the bench.

---

## What this repo is

A folder of `.qmd` files that together form a navigable lab notebook website.
Each file is one of two things:

| Type | Lives in | What it is |
|------|----------|------------|
| **Protocol** | `protocols/` | Interactive OJS document — set inputs, volumes update live, print to PDF |
| **Experiment entry** | `experiments/` | Static record of one experiment run, filled in day-by-day |

---

## Repo structure

```
lab_notebook/
├── _quarto.yml                          # site config, navbar, sidebar
├── index.qmd                            # homepage
├── styles.css                           # global styles
│
├── protocols/
│   ├── index.qmd                        # protocol index page
│   ├── hic-protocol.qmd                 # in situ Hi-C
│   ├── cnr-protocol.qmd                 # CUT&RUN
│   └── ens-2d-protocol.qmd              # 2D ENS differentiation
│
├── experiments/
│   ├── index.qmd                        # experiment table
│   ├── _templates/
│   │   ├── HiC_entry_template.qmd
│   │   └── CnR_entry_template.qmd
│   └── YYYY-MM-DD_protocol-target-celltype.qmd
│
└── docs/                                # built site (GitHub Pages serves this)
```

---

## How it works

### Protocols — configure and print

Protocol QMDs use Observable JS (OJS) — they run entirely in the browser, no server
needed. Open a protocol on the site, set your parameters (samples, cell count, PCR
cycles, etc.), and all master mix volumes update live. Hit the print button and save
as PDF to take to the bench.

Works online at the site URL, and offline by opening `docs/protocols/hic-protocol.html`
directly in any browser.

Every volume calculation includes a 10% excess by default. Each protocol has
collapsible **math audit** sections showing the equations behind every number.

### Experiment entries — fill in as you go

Each experiment run is one `.qmd` file. Copy the relevant template to start:

```bash
cp experiments/_templates/HiC_entry_template.qmd \
   experiments/2025-08-12_HiC-SOX10-EGC.qmd
```

Each entry has:
- A metadata table (cell line, aim, protocol version, raw data path)
- One section per protocol day with pre-labeled QC tables
- Expected values pre-filled — you fill in what you observed
- Collapsed protocol reminder callouts for each day
- Fields for deviations and notes

Fill in each day's section at the end of the bench day. Don't commit until the
experiment is fully complete — the git commit is the "experiment complete" marker.

### Building and publishing the site

```bash
quarto render        # builds docs/ from all .qmd files
git add docs/
git commit -m "rebuild site"
git push             # GitHub Pages picks up docs/ automatically
```

---

## Adding a new protocol

Give Claude the protocol source (PDF, `.docx`, or paste the methods section) and say
**"make a QMD for this protocol."** The `/protocol-to-qmd` skill handles the
conversion — it produces an OJS file in `protocols/`.

Then:
1. Add a row to `protocols/index.qmd`
2. `quarto render`
3. `git add protocols/ docs/` → commit → push

## Starting a new experiment entry

Say **"make me an entry for a [protocol] run starting today"** to Claude.
The `/experiment-entry` skill generates a protocol-specific entry pre-filled
with the right day sections, QC expected values, and metadata fields.

---

## Requirements

- [Quarto](https://quarto.org/) ≥ 1.4
- R ≥ 4.3 with packages: `tidyverse`, `knitr`, `kableExtra`
- Git

```bash
install.packages(c("tidyverse", "knitr", "kableExtra"))
quarto check
```
---

## Copying this system

1. Fork this repo
2. Update `_quarto.yml`: change the site title and GitHub link
3. Replace `protocols/` with your own lab's protocols
4. Delete the example entries in `experiments/`
5. Update the templates in `experiments/_templates/`
6. Enable GitHub Pages: Settings → Pages → Branch: main, Folder: /docs
7. `quarto render` → `git push`

---

## Philosophy

- **One file per run.** Granular entries are annoying to create and invaluable
  to read six months later when you're debugging.
- **Commit when done, not daily.** The file accumulates across bench days;
  the git commit is the "experiment complete" marker.
- **Math lives in OJS, not in your head.** Protocol QMDs do all volume calculations
  in the browser — no R server, no Shiny, works offline.
- **Public protocols, private aims.** The site shows what's useful to other
  researchers. Aims and reading notes stay local.
- **Not a computational notebook.** Analysis code, figures, and downstream
  processing live in a separate repo. This is purely the bench record.
